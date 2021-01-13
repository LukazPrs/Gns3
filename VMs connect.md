## MAQUINA: CONTROLLER
 root:
 #### editar arquivo hostname

> nano /etc/hostname

    controller

#### editar arquivo ifcfg-enp0s3 

> nano /etc/sysconfig/network-scripts/ifcfg-enp0s3

    ONBOOT=yes


--
#### editar arquivo ifcfg-enp0s8  
 >nano /etc/sysconfig/network-scripts/ifcfg-enp0s8

    BOOTPROTO=static
    ONBOOT=yes

**[adicione]**

    IPADDR=10.0.0.11
    NETMASK=255.255.255.0
    GATEWAY=10.0.0.1

--
#### editar arquivo ifcfg-enp0s9

> nano /etc/sysconfig/network-scripts/ifcfg-enp0s9

    BOOTPROTO=none
    ONBOOT=yes
#### editar arquivo hosts: adicionar

> nano /etc/hosts

    10.0.0.11	controller
    10.0.0.31	compute
    10.0.0.41	storage

--

    ifup enp0s3
    dnf update -y

---
---
## MAQUINA: COMPUTE
 ---
 root:
#### editar arquivo hostname

> nano /etc/hostname

    compute

#### editar arquivo hosts: adicionar

> nano /etc/hosts

    10.0.0.11	controller
    10.0.0.31	compute
    10.0.0.41	storage

--
#### editar arquivo ifcfg-enp0s3

> nano /etc/sysconfig/network-scripts/ifcfg-enp0s3

    ONBOOT=yes


--
#### editar arquivo ifcfg-enp0s8  

> nano /etc/sysconfig/network-scripts/ifcfg-enp0s8

    BOOTPROTO=static
    ONBOOT=yes

**[adicione]**

    IPADDR=10.0.0.31
    NETMASK=255.255.255.0
    GATEWAY=10.0.0.1

--
#### editar arquivo ifcfg-enp0s9

> nano /etc/sysconfig/network-scripts/ifcfg-enp0s9

    BOOTPROTO=none
    ONBOOT=yes


ifup enp0s3
dnf update -y

## MAQUINA: STORAGE
 root:
 #### editar arquivo hostname

> nano /etc/hostname

    storage

#### editar arquivo hosts: adicionar

> nano /etc/hosts

    10.0.0.11	controller
    10.0.0.31	compute
    10.0.0.41	storage

#### editar arquivo ifcfg-enp0s3 

> nano /etc/sysconfig/network-scripts/ifcfg-enp0s3

    ONBOOT=yes

#### editar arquivo ifcfg-enp0s8  

> nano /etc/sysconfig/network-scripts/ifcfg-enp0s8

    BOOTPROTO=static
    ONBOOT=yes

**[adicione]**

    IPADDR=10.0.0.41
    NETMASK=255.255.255.0
    GATEWAY=10.0.0.1

#### editar arquivo ifcfg-enp0s9

> nano /etc/sysconfig/network-scripts/ifcfg-enp0s9

    BOOTPROTO=none
    ONBOOT=yes

ifup enp0s3
dnf update -y

---
---

---
---




