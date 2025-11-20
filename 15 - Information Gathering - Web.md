
 - “Olhar código fonte da página é sempre uma boa prática”


Talvez desse pra tornar o script mais eficiente sendo que quando ele encontra uma página ele retorna 200 e se tu pesquisar por um dir sem por o / no final ele retorna 301

Como criar um whatweb teu???

# Introdução

No módulo a seguir revisaremos como coletar informações do lado de Web, muitas vezes essa verificação é ignorada mas de acordo com o repassado aqui será possível entender a necessidade dessa verificação

**Quais serão os objetivos a serem alcançados?**

- Coleta de páginas e diretórios
- Identificar tecnologias, servidores, OSs
- Observar métodos permitidos para o alvo
- Entender estrutura da página
- Análise de **código fonte** de **javascript**

Itens de atenção:

- Login possível na página?
- Qual o retorno de erros?
- Existe redirecionamento?
- Existe cadastro ou envio de dados?

# Robots e Sitemap

As páginas **robots.txt** e **sitemap(.txt ou .xml)** em muitos casos estão presentes na maioria das páginas da internet. Isso ocorre pois elas tem funções especificas que muitas vezes auxiliam o programadores a controlar itens muito importantes da navegações, como veremos a seguir:

### Robots.txt

A página robots.txt tem como principal função enviar mensagens para os crawlers do google que realizam as pesquisas para mostrar as páginas ao usuário. Essas mensagens muitas vezes se resumem a quais diretórios não devem aparecer na pesquisa e quais User Agents podem acessar as páginas. Repare na estrutura abaixo:

```bash
User-agent: *            :Informa qual User Agent pode acessar, nesse caso any
Disallow: /_restrito     : Diretório que não deve ser achado por crawler
Disallow: /_docs         : Diretório que não deve ser achado por crawler
Disallow: /admin         : Diretório que não deve ser achado por crawler
Disallow: /bkp           : Diretório que não deve ser achado por crawler
Allow: /configuracoes/comunicacao/projeto.txt : Diretório liberado para acesso
```

### Sitemap

O Sitemap é uma página que muitas vezes desenvolvedores utilizam para manter a organização da árvore de páginas do site. Caso você seja capaz de encontrar o sitemap da página ele será muito útil porque basicamente lista todas as páginas que queremos encontrar

Sua estrutura geralmente é assim:

```bash
<sitemapindex>
<sitemap>
<loc><https://www.google.com/gmail/sitemap.xml></loc>
</sitemap>
<sitemap>
<loc><https://www.google.com/forms/sitemaps.xml></loc>
</sitemap>
<sitemap>
<loc><https://www.google.com/slides/sitemaps.xml></loc>
</sitemap>
<sitemap>
<loc><https://www.google.com/sheets/sitemaps.xml></loc>
</sitemap>
.....
```

# Listagem de diretórios e Mirror Website

### Listagem de diretórios

Muitas páginas quando mal configuradas tem o nome de Index of, isso ocorre pois a página citada funciona como diretório aberto apontando para arquivos, mas não foi restringido acesso a ela. Em casos como esse é ótimo achar páginas assim pois é como entrar em um dir do apache via Browser.

Sendo assim lembre-se de incluir nas pesquisar tanto de Web quanto de Business sempre o alias Index of. Pode ser muito útil no futuro.

### Mirror Website

Mirror website é uma técnica utilizando o comando:

`wget -m <página web>`

Essa técnica consiste em basicamente baixar a página alvo inteira para seu Linux de forma que ela esteja utilizável (exatamente como a que está na web).

O Curl criará um novo diretório contendo todas os itens dessa nova página para que possamos analisar ela como um todo, itens de atenção ao fazer a análise:

- Muitas vezes é necessário abrir o index e revisar TUDO, funcionamento por funcionamento
- Diretórios `js` geralmente pode conter as funções da página e geralmente merecem mais atenção
- Você pode simular a página a partir da sua máquina
- `wget -m -e robots=off <endereço>` : Caso a página não tenha robots.txt

# Análise de erros, códigos e extensões (PaP)

Nessa sessão exploraremos um passo a passo que deve ser feito em basicamente todas as páginas que analisaremos em nosso alvo. Esses itens são essenciais para o Pentest e muitas vezes podemos tirar informações valiosas aplicando o seguinte

1. **`Abrir código fonte da página (f12 / ctrl + shift + i)`**
    
    Muitas vezes no código fonte de uma página podemos encontrar comentários, funções e diversos itens deixados pelo desenvolvedor que podem ser a chave para um pentest bem feito. Além disso, em casos mais difíceis onde o pentest estiver complicado vale muito a pena abrir a página e entender todo o funcionamento da página
    
2. **`Fazer requisições para página ou diretório que não existe de propósito`**
    
    Uma técnica útil também é simular um erro de propósito para uma página ou diretório. Dessa maneira muitas vezes conseguimos informações sobre a aplicação rodando a página, sua versão, OS e muitos outros itens.
    
3. `Fazer download das páginas com wget -m e ler todas às páginas.`
    
    Caso o Pentest esteja complicado vale a pena criar um entendimento completo sobre o funcionamento do site. Para isso baixe a página e análise cada um dos itens para maior aprofundamento
    
    ****MUITO IMPORTANTE: Na maioria dos casos vale muito a pena forçar um erro para uma página inexistente mas uma página com a mesma linguagem de programação do alvo.**
    
    EX: [http://alvo.com.br/teste.php](http://alvo.com.br/teste.php) ou teste.aspx
    
    Fazendo isso se for a linguagem que o server usa podemos ter retorno de versão dessa linguagem entre outras infos interessantes.
    

# Dirb e a lógica do Bruteforce Web

Para encontrarmos ainda mais páginas é possível utilizar a aplicação Dirb no Linux, que utiliza uma wordlist pré-definida e dessa forma faz um bruteforce descobrindo páginas e diretórios desconhecidos, observe abaixo:

```bash
dirb http://<dominio>

dirb http://<dominio> -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:131.0) Gecko/20100101 Firefox/131.0"
```

Perceba que acima utilizamos o comando `-a` para definir quem seria o **User Agent** da nossa comunicação com o sites. Fizemos isso pois 1 dos grandes calcanhares de Aquiles do Dirb são firewalls WAF que necessitam de User Agents sendo browsers cotidianos.

A seguir entenderemos melhor sobre isso:

### Lógica do bruteforce Web

A lógica que o problema Dirb utiliza para fazer a varredura de sites no alvo é pretty much a mesma que usaremos em um script criado de forma personalizada, com exceção de alguns refinamentos.

Para encontrar as páginas o Dirb segue o seguinte passo a passo:

1. Faz requisições curl para o alvo selecionando com a opção -w o retorno do status da requisição. Além disso é usado o -o para que apenas isso apareça na tela.
    
    `curl -s -o /dev/null -w “{http_code}” https://<página>/<palavra>`
    
    A palavra acima roda para toda a wordlist
    
2. Logo após caso o retorno seja 202, ele mostra na tela e reporta que é uma página existente, caso não seja ele descarta e não mostra nada
    
3. Logo após ele faz o mesmo teste mas dessa vez pesquisando se a palavra não pode ser um diretório, php ou HTML. Repare:
    
    `curl -s -o /dev/null -w “{http_code}” https://<página>/<palavra>/`
    
    `curl -s -o /dev/null -w “{http_code}” https://<página>/<palavra>.php`
    
    `curl -s -o /dev/null -w “{http_code}” https://<página>/<palavra>.html`
    

### Comando curl

Vale a pena também realizar um aprofundamento sobre o comando Curl que é tão necessário em nosso código de varredura.

**Função**: Permite fazer requisições baseadas no protocolo HTTP/HTTPS

Variações possíveis:

```bash
curl -v <site> :Mostra com mais detalhes todo o processo do Curl (Mtbom)

curl -s <site> : Faz requisição ao site de forma silenciosa

curl -s --head <site> | grep 200 : Faz requisição do head do site caso seja 200

curl -s -H "UserAgent: <Browser completo>" <site> : Altera campo User-Agent

curl -s -o /dev/null -w “{http_code}” https://<página>/<palavra>
: Pega código de retorno da página e manda o restante para /dev/null

-o --> Trata saída desnecessária 
-w --> Retorna item específico do Curl
-H --> Permite mudar item que o Curl envia
```

# Script para Bruteforce Web Recon

Abaixo criaremos um script personalizado para bruteforce em página alvo online, primeiro vamos a lógica do código

### Lógica do código

1. Criação de wordlist
2. Criar for que passa `<site>/<wordlist>`
3. Fazer requisições com Curl e colocar código retorno em variável
4. 4. Caso a variável seja “200” retorne a página encontrada
5. Repita o processo para diretórios e para .php e .html

**Código:**

[https://github.com/m4EDRE/web2/blob/main/brutefweb.sh](https://github.com/m4EDRE/web2/blob/main/brutefweb.sh)

OBS:

- Perceba que ele faz a revisão de dirs /`<palavra>`/, e páginas /`<palavra>` 
- 3. Fazer requisições com Curl e colocar código retorno em variável
4. Caso a variável seja “200” retorne a página encontrada
5. Repita o processo para diretórios e para .php e .html

**Código:**

[https://github.com/m4EDRE/web2/blob/main/brutefweb.sh](https://github.com/m4EDRE/web2/blob/main/brutefweb.sh)

OBS:

- Perceba que ele faz a revisão de dirs /`<palavra>`/, e páginas /`<palavra>`
- Perceba que com -H definimos um User Agent para burlar firewalls WAF
- `Principal ponto` = Tratamento de valor de comando pondo ele em variável
- wordlistweb.txt foi feito a partir de várias wordlists do Dirb
- `IMPORTANTE:`Vale a pena entrar na página e com F12 ou ctrl +shift + i revisar não apenas as infos da página mas a parte de redes, lá você confere o User Agent que está usando

# Whatweb e Wappalyzer

Aqui exploraremos duas ferramentas já criadas que quando necessário podem ser integradas no Pentest afim de completar ainda mais nossa análise Web, são elas:

### Wappalyzer

Ferramenta que funciona como extensão do browser, permite encontrar diversas infos sobre o alvo como:

- Linguagem de programação da página
- Aplicação rodando a página
- Sistema operacional, versões
- Entre outros

### Whatweb

Ferramenta que deve ser instalada no Linux e faz o trabalho de fazer várias pesquisas sobre o site alvo, como:

- Sites
- Server e OS
- Jquery e páginas

(Não tão prático quanto o Wappalyzer)

`whatweb <site>

# Script para identificar páginas Web

No script que exploraremos abaixo faremos uma pesquisa pelo alvo utilizando o Lynx onde a ideia é utilizar a pesquisa em browser para encontrar mais alguns sites perdidos. Repare na lógica a seguir

### Lógica do código

1. Chamada do programa deve ser feita com $1 sendo o alvo e $2 sendo a extensão a ser procurada
    
2. Usa Lynx -dump para fazer a chamada dessa pesquisa usando google dorks também
    
    `lynx --dump `<https://google.com/search?num=200&q=site:$1+ext:.$2”`>`
    
    Perceba que na chamada acima filtramos para apenas respostas com código 200
    
3. Filtra o retorno do dump para que apareçam apenas as os sites com a extensão desejada
    
    `lynx --dump [`<https://google.com/search?num=200&q=site:$1+ext:.$>](<https://google.com/search?num=200&q=site:$1+ext:.$2”>`)2" | cut -d "=" -f2 | grep ".$2"`
    
    Dessa forma temos o retorno desejado e podemos aprofundar ainda mais a pesquisa por páginas com extensões específicas
    

# Explicando labs

LAB 1

Primeira verificação pelo o que está no Robots.txt

Dentro existia um dir /bkp/ que continha uma operação chamada dentro

Operação simples de cópia de diretório… mas o interessante eram os diretórios envolvidos

cp /var/www/db/update.sql para outro

Já tinha feito o bruteforce e achado o /db/, entrei nele e fiz o teste pra tentar achar o update.sql

BINGO!

LAB2

Achei o site sitemap, ele direciona dizendo que a localização é outro site

Chegando lá não tem nada

> “Olhar código fonte da página é sempre uma boa prática”

Bingo!

LAB3

Bruteforce simples

LAB4

Bruteforce no dirb [http://businesscorp.com.br/info](http://businesscorp.com.br/info)

LAB 5

BAIXEI A PAGINA COM WGET - M [BUSINESSCORP.COM.BR](http://BUSINESSCORP.COM.BR)

DENTRO DELA TEM A PASTA JS COM OS METODOS

DENTRO DESSA PASTA TEM —> getClient.js

ABRE ISSO TEM KEY

LAB 6

Criação do script de [brutefweb.sh](http://brutefweb.sh) versão 3. Na versão 3 ele corrige erros e agora é capaz de mascarar o User Agent usando o Firefox.

Quando tu usa isso acha de boa

IMPORTANTE: Sempre testar no teu browser pra ver qual user agent está usando

LAB 7

LAB 8

Bruteforce simples, meu script comporta pesquisas pra PHP

LAB 9

Nesse lab para achar as versões utilizei as técnicas de forçar um erro pesquisando por página que não existe.

Peguei a aplicação e a versão dela forçando esse erro

LAB 10

Utilizei para pegar essas infos da página o Weppalyzer que é uma extensão do browser para pegar as infos referentes a esse lab

VPN

LAB 1

Pra descobrir versão do do server usei o whatweb

LAB2

Pra descobrir linguagem de programação usei o Wappalyzer

LAB3

Longatto explica em um vídeo que pra pegar a versão da linguagem de programação tu tem que fazer uma requisição http do mesmo tipo, daí como o server conhece a linguagem ele te retorna a linguagem com a versão

Aqui a linguagem é [ASP.NET](http://ASP.NET)

Então fiz um Curl -v `<IP>`/testequalquer.aspx (Extensão [ASP.NET](http://ASP.NET))

