**Conectar 2 Vlans atravéz de 2 switches e 1 roteador.**
Os PCs de

----------


![enter image description here](https://uploaddeimagens.com.br/images/001/141/362/original/vlan005.png?1508441434)


----------


Usaremos 2Vlans(Vlan10 e Vlan20) e 2 PCs em cada switch. 
Primeiro colocamos os IPs nos 4 PCs.
**Adicionando IP no PC1:**

    PC1> ip 192.168.1.10 255.255.255.0 192.168.1.1

**Adicionando IP no PC2:**

    PC2> ip 192.168.2.10 255.255.255.0 192.168.2.1

**Adicionando IP no PC3:**

    PC3> ip 192.168.2.20 255.255.255.0 192.168.2.1

**Adicionando IP no PC4:**

    PC4> ip 192.168.1.20 255.255.255.0 192.168.1.1

Criar as Vlans 10 e 20 no switch1.
**No switch1, digite:**

    switch1# vlan database
    switch1# vlan 10
    switch1# vlan 20
    switch1# exit


Depois de adicionar os IPs e criar as Vlans, adicionar o PC1 e PC3 em suas Vlans no switch1. Colocar o PC1 na Vlan10 e PC3 na Vlan20.

**No switch1, digite:**

    switch1# conf t
    switch1# int f1/2
    switch1# switchport mode access
    switch1# switchport access vlan 10
    switch1# exit

    switch1#int f1/5 
    switch1# switchport mode access
    switch1# switchport access vlan 20
    switch1# exit (2x)

**Configurar a interface com outro switch2(1/1) com modo TRUNK e também com o R1(1/0).**

    switch1# conf t
    switch1# int f1/1
    switch1# switchport mode trunk
    switch1# exit
    switch1# int f1/0
    switch1# switchport mode trunk
    switch1# exit (2x)

Finalizado o switch1, configurar o switch2 igualmente.

Criando as Vlan 10 e 20 no switch2.
**No switch2, digite:**

    switch1# vlan database
    switch1# vlan 10
    switch1# vlan 20
    switch1# exit
    
**Adicionando o PC2 na Vlan20 e PC4 na Vlan10.**

switch2# conf t
switch2# int f1/0
switch2# switchport mode access
switch2# switchport access vlan 20
switch2#  exit

switch2# int f1/5
switch2# switchport mode access
switch2# switchport access vlan 20
switch2# exit

switch2# int f1/1
switch2# switchport mode trunk
switch2# CTRL + Z

Finalizado o switch2, falta somente configurar as interfaces da Vlan10 e 20 em R1. Usaremos os IPs dos gateways das rede 1 e rede 2.

**Em R1:**
R1# conf t
R1# int f0/0
R1#  no sh
R1# exit

R1# int f0/0.10
R1# encapsulation dot1q 10
R1# ip add 192.168.1.1255.255.255.0
R1# exit

R1# int f0/0.20
R1# encapsulation dot1q 10
R1# ip add 192.168.2.1255.255.255.0
R1# exit
R1# ip routing
R1# exit

Feito isso, os PCs de uma Vlan ja conseguiraõ se comunicar com outros PCs de outra Vlan.


----------


Download ISOs dos switch e roteador: [Roteador c3725](http://www.mediafire.com/file/f57mccrqfdpeiin/c3725-adventerprisek9-mz124-15.bin) || [Switch c3725](http://www.mediafire.com/file/p9m86m044yncsmm/c3745-advipservicesk9-mz.124-25d.bin)
[Ambos podem simular roteador e também switch]
