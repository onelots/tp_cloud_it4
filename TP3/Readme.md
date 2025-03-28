# TP3 : Automatisation et gestion de conf

## Part 1 : Ansible first steps

[main.tf : c'est par ici](main.tf)

Ensuite ben

[Le cloud-init](cloud-init.txt)

---

### 1 le first playbook

```yml
- name: Install nginx
  hosts: le_tp
  become: true

  tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present

  - name: move certificate
    template:
      src: localhost.crt
      dest: /

  - name: move key
    template:
      src: localhost.key
      dest: /

  - name: create directory 
    ansible.builtin.file:
      path: /var/www/site_chouette
      state: directory
      mode: '0755'
      owner: www-data
      group: www-data

  - name: move index
    template:
      src: index.html
      dest: /var/www/site_chouette

  - name: move nginx conf in the right folder
    template:
      src: static-site-config
      dest: /etc/nginx/sites-enabled/default

  - name: Start NGiNX
    service:
      name: nginx
      state: started
```

voilà. Maintenant on curl.

`curl --insecure https://4.212.13.80/`
> It works !

donc ça work j'imagine ?

### 2 - la souffréance et l'anéantissement de tout espoir de rédemption (mysql)

```yml
- name: MySQL my beloved
  hosts: db
  become: true

  tasks:
    - name: Install MySQL
      package:
       name: "{{item}}"
       state: present
       update_cache: yes
      loop:
       - mysql-server
       - mysql-client 
       - python3-mysqldb
       - libmysqlclient-dev
      become: yes
  
    - name: start mysql
      service:
        name: mysql
        state: started
        enabled: yes    

    - name: create user
      mysql_user:
        name: "onelots"
        password: "1234ABCD!"
        priv: '*.*:ALL'
        host: '%'
        state: present    

    - name: create db
      mysql_db:
        name: "dummy_database"
        state: present    

    - name: remote login to db
      lineinfile:
        path: /etc/mysql/mysql.conf.d/mysqld.cnf
        regexp: '^bind-address'
        line: 'bind-address = 0.0.0.0'
        backup: yes
      notify:
        - Restart mysql  

  handlers:
    - name: Restart mysql
      service:
        name: mysql
        state: restarted
```

```logs
❯ mysql --host=4.212.13.80 --user=onelots --password=1234ABCD! --skip-ssl dummy_database

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 16
Server version: 8.0.41-0ubuntu0.22.04.1 (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Support MariaDB developers by giving a star at https://github.com/MariaDB/server
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [dummy_database]>

```

## Part 2

Ma chambre est pas rangée. RIP. Elle est toujours dans un état désastreux.

## Part 3

#### 1. Le ène-jaïne-ixe.


> virtual_hosts.yml

```yaml

- name: ène-jaïne-ixe
  become: true
  template:
    src: virtual_host.conf.j2
    dest: /etc/nginx/conf.d/{{ item.nginx_servername }}.conf
  loop: "{{ vhosts }}"
  notify: Restart Nginx

- name: vhost folders
  become: true
  file:
    path: "{{ item.nginx_webroot }}"
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'
  loop: "{{ vhosts }}"

- name: dummy index html foreach 
  become: true
  copy:
    dest: "{{ item.nginx_webroot }}/index.html"
    content: "{{ item.nginx_index_content }}"
    owner: www-data
    group: www-data
    mode: '0744'
  loop: "{{ vhosts }}"
  
  ```


> virtual_hosts.conf.j2


```jinja
server {
    listen {{ item.nginx_port }};
    server_name {{ item.nginx_servername }};
    root {{ item.nginx_webroot }};

    location / {
        index index.html;
    }
}
```

> handlers/main.yaml

```yaml
- name: reload ène-jaïne-ixe
  become: true
  service:
    name: nginx
    state: reload
```

#### 2. users ? users. (Common)

create_users.yml

```yaml

- name: create users with ssh access
  become: true
  block:
    - name: create users
      user:
        name: "{{ item.username }}"
        state: present
        password: "{{ item.password }}"
        home: "/home/{{ item.username }}"
        shell: /bin/bash
        groups: admin
        append: true
      loop: "{{ users }}"

    - name: .ssh ?
      file:
        path: "/home/{{ item.username }}/.ssh"
        state: directory
        owner: "{{ item.username }}"
        group: "{{ item.username }}"
        mode: '0700'
      loop: "{{ users }}"

    - name: hop adding ssh key to authorized
      copy:
        dest: "/home/{{ item.username }}/.ssh/authorized_keys"
        content: "{{ item.ssh_key }}"
        owner: "{{ item.username }}"
        group: "{{ item.username }}"
        mode: '0600'
      loop: "{{ users }}"
    
```

--- 

Là j'me suis endormi

```yml
rgjoemqivqoi m
e tbd




e qvf ez v

qvfs
 
 vfsgzet)-i
 ```

 --- 

`4.212.13.80.yml`

```yml

users:
  - name: roberto
    username: roberto
    password: un password
    ssh_key: ssh-rsa ma clé
  - name: marco
    username: marco
    password: un password
    ssh_key: ssh-rsa ma clé

```

#### Dynamic loadbalancer

Une 3eme machine 

```json

resource "azurerm_linux_virtual_machine" "la_troisieme_machine" {
  name                = "${var.prefix}-vm3"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B2s"
  admin_username      = "onelots"
  
  network_interface_ids = [azurerm_network_interface.internal_vm3.id]

  admin_ssh_key {
    username   = "onelots"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  custom_data = filebase64("cloud-init.txt")
}```

reverse_proxy.yml

```yml

- name: reverse_proxy
  hosts: reverseproxy
  become: true

  tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present

  - name: copy file
    become: true
    template:
      src: reverse_proxy.conf.j2
      dest: /etc/nginx/conf.d/rproxy.conf

  - name: delete default website
    file:
      path: /etc/nginx/sites-enabled/default
      state: absent
  
  - name: Start NGiNX
    service:
      name: nginx
      state: started
```

jinja conf pour le reverseproxy (reverse_proxy.conf.j2)

```
upstream onelots {
{% for ip in groups['webapp'] %}
    server {{ ip }};
{%  %}
}

server {
    server_name onelots.com;

    location / {
        proxy_pass http://onelots;
    }
}