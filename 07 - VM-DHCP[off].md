**Criar um servidor DHCP simples em uma VM Linux Ubuntu**

Usaremos o Virtual Box como VM que será integrada ao GNS3 e usaremos a seguinte topologia, imagem abaixo.

...

Para o funcionamento deste tutorial, será necessário a instalação do programa  *Virtual Box*, ter um sistema operacional Ubuntu(ou derivados). Usaremos a VM para configurar um servidor DHCP simples que distribuirá IPs para PCs(VPCS do Gns3).
Ao baixar o Virtual Box(ou outra VM) é necessário criar uma nova maquina virtual com o sistema linux ubuntu, depois abrir o Gns3 e abrir as *Preferencias* e ir na aba **Virtual Box** e clicar em **Novo**, sera aberto uma janela com os sistemas operacionais que foram adicionados ao Virtual Box, escolha a maquina Ubuntu e finalize com Aplicar. A maquina virtual estará disponível em seus dispositivos no Gns3, restando somente montar a topologia seguindo a nossa imagem acima.
Ao iniciar a topologia no Gns3, o virtual box iniciará o sistema operacional Ubuntu junto com os PCs virtuais.


![Adicionando Iso no Gns3](https://uploaddeimagens.com.br/images/001/198/604/original/menuVM.png?1512413307)

Ao abrir a máquina virtual ubuntu, siga os passos do link do tutorial abaixo para configurar dentro do ubuntu um servidor dhcp simples, que distribuirá IPs para os VPCS do gns3.   [**LINK DO TUTORIAL**](http://www2.unemat.br/robinho/LABREDES/material/DhcpServer.txt)


Depois de configurado o servidor DHCP no ubuntu, entro no terminal dos VPCS e digitem o seguinte comando abaixo para que os PCs possam procurar um servidor dhcp na rede e fazer uma requisição de configuração IP.

Em todos VPCS, digite:

> PC> ip dhcp

Feito isso, os VPCs pegarão uma configuração que foi definida no servidor dhcp e o tutorial estará finalizado.
