## Salt tools for bare-metal provisioning

Other projects that use this repository:
- [kilo-saltstack](https://github.com/dkilcy/kilo-saltstack) - OpenStack 3+ node architecture on CentOS 7
- [juno-saltstack](https://github.com/dkilcy/juno-saltstack) - OpenStack 3+ node architecture on CentOS 7

### Introduction

Use SaltStack (Salt) in conjunction with PXE server/kickstart to install and provision multiple bare-metal machines running CentOS.

In this project, the Salt masters are installed manually, and the minions are installed via PXE/kickstart.  

Bare-metal machines take on one of two roles:
- Salt master 
- Salt minion 

### Lab Infrastructure

- 3 [MintBox2](http://www.fit-pc.com/web/products/mintbox/mintbox-2/)
- 10 [Supermicro SYS-5108A](http://www.newegg.com/Product/Product.aspx?Item=N82E16816101837)
- 2 [TP-Link TL-SG-3216 L2 Switches](http://www.tp-link.com/lk/products/details/cat-39_TL-SG3216.html)
- 2 [TP-Link TL-SG-3424 L2 Switches](http://www.tp-link.com/lk/products/details/cat-39_TL-SG3424.html)
- 2 [Dell Powerconnect 6224 L3 Switches](http://www.dell.com/us/business/p/powerconnect-6200-series/pd)

The MintBox2 machines are the Salt masters running CentOS 7 with the MATE desktop.  The Supermicros are the Salt minions running CentOS 6 or 7.

- [/etc/hosts](states/network/files/hosts) file
- [/etc/dhcp/dhcpd.conf](states/dhcp/files/dhcpd.conf) file

Network infrastructure is described [here](notes/network-setup.md)

### Lab Setup

1. [Install CentOS 7 on MintBox2](notes/centos-7-manual.md)
2. [Setup Salt Master and Minion on MintBox2](notes/setup-salt.md) 
2. [Setup Git and saltstack-base repository on MintBox2](notes/saltstack-base-setup.md)
3. [Setup PXE Server on MintBox2](notes/pxe-server-setup.md)
4. [Install Supermicros (or other MintBox2) via PXE Server](notes/pxe-install.md)

#### Assigning Roles to Machines

 ```bash
salt '<id>' grains.setvals "{'saltstack-base:{'role':'master'}}"
salt '<id>' grains.setvals "{'saltstack-base:{'role':'minion'}}"
```

#### Useful Commands

Common:
- Run a command: `salt '*' cmd.run 'date'`
- Service restart: `salt '*' service.restart ntp`
- View a file: `salt '*' cp.get_file_str /etc/hosts`
- Force a pillar refresh:  `salt '*' saltutil.refresh_pillar`
- Sync all: `salt '*' saltutil.sync_all`
- Calling Highstate: `salt '*' state.highstate`
- Copy a file: `salt-cp '*' /local/file /remote/file`

Debug and output options:
- Output data using pprint: `salt 'store1' grains.items --output=pprint`
- Output data using json: `salt 'store1' grains.items --output=json`
- State output mixed: `salt 'store1' test.ping --state-output=mixed`
- Debug level: `salt 'store1' --log-level=debug --state-output=mixed state.highstate test=True`'
- Verbose: `salt -v --log-level=debug --state-output=mixed 'store1' state.highstate test=True`
- Version: `salt --version`

Jobs:
- Lookup result of a job: `salt-run jobs.lookup_jid 20150627120734094928`

### References

- [Salt Module Index](http://docs.saltstack.com/en/latest/salt-modindex.html)

 
