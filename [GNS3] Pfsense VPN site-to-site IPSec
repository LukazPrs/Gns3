**

## VPN site-to-site IPSec - PfSense

![pfsense com 1 interface rede(lan)](https://i.ibb.co/Tvj5Xp1/PFSENSE-1-INTERFACE.png)

**
Cenário.
![cenario vpn](https://uploaddeimagens.com.br/images/002/552/931/original/VPN_site-to-site.png?1585260731) Ligar duas rede LANs através da rede WAN(net).

Configurar o pfsense1 com a rede local 10.10.10.35 com dhcp 10.10.10.20-45.
Configurar o pfsense2 com a rede local 20.20.20.35 com dhcp 20.20.20.10-45.

Os PCs ubuntu vão pegar IP via dhcp do pfsense.

No pfsense1, criar o Tunel em **VPN>IPSec**, adicionar IP do gateway e uma senha para o tunel.
![enter image description here](https://uploaddeimagens.com.br/images/002/552/954/original/tunel-pfsense1.png?1585261435)
Realizar o mesmo processo com o pfsense2.
![enter image description here](https://uploaddeimagens.com.br/images/002/552/957/original/tunel-pfsense2.png?1585261447)
Adicionar EP2 com a rede a ser acessada, neste caso a rede LAN.
No pfsense1 adicione a rede a ser descoberta do outro lado.
![enter image description here](https://uploaddeimagens.com.br/images/002/552/980/original/ep1-pfsense1.png?1585261903)
E realizar o mesmo no pfsense2.
![enter image description here](https://uploaddeimagens.com.br/images/002/552/979/original/ep1-pfsense2.png?1585261900)
Feito isto, o tunel ja foi criado e as redes locais descobertas, faltando somente liberar o acesso via VPN na configuração do firewall.
Criando acesso IPSec em rules.
![enter image description here](https://uploaddeimagens.com.br/images/002/552/995/original/regra_pra_acesso_vpn.png?1585262219)
**feito isto, tudo estará funcionando, havera conexão entre as duas LANs distintas tendo seus dados trafegando pela net com um túnel vpn.**
![enter image description here](https://uploaddeimagens.com.br/images/002/553/000/original/exemplo-ping.png?1585262467)







