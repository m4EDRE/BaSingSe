# Ataques de força bruta

Um ataque de força bruta basicamente é uma automação que testa várias senhas em uma wordlist para tentar adivinhar qual a senha do usuário alvo.

Por mais que essa lógica seja aparentemente simples é importante revisarmos alguns pontos que podem ter ligação direta com um ataque bem ou mal sucedido…

### Tipos de Bruteforce

- **Clássico**: É utilizada uma wordlist padrão gigante para tentar adivinhar a senha do um usuário encontrado em qualquer momento do Pentest
- **Guessing:** Pentest onde é feita a mesma automação mas agora com uma wordlist personalizada para o user alvo, utilizando gostos ou caracteristicas para fazer a adivinhação
- **Acesso Default**: Bruteforce onde são aplicados a todos os hosts com que se tem conexão senhas default de diversos equipamentos para tentar explorar essa falha

### Wordlists

Abaixo revisaremos as principais wordlists a serem utilizadas… mas primeiro porque não usar uma **Wordlist com 1 bilhão de senhas**?

- Tráfego no servidor alvo
- Te faz ser facilmente descoberto
- Pode até derrubar o servidor com o alto acesso

**`Principais wordlists a serem utilizadas`**

`/usr/share/wordlists` : Pasta com principais wordlists como **nmap**, **rockyou** e etc

`/usr/share/seclists/passwords` : Pasta principal de senhas das seclists pra bruteforce, pode ser necessário instalar seclists no kali

`wl.txt` : Arquivo recomendado pelo Longatto para você usar como lista

[https://github.com/ricardolongatto/loncrack/blob/master/wl.txt](https://github.com/ricardolongatto/loncrack/blob/master/wl.txt)




----





# Wordlists mutáveis e personalizadas

### Wordlist mutáveis

A seguir está o passo a passo para criar uma wordlist mutável baseado em algumas regras que geralmente são utilizadas na criação de senhas por usuários…

1. Criar wordlist com variações do nome foco
    
    1. Roberto
    2. Beto
    3. Rob
    4. roberto
2. Com o John aplique as regras de mutação e crie um novo arquivo
    
    `john —wordlist=<arquivocriado> —rules —stdout > newarchive.txt`
    
    *Essas regras tem boas possibilidades e alterações para senhas
    
3. No arquivo `/etc/john/john.conf` você pode fazer uma busca por rules e observar os padrões que ele utiliza para alterar a sua lista de possibilidades, além de configurar outros padrões para alteração.
    

### Gerando Wordlists personalizáveis

Nesse item verificaremos como criar uma wordlist personalizada de acordo com um alvo. Para isso, é importante reforçar que deve ter sido feito um **Information Gathering** bem feito sobre o alvo para poder ter as melhores informações sobre essa lista e identificar padrões do sistema

`cewl` : Gera lista de palavras baseado em página na WEB

`cewl <página> -m <quantidade de palavras>`

*Interessante pegar as palavras do cewl e utilizar o john para criar a lista

`crunch` : Aplicação que gera alterações em lista de palavras de acordo com a sua preferência

`crunch 10 10 -t <palavra><operadores> -o <novoarquivo.txt>`

`-t` : Palavra a ser alterada

Primeiro 10 —> Tamanho mínimo da linha gerada

Segundo 10 —> Tamanho máximo da linha gerada

**Operadores**

`%` - Todos os dígitos

`^` - Todos os caracteres especiais

`@` - Todos os caracteres minúsculos

`,` - Todos os caracteres maiúsculos



----





# Key Space Bruteforce

O método Key Space Bruteforce tem uma característica em particular ao outros ataques de bruteforce que vale a pena ser citado nessa documentação

Basicamente o KSB se reforça de dois itens em especial para achar uma senha, são eles: **Identificação de um padrão + Exploração completa do padrão**

Dessa forma esgotamos todas as possibilidades de uma senha até encontrar o acesso desejado

Muitas vezes o KSB é utilizado em acessos **Token ou qualquer contexto onde os limites da senha tem uma regra ou padrão identificável**

Exemplo de caso de uso: Sabemos que as senhas de Token do usuário é um token de 4 dígitos

Para esse caso usaremos o crunch para criar uma lista de todos os dígitos com até 4 caracteres

`crunch 4 4 0123456789 -o pin.txt`

`crunch 6 6 -f [charset.](<http://charset.st>)lst numeric -o pin.txt` : Nesse exemplo usamos a lista de 0-9 do charset.lst para gerar o mesmo resultado

Exemplo mais manipulável: `crunch 8 8 0123456789 -t dev01@@@ -o dev01_3dig.txt`

Acima ele cria senhas com 8 digitos sendo os primeiros dev01 e os ultimos 3 todas as possibilidades dentro de 0-9

_Operadores:_

- % - Para números
- ^ - Para caracteres especiais
- @ - Caracteres minúsculos
- , - Caracteres maiúsculos

Com isso temos um baita exemplo de bruteforce direcionado



-----





# Bruteforce com Hydra

Como principal ferramenta para bruteforces temos o Hydra, essa ferramenta é conhecida no mundo do pentest tanto quanto o John ou o NMAP e tem diversos recursos relacionados a bruteforce que podem ser extremamente úteis.

Exemplo utilizando lista de users e lista senhas:

`hydra -v -l <user> -p <senha> <IP>`

### Itens úteis

- `-L` : Passa lista de users `(cuidar)`
    
- `-P` : Passa lista de senhas `(cuidar)`
    
- `-p` : Passa 1 senha `(cuidar)`
    
- `-l` : Passa 1 user `(cuidar)`
    
- `-s` : Especifica porta para conexão
    
- `-t` : Especifica quantidade de tasks (tentativas) ao mesmo tempo, MAX 16, muito bom pra evitar IDS
    
- `-w` : Especifica tempo entre tentativas
    
- `-f` : Para quando senha é encontrada
    
- `-M` : Passa lista de servers a serem atacados
    
- `-q` : Não printa mensagens de erro de conexão
    
- `botar nome do serviço no final do comando pode ajudar`
    
- Para ftp por exemplo use:
    
    `hydra -l user -P passlist.txt [<ftp://192.168.0.1>](<ftp://192.168.0.1/>)`
    
    `hydra -l user -P passlist.txt ssh[://192.168.0.1](<ftp://192.168.0.1/>)`
    



----




# Reverse Bruteforce e Low Hanging Fruit

Importante também registrar o conceito de Reverse Bruteforce e Low Hanging Fruit, como exploraremos a seguir. Esses conceitos por mais que simples **sempre devem ser empregados em um cenário amplo de pentest onde existem muitas possibilidades**

### Reverse Bruteforce

Em essência o processo aqui é encontrar formas de fazer um bruteforce focado em achar usuários no sistema, e não necessariamente encontrar senhas especificas. Para isso temos alguns caminhos possíveis

- Criar script que teste lista de usuários e retorne caso o erro do sistema seja referente a erro de senha, confirmando que o usuário existe na database da aplicação
    
- Aplicar o Hydra com uma lista de users e senhas default para confirmar que não existem usuários desavisados com senhas fáceis no sistema
    
    `hydra -v -L <listausers.txt> -p admin <IP> ssh`
    

### Low Hanging Fruit

Aliado ao conceito de Reverse BruteForce podemos citar o Low Hanging Fruit, que é um método que foca em varrer e vasculhar por itens que estejam mais suscetíveis ao ataque. Para isso utilizamos senhas padrão muito utilizadas por usuários, ou também podemos utilizar senhas default de aplicações, softwares ou dispositivos para verificar que não existem itens disponíveis.

**Foco: Elo + fraco**

Exemplo: `hydra -v -l root -p root -M <listadehosts.txt>`

**`OBS`**: Esse item pode ter grande valida também se utilizadas credenciais encontradas no pós exploração

**`OBS2`**: Vale a pena fazer isso também para `outros serviços` como FTP e etc



----




# Códigos feitos no módulo Bruteforce

## Construindo a própria ferramenta

Passo a passo

*Entender como aplicação funciona (Nesse caso FTP)

para variar esse script será feito no python o Windows

1. Conectar no FTP porta 21
    
2. Passa USER `<user>`
    
3. Passa PASS `<senha>`
    
4. Recebe retorno do FTP
    
5. De acordo com o código de retorno trata resposta ao user
    
    [https://github.com/m4EDRE/bruteforcecodes/blob/main/btf1.py](https://github.com/m4EDRE/bruteforcecodes/blob/main/btf1.py)
    

## Criando bruteforce SSH em python

*Para esse caso usaremos Paramiko para simplificar a autenticação SSH

Passo a Passo

1. Instalação do paramiko e import para o código e lê introdução da doc
    
2. Inicia criando conexão SSH
    
3. Adiciona envio de comando na comunicação SSH
    
4. Trata erros possíveis da conexão SSH
    
5. Fecha conexão
    
6. Retorna ao usuário retorno do comando enviado
    
    [https://github.com/m4EDRE/bruteforcecodes/blob/main/btf2.py](https://github.com/m4EDRE/bruteforcecodes/blob/main/btf2.py)
    

## Criando Bruteforce script em bash

*A seguir temos um script rápido para bruteforce, muitas vezes acessamos máquinas em que não se é possível fazer download de ferramentas, sendo assim é prático termos scripts rápidos em mãos para fazer essas validações