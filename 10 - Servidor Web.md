Fazer um servidor Web usando Nginx no Ubuntu, usando a topologia do tutorial anterior e adicionando mais uma maquina virtual para ser nosso servidor web.

![enter image description here](https://uploaddeimagens.com.br/images/001/264/201/original/ServerWEB.png?1516904426)
Primeiramente baixaremos o servidor web(Nginx).

	#> sudo apt-get install nginx

Depois de baixado, iremos editar a pagina index.html padrão do servidor web. Entre no seguinte diretório: **cd /var/www/html**
Entre como root usando a interface gráfica: **sudo nautilus /var/www/html/**
Edite o arquivo index.html como desejar, essa será sua página padrão.
Depois de construído a página, adicionaremos um IP estático para o endereço do servidor web como feito no servidor DNS.
	
	#> sudo nano /etc/network/interfaces

![enter image description here](https://uploaddeimagens.com.br/images/001/264/235/original/IpWeb.png?1516906938)
Ip adicionado para o servidor web é 192.168.1.100.
Reinicie o ubuntu para o IP ser adicionado.
Depois disso vamos colocar em nosso servidor DNS, o domínio para que possamos acessar do nosso PC01 ou PC02 a página do servidor web via domínio.
Para isso, vamos adicionar uma linha em dois arquivo do nosso DNS.
No servidor DNS, entre no diretório: **cd /etc/bind/**  e depois abra o arquivo de zona direta que você escolheu o nome: **sudo nano db.risc.net**

![enter image description here](https://uploaddeimagens.com.br/images/001/264/284/original/web-IP1.png?1516908002)
Adicionamos um nome para o servidor(web) e adicionamos o seu IP(192.168.1.100)
Agora adicionaremos também no arquivo de zona reversa: **sudo nano db.192**.

![enter image description here](https://uploaddeimagens.com.br/images/001/264/301/original/web-IP2Reverse.png?1516908416)
Reinicie o servidor dns(bind): **sudo /etc/init.d/bind9 restart**
Agora os PCs ja conseguem acessar o servidor web usando o domínio e seu nome que foi adicionado no DNS.
Não esqueça de adicionar o servidor criado como servidor DNS de nossos PCs no arquivo resolv.conf como no tutorial passado.
Feito isso, abra o navegador e acesso a página do servidor web via domínio:
**web.risc.net**

![enter image description here](https://uploaddeimagens.com.br/images/001/264/352/original/paginaWEb.png?1516909959)
Feito isso, tudo estará funcionando normalmente e você poderá criar outros domínios para outros serviços e dispositivos.
