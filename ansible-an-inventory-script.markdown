---
title: Ansible  inventory script
date: 2018-01-28 12:07:00 -08:00
---



# Ansible Dynamic Inventory
## Getting API Key

We need to put to the server which can run PHP, two files: admin.php http://files.virtualizor.com/sdk.zip and  something like below

```php

<?php

    require_once('/usr/local/virtualizor/sdk/admin.php');

    $key =  'some key';

    $pass = 'some pass;

    $ip = 'some IP';

    $mainFunction=$_GET['f'];
    $numParametr = count($_GET);

$post = array_slice($_GET,1);
foreach ($post  as $key => $value) {
    echo "Key: $key; Value: $value<br />\n";}


    $admin = new Virtualizor_Admin_API($ip, $key, $pass);
if ($numParametr <=1) {

   $output = $admin->$mainFunction(); 
}
else{ 

$output = $admin->$mainFunction($post);
}
print_r(json_encode($output));
?>  

```
In  my  case I just copy my script to ``` cp efim.php /usr/local/virtualizor/enduser/``` then execute the query ``` https://your_server_domain:4085/efim.php?f=listservers```. Please don't forget to remove the script ```rm /usr/local/virtualizor/enduser/efim.php after your get the key.

To understand my script you need a  base understanding of PHP and this page http://virtualizor.com/admin-api/ that explains how to work with Virtualizator API.

The  script I provided,  consist: an IP the main node,  its key and its pass.The key and the pass can be found here ``` /usr/local/virtualizor/universal.php```

Next step would be finding tthe key,  just check your server log for the queries  you sent to the server by exucuting my scrpt ``` tail -f /usr/local/emps/var/log/web.access.log | grep -v '?reverse_sync=1'| grep listservers ```

## Writing a library to work with API

Here it is my library as you see I provide the apikey that I gained at the previous step

```python
import requests
import json

def sendRecieveDCIM(mainFunctionTosend,funcToSend):

    baseUrl = "https://your_server_domain:4085//index.php?api=json&apikey=your_api_key"

   
    firstPart = {"act":  mainFunctionTosend}
    firstPart = list(firstPart.items())
    secondPart = list(funcToSend.items())
    payload  =  dict( firstPart + secondPart)
    #print payload           
    s = requests.get(baseUrl,params=payload)
#    print (s.url)
    try:
        return s.json()
    except ValueError:
        return "What????"
if __name__ == "__main__":
    import  json

    from pprint import pprint
    
    servers = sendRecieveDCIM("servers",{})
#    print(servers)
    serverGroups =  sendRecieveDCIM("servergroups",{})  
#    
    group = {}
    for server in servers['servers']:
        
        if servers['servers'][server]['sgid'] not in group.keys():
            group[servers['servers'][server]['sgid']] = []
            group[servers['servers'][server]['sgid']].append({servers['servers'][server]['server_name']: servers['servers'][server]['ip']})
        else:
            group[servers['servers'][server]['sgid']].append({servers['servers'][server]['server_name'] : servers['servers'][server]['ip']})

            
        #names[servers['servers'][server]['server_name']] =  {"hostname": servers['servers'][server]['ip']}
    # print(group.keys())
    # print(serverGroups['servergroups'][0])

    for servergroup in serverGroups['servergroups']:
        if servergroup['sgid'] in group:
            group[servergroup['sg_name']] = group.pop(servergroup['sgid'])


    print(json.dumps(group))        
    #names = json.dumps(names)
    #print(names)
    #pprint(sendRecieveDCIM("services",{}))
```

## The dynamic inventory script itself

```python
import json
import argparse
from collections import OrderedDict
import json_p_nRH




def get_server_dict():
    ''' Send reqests to the main node and populate the dictionary we will work with'''
    servers = json_p_nRH.sendRecieveDCIM("servers",{})
    serverGroups = json_p_nRH.sendRecieveDCIM("servergroups",{}) 
    group = {} 
 
    
    for server in servers['servers']:
        '''fill a  dictionary {sgid:[{serve_name:server_ip}]}'''
        if servers['servers'][server]['sgid'] not in group.keys():
            group[servers['servers'][server]['sgid']] = []
            
            group[servers['servers'][server]['sgid']].append({servers['servers'][server]['server_name']: servers['servers'][server]['ip']})
           
        else:
            group[servers['servers'][server]['sgid']].append({servers['servers'][server]['server_name'] : servers['servers'][server]['ip']})
   

    for servergroup in serverGroups['servergroups']:
        ''' update dictionary's key to something  meaningful {Los angles servers:[{serve_name:server_ip}]}'''
        if servergroup['sgid'] in group:
            group[servergroup['sg_name']] = group.pop(servergroup['sgid'])
    
    
    return group
def print_host(host):
    '''print info for a particular host'''  
    group = get_server_dict()
    for local_group in list(group.values()):
        
        for server in local_group:
            if host in server:
                print(json.dumps({'ip':server[host]}))

    
def print_hosts():
    '''print all hosts'''  
    group = get_server_dict()
    holder0 = OrderedDict()
    
    meta = {'hostvars':{}}

    for k,value in group.items():
        
        holder0[k] = {'hosts':[]}
        temp = []
        for key in value:
            value = list(key.values())
            key = list(key.keys())
            temp.append(key[0])
            
            meta['hostvars'][key[0]] = {'ansible_ssh_host':  value[0]}
        holder0[k] = {'hosts':temp}
    holder0.update({'_meta':meta})    
    print (json.dumps(holder0))

  
   
 
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument('--list', help='show all the inventory',  action='store_true')
    parser.add_argument('--host', help='show a particular host')
    args = parser.parse_args()
    if args.list:
        print_hosts()
    if args.host:
        print_host(args.host)    
    if not args.list and not  args.host:
        print("One argument should be passed")    
if __name__ == '__main__':
    main()


```
