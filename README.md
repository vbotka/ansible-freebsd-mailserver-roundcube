freebsd-mailserver-roundcube
============================

[![Build Status](https://travis-ci.org/vbotka/ansible-freebsd-mailserver-roundcube.svg?branch=master)](https://travis-ci.org/vbotka/freebsd-mailserver-roundcube)

[Ansible role.](https://galaxy.ansible.com/vbotka/freebsd-mailserver-roundcube/) FreeBSD. Install and configure [Roundcube](https://roundcube.net/) webmail.


Requirements
------------

Required:
- [vbotka.ansible-freebsd-mailserver](https://galaxy.ansible.com/vbotka/ansible-freebsd-mailserver/)
- [vbotka.freebsd-mysql](https://galaxy.ansible.com/vbotka/freebsd-mysql/)
- [vbotka.apache](https://galaxy.ansible.com/vbotka/apache/)

Recommended:
- [vbotka.ansible-freebsd-mailserver-spamassassin](https://galaxy.ansible.com/vbotka/ansible-freebsd-mailserver-spamassassin/)
- [vbotka.ansible-freebsd-mailserver-sieve](https://galaxy.ansible.com/vbotka/freebsd-mailserver-sieve/)


Variables
---------

Review the defaults and examples in vars.


MySQL password for user *roundcube*
---------------------------------

roundcube_mysql_password: "MYSQL-PASSWORD"

```
GRANT ALL PRIVILEGES ON roundcube.* TO roundcube@localhost IDENTIFIED BY 'MYSQL-PASSWORD';
```

Defaults
--------

```
fm_roundcube_debug: False
fm_roundcube_initial_sql: False

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

By default the database is not populated *fm_roundcube_initial_sql=False*. Let's configure Roundcube first (1-5) and populate the database separately (6).

1) Change shell to /bin/sh

```
# ansible mailserver -e 'ansible_shell_type=csh ansible_shell_executable=/bin/csh' -a 'sudo pw usermod freebsd -s /bin/sh'
```

2) Install role

```
# ansible-galaxy install vbotka.freebsd-mailserver-roundcube
```

3) Fit variables

```
# editor vbotka.freebsd-mailserver-roundcube/vars/main.yml
```

4) Create playbook and inventory

```
# cat freebsd-mailserver-roundcube.yml

- hosts: mailserver
  roles:
    - role: vbotka.freebsd-mailserver-roundcube
```

```
# cat hosts
[mailserver]
<MAILSERVER-IP-OR-FQDN>
[mailserver:vars]
ansible_connection=ssh
ansible_user=freebsd
ansible_python_interpreter=/usr/local/bin/python2.7
ansible_perl_interpreter=/usr/local/bin/perl
```

5) Install and configure Roundcube webmail

```
# ansible-playbook freebsd-mailserver-roundcube.yml
```

6) Populate Roundcube database

```
# ansible-playbook -e fm_roundcube_initial_sql=True freebsd-mailserver-roundcube.yml -t fm_roundcube_initial_sql
```

7) Consider to test the webmail

   - http://validator.w3.org
   - https://www.ssllabs.com
		

References
----------

- [FreeBSD Postfix – Page 13 – Roundcube Install](http://www.purplehat.org/?page_id=20)
- [Guide On How To Install Roundcube On FreeBSD](http://www.xfiles.dk/guide-on-how-to-install-roundcube-on-freebsd/)
- [Roundcube Community Forum](http://www.roundcubeforum.net/)


TODO
----

- add automatic_addressbook plugin
- configure sieve
- configure pspell


License
-------

[![license](https://img.shields.io/badge/license-BSD-red.svg)](https://www.freebsd.org/doc/en/articles/bsdl-gpl/article.html)


Author Information
------------------

[Vladimir Botka](https://botka.link)
