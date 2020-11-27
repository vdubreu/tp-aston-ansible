# Installation du bac a sable ansible-examples

## Mise en place des containers et du fichier inventory  
Demarrer des containers pour simuler plusieurs machines.    
```shell script
docker run -d --name target1 systemdevformations/ubuntu_ssh:v2  
docker run -d --name target2 systemdevformations/centos_ssh:v4  
docker run -d --rm --name target3 --env ROOT_PASSWORD=Passw0rd systemdevformations/alpine-ssh:v1   
```
Retrouver le nom des containers  
Faire un ```docker ps | grep systemdevformation ``` 

```shell script
CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS                    PORTS                  NAMES
f5034036dc56        systemdevformations/alpine-ssh:v1   "/entrypoint.sh"         11 hours ago        Up 11 hours               22/tcp                 target3
c714f0b92509        systemdevformations/centos_ssh:v4   "/usr/bin/supervisor…"   23 hours ago        Up 23 hours (unhealthy)   22/tcp                 target2
6051c68c1712        systemdevformations/ubuntu_ssh:v2   "/usr/sbin/sshd -D"      23 hours ago        Up 23 hours               22/tcp                 target1              target1  
```  
 et retrouver l'adresse IP des containers  
 ```shell script
docker network inspect bridge
```

en fonction de l'adresses IP fournie pendant le cours     
modifier egalement la VM centos-remote  

Dans votre home directory,  faire  
```ssh-keygen -t rsa -b 4096 -C "votre adresse mail"```  
Valider les parametres par defaut  
sans passphrase  
Et  
```ssh-copy-id centos@<remote_id_address>```  

## Installation de VirtualEnv Python et faire des Tests avec les Ad-Hoc commandes  

Dans votre home directory faire un git clone de https://github.com

Dans votre projet ansible-examples, faire  
`` python3 -m venv venv``  
cela installe le systeme virtualenv Python dans la directory venv    
Faire  
```source venv/bin/activate```   
Votre prompt change   
```(venv) [centos@ansible-oxiane-controller-2 ~]$```  
Faire   
```pip3 install wheel```    
installe le package wheel qui gere les permission pour installer les packages Pip     
et
```pip3 install ansible```

Vous devez egalement installer le package sshpass     
pour utiliser ssh avec un password, c'est-a-dire sans une cle ssh propagee.    
```sudo apt-get -y install sshpass```  
avec centos  
```sudo yum -y install sshpass```

changer le ficher inventory avec les nouvelles adresses IP 

et faire la commande Ansible Ad-Hoc pour verifier si votre fichier inventory est correct.    
```ansible all -m ping -i inventory```
Faire ensuite  les Ad-Hoc commandes suivantes:  
```shell script 
ansible centos -m yum -a "name=elinks state=latest" -i inventory
ansible centos-remote -m yum -a "name=elinks state=latest" -i inventory
ansible centos-remote -b -m yum -a "name=elinks state=latest" -i inventory
ansible all --list-hosts -i inventory
ansible all -m setup -a "filter=ansible_default_ipv4"  -i inventory
ansible all -m setup -a "filter=ansible_distribution"  -i inventory 
ansible all -m setup -a "filter=ansible_distribution_version"  -i inventory 
ansible all -m command -a "df -h" -i inventory
ansible centos -b -m yum -a "name=* state=latest" -f 100  -i inventory
ansible centos -m file -a "dest=/home/centos/testfile state=touch" -i inventory 
```
## Presentation des groupes
Mettre a jour le ficher inventory_children  sans en modifier la structure  

## Premier script YAML
Dans la directory ansible-examples editez le fichier ansible_ping.yml, et etudiez le code. 
## Premieres commandes ansible-playbook
 ```shell script
ansible-playbook  -i inventory_children ansible_ping.yml  --limit centosdocker
ansible-playbook  -i inventory_children ansible_ping.yml  --limit centos
````
