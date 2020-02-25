### Utilizando pfsense com 1 interface de rede e switch gerenciável

Cenário
![pfsense com 1 interface rede(lan)](https://i.ibb.co/Tvj5Xp1/PFSENSE-1-INTERFACE.png)
1 - Criar projeto no gns3, inserir CLOUD com utilização da interface cabeada **[Interface 1/8]**

2 - Switch gerenciavel com ao menos 8 portas 

3 - Pfsense(2.4.4) instalado na VM VirtualBox  **[Interface 1/1]**

4 - PCs para LAN   **[Interfaces 1/2-4]**

5 - Roteador para rede wireless    **[Interfaces 1/5-7]**

### Configurar pfsense
Inserir o pfsense ao gns3, fazer sua instalação, ao iniciar criar a **VLAN20(em0.20)** onde esta **interface virtual** sera responsável pela conexão **WAN** ficando a interface física(em0) para utilização da LAN.

    WAN -> em0.20
    LAN   -> em0

### Configurar switch

    SWITCH - PORTAS
    porta 1 - trunk (PFSENSE)
    porta 2-4 vlan 1 (REDE LOCAL)
    porta 5-7 vlan 30 (REDE WIFI)
    porta 8 vlan 20 (WAN)
Criando vlans e configurando as portas do switch
```
switch1# vlan database
switch1# vlan 20 name WAN
switch1# vlan 30 name WIFI
switch1# exit
```

**TRUNK**
```
switch1# conf t
switch1# int f1/1
switch1# switchport mode trunk
switch1# exit
```
**WAN(net) na vlan20**
```
switch1# conf t
switch1# int f1/8
switch1# switchport mode access
switch1# switchport access vlan 20
switch1# exit
```
**WIFI na vlan30**
```
switch1# conf t
switch1# int f1/5-7     (interface 5, 6 e 7)
switch1# switchport mode access
switch1# switchport access vlan 30
switch1# exit
```
Vlan da LAN será a vlan padrão 1.

A interface WAN do pfsense(em0.20) já deve obter IP que vem da CLOUD(cabeada/modem)

**Logar na interface do pfsense**
Criar a vlan30 no pfsense
![enter image description here](https://i.ibb.co/gJjQJhV/vlans-imag.png)
Adicionando a interface WIFI
![enter image description here](https://i.ibb.co/Bw3gqrJ/pfsense-interface-imag.png)
**Configurando a interface WIFI**

    Habilitar interface
    Adicionar modo Estatico IPV4
    Adicionar IP (192.168.2.1/24)
    Salvar

**Habiltar DHCP Interface WIFI**
![enter image description here](https://i.ibb.co/mRHGt23/pfsense-dhcp-wifi-imag.png)
Configurar regra no pfsense para interface WIFI tenha acesso externo
![enter image description here](https://i.ibb.co/f8f0jmr/pfsense-regra-wifi-imag.png)

Finalizado esta configuração, a rede WIFI tera acesso externo e assim finalizando este cenário.


