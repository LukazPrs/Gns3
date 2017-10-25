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


Configurar o Servidor DHCP para atribuir os IPs para os PCs nas Vlans, cada vlan como sua faixa de IP.

