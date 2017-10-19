**Conectar 2 Vlans no mesmo switch.**

Iremos conectar 2 vlans utilizando 1 switch, os PCs de uma Vlan consiguirá se comunicar com outros PCs da mesma Vlan atravéz do switch, para que os PCs de uma Vlan possa se comunicar com outros de Vlans distintas, usaremos um roteador para as Vlans possam se comunicar.
Usaremos a topologia abaixo.


----------


![enter image description here](https://uploaddeimagens.com.br/images/001/141/226/original/VlanSimples.png?1508436659) 


----------


**Começaremos adicionando os IPs nos 4 PCs.**

No console do PC1, digite:

    PC1> ip 192.168.1.10 255.255.255.0 192.168.1.1
No console do PC2, digite:

    PC2> ip 192.168.2.10 255.255.255.0 192.168.2.1
No console do PC3, digite:

    PC3> ip 192.168.1.20 255.255.255.0 192.168.1.1
No console do PC4, digite:

    PC4> ip 192.168.2.20 255.255.255.0 192.168.2.1

Com os IPs adiciodos, criamos as Vlans 10 e Vlan 20 e colocaremos os PCs em suas respectivas Vlans.

**Criando as Vlans 10 e 20 no switch1. Digite no console:**

    switch1# vlan database
    switch1# vlan 10
    switch1# vlan 20
    switch1# exit

PC1 e PC3 na Vlan10, PC2 e PC4 na Vlan 20.

**Adicionando os PC1 e PC3 na Vlan10. No switch1, digite:**

    switch1# conf t
    switch1# int f1/2
    switch1# switchport mode access
    switch1# switchport access vlan 10
    switch1# no sh
    switch1# exit
    
    switch1#int f1/4 
    switch1# switchport mode access
    switch1# switchport access vlan 10
    switch1# no sh
    switch1# exit


**Adicionando os PC2 e PC4 na Vlan10. No switch1, digite:**

    switch1# conf t
    switch1# int f1/3
    switch1# switchport mode access
    switch1# switchport access vlan 20
    switch1# no sh
    switch1# exit
    
    switch1#int f1/5
    switch1# switchport mode access
    switch1# switchport access vlan 20
    switch1# no sh
    switch1# exit (2x)
Feito isso, os PCs da mesma Vlan ja conseguem se comunicar, para se comunicar agora com outras Vlan devemos configurar a interface do switch1 como TRUNK e configurar as interfaces das Vlan 10 e 20 no R1.

**Colocando a interface com R1 como TRUNK, no switch1, digite:**

    switch1# conf t
    switch1# int f1/1
    switch1# switchport mode trunk
    switch1# CTRL + Z
    
As interfaces que serão adicionada nas Vlan pelo R1, serão com os IPs de Gateways dos PCs, seja da rede 1(PC1 e PC3) com a Vlan10, e também com a rede 2(PC2 e PC4) com a Vlan20.

**Configurar as Interfaces das Vlan em R1, digite:**

    R1# conf t
    R1# int f0/0
    R1# no sh
    R1# exit

    R1# int f0/0.10 
    R1# encapsulation dot1q 10
    R1# ip add 192.168.1.1 255.255.255.0
    R1# exit

    R1# int f0/0.20
    R1# encapsulation dot1q 20
    R1# ip add 192.168.2.1 255.255.255.0
    R1# exit 
    R1# ip routing
    R1# exit

Feito isso, os PCs conseguem se comunicar com a Vlan 10 e Vlan 20. Faça um teste de ping com PCs da mesma Vlan e também em Vlans diferentes.
