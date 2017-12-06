**Criar um servidor DHCP simples em uma VM Linux Ubuntu**

Usaremos o Virtual Box como VM que será integrada ao GNS3 e usaremos a seguinte topologia para montagem do servidor DHCP.

![enter image description here](https://uploaddeimagens.com.br/images/001/201/294/original/VM_DHCP.png?1512574166)

Para o funcionamento deste tutorial, será necessário a instalação do programa  *Virtual Box* e ter um sistema operacional Ubuntu(ou derivados). Usaremos a VM Ubuntu para configurar um servidor DHCP simples que distribuirá IPs para os PCs(VPCS do Gns3).

![enter image description here](https://uploaddeimagens.com.br/images/001/201/384/original/vm.png?1512577996)

Ao baixar o Virtual Box é necessário criar uma nova máquina virtual com o sistema linux ubuntu, depois abrir o Gns3 e abrir as ***Preferencias*** e ir na aba ***Virtual Box*** e clicar em ***Novo**,* sera aberto uma janela com os sistemas operacionais que foram adicionados ao Virtual Box, escolha a maquina Ubuntu e finalize com Aplicar. 

A maquina virtual estará disponível em seus dispositivos no Gns3, restando somente montar a topologia seguindo a nossa imagem acima.
Ao iniciar a topologia no Gns3, o virtual box iniciará o sistema operacional Ubuntu junto com os PCs virtuais.


A imagem abaixo ilustra como adicionar uma ISO do Virtual Box no Gns3.
![Adicionando Iso no Gns3](https://uploaddeimagens.com.br/images/001/198/604/original/menuVM.png?1512413307)

Ao abrir a máquina virtual ubuntu, siga os passos do link do tutorial abaixo para configurar dentro do ubuntu um servidor dhcp simples, que distribuirá IPs para os VPCS do gns3.   [**LINK DO TUTORIAL 1/2**](https://www.youtube.com/watch?v=hqS_EuQA6pQ) | [**LINK DO TUTORIAL 2/2**](https://www.youtube.com/watch?v=0hfJEnYk_6A)


Depois de configurado o servidor DHCP no ubuntu, entre no terminal dos VPCS e digitem o seguinte comando abaixo para que os PCs possam procurar um servidor dhcp na rede e fazer uma requisição de configuração IP.

Em todos VPCS, digite:

> PC> ip dhcp

Feito isso, os VPCs pegarão uma configuração que foi definida no servidor dhcp e o tutorial estará finalizado.
