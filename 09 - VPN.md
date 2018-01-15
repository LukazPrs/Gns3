**Conexão entre dois PCs(VM UBUNTU) em redes diferentes.**

Usaremos a topologia abaixo para montar uma conexão VPN entre dois PCs usando o OpenVPN.
![](https://uploaddeimagens.com.br/images/001/250/977/original/VPN_TOPOLOGIAt.png?1516038467)

Usaremos duas máquinas virtuais Ubuntu para fazer a conexão, cada maquina estará em uma rede distinta da outra. Cada Máquina esta ligada em um switch simples e este switch a um roteador. Antes de começar a instalar e configurar a VPN nos PCs com Ubuntu, teremos que atribuir os IPs aos PCs e interfaces do roteador, depois fazer um roteamento para que os PCs da Lan1 consiga se comunicar com os PCs da Lan2.
Primeiro vamos adicionar IP nos PCs da Lan1:

*Linux_ubu1: Abra o terminal na máquina virtual e digite o seguinte comando para adicionar IP na interface enp0s3(ou outra de sua escolha)*

	Linux_ubu-1 > sudo ifconfig enp0s3 192.168.1.10
	Linux_ubu-1 > sudo route add default gw 192.168.1.1   (adicionar gateway do R1)

	PC1 > ip 192.168.1.20 255.255.255.0 192.168.1.1

*Linux_ubu-2: Abra o terminal na máquina virtual e digite o seguinte comando para adicionar IP na interface enp0s3(ou outra de sua escolha)*
	
	Linux_ubu-2 > sudo ifconfig enp0s3 192.168.3.30
	Linux_ubu-2 > sudo route add default gw 192.168.3.2 (adicionar gateway do R2)
	
	PC2 > ip 192.168.3.20 255.255.255.0 192.168.3.2

Adicionando IP nas interfaces dos roteadores.

**Roteador 1 - R1#**

	R1# conf t
	R1# int f0/1
	R1# ip add 192.168.1.1 255.255.255
	R1# exit

	R1# int 0/0
	R1# ip add 192.168.2.1 255.255.255
	R1# end

**Roteador 2 - R2#**

	R2# conf t
	R2# int f0/1
	R2# ip add 192.168.3.2 255.255.255
	R2# exit

	R2# int 0/0
	R2# ip add 192.168.2.2 255.255.255
	R2# end

**Fazer o roteamento.**

	R1# conf t
	R1# ip route 192.168.3.0 255.255.255.0 192.168.2.2
	R1# end

	R2# conf t
	R2# ip route 192.168.1.0 255.255.255.0 192.168.2.1
	R2# end

Feito isso, a Lan 1 consiguirá acessar a Lan 2 e vice versa.

**Baixar o OpenVPN no Linux_ubuntu-1(servidor).**

*Baixando o OpenVPN*

	Linux_ubu-1 > sudo apt-get install openvpn

**Baixar o OpenVPN no Linux_ubuntu-2(cliente).**

*Baixando o OpenVPN*
	
	Linux_ubu-2 > sudo apt-get install openvpn

**Criando um Túnel Simples entre os dois PCs.**

**No servidor(Linux_ubu-1):**

	Linux_ubu-1 > openvpn --dev tun --ifconfig 10.0.0.1 10.0.0.2

**No cliente(Linux_ubu-2):**
	
	Linux_ubu-2 > openvpn --remote 192.168.1.10 --dev tun --ifconfig 10.0.0.2 10.0.0.1

**Faça um teste pingando do cliente no tunel do servidor(10.0.0.1)**

	Linux_ubu-2 > ping 10.0.0.1
