**Conectar 2 PCs utilizando um switch padrão**
----------------------------------------------

Primeiramente abrir o GNS3 e montar a topologia(Foto abaixo).
Usaremos os dois PCs na mesma rede, esses PCs estarão ligados diretamente a um switch somente.

![enter image description here](https://uploaddeimagens.com.br/images/001/133/776/original/Conectar_2Pcs2.png?1507922374)

No PC 1 e 2, utilizaremos IPs de classe C com o seguinte IP, PC1: 192.168.1.1 e PC2: 192.168.1.2, ambos com mascara de rede 255.255.255.0. 

## Adicionando os IPs nos PCs ##
Iniciar todos dispositivos e abrir os Consoles dos PC1 e PC2, no console do PC1 será colocado o IP: 192.168.1.1.
Digitar o seguinte comando no console do PC1:

> **PC1:   ip 192.168.1.1 255.255.255.0**

O IP ao PC1 foi adicionado. Fazer o mesmo com PC2.
Digitar o seguinte comando no console do PC2:

> **PC2:   ip 192.168.1.2 255.255.255.0**

Feito isso, os PCs estão com seus respectivos IPs e já podem fazer um teste de conexão através do comando ping.

Para pingar do PC1 ao PC2, utilize o comando ping com o IP do PC2.

>  **PC1:  ping 192.168.1.2**

Para pingar do PC2 ao PC1, utilize o comando ping com o IP do PC1.

>  **PC2:  ping 192.168.1.1**


Feito isso, os PCs já estarão conectados!
