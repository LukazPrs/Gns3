## Criar GPG-Key para assinar o repositório

    > gpg --gen-key

Salve sua chave em um diretório(será colocada no caminho do repositório)

    >  gpg --armor --export you@example.com_ > minhachave.asc

Podemos usar o programa dpkg-sig para assinar os pacotes **.deb**

    apt-get install 
    dpkg-sig dpkg-sig --sign builder mypackage_0.1.2_amd64.deb

### Criando repositório reprepro
Escolha o diretório do repositório e crie uma pasta, onde será montado o diretório do repositório reprepro.

    > mkdir /home/lucas/reprepro

Criar uma pasta com nome de **conf** e dentro dela criar um arquivo com nome de **distributions** e coloque as informações no arquivo.

    > mkdir /home/lucas/reprepro/conf nano
    > nano /home/lucas/reprepro/conf/distributions

    Origin: risc  
    Label: apt repository  
    Codename: bionic  
    Architectures: i386 amd64 source  
    Components: main  
    Description: repositorio reprepro 
    SignWith: yes  
    Pull: bionic

### Instale o reprepro e configure o mesmo no diretório criado

    > apt install reprepro reprepro --ask-passphrase -Vb . includedeb bionic
    > /home/lucas/reprepro/PACOTE_amd64.deb
   
  ** Colocar a chave no diretório do reprepro**

### BAIXAR E CONFIGURAR APACHE

apt install apache2

Configurar diretório apache([aqui](https://wiki.debian.org/DebianRepository/SetupWithReprepro))
#### Configurar caminho padrão do apache para seu diretorio reprepro

> editar arquivo em /etc/apache2/apache2.conf

    <Directory /home/mirror/reprepro>
            Options Indexes FollowSymLinks
            AllowOverride None
            Require all granted
    </Directory>

### Adicionando pacotes .deb ao repositório

    reprepro includedeb <osrelease> <debfile>

....
