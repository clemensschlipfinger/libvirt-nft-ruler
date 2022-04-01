# libvirt-nft-ruler
```libvirt-nft-ruler``` is script written in Python for automatically applying/removing **your own** firewall rules with ```nft``` when a virtual network in libvirt starts or stops.
## Motivation
Libvirt uses ```iptables``` for the firewall configuration of the virtual networks. It has no good compatibility to ```nftables``` (the successor of iptables) which I use myself.  
I wanted to insert the input rules from libvirt into my own input chain in a specific order which was not possible.  
So I wrote this script.
## Installation
If you are runnig ```Arch Linux``` on your computer, just use the ```PKGBUILD``` script otherwise do the [`Manual Install`](#manual-install).  
After the installation restart ```libvirtd``` with ```systemctl```.
### Dependencies
 - nftables python library (in Arch Linux shipped with the ```nftables``` package) 
 - python-jsonschema (used for validating templates)
### Manual Install
Clone the repo and copy ```libvirt-nft-ruler``` to ```/etc/libvirt/hooks/network.d```. Additionally, copy the ```templates``` folder into ```/etc/libvirt-nft-ruler```.
## Usage
Create a new virtual network in libvirt and name it ```<network name>-nft-ruler-<template name>``` for example ```network-nft-ruler-nat```. 
 - ```<network name>``` is just the normal name of the network.
 - ```-nft-ruler-``` tells the script to apply rules for this network.
 - ```<template name>``` the name of the template to use, equivalent to the beginning of the files in ```templates```.

When you start this network in Libvirt the script will apply the rules in the specified template. If you stop this network the script will remove the applied rules. You can use the predefined templates but be **careful** see [`Predefined Templates`](#predefined-templates) or you can create your own templates see [```Creating your own Templates```](#creating-your-own-templates).
## Templates
Templates are ```json``` files in the ```templates``` folder which contain the rulesets.  
In every file there are placeholders which are replaced when the script runs.
 - ```$virtualInterface``` specifies the interface for the virtual network.
 - ```$networkName``` is the name of the virtual network.  

Additionally every rule has the property ```"comment": "libvirtd $networkName"``` which has the purpose to identify the rules on deletion.

### Predefined Templates
I have already made templates with rules for a nat and a private network. The rules are based on the [```default ruleset```](https://libvirt.org/firewall.html) libvirt uses.  

**Attention:** The templates assume that a ```insert-input``` chain is already defined. The reason behind it is that I want to easily insert my input rules in a specific order in my standard ```input base chain```.  
#### Create your own ```insert-input``` chain
 - create ```insert-input``` chain using ```nft```
 - call the chain in the ```base input chain``` using the verdict statement ```jump``` in ```nft```

### Creating your own Templates
You can easily create your own template by do the following steps:
 1. ```flush``` your current ruleset
 2. ```apply``` the rules you want the script to apply automatically
 3. ```save``` the active ruleset as json with ```nft -j list ruleset > <template name>-template.json``` and edit it:
    - remove unwanted rules
    - remove ```metainfo```
    - remove the ```handle``` properties
    - replace the network interface name with ```$virtualInterface``` 
    - add the property ```"comment": "libvirtd $networkName"``` to every rule
 4. ```move``` it to the templates folder
