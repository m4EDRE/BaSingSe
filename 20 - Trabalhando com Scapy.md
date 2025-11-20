
Nessa sessão observaremos as funcionalidades do Scapy, ferramenta que permite construir o seu pacote IP/TCP para envio na rede.

# Introdução ao Scapy

Para iniciar o Scapy após baixa-lo para o seu linux basta escrever **scapy** no terminal e automaticamente você será levado para um terminal completamente novo contendo apenas comandos para a aplicação.

Para ver comandos disponíveis no Scapy: `lsc()`

Para ver todos os campos personalizáveis de um protocolo: `ls(<protocolo>)`

EX: `ls(IP) , ls(TCP)`

Comando para descobrir funcionalidades e modificações de pacote: `explore()`

EX: `explore()` —> via GUI seleciona IPV4 : Ele retorna várias opções de headers para construir o seu pacote

---
# Manipulando pacotes com Scapy

Para manipular pacotes com Scapy o primeiro passo é criar os headers do pacote individualmente de acordo com a layer, abaixo está especificado como fazer esse processo para as camadas 3 e 4.

### Criando header IP

```bash
pIP = IP(dst="192.168.1.101")

pIP = Váriavel header que construimos
IP  = Tipo do header
(dst="192.168.1.101") = Alteração nas opções do pacote

Alterações possíveis:
  version   =
  ihl       =
  tos       =
  len       =
  id        =
  flags     =
  frag      =
  ttl       =
  proto     =
  chksum    =
  src       =
  dst       =
```

### Criando header TCP

```bash
pTCP = TCP(dport=[80,443,5858],flags="S")

pTCP = Variável do header TCP que estamos criando
TCP = Protocolo selecionado para  variável
(dport=80,flags="S") = Opções selecionadas para o header

Alterações possíveis
     sport     =
     dport     =
     seq       =
     ack       =
     dataofs   =
     reserved  =
     flags     = S,SA,RA etc
     window    =
     chksum    =
     urgptr    =
     options   =
```

### Formando pacote e tratando envio e retorno

```bash
pacote = pIP/pTCP : Construção da variável pacote que une dois headers criados

sr1(pacote) : Envio do pacote criado acima

resposta =  sr1(pacote) : variável resposta recebe o retorno do envio do pacote

---------------------------------------------------------------------
Ponto relevante:

**sr // sr1** : Quando tivermos mais de um tipo de pacote sendo enviado, como qunaod temos mais de uma porta de destino, precisamos usar sr porque ele engloba todos os envios. sr1 é referente apenas ao primeiro envio
```

****Função show()**: Essa é uma das ferramentas mais importantes do Scapy, com ela você pode pegar qualquer construção que você tenha feito e analisar o que está feito até agora, perceba abaixo alguns exemplos

`pTCP.show()`

`pIP.show()`

`pacote.show()`


--- 

# Criando ICMPs alterados e modificando seus payloads

Para prosseguir observando alterações possíveis é interessante citarmos rapidamente a criação de ICMPs com seus payloads modificados.

Lembrando o último módulo de evasão de defesas, é interessante que utilizemos esse recurso para burlar ferramentas como IDS ou IPS.

```bash
Criando pacote com header ICMP:

pIP = IP(dst="X")
pacote = pIP/ICMP()
resposta = sr1(pacote)
resposta.show()

Alterando payload do pacote:
pacote = pIP/ICMP()/"<O que você quiser escrever>"

Aqui ele vai substituir o payload por o que você quiser escrever, e não precisa ser ICMP pra ele mudar o payload

```

**EXTRA:** Tratando informações específicas de pacotes retorno

Para Scripts é muito interessante sermos capazes de tratar os pacotes de retorno das requisições para automatizar tarefas, com isso o Scapy passa algumas formas para que coletemos informações especificas desse retorno, como é possível ver abaixo:

```python
pIP = IP(dst=ip)
pTCP = TCP(dport=porta)
pacote = pIP/pTCP
resp,noresp = sr(pacote)

if (resp[0][1][TCP].flags == "SA"):

resp : Variável que recebe quando há retorno de pacotes enviados
[0] : Referente a primeira comunicação feita
[1] : Referente a primeira comunicação feita mas o retorno dela
[TCP] : Referente a primeira comunicação feita mas o retorno dela e apenas o header TCP
.flags : Referente a primeira comunicação feita mas o retorno dela e apenas o header TCP e apenas o campo de flags
```

--- 

# Portscan com Scapy

Abaixo está descrito o processo e a lógica usada para criar um Portscan usando Scapy, juntamente com um código criado e posto no GitHub.

### Passo a passo utilizado:

1. Criaremos um script usando python então primeiramente precisamos importar o python para o arquivo .py juntamente com a biblioteca do Scapy
    
    `#!/usr/bin/python3`
    
    `from scapy.all import *`
    
2. A seguir precisamos criar os pacotes IP e TCP para o Portscan com Scapy, crie o código para que ele solicite o IP a ser analisado e crie o pacote com as informações necessárias
    
    ```python
    conf.verb = 0
    #Remove infos desnecessárias da conexão
    
    ip = input("Input the host to be attacked:")
    pIP = IP(dst=ip)
    pTCP = TCP(dport=porta)
    pacote = pIP/pTCP
    resp,noresp = sr(pacote)
    ```
    
3. Feito isso precisamos de um loop para que todas as portas sejam verificadas, para isso utilizaremos um loop com for in range
    
    ```python
    for porta in range(1,65535):
            pIP = IP(dst=ip)
            pTCP = TCP(dport=porta)
            pacote = pIP/pTCP
            resp,noresp = sr(pacote)
    ```
    
4. Para finalizar precisamos tratar a saída das tentativas de conexão a portas do host alvo, para isso precisamos que toda vez que a resposta for positiva (SA) ele retorne que a porta está aberta. Então criamos um if verificando a flag recebida e retornando uma mensagem.
    
    ```python
    for porta in range(1,65535):
            pIP = IP(dst=ip)
            pTCP = TCP(dport=porta)
            pacote = pIP/pTCP
            resp,noresp = sr(pacote)
            if (resp[0][1][TCP].flags == "SA"):
                    print("Porta "+str(porta)+ " aberta no host")
    ```
    
    ### Código final:
    
    [https://github.com/m4EDRE/scapy/blob/main/ScapPScan](https://github.com/m4EDRE/scapy/blob/main/ScapPScan)