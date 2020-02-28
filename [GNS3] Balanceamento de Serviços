
Balanceamento de serviço com pfsense 2.4.4
Os clientes terão acesso ao servidor pela interface WAN do pfsense, onde o serviço esta na LAN.

Cenario:

![enter image description here](https://i.ibb.co/sQcPN3F/Pfsense-Load-Balance-Service.png)

Criar dois serviços em funcionamento(neste caso 2Server Web), colocalos na rede LAN do PfSense.

Criar no Pfsense um POOL(services>loadbalance>pool) para adicionar os servidores web para serem usados em conjunto.

![enter image description here](https://i.ibb.co/S0JFB0J/pfsense-POOL-image.png)


Criar Virtualserver(services>loadbalance>virtualserver) que irá representar o pool com os servidores adicionados anteriormente, usando o IP da wan do pfsense.


![enter image description here](https://i.ibb.co/kBk4W6B/pfsense-Virtual-Server-imag.png)criar regra(wan) de firewall para acesso ao serviço interno pela wan.

![enter image description here](https://i.ibb.co/NLbjqC8/pfsense-REGRA-imag.png)Com isso o acesso pela wan será permitido.
Caso o SV1 caia, o trafego é redirecionado para o SV2.
