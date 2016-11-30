bvansomeren.jails
=================

Creates jails based on previously created jail templates (for example bvansomeren.freebsd-jail-template)  
The role doesn't use any extra utilities or scripts and just relies on ZFS and jail.conf(5)  
Extra packages can be installed from the host as required, further configuration should be done using Ansible into the created jail (TODO: Add examples)  

This role makes extensive use of the /etc/jail.conf file. In general at least one invocation of this role should set "freebsd\_jails\_set\_globals" to ensure the global variables are set at least once.  
Because this role uses ansible blockinfile it will only change the blocks in the configuration, leaving the other ones alone.  
The idea is that you can have many playbooks which invoke their own jails to the same host without collision (except of course on name, IP and locations)  

Requirements
------------

A Basic template must be available. In general I create the templates on one machine and use zfs send | recv to copy them around. You can easily and manually create a simple template without bvansomeren.freebsd-jail-template with the following instructions:

```
zfs create -o mountpoint=/usr/local/jails zroot/jails
zfs create -p zroot/jails/templates/11.0-RELEASE
fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/11.0-RELEASE/base.txz -o /tmp/base.txz
fetch ftp://ftp.freebsd.org/pub/FreeBSD/releases/amd64/11.0-RELEASE/lib32.txz -o /tmp/lib32.txz
tar -xvf /tmp/base.txz -C /usr/local/jails/templates/11.0-RELEASE
tar -xvf /tmp/lib32.txz -C /usr/local/jails/templates/11.0-RELEASE
cp /etc/resolv.conf /usr/local/jails/templates/11.0-RELEASE/etc/resolv.conf
cp /etc/localtime /usr/local/jails/templates/11.0-RELEASE/etc/localtime
env UNAME_r=11.0-RELEASE freebsd-update -b /usr/local/jails/templates/11.0-RELEASE fetch install
env UNAME_r=11.0-RELEASE freebsd-update -b /usr/local/jails/templates/11.0-RELEASE IDS
zfs snapshot zroot/jails/releases/11.0-RELEASE@latest
```

Role Variables
--------------

```
freebsd_jails_location:             "/usr/local/jails"
freebsd_jails_dataset:              "zroot/jails"
freebsd_jails_template_dataset:     "zroot/jails/templates"
```
```
freebsd_jails: []
```

A valid freebsd_jails should look like
```
freebsd_jails:
- name: newjail
  hostname: newjail.local.domain
  extra_opts:
  - name: exec.start
    option: ""
```
```
freebsd_jails_action:               "create,start"
```
allows finer grained control on what action to execute  
The default create,start are harmless to containers not listed and running containers in general  
create executes create actions  
start starts the jails that are configured  
stop stops the jails that are configured  
destroy destroys the configured jails  

Dependencies
------------

No hard dependencies other than obviously FreeBSD (and less obviously ZFS. Design choice)  
Could be useful to build the templates using bvansomeren.freebsd-jails-template
This was only tested on FreeBSD 11, but might work on 10

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: all
      user: barry
      sudo: true
      roles:
      - role: bvansomeren.freebsd-jails
        freebsd_jails_set_globals: yes
        freebsd_jails:
        - name: newjail
        hostname: newjail.local.domain
        extra_opts:
        - name: ip4.addr
          option: 10.0.1.53

License
-------

BSD

Author Information
------------------

Just reach out via Github
