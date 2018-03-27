**Rede Mesh Unleashed - Ruckus R600**
Este tutorial tem o objetivo de auxiliar o leitor na criação de uma rede Mesh usando rádio R600 da Ruckus, tendo como exemplo uma rede de teste com apenas 3 equipamentos.

Primeiro configurar AP-Master que será o AP ligado a internet(via cabo) e terá acesso a todos APs que serão Mesh-APs. A rede mesh a ser implementada seguirá a topologia abaixo.

![enter image description here](https://uploaddeimagens.com.br/images/001/348/122/original/topologia2.png?1522179556)

**Iniciando a configuração do zero.**
Instalar em cada R600 a versão mais atual do Firmware Unleashed(**200.5**).

**1 - Ao iniciar o *`setup`* pela primeira vez do Unleashed, ligue um cabo do radio até seu PC e coloque manualmente um IP da rede padrão do radio(192.168.0.1). Ao colocar o IP manualmente, acesse o radio através do IP 192.1698.0.1 usando um navegador.**

 - Na configuração será necessário adicionar um IP ao radio(ou deixar o padrão), ativar o modo gateway e escolher uma SSID e senha Mesh(tem que ser igual em todos APs), além de escolher uma rede que será a rede responsável pela conexão dos APs e clientes.
 
 - Escolher também um nome para SSID e senha da rede wireless.
 - Feito isso aguarde o radio criar a rede(5~10min).
 - Ao aparecer o SSID criado, o Master estará configurado já poderá ser acessado sem cabo.


**2- Conectar o AP-Master a internet**

 - Ligar o AP-Master a um switch com internet.
 

**3- Configurar os APs que serão integrados a rede do AP-Master.**

 - Instalar o Unleashed nos APs e fazer a mesma configuração do Master, colocando os APs na mesma rede do AP-Master.

 - Ativar o modo gateway e também o modo Mesh, usando a **mesma SSID e senha do Mesh** usada no AP-Master.
 - Feito isso aguarde os APs criarem a rede.



**4- Com o Master e os APs configurados na mesma rede, conecte todos em um mesmo switch para que  rede Mesh seja configurada automaticamente pelo AP-Master.**

 - Ao colocar o Master e os APs no mesmo switch, o Master deverá adicionar os APs a sua rede automaticamente.
 - Conectado a SSID do Master, você poderá ver em sua interface se os APs estão fazendo parte da rede do AP-Master como APs adicionais(como na imagem abaixo).
 
![enter image description here](https://uploaddeimagens.com.br/images/001/345/305/original/1.png?1522015512)

 - Depois dos APs estarem fazendo parte do AP-Master, o cabo ligado ao switch poderá ser desconectado e será somente necessário o cabo **PoE** ligado em cada  AP(se tornando a topologia abaixo).
 
 ![enter image description here](https://uploaddeimagens.com.br/images/001/348/119/original/topologia1.png?1522179539)
 
 - Somente o AP-Master estará ligado com cabo PoE e com o cabo ao switch com internet.
 - Na aba de Pontos de acesso, os APs estarão atribuídos como Mesh, isso para cada um dos APs adicionados.
 
![enter image description here](https://uploaddeimagens.com.br/images/001/347/886/original/4-aps.png?1522172880)

Feito isso, a rede mesh estará configurada e em funcionamento.


