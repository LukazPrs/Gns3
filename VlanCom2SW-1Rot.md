Conectar 2 Vlans diferentes

Primeiro passo, adicionar IP nos PC1 e PC2. No PC1 usaremos o IP 192.168.1.10 com gateway 192.168.1.1. No PC2 usaremos o IP 192.168.2.10 e Gateway 192.168.2.1.

**No console do PC1, Digite:**

    PC1> ip 192.168.1.10 255.255.255.0 192.168.1.1

**No console do PC2, Digite:**

    PC1> ip 192.168.2.10 255.255.255.0 192.168.2.1

É necessário criar as Vlans que iremos utilizar, neste caso 2Vlan(Vlan 10 e Vlan 20).
No console do Switch1, digite:
switch1# vlan database
switch1# vlan 10
switch1# vlan 20
exit

Configurar a interface do switch1 para o PC1 com a Vlan 10 e do switch para o switch2 e para o roteador como trunk.
**Adicionando a interface do switch com PC1 na Vlan10, No switch1 digite:**
switch1# conf t
switch1# int f1/2
switch1# switchport mode access
switch1# switchport access vlan 10
switch1#  CTRL+Z

**Na interface com o switch2, faça o modo trunk:**
switch1# conf t
switch1# int f1/1
switch1# switchport mode trunk
switch1# no sh
switch1# 
switch1#  CTRL+Z

**Faça o mesmo com a interface para o R1:**
switch1# conf t
switch1# int f1/0
switch1# switchport mode trunk
switch1# no sh
switch1#  CTRL+Z

Finalizado o switch1, configurar o switch2 da mesma maneira.

**Adicionando a interface do switch com PC2 na Vlan20, No switch2 digite:**
switch1# conf t
switch1# int f1/0
switch1# switchport mode access
switch1# switchport access vlan 20
switch1#  CTRL+Z

**Na interface com o switch1, faça o modo trunk:**
switch1# conf t
switch1# int f1/1
switch1# switchport mode trunk
switch1# no sh
switch1#  CTRL+Z
Finalizado o switch2.
No roteador1(R1) será necessário configurar a porta TRUNK na interface com o switch1 e colocar IP nas interface da Vlan 10 e Vlan20.
