# DNS (BIND / NAMED)
O Domain Name System (DNS) é um sistema de gerenciamento de nomes hierárquico e distribuído para computadores, serviços ou qualquer recurso conectado à Internet ou em uma rede privada.

## Instalação
Toda a instalação é feita como **root**

### Debian
Instale o bind server com o comando:

`apt-get install bind9`

Configurar o encaminhamento de DNS edite o arquivo **/etc/bind/named.conf.options**, remova o comentário da linha onde está o item forwarders abaixo segue um exemplo:

```
forwarders {
IP da sua máquina;
8.8.8.8;
};
```
Após editar o arquivo, salve-o e feche-o.

Definindo zonas de DNS edite o arquivo **/etc/bind/named.conf.local**, inclua uma linha em branco e após esta inclua as suas zonas (normal e reverso), veja um exemplo nas **Configurações** item **Definição de Zonas de DNS**

Criar um diretório onde vão ficar as suas zonas, digite o comando:

`mkdir /etc/bind/zones`

Crie os arquivo da zona .db, usando o editor de sua preferência:

`editor /etc/bind/zones/home.lan.host`

Veja um exemplo nas **Configurações** item **Zona de DNS**

Crie os arquivo da zona reversa .db, usando o editor de sua preferência:

`editor /etc/bind/zones/rev.(IP_REVERSO).host`

Veja um exemplo nas **Configurações** item **Zona Reversa**

Eeinicie o serviço de DNS:

`service bind9 restart`

### CentOS
Instale o bind server e as ferramentas do bind com o comando:

`yum -y install bind bind-utils`

Coloque o IP do Servidor e range de rede que o DNS atenderá, edite o arquivo **/etc/named.conf**:

`após:` **listen-on port 53 { 127.0.0.1;** `coloque o IP do servidor, termine com` **;** `e` **}**
`após:` **allow-query     { localhost;** `coloque o range da rede, termine com` **;** `e` **}**

Após editar o arquivo, salve-o e feche-o.

Definindo zonas de DNS edite o arquivo **/etc/named.conf**, inclua uma linha em branco antes das linhas de include, veja um exemplo nas **Configurações** item **Definição de Zonas de DNS**

Crie os arquivo da zona .db, usando o editor de sua preferência:

`editor /etc/named/home.lan.host`

Veja um exemplo nas **Configurações** item **Zona de DNS**

Crie os arquivo da zona reversa, usando o editor de sua preferência:

`editor /etc/(IP_REVERSO).in-addr.host`

Veja um exemplo nas **Configurações** item **Zona Reversa**

Habilite e inicie o serviço com os comandos:

```
systemctl enable named
systemctl start named
```

## Configurações
### Definição de Zonas de DNS

```
zone "home.lan" IN {
        type master;
        file "/etc/bind/zones/home.lan.host";
}; 

zone "(IP_REVERSO).in-addr.arpa" {
        type master;
        file "/etc/bind/zones/rev.(IP_REVERSO).host";
};
```
Onde: **home.lan** é o nome da sua zona, e **IP_REVERSO** é o seu IP invertido (Ex.: 0.168.192)

### Zona de DNS

```
$TTL 1D
@       IN      SOA    ns.home.lan. (NOME_DO_SERVER).home.lan. (
                       20   ; serial
                       8H   ; refresh
                       2H   ; retry
                       4W   ; expire
                       1D ) ; minimum

        IN      NS     ns.home.lan.
        IN      MX     10    mail.home.lan. ; Opcional

router  IN      A    (IP_DO_ROTEADOR) ; Opcional

        IN      (IP_DO_SERVER)
ns      IN      (IP_DO_SERVER)
(NOME_DO_SERVER) IN      A    (IP_DO_SERVER)
mail    IN      A   (IP_DO_SERVER) ; Opcional
www     IN      A   (IP_DO_SERVER) ; Opcional
ftp     IN      A   (IP_DO_SERVER) ; Opcional
smtp    IN      A   (IP_DO_SERVER) ; Opcional
imap    IN      A   (IP_DO_SERVER) ; Opcional
pop3    IN      A   (IP_DO_SERVER) ; Opcional

gateway CNAME   router ; Opcional
gw      CNAME   router ; Ocpional
```

Onde: **NOME_DO_SERVER** é o nome do servidor e **IP_DO_SERVER** é o IP do servidor

### Zona Reversa

```
$TTL    1D
@       IN    ns.home.lan. (NOME_DO_SERVER).home.lan. (
              20   ; Serial
              8H   ; Refresh
              2H   ; Retry
              4W   ; Expire
              1D ) ; Negative Cache TTL

@       IN    NS    ns.home.lan.

(FINAL_DO_IP)   IN      PTR     (NOME_DO_SERVER)
```
Onde: **NOME_DO_SERVER** é o nome do servidor e **IP_REVERSO** é o seu IP invertido (Ex.: 0.168.192)
E **FINAL_DO_IP** o final do IP (Ex: 11 = 192.168.1.11)

## Testes

Antes de testar o DNS, como **root** edite o arquivo **/etc/resolv.conf** e coloque na primeira linha conforme exemplo abaixo:

```
nameserver IP_DO_SERVER
nameserver 8.8.8.8

```
Após editar o arquivo, salve-o e feche-o.

Onde: **IP_DO_SERVER** é o IP do servidor

Obs: Se este arquivo é modificado a cada inicialização proteja-o dando o comando:

`chattr +i /etc/resolv.conf`

Obs2: Para tirar a proteção altere de **+i** para **-i**.

Testando o DNS, como **root** digite o comando:

`nslookup (NOME_DO_SERVER).home.lan`

Que dever retornar algo como:

```
Server:  IP_DO_SERVER
Address: IP_DO_SERVER#53

Name:   (NOME_DO_SERVER).home.lan
Address: IP_DO_SERVER
```

Onde: **NOME_DO_SERVER** é o nome do servidor e **IP_DO_SERVER** é o IP do servidor