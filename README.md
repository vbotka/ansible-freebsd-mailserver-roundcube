# freebsd_mailserver_roundcube

[![Build Status](https://travis-ci.org/vbotka/ansible-freebsd-mailserver-roundcube.svg?branch=master)](https://travis-ci.org/vbotka/freebsd-mailserver-roundcube)

[Ansible role.](https://galaxy.ansible.com/vbotka/freebsd_mailserver_roundcube/) FreeBSD. Install and configure [Roundcube](https://roundcube.net/) webmail.

Please feel free to [share your feedback and report issues](https://github.com/vbotka/ansible-freebsd-mailserver-roundcube/issues).

## Requirements

Only Apache and MySQL is supported by this role. Other servers (Lighttpd, Nginx, PostgreSQL, SQLite) are WIP.


## Dependencies

- [vbotka.freebsd_mailserver](https://galaxy.ansible.com/vbotka/freebsd_mailserver/)
- [vbotka.freebsd_mysql](https://galaxy.ansible.com/vbotka/freebsd_mysql/)
- [vbotka.apache](https://galaxy.ansible.com/vbotka/apache/)

The dependencies are not listed in the meta file. Install the roles manually.


## Recommended

- [vbotka.freebsd_mailserver_spamassassin](https://galaxy.ansible.com/vbotka/freebsd_mailserver_spamassassin/)
- [vbotka.freebsd-mailserver_sieve](https://galaxy.ansible.com/vbotka/freebsd_mailserver_sieve/)


## Variables

Review the defaults and examples in vars.


1) Configure MySQL password for user *roundcube*

```
roundcube_mysql_password: "MYSQL-PASSWORD"
```
This password has been used by [vbotka.freebsd_mysql](https://galaxy.ansible.com/vbotka/freebsd_mysql/) to grant privileges to user *roundcube@localhost*
```
GRANT ALL PRIVILEGES ON roundcube.* TO roundcube@localhost IDENTIFIED BY 'MYSQL-PASSWORD';
```


## Defaults

```
fm_roundcube_debug: false
fm_roundcube_initial_sql: false

roundcube_zoneinfo: "UTC"
roundcube_mysql_password: "MYSQL-PASSWORD"
roundcube_debug_level: "5"
roundcube_smtp_server: "localhost"
roundcube_support_url: "www.example.com/support/"
roundcube_product_name: "Roundcube Webmail"
roundcube_plugins_conf:
  archive:
    enabled: true
  managesieve:
    enabled: true
    conf:
      - {key: managesieve_default, val: "'/usr/local/virtual/home/default.sieve'"}
  password:
    enabled: true
    conf:
      - key: password_minimum_length
        val: 8
      - key: password_db_dsn
        val: "'mysql://postfix:postfix_sql_password@localhost/postfix'"
      - key: password_query
        val: "'UPDATE mailbox SET password=%c, modified=now() WHERE username=%u'"
  zipdownload:
    enabled: true
    conf: []
```


## Workflow

By default the database is not populated *fm_roundcube_initial_sql=False*. Let's configure Roundcube first (1-5) and populate the database later (6).

1) Change shell to /bin/sh

```
shell> ansible mailserver -e 'ansible_shell_type=csh ansible_shell_executable=/bin/csh' -a 'sudo pw usermod freebsd -s /bin/sh'
```

2) Install role

```
shell> ansible-galaxy install vbotka.freebsd_mailserver_roundcube
```

3) Fit variables

```
shell> editor vbotka.freebsd_mailserver_roundcube/vars/main.yml
```

4) Create playbook and inventory

```
shell> cat freebsd-mailserver-roundcube.yml

- hosts: mailserver
  roles:
    - role: vbotka.freebsd_mailserver_roundcube
```

```
shell> cat hosts
[mailserver]
<MAILSERVER-IP-OR-FQDN>
[mailserver:vars]
ansible_connection=ssh
ansible_user=freebsd
ansible_python_interpreter=/usr/local/bin/python3.7
ansible_perl_interpreter=/usr/local/bin/perl
```

5a) Optionally create plugins default configuration files from *config.inc.php.dist*. Disable webserver to prevent starting the webserver with the default configuration.

```
shell> ansible-playbook freebsd-mailserver-roundcube.yml -t fm_roundcube_plugins_conf_create
```

5b) Optionally run playbook in check and diff mode. The command will fail if the plugins' configuration files are missing.

```
shell> ansible-playbook freebsd-mailserver-roundcube.yml --check --diff
```

5c) Install and configure Roundcube webmail. Run the command twice to make sure it is idempotent.

```
shell> ansible-playbook freebsd-mailserver-roundcube.yml
```

6) Populate Roundcube database

```
shell> ansible-playbook freebsd-mailserver-roundcube.yml -t fm_roundcube_initial_sql -e "fm_roundcube_initial_sql=True"
```

7) Consider to test the webmail

- http://validator.w3.org
- https://www.ssllabs.com
		

## References

- [Roundcube webmail](https://roundcube.net/)
- [FreeBSD Postfix – Page 13 – Roundcube Install](http://www.purplehat.org/?page_id=20)
- [Guide On How To Install Roundcube On FreeBSD](http://www.xfiles.dk/guide-on-how-to-install-roundcube-on-freebsd/)
- [Roundcube Community Forum](http://www.roundcubeforum.net/)


**TODO**

- add automatic_addressbook plugin
- configure sieve
- configure pspell


## License

[![license](https://img.shields.io/badge/license-BSD-red.svg)](https://www.freebsd.org/doc/en/articles/bsdl-gpl/article.html)


## Author Information

[Vladimir Botka](https://botka.link)
