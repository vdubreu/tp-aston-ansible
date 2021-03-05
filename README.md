# Installation du bac a sable ansible-examples
## Pre-requis pour Centos 7
```
sudo yum -y update   # update all packages 
sudo yum -y install git wget   # install git and wget 
sudo yum -y install epel-release  # added extra packages
sudo yum -y install htop iotop iftop  # added monitoring tools
sudo yum -y install python3 
```
## installation de Docker
### Installation de la derniere version de Docker sous Centos 
L'installation de Docker necessite certains packages.
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
Ensuite nous devons mettre en place le lien vers le repository Docker.
```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
Installer la derniere version de Docker et de ses packages client et containerd.io
```
 sudo yum install -y docker-ce docker-ce-cli containerd.io
```
Lancer le daemon Docker 
```
sudo systemctl start docker
```
Placer un lien symbolique pour que le daemon Docker demarre automatiquement meme si le host est reboote. 
```
sudo systemctl enable docker
sudo usermod -aG docker centos  # fermer votre shell pour les chargements soient appliques
```

## Mise en place des containers et du fichier inventory  
Demarrer des containers pour simuler plusieurs machines.    
```shell script
docker run -d --name target1 systemdevformations/ubuntu_ssh:v2  
docker run -d --name target2 systemdevformations/centos_ssh:v5 
docker run -d --name target3 --env ROOT_PASSWORD=Passw0rd systemdevformations/alpine-ssh:v1   
```
Check si les containers sont presents  
Faire un ```docker ps | grep systemdevformation ``` 

```shell script
CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS                    PORTS                  NAMES
f5034036dc56        systemdevformations/alpine-ssh:v1   "/entrypoint.sh"         11 hours ago        Up 11 hours               22/tcp                 target3
c714f0b92509        systemdevformations/centos_ssh:v5   "/usr/bin/supervisor…"   23 hours ago        Up 23 hours (unhealthy)   22/tcp                 target2
6051c68c1712        systemdevformations/ubuntu_ssh:v2   "/usr/sbin/sshd -D"      23 hours ago        Up 23 hours               22/tcp                 target1              target1  
```  
 et retrouver l'adresse IP des containers, et notez les 
 ```shell script
docker network inspect bridge
```

en fonction de l'adresses IP fournie pendant le cours     
modifier egalement la VM ansible-remote  

Dans votre home directory,  faire  
```ssh-keygen -t rsa -b 4096 ```  
Valider les parametres par defaut en tapant enter a chaque etape 
sans passphrase  
Et  
```ssh-copy-id centos@<remote_id_address>```  

## Installation de VirtualEnv Python et faire des Tests avec les Ad-Hoc commandes  
Dans votre home directory faire un git clone de https://github.com/<votre_repo>/ansible-examples.git

Dans votre projet ansible-examples, faire  
```shell script
cd ansible-examples 
python3 -m venv venv
```  
cela installe le systeme virtualenv Python dans la directory venv    
Faire  
```source venv/bin/activate```   
Votre prompt change   
```(venv) [centos@ansible-oxiane-controller-2 ~]$```  
Faire
```shell
pip3 install natsort # library pour l'utilisation des filtres
pip3 install wheel  # pour mettre en place des permissions
pip install --upgrade pip  # pour mettre a jour pip et supporter les libs de crypto
```
installe la derniere version d'Ansible     
et
```shell
pip3 install ansible
ansible --version # should be the version 2.10.6
```

Vous devez egalement installer le package sshpass     
pour utiliser ssh avec un password, c'est-a-dire sans une cle ssh propagee. 
avec Ubuntu
```sudo apt-get -y install sshpass```  
avec centos  
```sudo yum -y install sshpass```

changer le ficher inventory avec les nouvelles adresses IP 

et faire la commande Ansible Ad-Hoc pour verifier si votre fichier inventory est correct.    
```ansible all -m ping -i inventory```
Faire ensuite  les Ad-Hoc commandes suivantes une par une lentement:  
```shell script 
ansible centos -m yum -a "name=elinks state=latest" -i inventory
ansible centosremote -m yum -a "name=elinks state=latest" -i inventory
ansible centosremote -b -m yum -a "name=elinks state=latest" -i inventory
ansible all --list-hosts -i inventory
ansible all -m setup -a "filter=ansible_default_ipv4"  -i inventory
ansible all -m setup -a "filter=ansible_distribution"  -i inventory 
ansible all -m setup -a "filter=ansible_distribution_version"  -i inventory 
ansible all -m command -a "df -h" -i inventory
NE PAS FAIRE LA COMMANDE SUIVANTE ELLE PREND 10 MINUTES
# -- ansible centos -b -m yum -a "name=* state=latest" -f 10  -i inventory  # default = 5
ansible centos -m file -a "dest=/home/centos/testfile state=touch" -i inventory 
```
## Presentation des groupes de hosts
Mettre a jour les adresse IP dans le ficher inventory_children sans en modifier 
la structure.  

## Premier script YAML
Dans la directory ansible-examples editez le fichier ansible_ping.yml, et etudiez le code. 
## Premieres commandes ansible-playbook
 ```shell script
ansible-playbook  -i inventory_children ansible_ping.yml  --limit centosdocker
ansible-playbook  -i inventory_children ansible_ping.yml  --limit centos
````
