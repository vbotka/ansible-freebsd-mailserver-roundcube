# freebsd_mailserver_roundcube

[![quality](https://img.shields.io/ansible/quality/27910)](https://galaxy.ansible.com/vbotka/freebsd_mailserver_roundcube)[![Build Status](https://travis-ci.org/vbotka/ansible-freebsd-mailserver-roundcube.svg?branch=master)](https://travis-ci.org/vbotka/freebsd-mailserver-roundcube)

[Ansible role.](https://galaxy.ansible.com/vbotka/freebsd_mailserver_roundcube/) FreeBSD. Install and configure [Roundcube](https://roundcube.net/) webmail.

Feel free to [share your feedback and report issues](https://github.com/vbotka/ansible-freebsd-mailserver-roundcube/issues).

[Contributions are welcome](https://github.com/firstcontributions/first-contributions).


## Requirements and dependencies

### Packages

- PHP
- Only Apache and MySQL is supported by this role.
- Other servers (Lighttpd, Nginx, PostgreSQL, SQLite) are WIP.

See the default versions of the packages in defaults/main.yml

### Collections

- community.general

### Roles

- [vbotka.freebsd_mailserver](https://galaxy.ansible.com/vbotka/freebsd_mailserver/)
- [vbotka.freebsd_mysql](https://galaxy.ansible.com/vbotka/freebsd_mysql/)
- [vbotka.apache](https://galaxy.ansible.com/vbotka/apache/)

The dependencies are not listed in the meta file. Install the roles manually.

### Recommended

- [vbotka.freebsd_mailserver_spamassassin](https://galaxy.ansible.com/vbotka/freebsd_mailserver_spamassassin/)
- [vbotka.freebsd-mailserver_sieve](https://galaxy.ansible.com/vbotka/freebsd_mailserver_sieve/)


## Variables

See the defaults and examples in vars.


1) Configure MySQL password for user *roundcube*

```
roundcube_mysql_password: "MYSQL-PASSWORD"
```

This password has been used by [vbotka.freebsd_mysql](https://galaxy.ansible.com/vbotka/freebsd_mysql/) to grant privileges to user *roundcube@localhost*

```
GRANT ALL PRIVILEGES ON roundcube.* TO roundcube@localhost IDENTIFIED BY 'MYSQL-PASSWORD';
```

## Defaults (selected)

```
fm_roundcube_install: true
fm_roundcube_debug: false
fm_roundcube_debug_classified: false
fm_roundcube_backup_conf: false
fm_roundcube_initial_sql: false

roundcube_mysql_password: MYSQL-PASSWORD
roundcube_support_url: www.example.com/support/
roundcube_product_name: Roundcube Webmail
roundcube_zoneinfo: UTC
roundcube_debug_level: 5
roundcube_smtp_server: localhost
```


## Workflow

By default the database is not populated *fm_roundcube_initial_sql=False*. Let's configure Roundcube first (1-5) and populate the database later (6).

1) Change shell to /bin/sh

```
shell> ansible mailserver -e 'ansible_shell_type=csh ansible_shell_executable=/bin/csh' -a 'sudo pw usermod freebsd -s /bin/sh'
```

2) Install the role and collections

```
shell> ansible-galaxy role install vbotka.freebsd_mysql
shell> ansible-galaxy role install vbotka.freebsd_apache
shell> ansible-galaxy role install vbotka.freebsd_mailserver
shell> ansible-galaxy role install vbotka.freebsd_mailserver_roundcube
shell> ansible-galaxy collection install community.general
```

3) Fit variables, e.g. in vars/main.yml

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

5a) Check syntax and install packages or ports

```
shell> ansible-playbook freebsd-mailserver-roundcube.yml --syntax-check
shell> ansible-playbook freebsd-mailserver-roundcube.yml -t fm_roundcube_packages -e fm_roundcube_install=true
```

5b) Copy /usr/local/etc/php.ini-production to /usr/local/etc/php.ini if the target does not exist

```
shell> ansible-playbook freebsd-mailserver-roundcube.yml -t fm_roundcube_conf_php_ini_create
```

5c) Optionally create plugins default configuration files from *config.inc.php.dist*. Disable
webserver to prevent starting the webserver with the default configuration.

```
shell> ansible-playbook freebsd-mailserver-roundcube.yml -t fm_roundcube_plugins_conf_create
```

5d) Optionally run playbook in check and diff mode. The command will fail if the plugins'
configuration files are missing.

```
shell> ansible-playbook freebsd-mailserver-roundcube.yml --check --diff
```

5e) Install and configure Roundcube webmail. Run the command twice to make sure it is idempotent

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


## Plugins

By default these plugins are enabled: archive, enigma, managesieve, password, zipdownload. See other plugins in the directory */usr/local/www/roundcube/plugins/*.

### Archive

### Enigma

User's GnuPG data will be created in the dictionary stored in the variable *roundcube_enigma_pgp_homedir*

```
shell> tree /var/db/roundcube/enigma/
/var/db/roundcube/enigma/
└── user1
    ├── pubring.gpg
    └── secring.gpg
```

### Managesieve

### Password

### Zipdownload


## References

- [Roundcube webmail](https://roundcube.net/)
- [Roundcube - ArchLinux Wiki](https://wiki.archlinux.org/index.php/Roundcube)
- [FreeBSD Postfix – Page 13 – Roundcube Install](http://www.purplehat.org/?page_id=20)
- [Guide On How To Install Roundcube On FreeBSD](http://www.xfiles.dk/guide-on-how-to-install-roundcube-on-freebsd/)
- [Roundcube Community Forum](http://www.roundcubeforum.net/)
- [Enigma plugin (PGP encryption) Roundcube signature](https://www.saic.it/enigma-plugin-pgp-encryption-roundcube-signature/)


**TODO**

- add automatic_addressbook plugin
- configure sieve
- configure pspell


## License

[![license](https://img.shields.io/badge/license-BSD-red.svg)](https://www.freebsd.org/doc/en/articles/bsdl-gpl/article.html)


## Author Information

[Vladimir Botka](https://botka.link)
