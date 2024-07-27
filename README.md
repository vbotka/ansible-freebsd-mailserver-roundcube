# freebsd_mailserver_roundcube

[![quality](https://img.shields.io/ansible/quality/27910)](https://galaxy.ansible.com/vbotka/freebsd_mailserver_roundcube)
[![Build Status](https://travis-ci.org/vbotka/ansible-freebsd-mailserver-roundcube.svg?branch=master)](https://travis-ci.org/vbotka/freebsd-mailserver-roundcube)
[![GitHub tag](https://img.shields.io/github/v/tag/vbotka/ansible-freebsd-mailserver-roundcube)](https://github.com/vbotka/ansible-freebsd-mailserver-roundcube/tags)

[Ansible role.](https://galaxy.ansible.com/vbotka/freebsd_mailserver_roundcube/) FreeBSD. Install and configure [Roundcube](https://roundcube.net/) webmail.

Feel free to [share your feedback and report issues](https://github.com/vbotka/ansible-freebsd-mailserver-roundcube/issues).

[Contributions are welcome](https://github.com/firstcontributions/first-contributions).


## Requirements and dependencies

### Packages

- PHP
- Only Apache and MySQL are supported by this role.
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

1) Configure MySQL password for user *roundcube*

```yaml
roundcube_mysql_password: "MYSQL-PASSWORD"
```

This password has been used by [vbotka.freebsd_mysql](https://galaxy.ansible.com/vbotka/freebsd_mysql/) to grant privileges to user *roundcube@localhost*

```bash
GRANT ALL PRIVILEGES ON roundcube.* TO roundcube@localhost IDENTIFIED BY 'MYSQL-PASSWORD';
```

## Defaults (selected)

```yaml
fm_roundcube_install: true
fm_roundcube_debug: false
fm_roundcube_debug_classified: false
fm_roundcube_backup_conf: false
fm_roundcube_copy_favicon: true
fm_roundcube_initial_sql: false

roundcube_mysql_password: MYSQL-PASSWORD
roundcube_imap_host: imap.example.com
roundcube_smtp_host: smtp.example.com
roundcube_des_key: 'MY_DES_KEY'
roundcube_support_url: www.example.com/support/
roundcube_product_name: Roundcube Webmail
roundcube_skin: elastic

roundcube_zoneinfo: UTC
```

Notes:

 * Change passwords and fir URL
 * Make sure roundcube_skin is available in roundcube/skins


## Workflow

By default the database is not populated *fm_roundcube_initial_sql=False*. Let's configure Roundcube first (1-5) and populate the database later (6).

1) Change shell to /bin/sh

```bash
shell> ansible mailserver -e 'ansible_shell_type=csh ansible_shell_executable=/bin/csh' -a 'sudo pw usermod freebsd -s /bin/sh'
```

2) Install the role and collections

```bash
shell> ansible-galaxy role install vbotka.freebsd_mysql
shell> ansible-galaxy role install vbotka.freebsd_apache
shell> ansible-galaxy role install vbotka.freebsd_mailserver
shell> ansible-galaxy role install vbotka.freebsd_mailserver_roundcube
shell> ansible-galaxy collection install community.general
```

3) Fit variables to your needs.

4) Create playbook and inventory

```bash
shell> cat freebsd-mailserver-roundcube.yml
- hosts: mailserver
  roles:
    - role: vbotka.freebsd_mailserver_roundcube
```

```ini
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

```bash
shell> ansible-playbook freebsd-mailserver-roundcube.yml --syntax-check
shell> ansible-playbook freebsd-mailserver-roundcube.yml -t fm_roundcube_packages -e fm_roundcube_install=true
```

5b) Copy /usr/local/etc/php.ini-production to /usr/local/etc/php.ini if the target does not exist

```bash
shell> ansible-playbook freebsd-mailserver-roundcube.yml -t fm_roundcube_conf_php_ini_create
```

5c) Optionally create plugins default configuration files from *config.inc.php.dist*. Disable
webserver to prevent starting the webserver with the default configuration.

```bash
shell> ansible-playbook freebsd-mailserver-roundcube.yml -t fm_roundcube_plugins_conf_create
```

5d) Optionally run playbook in check and diff mode. The command will fail if the plugins'
configuration files are missing.

```bash
shell> ansible-playbook freebsd-mailserver-roundcube.yml --check --diff
```

5e) Install and configure Roundcube webmail. Run the command twice to make sure it is idempotent

```bash
shell> ansible-playbook freebsd-mailserver-roundcube.yml
```

6) Populate Roundcube database

```bash
shell> ansible-playbook freebsd-mailserver-roundcube.yml -t fm_roundcube_initial_sql -e fm_roundcube_initial_sql=true
```

### Install Roundcube

The directory `/usr/local/www/roundcube/installer/` keeps the files of the Roundcube installation wizard. Manually enable the installation in `/usr/local/www/roundcube/config/config.inc.php`

```ini
shell> grep enable_installer /usr/local/www/roundcube/config/config.inc.php
$config['enable_installer'] = true;
```

Then, open the installer in browser `https://mail.example.com/installer`

When you finish disable the installer and the whole installer folder
from the document root. Quoting:

> After completing the installation and the final tests please remove
  the whole installer folder from the document root of the webserver
  or make sure that enable_installer option in config.inc.php is
  disabled. These files may expose sensitive configuration data like
  server passwords and encryption keys to the public. Make sure you
  cannot access this installer from your browser.

### Audit

Optionally, run audit before you publish the site

```bash
shell> ansible-playbook freebsd-mailserver-roundcube.yml -t fm_roundcube_audit -e fm_roundcube_audit=true
```

### Test the webmail

* http://validator.w3.org
* https://www.ssllabs.com


## Plugins

By default these plugins are enabled: archive, enigma, managesieve, password, zipdownload. See other plugins in the directory */usr/local/www/roundcube/plugins/*.

### Archive

### Enigma

User's GnuPG data will be created in the dictionary stored in the variable *roundcube_enigma_pgp_homedir*

```bash
shell> tree /var/db/roundcube/enigma/
/var/db/roundcube/enigma/
└── user1
    ├── pubring.gpg
    └── secring.gpg
```

### Managesieve

### Password

### Zipdownload


## Ansible lint

Use the configuration file *.ansible-lint.local* when running
*ansible-lint*. Some rules might be disabled and some warnings might
be ignored. See the notes in the configuration file.

```bash
shell> ansible-lint -c .ansible-lint.local
```


## References

- [Roundcube webmail](https://roundcube.net/)
- [Roundcube - ArchLinux Wiki](https://wiki.archlinux.org/title/Roundcube)
- [Roundcube Webmail Project](https://github.com/roundcube)
- [Roundcube Setup - Purplehat](https://www.purplehat.org/?page_id=1593)
- [Roundcube Community Forum](http://www.roundcubeforum.net/)


**TODO**

* configure config/defaults.inc.php
- add automatic_addressbook plugin
- configure sieve
- configure pspell


## License

[![license](https://img.shields.io/badge/license-BSD-red.svg)](https://www.freebsd.org/doc/en/articles/bsdl-gpl/article.html)


## Author Information

[Vladimir Botka](https://botka.info)
