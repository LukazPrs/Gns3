**Criando um servidor DNS no Ubuntu.**

Usaremos a topologia abaixo para criar um domínio para que dois PCs consigam se comunicar por este domínio e seu nome e não usando o endereço IP.
![enter image description here](https://uploaddeimagens.com.br/images/001/253/597/original/DNS.png?1516209228)
Usaremos duas máquinas virtuais para demonstrar este servidor Dns. Na figura acima usamos duas Máquinas Virtual com sistema Ubuntu e o servidor DNS com as mesmas especificações.

Nas Maquinas Virtuais:

	PC1 #> sudo ifconfig enp0s3 192.168.1.10
	PC2 #> sudo ifconfig enp0s3 192.168.1.30


Configurando o servidor DNS.
O servidor DNS precisa de um IP estático para seu funcionamento, antes de baixar o servidor Dns(bind9), devemos colocar um Ip estático no nosso Ubuntu(confira sua interface de rede com ifconfig).

Abra o arquivo:
	#> sudo nano /etc/network/interfaces

Adicione na sua interface um IP de sua escolha como no exemplo abaixo.

![enter image description here](https://uploaddeimagens.com.br/images/001/253/725/original/interfaces.png?1516212734)

Depois de adicionar IP fixo, editaremos nosso domínio.

Abra o arquivo:  
		#> sudo nano /etc/hosts

Adicione uma nova linha com IP do servidor e o nome da máquina com seu domínio escolhido.
![enter image description here](https://uploaddeimagens.com.br/images/001/253/749/original/hosts.png?1516213813)
		
		IP do servidor:192.168.1.200
		Nome da Máquina: lucas-VirtualBox
		Domínio: risc.net
		(Use TAB para das espaço)


Usaremos o servidor DNS Bind9, para instalar este programa digite no terminal:
	
	#> sudo apt-get install bind9

Adicionaremos nossas Zonas de Dns.
	Entre no diretório do bind9: cd /etc/bind/
	Abra o arquivo: sudo nano /named.conf.local

![enter image description here](https://uploaddeimagens.com.br/images/001/253/775/original/zonas.png?1516215186)
Foi criado uma zona direta e uma reversa como mostra na imagem acima.([Vídeo completo sobre o passa a passo](https://www.youtube.com/watch?v=0SSSfyy7bO4))

Fazer uma copia para as duas zonas criadas, essa copia será de um arquivo já existente e pré-configurado.

	#> sudo cp db.local db.risc.net
	#> sudo cp db.127 db.192

Temos que editar os dois arquivos, começando pelo **db.risc.net**.

	#> sudo nano db.risc.net
![enter image description here](https://uploaddeimagens.com.br/images/001/253/841/original/zonaDireta.png?1516217722)
[\[Siga o passo a passo aqui\]](https://www.youtube.com/watch?v=xZcf7TaxKHU)
Editando o arquivo de zona reversa.
	#> sudo nano db.192
![enter image description here](https://uploaddeimagens.com.br/images/001/253/857/original/zonaReversa.png?1516218271)
Feito isso reinicie a maquina virtual.

#> sudo reboot

Testar se comunicar do PC1 ao PC2 usando domínio.
PC1 > ping vm3.risc.net

