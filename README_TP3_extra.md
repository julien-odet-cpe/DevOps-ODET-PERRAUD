# **TP3 DevOps - Compte rendu**

Filière IRC - 4ème année - Promotion 2024

## Auteurs :
- Julien ODET
- Alex PERRAUD

---
### 3-1 Document your inventory and base commands


```code

all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: ~/.ssh/devops
  children:
    prod:
      hosts: alex.perraud.takima.cloud
```
On teste la connexion :
```code
ansible all -i inventories/setup.yml -m ping
```
on regarde l'os :
```code
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"

```
On retire le serveur apache :
```code
ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become
```

```code
- hosts: all
  gather_facts: false
  become: yes

  tasks:
    - name: Test connection
      ping:
```
```code

ansible-playbook -i inventories/setup.yml playbook.yml
```
Exécution du playbook (playbook.yaml) avec la config de setup.yaml

```code
- hosts: all
  gather_facts: false
  become: yes

  # Install Docker
  tasks:
    - name: Clean packages
      command:
        cmd: dnf clean -y packages

    - name: Install device-mapper-persistent-data
      dnf:
        name: device-mapper-persistent-data
        state: latest

    - name: Install lvm2
      dnf:
        name: lvm2
        state: latest

    - name: add repo docker
      command:
        cmd: sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

    - name: Install Docker
      dnf:
        name: docker-ce
        state: present

    - name: install python3
      dnf:
        name: python3

    - name: Pip install
      pip:
        name: docker

    - name: Make sure Docker is running
      service: name=docker state=started
      tags: docker
```


Les playbooks et les rôles ont été vérifiés, durant le déploiement ansible la VM a crash et je n'ai jamais pu réessayer.
Elle a sûrement crash à cause du Load Balancer, le fait que j'ai deux serveurs JAVA.
Donc je ne suis pas apparu sur le git. Elle n'a jamais redémarré.
Il aurait fallu que je réduise la ram des deux Java mais dans tous les cas la VM n'a plus jamais été accessible.
6 rôles ont été créés :

- docker -> installation de docker :
```code
- name: Clean packages
  command:
    cmd: dnf clean -y packages

- name: Install device-mapper-persistent-data
  dnf:
    name: device-mapper-persistent-data
    state: latest

- name: Install lvm2
  dnf:
    name: lvm2
    state: latest

- name: add repo docker
  command:
    cmd: sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

- name: Install Docker
  dnf:
    name: docker-ce
    state: present

- name: install python3
  dnf:
    name: python3

- name: Pip install
  pip:
    name: docker

- name: Make sure Docker is running
  service: name=docker state=started
  tags: docker

```

- network -> création du réseau app-network :
```code

- name: Create Network
  command:
    cmd: docker network create app-network
```
- database -> création du conteneur docker database :
```code
- name: Run database
  docker_container:
    name: database
    image: alexcpe/database:1.0
    network_mode: app-network
```
- app -> les deux serveurs Java car il y'a un load Balancer (on aurait pu utiliser la même image) :
```code
- name: Run backend-green
  docker_container:
    name: backend-green
    image: alexcpe/backend-green:1.0
    network_mode: app-network

- name: Run backend-blue
  docker_container:
    name: backend-blue
    image: alexcpe/backend-blue:1.0
    network_mode: app-network
```
- front ->  conteneur dockeur front :
```code
- name: Run FRONT
  docker_container:
    name: front
    image: alexcpe/front:1.0
    network_mode: app-network
    env:
      - VUE_APP_API_URL : "http://alex.perraud.takima.cloud:8080"
```
- proxy -> conteneur dockeur httpd -> apache :
```code
- name: Run HTTPD
  docker_container:
    name: httpd
    image: alexcpe/httpd:1.0
    network_mode: app-network
    ports:
      - "80:80"
      - "8080:8080"

```

## Load Balancer


## Front