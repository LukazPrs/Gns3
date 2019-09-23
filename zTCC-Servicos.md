

## Virtualização de serviços no contexto de NFV.

Artigo: Funções de Rede Virtualizadas em Plataforma de Computacao em Nuvem para Cidades Inteligentes. Autor: Alexandre Heideker , Carlos Alberto Kamienskl Ano: 2015

> Tecnologias: Comp Urbana , Comp NUVEM OpenStack , NFV, IaaS

Serviços Virtualizados: Roteador e servidor DNS
Rotedor = Neutron já virtualiza uma das funções utilizadas nos experimentos: o roteador
DNS: Maquina Virtual KVM(Este mesmo método pode ser utilizado para criação de novas VNFs, tais como proxy, NAT, Firewall, etc)

Este artigo propõe e avalia o desempenho de uma solução para fornecer Funções de Rede Virtualizadas (VNF) utilizando a plataforma de computação em nuvem OpenStack, apresentando o ganho significativo de desempenho esperado.

Virtualização: O serviço de infraestrutura é provido pelo OpenStack 
fornecendo dois tipos de VNFs para os experimentos: roteador e servidor DNS. 
Neste trabalho utilizou-se a versão Juno do OpenStack, com a nova arquitetura de rede chamada Neutron.

Cenário com três componentes: consumidor, produtor e serviço. 
O produtor é um servidor web fornecendo dados para os experimentos realizados. 
O consumidor, representando os usuários em um ambiente urbano.
Finalmente o serviço de infraestrutura que é promovido pelo OpenStack. 

A característica principal do cenário de Computação Urbana que deve ser 
representado no ambiente experimental é a grande quantidade de usuários 
utilizando os serviços de rede: desde algumas dezenas até dezenas de milhares.

Cada processo faz uma requisição ao servidor DNS, registrando o tempo necessário 
para esta função. Em seguida é realizada a requisição do arquivo selecionado utilizando
o protocolo HTTP ao servidor Web.

Este trabalho apresenta a implementação do conceito de NFV utilizando a plataforma de Computação em Nuvem OpenStack, através da criação de máquinas virtuais específicas para a manipulação e tratamento do tráfego, além da utilização dos funções de rede nativas do OpenStack. Este conjunto de ferramentas, aplicado em um cenário de cidades inteligentes usando computação urbana com demanda crescente de recursos, mostra a melhoria do desempenho com a distribuição dinâmica de recursos virtuais de infraestrutura. Ambiciona-se responder se é possível otimizar e automatizar o fornecimento de recursos de telecomunicações neste ambiente altamente dinâmico, obter um equilíbrio entre custo e performance para uma qualidade de serviço adequada
--------------------------------------------------------------------------------------------------------------------------

Artigo: Emprego de NFV e Aprendizagem por Reforc¸o para Detectar e Mitigar Anomalias em Redes Deﬁnidas por Software
Autor: Pedro H. A. Faustini1, Anderson S. Silva1, Lisandro Z. Granville1, Alberto E. Schaeffer-Filho1
Ano:2017 

Tecnologias: SDN, NFV, Aprendizagem de Maquina

Serviços Virtualizados: Firewall -Docker(Agente determina a instanciação de um novo firewall para barrar ataques)


Resumo: Uma rede de computadores esta sujeita a diversas anomalias, é necessario coordenar tecnicas de detecçao e mitigaçao 
para mante-la operacional. Este artigo propoe aprendizagem por reforço para promover resiliencia em redes deﬁnidas por software (SDN).
Propoe-se que metricas de rede sejam coletadas e agrupadas em perﬁs, cada um com um conjunto de açoes que trate problemas usando 
aprendizagem por reforço, virtualizaçao de funçoes de rede (NFV) e um controlador SDN.

Neste artigo, propomos um modelo de aprendizagem por reforço integrado a arquitetura de NFV por meio de um agente que reside no 
orquestrador. O agente recebe metricas de rede e constroi perﬁs de rede. Cada perﬁl busca tratar uma anomalia. Cada perﬁl e formado
por um conjunto de açoes, bem como uma tabela de estados e uma tabela estado-açao.

Foi utilizada a plataforma Mininet, que emula uma SDN, e o framework Pox, que implementa a versao 1.0 do OpenFlow, para o 
desenvolvimento do controlador. Contudo, Mininet nao prove suporte nativo para NFV. As funçoes de rede portanto foram 
implementadas em conteineres Docker.


---------------------------------------------------------------------------------------------------------------------------------

Artigo: Gerenciamento Flexível de Infraestrutura de Acesso Público à Internet com NFV 
Autor: Alexandre Heideker, Carlos Kamienski 
Ano: 2016

NAT* controlador de elasticidade com NFV(https://github.com/heideker/elasticNFV) - (Praças SP)

Este trabalho utiliza NFV para solucionar o problema de fornecimento dinâmico e elástico de acesso público à internet nas 
grandes cidades.

O elemento fundamental de NFV é a técnica de virtualização e esta pode influenciar diretamente na performance e versatilidade da 
implementação das Funções de Rede Virtualizadas (VNF). Entre as técnicas disponíveis, este trabalho avalia  a virtualização total, 
a paravirtualização e os containers Linux.


----------------------------------------------------------------------------------------------------------------------------------

Artigo>: Uma Arquitetura de Virtualização de Funções de Rede para Proteção Automática e Eﬁciente contra Ataques
Autor: Leopoldo A.F.Mauricio, Igor D.Alvarenga, Marcelo G.Rubinstein e Otto CarlosM.B.Duarte
Ano: 2017

Virtualização: Firewall e IDS

Este artigo propõe uma arquitetura de virtualização de funções de rede (Network Functions Virtualization - NFV) 
que oferece proteção automática e eﬁciente contra ataques. Os ﬂuxos maliciosos são dinamicamente desviados por uma 
cadeia de funções virtuais de rede (Virtual Network Functions VNFs)contendoumﬁrewalldeaplicaçãoeumIDS.
