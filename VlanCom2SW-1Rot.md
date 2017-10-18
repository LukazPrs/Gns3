###**Conectar 2 Vlans diferentes**

Usaremos a topologia abaixo para conectar 2 Vlans distintas usando 2 switch e 1 roteador. Adicionaremos 4 PCs, dois em cada switch.

![enter image description here](https://uploaddeimagens.com.br/images/001/140/014/original/vlan005.png?1508353825)

Primeiro passo, adicionar IP nos PC1, PC2, PC3 e PC4 com seus respectivos gateways.

**No console do PC1, Digite:**

    PC1> ip 192.168.1.10 255.255.255.0 192.168.1.1
    
**No console do PC4, Digite:**

    PC4> ip 192.168.1.20 255.255.255.0 192.168.1.1

**No console do PC2, Digite:**

    PC2> ip 192.168.2.10 255.255.255.0 192.168.2.1
    
**No console do PC3, Digite:**

    PC3> ip 192.168.2.20 255.255.255.0 192.168.2.1

É necessário criar as Vlans que iremos utilizar, neste caso 2Vlan(Vlan 10 e Vlan 20).

**No console do Switch1, digite:**

    switch1# vlan database
    switch1# vlan 10
    switch1# vlan 20
    exit

Configurar a interface do switch1 para o PC1 com a Vlan 10, PC3 com a Vlan 20, switch1 para o switch2 e para a interface com o roteador como trunk.

**Adicionando a interface do switch1 com PC1 na Vlan10. No switch1 digite:**

    switch1# conf t
    switch1# int f1/2
    switch1# switchport mode access
    switch1# switchport access vlan 10
    switch1#  CTRL+Z


**Adicionando a interface do switch1 com PC3 na Vlan20. No switch1 digite:**

    switch1# conf t
    switch1# int f1/5
    switch1# switchport mode access
    switch1# switchport access vlan 20
    switch1#  CTRL+Z


----------
**Na interface com o switch2, faça o modo trunk:**

    switch1# conf t
    switch1# int f1/1
    switch1# switchport mode trunk
    switch1# no sh
    switch1#  CTRL+Z

**Faça o mesmo com a interface para o R1:**

    switch1# conf t
    switch1# int f1/0
    switch1# switchport mode trunk
    switch1# no sh
    switch1# CTRL+Z

Finalizado o switch1, configurar o switch2 da mesma maneira.

**Crie as Vlan 10 e 20 no switch2:**

    switch1# vlan database
    switch1# vlan 10
    switch1# vlan 20
    switch1# exit



**Adicionando a interface do switch com PC2 na Vlan20. No switch2 digite:**

    switch2# conf t
    switch2# int f1/0
    switch2# switchport mode access
    switch2# switchport access vlan 20
    switch2#  CTRL+Z


**Adicionando a interface do switch com PC4 na Vlan10. No switch2 digite:**

    switch2# conf t
    switch2# int f1/5
    switch2# switchport mode access
    switch2# switchport access vlan 10
    switch2#  CTRL+Z
    
----------
 

No roteador1(R1) será necessário configurar a porta TRUNK na interface com o switch1 e colocar IP nas interface da Vlan 10 e Vlan20.

**Adicionando IP para Vlan10:**

    R1# conf t
    R1# int f0/0
    R1# no sh
    R1# exit

    R1# int f0/0.10
    R1# encapsulation dot1q 10
    R1# ip add 192.168.1.1 255.255.255.0
    R1# exit

**Adicionando IP para Vlan20:**

    R1# int f0/0.20
    R1# encapsulation dot1q 20
    R1# ip add 192.168.2.1 255.255.255.0
    R1# CTRL + Z
    R1# conf terminal
    R1# ip routing
    R1# exit

Agora o PC1 como se comunicar com o PC2 que está em uma Vlan diferente.
Faça o ping do PC1 para o PC2 e vice-versa.

**Download ISOs dos switch e roteador:** [Roteador c3725](http://www.mediafire.com/file/f57mccrqfdpeiin/c3725-adventerprisek9-mz124-15.bin) | [Switch c3745](http://www.mediafire.com/file/p9m86m044yncsmm/c3745-advipservicesk9-mz.124-25d.bin)
[Ambos podem ser configurados como Roteador e Switch]
