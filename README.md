Ansible Role: Exim4
===================

[![Build Status](https://travis-ci.org/degtyarevalexey/ansible-role-exim4.svg?branch=master)](https://travis-ci.org/degtyarevalexey/ansible-role-exim4)

Ansbile role to install and configure Exim4 on Debian or Ubuntu system.

With that role you may fine tune your Exim4 installation using variables.

Role Variables
--------------

Variable | Default value | Description 
-------- | ------------- | -------------
**exim4_package_name** |  exim4-daemon-light | Sets the package name to install
**exim4_conf_keyvalue** | _empty_ | A lists of dictionaries of key-values (see below)
**exim4_conf_values** | _empty_ | A lists of dictionaries with one-per-line values (see below)

For **exim4_package_name** value good choice is also `exim4-daemon-heavy` or any other name available for your system.

### Using lists of dictionaries

**exim4_conf_keyvalue** used to create config files with key-value pairs of data.  For example, to configure a list of route_data records which can be used to override or augment MX information from the DNS: 
```yaml
exim4_conf_keyvalue:
  - name: hubbed_hosts
    data:
      example.com: mail.example.com
      example.net: mail.example.net
```
As a result, file `/etc/exim4/hubbed_hosts` will be created with key-value pairs of domain pattern and route data.

**exim4_conf_values** used to create config files with flat list values.  For example, to configure a list of envelope
recipients for which incoming messages are subject to recipient verification with a callout:
```yaml
exim4_conf_values:
  - name: local_rcpt_callout
    data:
      - "*@example.com"
      - "*@example.net"
```
As a result, file with address list `/etc/exim4/local_rcpt_callout` will be created.

For more info about files in use by the Debian exim4 package, please consult `man exim4-config_files`

### Maintaining update-exim4.conf.conf

The following variables and their default values are used for the content of
the `/etc/exim4/update-exim4.conf.conf` file:

Variable | Default value | Description 
-------- | ------------- | -------------
**exim4_dc_eximconfig_configtype** | internet | Mail server configuration type
**exim4_dc_other_hostnames** | ansible_hostname | Other destinations for which mail is accepted
**exim4_dc_local_interfaces** | 127.0.0.1 ; ::1 | IP-addresses to listen on
**exim4_dc_minimaldns** | false | Keep number of DNS-queries minimal |
**exim4_dc_localdelivery** | mail_spool | Delivery method for local mail
**exim4_dc_use_split_config** | false | Split configuration into small files?
**exim4_dc_mailname_in_oh** | true | Internal  use only
**exim4_dc_relay_nets** | _empty_ | Machines to relay mail for
**exim4_dc_relay_domains** | _empty_ | Domains to relay mail for
**exim4_CFILEMODE** | 644 | The octal file mode of the generated file

Smarthost-specific variables:

Variable | Default value | Description 
-------- | ------------- | -------------
**exim4_dc_smarthost** | _empty_ | IP address or host name of the outgoing smarthost
**exim4_dc_hide_mailname** | _empty_ | Hide local mail name in outgoing mail?
**exim4_dc_readhost** | _empty_ | Visible domain name for local users

For more info about the meaning of these configuration variables please consult
man page for `update-exim4.conf`.

The following variables are used in default template to configure Exim4:

* *exim4_custom_options*
* *exim4_features_enable*
* *exim4_features_disable*
* *exim4_passwd_client*: Account and password data for SMTP authentication when exim is authenticating as a client to some remote server as a list.

The following features are built into this role:

* *00_exim4-config_tls*: Enable TLS in Exim
* *02_exim4-custom_options*: Custom options to add into config
...

Usage
-----

Download a role onto your Ansible host using the ansible-galaxy command that
comes bundled with Ansible.

```shell
$ ansible-galaxy install degtyarevalexey.exim4
```

Define a role in your playbook and setup desired options.  For example:

```yaml
roles:
  - role: degtyarevalexey.exim4
```

The defaults installs `exim4-daemon-light` package and no additional
configuration made except the defaults for OS.

Note that this role adds default Exim user `Debian-exim` into group `ssl-cert`
to let the daemon to access SSL certificates and keys.

Setup desired options.  For example:
```yaml
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

exim4_conf_keyvalue:
  - name: hubbed_hosts
    data:
      example.com: mail.example.com

exim4_conf_values:
  - name: local_rcpt_callout
    data:
      - "*@example.com"
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
