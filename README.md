freebsd-mailserver-roundcube
============================

[![Build Status](https://travis-ci.org/vbotka/ansible-freebsd-mailserver-roundcube.svg?branch=master)](https://travis-ci.org/vbotka/freebsd-mailserver-roundcube)

Ansible role. Install and configure Roundcube webmail at FreeBSD. In development. See TODO below.

[galaxy.ansible.com/vbotka/freebsd-mailserver-roundcube/](https://galaxy.ansible.com/vbotka/freebsd-mailserver-roundcube/)

Tested with FreeBSD 10.3 at [digitalocean.com](https://cloud.digitalocean.com)


Requirements
------------

Requires:
- [vbotka.ansible-freebsd-mailserver](https://galaxy.ansible.com/vbotka/ansible-freebsd-mailserver/)
- [vbotka.apache](https://galaxy.ansible.com/vbotka/apache/)

Recommended:
- [vbotka.ansible-freebsd-mailserver-spamassassin](https://galaxy.ansible.com/vbotka/ansible-freebsd-mailserver-spamassassin/)
- [vbotka.ansible-freebsd-mailserver-sieve](https://galaxy.ansible.com/vbotka/ansible-freebsd-mailserver-sieve/)


Variables
---------

roundcube_mysql_password: "MYSQL-PASSWORD"

```
GRANT ALL PRIVILEGES ON roundcube.* TO roundcube@localhost IDENTIFIED BY 'MYSQL-PASSWORD';
```

Defaults

```
roundcube_zoneinfo: "UTC"
roundcube_mysql_password: "MYSQL-PASSWORD"
roundcube_debug_level: "5"
roundcube_smtp_server: "localhost"
roundcube_support_url: "www.example.com/support/"
roundcube_product_name: "Roundcube Webmail"
roundcube_plugins: "'archive', 'zipdownload', 'managesieve', 'password'"
```


Workflow
--------

1) Change shell to /bin/sh.

```
> ansible mailserver -e 'ansible_shell_type=csh ansible_shell_executable=/bin/csh' -a 'sudo pw usermod freebsd -s /bin/sh'
```

2) Install role.

```
> ansible-galaxy install vbotka.freebsd-mailserver-roundcube
```

3) Fit variables.

```
~/.ansible/roles/vbotka.freebsd-mailserver-roundcube/vars/main.yml
```

4) Create playbook and inventory.

```
> cat ~/.ansible/playbooks/freebsd-mailserver-roundcube.yml
---
- hosts: mailserver
  become: yes
  become_method: sudo
  roles:
    - role: vbotka.freebsd-mailserver-roundcube
```

```
> cat ~/.ansible/hosts
[mailserver]
<WEBSERVER-IP-OR-FQDN>

[mailserver:vars]
ansible_connection=ssh
ansible_user=freebsd
ansible_python_interpreter=/usr/local/bin/python2
ansible_perl_interpreter=/usr/local/bin/perl
```

5) Install and configure Roundcube webmail..

```
ansible-playbook ~/.ansible/playbooks/freebsd-mailserver-roundcube.yml
```

6) Consider to test the webmail

   - http://validator.w3.org
   - https://www.ssllabs.com
		

References
----------

- [FreeBSD Postfix – Page 13 – Roundcube Install](http://www.purplehat.org/?page_id=20))
- [Guide On How To Install Roundcube On FreeBSD](http://www.xfiles.dk/guide-on-how-to-install-roundcube-on-freebsd/)


TODO
----

- add mysql configuration

```
Create MySQL database and user for Roundcube:
mysql -u root mysql
mysql> CREATE DATABASE roundcube;
mysql> GRANT ALL PRIVILEGES ON roundcube.* TO roundcube@localhost IDENTIFIED BY '<sanitized>';
mysql> QUIT;
Populate the Roundcube database:
cd /usr/local/www/roundcube/SQL
mysql -u roundcube -p roundcube < mysql.initial.sql
```

- add automatic_addressbook plugin
- configure sieve
- configure pspell


License
-------

[![license](https://img.shields.io/badge/license-BSD-red.svg)](https://www.freebsd.org/doc/en/articles/bsdl-gpl/article.html)


Author Information
------------------

[Vladimir Botka](https://botka.link)
