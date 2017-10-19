**Conectar 2 Vlans no mesmo switch.**

Iremos conectar 2 vlans utilizando 1 switch, os PCs de uma Vlan consiguirá se comunicar com outros PCs da mesma Vlan atravéz do switch, para que os PCs de uma Vlan possa se comunicar com outros de Vlans distintas, usaremos um roteador para as Vlans possam se comunicar.
Usaremos a topologia abaixo.

![enter image description here](https://uploaddeimagens.com.br/images/001/141/226/original/VlanSimples.png?1508436659) 

Começaremos adicionando os IPs nos 4 PCs.

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
switch1# 
