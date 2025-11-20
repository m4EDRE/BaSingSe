> “Fase que determina o Pentest”

Nessa fase identificaremos o host em seu:

- Tipo
- Modelo
- OS
- Versão
- Portas
- E outros



# Tracking Route

Aqui revisaremos técnicas para analisar o caminho do tráfego até o host alvo que estamos analisando, para isso utilizaremos o `traceroute`

- TTL decrementa a cada hop pulado (-1)
- Quando o TTL chega a 0 o pacote volta a origem e informa o último host

Dessa forma o comando traceroute manda TTLs curtos de hop em hop para ir identificando os hosts entre source e alvo.

Assim conseguimos traçar esse trajeto.

**TTLs: - - - - - - - - - - - - - - - - - - - - - - - -**

`64 - Linux`

`128 - Windows`

`254 - Cisco`

**- - - - - - - - - - - - - - - - - - - - - - - - - - - -**

- - - —> Host não recebe UDP

### Comandos para personalizar traceroute

- `-w` Manipula tempo de espera
- `-m` Quantidade de TTLs
- `-f` Especifica a partir de qual salto começa a mostrar
- `-A` Mostra ASs
- `-n` Mostra apenas IPs
- `-I` Usa ICMP ao invés de UDP
- `-T` Usa TCP ao invés de UDP
- `-p` Escolhe porta para direcionar tráfego

**DICA:** Enviar por vários protocolos para achar vários hops

# Overview sobre Firewall

A seguir revisaremos um overview simples sobre o funcionamento do iptables em servidores Linux. Esse serviço basicamente é o firewall desses servidores e pode ser configurado para realizar as mais diversas tarefas.

Esse entendimento é primordial pois muitas vezes o iptables estar bem configurado pode ser a diferença entre o vazamento de informações importantíssimas na fase de Scanning ou não.

### Comandos:

- `iptables -nL` : Mostra regras ativas
    
- `iptables -F` : Apaga todas as regras do firewall
    
- `iptables -P INPUT DROP` : Regra primária que bloqueia toda a entrada de dados no servidor
    
    - -P demarca **regra padrão**
- `iptables -A INPUT -p tcp —dport 80 -j ACCEPT` : Regra que permite entrada de tráfego vindo para porta 80
    
    - -A adiciona regra no final da tabela, -j seleciona ação da regra, -p define protocolo
- `iptables -A INPUT -s 192.168.10.1 -d 192.168.1.2 -j ACCEPT` : Regra que aceita tráfego vindo de IP específico para outro
    
    *Importante pontuar que você precisa reiniciar o firewall após as mudanças!
    

# PingSweep e ARP e código rápido

A técnica chamada de PingSweep basicamente se consiste em realizar um tipo específico de requisição ou ping para um host ou range de network. Dessa forma conseguimos encontrar informações de **hosts ativos na rede** e informações como **OS ou portas abertas** dos mesmos.

> Não é porque não pingo que o host não está ativo

Isso ocorre porque em certos firewalls quando a regra é configurada de certa forma podemos ter comportamentos que fazem parecer que a porta não existe ao NMAP

- DROP: Não tem retorno para o host
- REJECT: Retorna porta unreachable para host]
- `fping -a -g <subnet/máscara>` : Comando de ping para subnet

### Descobrindo hosts internos

Uma vez dentro do ambiente podemos utilizar ideias para varrer a rede e utilizando protocolo ARP (L2) descobrir diversos hosts ativos.

`arping -c 1 <IP>` : Ping utilizando ARP para descobrir hosts ativos

Isso pode ajudar nas verificações por utilizar apenas ARP para busca

### Python

Código que mapeia IPs na rede

```python
 python3 -c "import os; [print(f'172.16.2.{i}') for i in range(1, 255) if os.system(f'ping -c 1 -W 1 172.16.2.{i} > /dev/null 2>&1') == 0]"
```

`python -c` : Executa python com 1 comando

`import os` : Importa lib necessária para execução de comandos na shell

`[ ]` : Mantém código em 1 linha

`f'172.16.2.{i}') for i in range(1, 255)` : Cria loop na rede /24 do 1 até o 255 de IPs para percorrermos

# PortScanning

Utilizamos a ideia de 3 way handshake como conceito base para fazer o PortScanning, sendo assim:

`SYN/SYN-ACK/ACK` Porta aberta

`SYN/RST-ACK` Porta fechada

Dessa forma então o pentester manda requisição para cada uma das portas dos sistema para identificar as que tiverem retorno positivo

### Como programas fazem esse envio? (Script)

`hping -c 1 —S -p <porta> <endereço>`

- `-c 1` : Envia apenas 1 pacote
- `—S` : Envia apenas pacote com flag SYN

Caso o retorno do haping seja SA (SYN ACK) a porta esta aberta, caso seja RA (RESET ACK) a porta está fechada.

**É assim que podemos criar um script de portscan**

OBS: Caso o firewall bloqueie a conexão podemos ter um retorno diferente:

- Sem retorno de flag alguma = filtered
- ICMP unrechable = filtered

### Diferenças no tipo de Scan (Connect x Sync Scan)

Quando fazemos o processo descrito acima para PortScanning é possível encontrar duas formas diferentes de mandar as requisições, estão são:

- **Connect:**
    - 3 Way Handshake
    - Mt consumo de rede
    - Facilmente detectável por seu comportamento
- **Sync Scan / Half Open**
    - Não usa 3 Way Handshake completo
    - Retorna um RST caso o alvo retorne SYN/ACK
    - - Uso de rede, + Silencioso
    - Não gera logs

Conforme é possível observar acima o Sync Scan consiste em realizar o envio do pacote sem a intenção de fechar o 3 way handshake, apenas para buscar um retorno do alvo.

Isso torna as capturas mais silenciosas utilizando menos recursos.

# NMAP

**“O que fazer quando firewall bloqueia ICMP para encontrar hosts ativos?”**

`nmap -sn 192.168.1.0/24` : Faz varredura simples no sistema utilizando todos os recursos do NMAP para achar hosts ativos

NMAP se utiliza de varreduras baseadas em:

- ICMP
- TCP
- HTTP/HTTPS

OBS:`-oN <arquivo>` : Opção do NMAP que permite enviar output para arquivo

### Outros recursos conhecidos

`-v` : Permite ver as operações na tela (verbose)

`-p` : Permite selecionar as portas a serem escaneadas

Permite ranges e várias ao mesmo tempo, EX :1,2,3,10-20

`-p-` : Escaneia todas as portas do host

`-Pn` : Pula a verificação pra ver se o host está ativo

`-sS` : Utiliza método Sync Scan para fazer Scan

`-sU` : Faz pesquisa por portas UDP

`—open` : Retorna apenas portas abertas

`-V` : Faz pesquisa para identificar serviço rodando na porta (Banner Grabbing)

`-iL` : Carrega lista de hosts

`-O` : Pede ao NMAP que identifique o OS rodando no alvo

Quanto mais infos e portas você informar do alvo, + ajuda ele

OBS: Quando a análise for UDP muitas vezes o retorno pode ser errado porque ele se baseia em flags que mudam. Sendo assim **SEMPRE** que for UDP usar `-sUV`

### Metodologia Scanning

Importante ressaltar qual a metodologia e finalizada para qual usamos o NMAP.

“Para identificar uma `vulnerabilidade` tem que identificar os `serviços`

E para identificar os `serviços` temos que identificar as `portas abertas` no sistema”

Para fazer isso precisamos de —> **Network Sweeping**

Desafios:

- Tempo do Scan
- Firewall
- Bloqueios, IPS e IDS

# Network Sweeping

A seguir veremos o processo de Network Sweeping que deve ser feito na maioria dos casos de Pentest. Esse processo tem grande importância para o Pentest como foi visto na fase de metodologia

1. Encontrar Hosts ativos

`nmap -sn 192.168.1.0/24 -oG ativos.txt` : Manda hosts ativos para ativos.txt

1. Com todos os IPs em um arquivo TXT crie uma automação pra ver os serviços rodando nos hosts

`nmap -v -sSV -p- —open -Pn - iL ativos.txt`

1. Faz a análise dos serviços de cada host na nova lista gerada atrás de vulnerabilidades.

OBS: No exemplo acima o Scan está sendo feito para -p- mas o correto é fazer para as top ports e ir alterando conforme a necessidade. Sempre direcionando de acordo com pentest.

# OS Fingerprint e identificando OSs

Um item que deve ser ressaltado em relação aos demais é a identificação de OSs do sistema alvo. Para isso reforçaremos alguns itens já vistos nas outras sessões.

**TTLs: - - - - - - - - - - - - - - - - - - - - - - - -**

`64 - Linux`

`128 - Windows`

`254 - Cisco`

**- - - - - - - - - - - - - - - - - - - - - - - - - - - -**

**NMAP: - - - - - - - - - - - - - - - - - - - - - - - -**

`-V` : Faz pesquisa para identificar serviço rodando na porta (Banner Grabbing)

`-O` : Pede ao NMAP que identifique o OS rodando no alvo

Quanto mais infos e portas você informar do alvo, + ajuda ele

**Serviços Especiais: - - - - - - - - - - - - - - - - - - - - - - - -**

SSH: Geralmente Linux

3389 (RDP) : Windows

# Laboratórios

OSINT

LAB 1

nmap -v -sS -p- [mail.businesscorp.com.br](http://mail.businesscorp.com.br/)

Resolve

LAB2

nmap -v -sU -p 1-1000 [mail.businesscorp.com.br](http://mail.businesscorp.com.br/) pra nao ficar mt grande

achei a 53

tirei o -V pq consome mt

LAB3

nmap -v -p 25 -sUV -Pn [mail.businesscorp.com.br](http://mail.businesscorp.com.br/)

Vai aparecer coisa afu, pega a repetição e percebe que tem algo como uma key

é ela

LAB4

Peguei nos labs antigos, tava no pastebin

LAB5

Fiz o 6 primeiro, dai pra fazer esse abri o email raw e analisei o cabeçalho

LAB6

Acessei pq dei uma tentada num programa conhecido de email depois da url

[http://mail.businesscorp.com.br/squirrelmail](http://mail.businesscorp.com.br/squirrelmail)

acesso com as credenciais da camila

fui nos sent e vi o email via raw

LAB7

Tranquilo de ver no email raw

LAB8

Fui nos emails recebidos e vi o mais antigo

LAB9

Só buscar data na inbox

LAb10

Tranquilo de ver no email raw

LAB11

Busquei o email que ela enviou no sent

LAB12

Deboas de achar na inbox

LAb13

Pega o user dela e a senha que ela mandou e entra na intranet da businesscorp

LAB14

So por a chave da intranet

Scanning

LAB 1

sudo nmap -v -sSV [businesscorp.com.br](http://businesscorp.com.br/)

LAB2

sudo nmap -v -sSV [businesscorp.com.br](http://businesscorp.com.br/)

LAB3

sudo nmap -v -sSV [businesscorp.com.br](http://businesscorp.com.br/)

LAB4

nmap -v -sn 172.30.0.0/24

LAB 5

sudo nmap -v -sn 172.30.0.0/24 | egrep -v "host down”

LAB 6

sudo nmap -v -sS -O 172.30.0.0/24 | egrep -v "host down”

-O salva os guri

LAB7

sudo nmap -v -sS -p- -Pn 172.30.0.103

LAB8

sudo nmap -v -p 25 -sS -Pn 172.30.0.0/24 | grep open

LAB9

sudo nmap -v -p 25 -sSV -Pn 172.30.0.128

LAB10

sudo nmap -v -sSV -O 172.30.0.128

LAB11

sudo nmap -v -p 3306 -sS 172.30.0.0/24

LAB12

sudo nmap -v -p 10000 -sS 172.30.0.0/24