# Introdução ao Python

Shebang do Python: `!/usr/bin/python3`

Tipo de arquivos em Python: `.py`

Itens solicitados para que o user escreva:

```python
ip = raw_input("Digite o IP")  //Variável IP recebe string passada pelo user

ip = input("Digite o IP")      //Variável IP recebe INT passado pelo user
```

Transformação de tipos de variável:

```python
int(<variável>) //Tranforma variável em int

str(<variável>) //Transforma variável em string
```

## Trabalhando com argumentos

Para que trabalhemos com argumentos passados pelo usuário na chamada do programa primeiro precisamos importar a `lib sys`

A partir disso, é criado um array chamado `sys.argv` que tem como primeiro item a string sys.argv e como seus itens a seguir todos os argumentos passados pelo user.

Exemplo:

```python
import sys

print(sys.argv[1], sys.argv[2])
```

É interessante sempre fazer um `if` contendo uma verificação caso não sejam passados argumentos e dessa forma ele retorno um manual de como usar o programa…

## Repasse de comandos no bash

Para isso necessitaremos da biblioteca OS para que seja possível realizar as atividades necessárias. Usaremos a função `system` para rodar os comandos definidos em uma string

Exemplo:
```python

import os 

os.system("netstat -nlpt")
```

--- 

# Condições e repetições

Para realizar as alternativas de condição utilizaremos a função `if` para realizar os comandos necessários.

Exemplo

```python
if (len(sys.argv)<2):
	print("Manual de como usar o programa criada")
else:
	print("Executa código")
```

Por outro lado podemos usar a função `for` para criar qualquer repetição necessária no código:
```python
for porta in range(1,65353): 
	print(porta)
```

----

# Trabalhando com Sockets

Para trabalhar com sockets primeiramente precisaremos de duas bibliotecas…

`import sockets e sys`

Exemplo de criação de socket:
```python
import socket 
import sys 

ip = "<ip>" #IP deve ser string 
porta = <porta> #IP deve ser um int 

MSBsocket = socket.socket(socket.AF_INET,socket.SOCK_STREAM) #SOCK STREAM pq é TCP 

resposta = MSBsocket.connect_ex((ip,porta)) 

if resposta == 0: 
	print("Socket bem sucedido") 
	meusocket.close() 
else: 
	print("Socket não conectado") 
	MSBsocket.close()

```
*Repare nos tipos necessários de IP e porta*

---- 

# Banner Grabbing

Muitas vezes se consegue informações valiosas sobre o serviço e sobre servidores no retorno de comandos dados. Essa técnica se chama **banner grabbing** e utilizaremos o script de sockets para exemplo sobre como fazer essa coleta

```python
import socket
import sys

ip = str(<ip>)               #IP deve ser string
porta = int(<porta>)         #IP deve ser um int

MSBsocket = socket.socket(socket.AF_INET,socket.SOCK_STREAM) #SOCK STREAM pq é TCP

resposta = MSBsocket.connect_ex((ip,porta))

if resposta == 0:
	print("Socket bem sucedido")
else:
	print("Socket não conectado")
	
banner = MSBsocket.recv(1024)         #1024 bytes coletados pelo banner
print (banner)

ou print("Resposta ao USER:", banner.decode(errors='ignore'))
	
```

Muito interessante utilizar essa técnica para revisar quais serviços estão rodando na porta!

---

# Interagindo com serviços e DNS Resolver

Na sessão a seguir revisaremos como enviar dados para para um host via socket e como criar um DNS Resolver com python

## Interagindo com serviços

Para mandar os comandos para um servidor conectado via socket usaremos a função `.send` vinculando ela ao seu socket criado

```python
meusocket.send(b"USER mytmsh \\r\\n")
```

*É interessante sempre coletar o banner após o envio de um comando para que fique evidente onde o processo está para o usuário

## DNS resolver em python

Para criarmos um DNS resolver além da necessidade óbvia dos comandos para coleta de argumentos do usuário com a biblioteca `sys` precisaremos usar a função `gethostbyname()` vinculada ao socket criado, veja no exemplo

```python
import sys
import socket

endereco = mysocket.gethostbyname(sys.argv[1])
print(endereco)
```

# Trabalhando com WEB

Para trabalhar com itens web é necessário primeiramente importar a biblioteca `requests`

A seguir utilizaremos a função `.get()` e a seguir utilizaremos **funções filhos** para pegar apenas i**tens específicos** desse get

```python
import requests

site = requests.get("http://<site>")  #Equivalente a Wget em python

print(site.content)  #Exibe o HTML da página
print(site.status_code)  #Pega o status da requisição EX:200 OK
print(site.headers)  #Mostra headers da página como versão do servers e outras
										 #Muito importante
										 

```


