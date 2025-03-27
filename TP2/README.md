YO

# Part 1

`❯ az vm create -g waf -n ubuntu1 --image Ubuntu2204 --admin-username onelots --generate-ssh-keys`

```log
Selecting "northeurope" may reduce your costs. The region you've selected may cost more for the same services. You can disable this message in the future with the command "az config set core.display_region_identified=false". Learn more at https://go.microsoft.com/fwlink/?linkid=222571 

Private SSH key file '/Users/onelots/.ssh/id_rsa' was found in the directory: '/Users/onelots/.ssh'. A paired public key file '/Users/onelots/.ssh/id_rsa.pub' will be generated.
SSH key files '/Users/onelots/.ssh/id_rsa' and '/Users/onelots/.ssh/id_rsa.pub' have been generated under ~/.ssh to allow SSH access to the VM. If using machines without permanent storage, back up your keys to a safe location.
{
  "fqdns": "",
  "id": "/subscriptions/5f780c1d-3f8e-4caa-96a8-c51420c96c8b/resourceGroups/waf/providers/Microsoft.Compute/virtualMachines/ubuntu1",
  "location": "francecentral",
  "macAddress": "60-45-BD-1B-0F-50",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "4.211.172.128",
  "resourceGroup": "waf",
  "zones": ""
}
```

`❯ az vm create \
  -g waf \
  -n ubuntu2 \
  --image Ubuntu2204 \
  --admin-username onelots \
  --vnet-name ubuntu1VNET \
  --subnet ubuntu1Subnet \
  --ssh-key-values ../../../.ssh/id_rsa.pub`

```log
Selecting "northeurope" may reduce your costs. The region you've selected may cost more for the same services. You can disable this message in the future with the command "az config set core.display_region_identified=false". Learn more at https://go.microsoft.com/fwlink/?linkid=222571 

{
  "fqdns": "",
  "id": "/subscriptions/5f780c1d-3f8e-4caa-96a8-c51420c96c8b/resourceGroups/waf/providers/Microsoft.Compute/virtualMachines/ubuntu2",
  "location": "francecentral",
  "macAddress": "00-0D-3A-89-57-F7",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "4.212.13.80",
  "resourceGroup": "waf",
  "zones": ""
}```

>> Même réseau local (10.0.0.5 et 10.0.0.4)

- vm 1 : 
    - ```onelots@ubuntu2:~$ cloud-init status
         status: done```
- vm 2 :
    - ```onelots@ubuntu1:~$ cloud-init status
    -    status: done```

## Ping ?

vm1 : 

```ping
onelots@ubuntu1:~$ ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=1.58 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=1.70 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=14.4 ms
^C
--- 10.0.0.5 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.577/5.891/14.399/6.016 ms```

vm2 :

```ping
onelots@ubuntu2:~$ ping 10.0.0.4
PING 10.0.0.4 (10.0.0.4) 56(84) bytes of data.
64 bytes from 10.0.0.4: icmp_seq=1 ttl=64 time=0.869 ms
64 bytes from 10.0.0.4: icmp_seq=2 ttl=64 time=0.994 ms
64 bytes from 10.0.0.4: icmp_seq=3 ttl=64 time=0.879 ms
^C
--- 10.0.0.4 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2052ms
rtt min/avg/max/mdev = 0.869/0.914/0.994/0.056 ms```

La classe




------------------


End of File
