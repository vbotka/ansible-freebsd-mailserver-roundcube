=====================================================
vbotka.freebsd_mailserver_roundcube 2.6 Release Notes
=====================================================

.. contents:: Topics
# BEGIN Commits 2.6.1
- Update python 3.11 in .travis.yml
- Update tests/test.yml playbook
- Start devel 2.6.1
# END Commits 2.6.1
# BEGIN Release notes 2.6.1
2.6.1
=====
Release Summary
---------------
Major Changes
-------------
Minor Changes
-------------
- Update python 3.11 in .travis.yml
- Update tests/test.yml playbook
- Start devel 2.6.1

Bugfixes
--------
Breaking Changes / Porting Guide
--------------------------------
# END Release notes 2.6.1


2.6.1
=====

Release Summary
---------------
Maintenance update

Major Changes
-------------

Minor Changes
-------------
* Update tests/test.yml playbook


2.6.0
=====

Release Summary
---------------
Ansible 2.17 update

Major Changes
-------------
* bsd_mysql_version: "81"
* bsd_php_version: "83"
* Supported 13.3, 14.0, 14.1

Minor Changes
-------------
* Update and fix lint.
* Update README
* Update debug, permissions
* Add var fm_roundcube_role_version
* Add files/favicon.ico; default fm_roundcube_copy_favicon=false
* Add tasks/audit.yml; default fm_roundcube_audit=false
* Add vars roundcube_data_dir

Bugfixes
--------

Breaking Changes / Porting Guide
--------------------------------
* Updated list roundcube_packages
* Updated template config-config.inc.php.j2
* Updated vars roundcube_* for config.inc.php / See defaults
