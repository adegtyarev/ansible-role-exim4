Ansible Role: Exim4
===================

Ansbile role to install and configure Exim4 on Debian based system.

With that role you may fine tune your Exim4 installation using variables.

Role Variables
--------------

* *exim4_package_name*: exim4-daemon-light 

Sets the package name to install.  Good choice is also `exim4-daemon-heavy` or
any other name available for your system.

The following variables and their default values are used for the content of
the /etc/exim4/update-exim4.conf.conf file

* *exim4_dc_eximconfig_configtype*: internet
* *exim4_dc_other_hostnames*: "{{ ansible_hostname }}"
* *exim4_dc_local_interfaces*: '127.0.0.1 ; ::1'
* *exim4_dc_readhost*: ''
* *exim4_dc_relay_domains*: ''
* *exim4_dc_minimaldns*: 'false'
* *exim4_dc_relay_nets*: ''
* *exim4_dc_smarthost*: ''
* *exim4_CFILEMODE*: '644'
* *exim4_dc_use_split_config*: 'false'
* *exim4_dc_hide_mailname*: ''
* *exim4_dc_mailname_in_oh*: 'true'
* *exim4_dc_localdelivery*: 'mail_spool'

The following variables are used in default templates to configure Exim4:

* *exim4_custom_options*
* *exim4_features_enable*
* *exim4_features_disable*
* *exim4_passwd_client*: Account and password data for SMTP authentication when exim is authenticating as a client to some remote server as a list.

Usage
-----

Download a role onto your Ansible host using the ansible-galaxy command that
comes bundled with Ansible.

```
$ ansible-galaxy install degtyarevalexey.exim4
```

Define a role in your playbook and setup desired options.  For example:

```
  roles:
    - role: degtyarevalexey.exim4
```

The defaults installs `exim4-daemon-light` package and no additional
configuration made except the defaults for OS.

Note that this role adds default Exim user `Debian-exim` into group `ssl-cert`
to let the daemon to access SSL certificates and keys.

Setup desired options.  For example:

```
  roles:
    - role: degtyarevalexey.exim4
      exim4_package_name: exim4-daemon-heavy
      exim4_dc_localdelivery: dovecot_lmtp
      exim4_dc_local_interfaces: "{{ ansible_all_ipv4_addresses | join(',') }}"
      exim4_custom_options:
        - daemon_smtp_ports: "25 : 465 : 587"
        - rfc1413_query_timeout: 0s
        - smtp_banner: "ESMTP server ready $tod_full"
      exim4_features_enable:
        - name: 02_exim4-custom_options
          group: main
        - name: 30_exim4-config_dovecot_lmtp
          group: transport
      exim4_features_disable:
        - name: 30_exim4-config_examples
          group: auth

```

The following tags may be used to re-configure Exim4:

* *exim4*: runs all tasks in role unless explicitly disabled
* *exim4-reconfigure*: run re-confguration to apply updated parameters (if any)

License
-------

BSD

Author Information
------------------

* Alexey Degtyarev <alexey@renatasystems.org>
