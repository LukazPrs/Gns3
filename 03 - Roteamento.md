**Comunicação entre dois PCs usando dois Roteadores.**

Usaremos dois roteadores centrais para fazer a comunicação entre dois PCs, as interfaces dos roteadores terão seus IPs, com isso faltando somente fazer o roteamento das redes fora de alcance.  

![imagem](https://uploaddeimagens.com.br/images/001/138/416/original/rede2rot.png?1508263501)

Como mostrado na figura, usaremos o IP 192.168.1.10 para o PC1 e 192.168.3.20 para o PC2. Vamos adicionar os IPs nos PCs.
O Gateway dos PCs deve ser o IP da interface conectada(ip do roteador) como mostra na figura acima.
**Adicionando IP ao PC1:**

    PC1>  ip 192.168.1.10 255.255.255.0 192.168.1.1
    

**Adicionando IP ao PC2:**

    PC2>  ip 192.168.3.20 255.255.255.0 192.168.3.1

É necessário configurar a interface dos roteadores, cada interface terá seu IP, sendo estas de redes diferentes.
Abrir o console do R1 e adicionar os IPs nas interfaces conectadas, neste caso interface fastEthernet 0/0 e 0/1.

**Adicionando IP nas interfaces do R1**

    R1# conf t
    R1# int f0/0
    R1# ip add 192.168.1.1 255.255.255.0
    R1# exit

    R1# conf t
    R1# int f0/1
    R1# ip add 192.168.2.1 255.255.255.0
    R1# exit

Os IPs foram adicionados e estas interfaces são visiveis ao PC1.

**Adicionando IP nas interfaces do R2**

    R2# conf t
    R2# int f0/0
    R2# ip add 192.168.2.2 255.255.255.0
    R2# exit

    R2# conf t
    R2# int f0/1
    R2# ip add 192.168.3.1 255.255.255.0
    R2# exit

Os IPs foram adicionados e estas interfaces são visíveis ao PC2, porém do PC2 não consegue ter acesso ao PC1 e vice-versa, para que o PC2 possa acessar o PC1 é necessário fazer um roteamento em R1 e R2 para que R1 possa acessar a rede 3.0 que esta distante, e o R2 ter acesso a rede 1.0.

Em R1 temos que fazer o roteamento até a rede 3.0 que está distante, para isso usamos o seguinte comando no roteador:

sintaxe: R1# ip route [ip da rede de acesso] [mascara] [ip do proximo salto]

    R1# conf t
    R1# ip route 192.168.3.0 255.255.255.0 192.168.2.2
    R1# exit

Em R2 temos que fazer o roteamento até a rede 1.0 que está distante, digitamos no console de R2:

    R2# conf t
    R2# ip route 192.168.1.0 255.255.255.0 192.168.2.1
    R2# exit

Feito isso, toda rede terá acesso a todos dispositivos.
Faça um teste de ping do PC1 ao PC2 ou qualquer outra interface e vice-versa.


Download da ISO usada: [Roteador c3725](http://www.mediafire.com/file/f57mccrqfdpeiin/c3725-adventerprisek9-mz124-15.bin)
