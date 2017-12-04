**Atribuir IPs para as Vlans 10 e 20, cada uma com suas faixas de IPs.**
Usaremos um roteador para simular um Servidor DHCP, um switch para configurar duas Vlans e 4 Pcs que estarão em suas Vlans. Usaremos a Topologia abaixo.

![enter image description here](https://uploaddeimagens.com.br/images/001/148/750/original/Topologia.png?1508953080)

Primeiramente iremos Criar as Vlans10 e 20, colocar os PCs em suas respectivas Vlans e colocar a interface com o roteador/servidor como TRUNK para haver comunicação entre Vlans.
Criando as Vlan10 e Vlan20. No switch1, digite:

    switch1# vlan database
    switch1# vlan 10
    switch1# vlan 20
    switch1# exit

**Colocando o PC1 e PC2 na Vlan10.** 

    switch1# conf t
    switch1# interface fastEthernet 1/1
    switch1# switchport mode access
    switch1# switchporta access vlan 10
    switch1# no sh

    switch1# interface fastEthernet 1/2
    switch1# switchport mode access
    switch1# switchporta access vlan 10
    switch1# no sh
    switch1# exit (2x)

**Colocando o PC3 e PC4 na Vlan20.** 

    switch1# conf t
    switch1# interface fastEthernet 1/3
    switch1# switchport mode access
    switch1# switchporta access vlan 20
    switch1# no sh
    switch1# exit
    
    switch1# interface fastEthernet 1/4
    switch1# switchport mode access
    switch1# switchporta access vlan 20
    switch1# no sh
    switch1# CTRL + Z

**Colocar a interface com servidor como TRUNK:**

    switch1# conf t
    switch1# interface fastEthernet 1/5
    switch1# switchport mode trunk
    switch1# no sh
    switch1# exit


Configurar o Servidor DHCP para atribuir os IPs para os PCs nas Vlans, cada vlan como sua faixa de IP. Será criado duas pools para ser distribuidas IPs para cada Vlan.
**Atribuir IP nas vlans. No servidor dhcp, digite:**

    dhcp# conf t
    dhcp# int f0/0
    dhcp# no sh
    dhcp# exit

    dhcp#int f0/0.10 
    dhcp# encapsulation dot1q 10
    dhcp# ip add 10.10.10.1 255.255.255.0
    dhcp# exit

    dhcp#int f0/0.20 
    dhcp# encapsulation dot1q 20
    dhcp# ip add 20.20.20.1 255.255.255.0
    dhcp# exit  (2x)

Criar as pools de distribuição de IPs para os PCs nas Vlans.

    dhcp# cont f
    dhcp# ip dhcp pool Vlan-10
    dhcp# default-router 10.10.10.1
    dhcp# network 10.10.10.0 255.255.255.0
    dhcp# exit

    dhcp# ip dhcp pool Vlan-20
    dhcp# default-router 20.20.20.1
    dhcp# network 20.20.20.0 255.255.255.0
    dhcp# exit (2x)

 Depois de configurado, para adicionar os IPs nos PCs, abra o console de cada PC e digite o comando:

     ip dhcp

 Será buscado o servidor dhcp e adiquirido o IP e mascara configurada, os PCs da Vlan 10 pegarão IPs do tipo 10.10.10.0 e Vlan 20 do tipo 20.20.20.0.
