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

MySQL [dummy_database]>```

## Part 2

## Part 3
