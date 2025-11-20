Na sessão a seguir entenderemos mais sobre os serviços de IDS e IPS presentes em quase em todos os firewalls de nova geração.

Entender o funcionamento desses serviços é essencial pois isso pode fazer com que nossas técnicas sejam efetivas ou não, tendo em vista que IPS e IDS são muito presentes nos ambientes de segurança.

# Bypass em Firewall

Iniciando o módulo temos algumas boas práticas para burlar FWs em geral. Faz muito sentido testar essas técnicas quando for identificado algum bloqueio nos sniffers de rede.

São elas:

1. Especificar porta de origem da conexão:
    
    `nmap -v -sS -Pn -g <porta de origem> <IP>`
    
    *É INTERESSANTE USAR COM SOURCE PORTAS COM STATUS CLOSED
    
2. Especificar MAC Address na conexão
    
    `nmap -v -sS -Pn -spoof-mac <mac> <IP>`
    
3. Caso você identifique algo a partir desses métodos utilize o netcat para capturar o banner dos serviços encontrados. Abaixo está o comando para especificar a porta source quando utilizar o netcat
    
    `nc -vn -p <porta source> <IP> <porta destino>`
    

# IDS - Identity Detection System

O IDS é responsável por baseado em alertas configurados ou pré-configurados realizar a gerência de logs gerados por eventos de segurança.

`sudo apt install snort`

**Principais pastas:**

- ~={red}/etc/snort/snort.conf=~ : Includes aqui dentro são as rules
- ~={red}/etc/snort/rules=~ : Arquivos .rules
- ~={red}/var/logs/snort=~ : Logs do snort

**Ativando serviço:**

`cd /etc/snort`

`snort -A fast -q -h <subnet/mask> -c snort.conf` : Armazena nas logs

`snort -A fast -q -h <subnet/mask> -c snort.conf` : Mostra no console

EX: snort -A console -q -h 192.168.1.0/24 -c snort.conf

## Criando regras para Snort

Para criarmos regras personalizadas para o snort precisamos primeiramente configurar o arquivo .rules, abaixo está o passo a passo para cria-lo e um exemplo de regra

```bash
cd /etc/snort/rules
nano msb.rules

alert tcp any any -> 192.168.1.106 any (msg:"Possível PortScan"; sid:10000001;rev:6;)

alert : tipo 
tcp : protocolo
any1 : IP Source
any2 : Porta Source
-> : Sentido da conexão
192.168.1.106 : IP Destino
any3 : Porta Destino
("Possível PortScan"; sid:10000001;rev:6;) : Options

**sid:10000001 deve ser incrementado em 1 a cada regra
```

1. Feito isso agora precisamos vincular esse novo alerta ao arquivo snort.conf, para isso siga o passo a passo abaixo
    
    ```bash
    cd /etc/snort
    nano snort.conf
    
    *Na sessão de includes
    include $RULE_PATH/msb.rules
    ```
    
    Options adicionais importantes nas regras IDS
    
    - `flags`: Podem definir qual flag o pacote tem que ter pro alertas
        
        (msg:"Possível PortScan";flags:s;sid:10000001;rev:6;)
        
    - `content`: Conteúdo repassado no payload ou na URL que o pacote tem
        
        (msg:"Possível PortScan";content:”robots.txt”;sid:10000001;rev:6;)
        

# Bypass em regras de IDS

A seguir revisaremos jeitos práticos para burlar a verificação IDS feita com Snort. Importante ressaltar que a forma mais efetiva de achar meios para burlar o Snort é **entendendo e procurando alertas utilizados**

1. Em muitos casos existem alertas que verificam de acordo com o **content** do pacote que o mesmo se refere a um ping de PortSweep. Sendo assim para contornar isso alteramos o conteúdo do payload enviado com o seguinte comando.
    
    `hping3 -c 1 -p “<novo conteúdo>” <IP>`
    
2. Para identificar o PortSweep o Snort também utiliza muito o **code 0** usado por pings para identificar o rastro da técnica em suas logs. Sendo assim é uma boa prática também realizar a alteração do code do pacote com o seguinte comando.
    
    `hping3 —icmp -c 1 -K <novo code> <IP>`
    
3. Para finalizar usaremos o **tamanho do pacote** para enganar o Snort. Muitas vezes existem regras configuradas na ferramenta que verificam justamente esse tamanho para fazer alertas. A seguir revisaremos como alterar isso
    
    `hping3 —icmp -c 1 -K <novo code> -d <novo tamanho> <IP>`
    
    EX: `hping3 —icmp -c 1 -K 1-d 24 -p "TEsteTeste" <IP>`
    

# IPS - Identity Prevention System

O IPS é um sistema que funciona em conjunto com o IDS para prevenir ataques e ações maliciosas de hackers no ambiente. Nessa sessão usaremos o serviço **Portsentry** como ferramenta para mostrar o funcionamento do IPS suas principais vantagens

**Portsentry**: Programa do Linux que abre portas “fakes” no sistema para que o atacante tente realizar um Portscan e seja automaticamente bloqueado

**Portsentry vem por padrão com uma regra de bloqueio via rota, é interessante que alteremos para bloqueio via IPtables em seu arquivo conf **(/etc/portsentry/portsentry.conf)**

```bash
# Newer versions of Linux support the reject flag now. This 
# is cleaner than the above option.
#KILL_ROUTE="/sbin/route add -host $TARGET$ reject" : Comenta essa sessão 

# iptables support for Linux
KILL_ROUTE="/sbin/iptables -I INPUT -s $TARGET$ -j DROP" : Descomenta essa!

BLOCK_UDP="1" : Habilita bloqueio de IPs
BLOCK_TCP="1" : Habilita bloqueio de IPs
```

**Muitos atacantes utilizam Syn Scan para fazer esses testes, para adaptamos o Portsentry precisamos seguir o passo a passo abaixo:

```bash
whereis portsentry 
cd /usr/sbin/

/usr/sbin/portsentry -stcp
```

**Configurações adicionais importante

```bash
cd /etc/portsentry/portsentry.conf

# Use these if you just want to be aware:
TCP_PORTS="1,11,15,79,111,119,143,540,635,1080,1524,2000,5742,6667,12345,12346,20034,27665,31337,32771,32772,32773,32774,40421,49724,54320"
UDP_PORTS="1,7,9,69,161,162,513,635,640,641,700,37444,34555,31335,32770,32771,32772,32773,32774,31337,54321"

: Portas sendo monitoradas

###############
# TCP Wrappers#
###############
# This text will be dropped into the hosts.deny file for wrappers
# to use. There are two formats for TCP wrappers:
#
# Format One: Old Style - The default when extended host processing
# options are not enabled.
#
#KILL_HOSTS_DENY="ALL: $TARGET$"

# Format Two: New Style - The format used when extended option
# processing is enabled. You can drop in extended processing
# options, but be sure you escape all '%' symbols with a backslash
# to prevent problems writing out (i.e. \\%c \\%h )
#
KILL_HOSTS_DENY="ALL: $TARGET$ : DENY"

Deixando esse opção habilitada quando o host for bloqueado no IPtables ele também é bloqueado no **/etc/hosts.deny**

```

# Bypass IDS/IPS/Firewall

Agora que estudamos IDS e IPS fica a dúvida, qual é a melhor forma de fazer os scaneamentos necessários do pentest de forma a burlar esses sistemas? Abaixo estão algumas formas para que consigamos contornar essas ferramentas:

1. É interessante muitas vezes fazer uma análise apenas nas portas que tem uma alta probabilidade de ter algum serviço vinculado (top ports), para que dessa forma fujamos das armadilhas dos sistemas de defesa
    
    `nmap -sSV —top-ports=10 (10 portas mais tops) <IP>`
    
2. Da mesma forma fazer essa requisição com um tempo maior entre os pacotes pode confundir os mecanismos de defesa e dar vantagem ao atacante. Use -T para essa função
    
    `nmap -sSV —top-ports=10 (10 portas mais tops) -T1 ou -T2 <IP>`
    
3. Por último o método mais eficaz para burlar esses sistemas é enviar _diversas sources diferentes_ para que as defesas não tenham certeza se o tráfego é legitimo ou não e não saibam quem bloquear, abaixo estão algumas formas de fazer isso
    
    `nmap -v -D -sSV <IP>` : Usa IPs pré definidos pelo NMAP para decoy
    
    `nmap -v -D <IP1> <IP2> <IP3> -sSV <IP>` : Usa IPs que você define para a tarefa
    
    `nmap -v -D RND:<numero> -sSV <IP>` : Usa quantos IPs aleatórios você quiser
    

# LABS

VPN ————————

1. Para conseguir revisar qual porta de source o firewall aceitaria criei um script simples, basicamente ele testa como source todas as portas e remove a linha de não ter achado 998 tcp ports. Quando ele não ter achado um numero diferente de 998 ele coloca num TXT.
    
    Depois é so procurar isso no txt com cat simples.txt | grep "filtered tcp”
    
2. Depois que eu descobri a porta source ele pede qual serviço esta rodando no que descobrimos usando a porta source. Me conectei no que eu achei (8080) usando o netcat com porta source. Uma vez conectado precisei capturar com GET / HTTP/1.0 pra pegar toda a página e a key
    

---

---

1. Quais portas estao abertas?

nmap -v [intranet.businesscorp.com.br](http://intranet.businesscorp.com.br)

1. Qual o OS do server?
    
    Debian 7 , quando tu te conecta no ssh da pra ver que ele tem uma versao de os ali pra ele
    
    pesquisando vi que aquela versao é do debian 7
    
2. Qual versão do serviço?
    

nc -v [intranet.businesscorp.com.br](http://intranet.businesscorp.com.br/) 80

HEAD / HTTP/1.0 —> ele retorna a versão

1. Versão do serviço de acesso remoto?
    
    nc -v 37.59.174.228 2222
    
2. Qual versão do serviço de webadmin?
    

sudo nmap -v -sSV -p 10000 [intranet.businesscorp.com.br](http://intranet.businesscorp.com.br/)

1. Quais os métodos de autenticação permitidos?
    
    sudo nmap -v -script=**auth** [intranet.businesscorp.com.br](http://intranet.businesscorp.com.br/)