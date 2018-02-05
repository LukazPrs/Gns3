**Comunicação entre dois PCs de redes diferentes.**

Para fazer a comunicação de dois PCs em redes diferentes, usaremos um ROTEADOR do modelo c3725 e fazer a ligação dos computadores a esse roteador, cada PC estará em uma rede diferente. (Figura abaixo)

![enter image description here](https://uploaddeimagens.com.br/images/001/137/061/original/Gns2.png?1508179251)

Primeiramente adicionaremos os IPs nos PC1 e PC2.

**CONFIGURAÇÃO DO PC1:**

> *IP: 192.168.1.10*.
> *MASCARA: 255.255.255.0*.
> *GATEWAY: 192.168.1.1*.

Abra o console do PC1 e coloque os dados acima.

    PC1>  ip 192.168.1.10 255.255.255.0 192.168.1.1

**CONFIGURAÇÃO DO PC2:**

> *IP: 192.168.2.20.*
> *MASCARA: 255.255.255.0.*
> *GATEWAY: 192.168.2.1.*

Abra o console do PC2 e coloque os dados acima.


    PC2>  ip 192.168.2.20 255.255.255.0 192.168.2.1

Feito isso, os IPs estarão configurados, assim precisando somente configurar o roteador para que uma rede possa enxergar a outra.
Como o roteador esta conectado a dois PCs diferentes, cada computador terá sua interface para o qual esta ligada, cada uma das interfaces terá que ser configurada com um ip que será o Gateway do PC da interface, assim adicionando dois IPs, um para cada interface.

**Configuração do Roteador**

    conf terminal
    interface fastEthernet 0/0
    ip add 192.168.1.1 255.255.255.0
    no shutdown
    exit
    
    interface fastEthernet 0/1
    ip add 192.168.2.1 255.255.255.0
    no shutdown
    exit (2x)

Feito isso, as redes 1 e 2 poderão se comunicar normalmente.
A rede 1 consiguirá se comunicar com a rede 2, pois o roteador possui ambas interfaces configuradas.

**Teste de ping do PC1 ao PC2.**

    PC1> ping 192.168.2.20

**Teste de ping do PC2 ao PC1.**

    PC2> ping 192.168.1.10


Download da ISO usada: [Roteador c3725](http://www.mediafire.com/file/f57mccrqfdpeiin/c3725-adventerprisek9-mz124-15.bin)
