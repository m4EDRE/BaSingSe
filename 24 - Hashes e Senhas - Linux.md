
Nesse módulo exploraremos como manipular e utilizar hashes para encontrar senhas em sistemas Linux.

Conhecimento imprescindível para pentests que necessitem de acesso certificados a ambientes

Sites:

- [hashes.com](https://hashes.com/en/tools/hash_identifier)
- [https://md5decrypt.net/en/](https://md5decrypt.net/en/)

# Introdução

**Hashes podem ser:**

- md5
- sha256
- sha512
- etc

**Casos de uso:** Quando passamos nossa senha para um site, o comportamento padrão de um site é salva-la em um banco de dados mas em formato de **hash** para maior proteção desses arquivos sensíveis.

1. Passa senha ao site
2. Aplicação coloca hash na senha
3. Hash é salvo no banco
4. No login é feita verificação com hash

Outro caso de uso é quando sites disponibilizam **hash específico para verificação de downloads**. Dessa forma é possível que o download foi feito sem alterações.


---




# Trabalhando com Hashes

Para iniciar, observe abaixo como verificar o hash de uma string ou de um arquivo

- `echo -n “<item>" | <cypher>` : Traduz item para seu modelo hash de acordo com a cypher
- `<cypher> <arquivo>` : Busca hash do arquivo de acordo com sua cypher
    - EX: `md5sum <arquivo>`

*Arquivos com o nome diferente mas mesmo conteúdo sempre terão o mesmo hash

*É impossível reverter funções usadas para criar hash

*Cópias de arquivos tem mesmo hash

### One way x Two way

Importante revisarmos rapidamente o que são funções one way e two way para termos mais clareza sobre o hash

- `One Way` - Funções que uma vez convertidas não permitem engenharia reversa para voltar a seu estado primário.
    - Ex: Hash
- `Two Way` - Funções que permitem engenharia reversa (decode) para encontrar valor primário que gerou a chave
    - Ex: Base64
    - `“<item>” | base64` (coda)
    - `“<itembase64>” | base64 -d` (decoda)

### Trabalhando com Hashes em Python

Para finalizar esse item uma explicação rápida de como usar tradução de itens em hashes via scripts em Python…

`import hashlib` : Importa lib

`hashlib.<cypher>(”texto”).hexdigest()` : Traduz texto em hash

`import base64` : Importa lib do base64

`base64.b64code(”texto”)` : Coda item para base64

`base64.b64decode(”texto”)` : Decoda item via base64



---





# Identificando Hashes

Como etapa muito importante desse módulo precisamos ter a habilidade de identificar o tipo de hash caso encontremos algum no meio de um pentest, para isso usaremos algumas aplicações:

- `hashid <hash>` : Serviço que retorna o tipo de hash
- `hash-identifier` : Serviço que abre opções para determinar hash

-----






# Estudo técnico, atacando hashes

Como não é possível realizar o decode de hashes, a prática mais usada para decodificar esses itens é com o que chamamos de `rainbow tables` , onde o processo é basicamente bater o hash encontrado com uma base de hashs para uma wordlist conhecida.

A seguir revisaremos como fazer isso primeiramente da forma mais manual possível

### Processo Manual

1. Descobrir hash e seu tipo (hashid)
2. Realizar o teste do hash em sites conhecidos com databases enormes para quebrar esses hashes, são eles
    1. **`md5decrypt.net`**
    2. **`hashes.com`**
3. Criar wordlist manual com senha e hash (mais a frente Longatto sugere wordlist)
4. Criar script que loopa hash alvo comparando com todos os hashes da wordlist
5. Realiza esse processo até encontrar o seu hash.

### Processo utilizando ferramentas

Para auxiliar nessas buscas podemos também utilizar ferramentas conhecidíssimas como o `john` e o `hashcat`abaixo explorados.

- **`john** <arquivo contendo hash alvo>` : Principal programa na quebra de hashs, tem database muito boa
- **`hashcat`** : Auxilia na quebra de hashes, necessita de wordlist para funcionar corretamente.
    - `—help` : Mostra opções
    - `-m <id do formato da cypher nas opções> <arquivo> <wordlist> --force`
    - `-w<wordlist>`



----





# Importância das senhas

Principais problemas causados por senhas…

- Sistema:
    - Senhas fracas
    - Reuso de senhas
    - Armazenamento inseguro de senhas
- Tecnologia
    - Ataques de força bruta
    - Vulnerabilidades
    - Senhas fracas

Encontrar senhas de acesso verificado no sistema farão você ir mais a fundo no pentest



----





# Senhas em sistemas Linux

Nesse item iremos a fundo para entender como funcionam as senhas em sistemas Linux, para dessa forma podermos explorara-las em diversos cenários e irmos mais a fundo nos pentests.

- `/etc/passwd` —> Arquivo que contém usuários no linux (Senhas são um x)
    
- `/etc/shadow` —> Arquivo onde ficam armazenadas as senhas em formato de hash do Linux
    
    - Necessita de acesso root para observar
    - Salva as senhas em formato de crypt como veremos a seguir
    
    ```bash
    Formato que é salvo as senhas no shadow....
    
    $id$salt$encrypted
    id --> Identificador do hash (da criptografia)
    salt --> Salt
    encrypted --> Senha em formato de hash
    
    openssl passwd -e -salt <salt> <senha> : Cria senha com salt
    ```
    
- **`Salt` : Item que aplica +1 função em cima do hash para modifica-lo ainda mais**
    
- Observe que ID identifica a criptografia utilizada!!!
    

### Descobrindo e quebrando senhas no Linux

**Passo a passo exemplo para quebrar senha possível no Linux:**

1. Salva conteúdo do /etc/passwd e /etc/shadow em um arquivo
    
2. `unshadow <arquivo passwd> <arquivo shadow> > hashes`
    
    Comando que junta dois arquivos criando apenas um no formato ideal pra John
    
3. `john <arquivo>` : Roda o John novamente para o arquivo criado da junção do passwd e shadow
    

**Quebrando senhas com Loncrack**

1. git clone `<path do loncrack>`
    1. **`wl.txt`** —> BAITA wordlist pra qualquer coisa
2. `gcc loncrack.c -lcrypt -o loncrack` : Compila Loncrack
3. `./loncrack wl.txt` : Executa loncrack com a lista txt
4. Passa hash completo e o salt dele
5. Realiza força bruta