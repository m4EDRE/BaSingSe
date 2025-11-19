> “Não existe passo inútil, tudo aqui será útil”

> “6 horas pra derrubar uma árvore? 4 horas afiando o machado!”

Sessão referente a primeira análise a ser feita em um alvo, sendo a coleta de informações relacionadas a **CORP** que muitas vezes pode revelar **domínios, usuários, senhas e versões** que serão a base para o ataque a seguir



# Mapeando colaboradores

A seguir revisaremos como coletar informações importantíssimas para o ataque, como:

- Vagas de emprego
- Colaboradores
- Emails e leaks
- Publicações e e-mails

## Linkedin

Para iniciar as buscas referentes a empresa alvo sempre se deve iniciar observando os itens expostos no Linkedin da empresa para observar:

- Pessoas que trabalham na empresa
- Tecnologias utilizadas (Observar via vaga de emprego)
    - Versão de firewall, versão de server, office e etc
- Github e projetos para observar a forma de trabalho
- Outras redes sociais, vale a pena pesquisar Instagram, Facebook e etc para encontrar demais infos para engenharia social

## Coletando endereços de e-mail com [Hunter.io](http://Hunter.io)

Acesse o site [Hunter.io](http://Hunter.io) para entender primeiramente a forma com que o endereço de e-mail da empresa alvo é construído. Repare que cada empresa pode ter sua forma de construir seu e-mail.

Além disso é importante também utilizar a ferramenta Finder que é capaz de repassar e-mails de colaboradores já mapeados na internet sendo essa uma grande ferramenta para investigação social do alvo.

----

# Leaks (Vazamento de dados)

A seguir revisaremos como encontrar informações vazadas da empresa alvo que estamos investigando.

Essas informações podem estar disponíveis na internet por vários motivos como descaso da equipe de segurança ou colaboradores mal intencionados.

### Onde achar as leaks?

- Sites de monitoramento
- Sites de leaks pagas
- Criminosos

Exemplo de sites:

—> haveibeenpwnd?

—> [pastebin.com](http://pastebin.com)

—> [trello.com](http://trello.com) (usando Google Hacking)

*Não é porque não está numa base de vazamento que não vazou

## Consultando leaks na Dark Web

Site darkweb: [http://pwndb2am4tzkvold.onion/](http://pwndb2am4tzkvold.onion/)

Acesso via tor

Você pode baixar o TOR via internet mesmo no site: [https://www.torproject.org/](https://www.torproject.org/) e realizar o processo de ativação no browser para mascarar sua localização ou configurar o firefox como abaixo

```bash
sudo apt install tor

sudo apt install proxychains

systemctl start tor
nano /etc/proxychains.conf

Ao final do arquivo põe: socks5 127.0.0.1 <9050 ou porta>
E descomenta ou o strict/dynamic/random
```

**Abre a GUI do firefox**: Network proxy > Settings > Socks5 >127.0.0.1 port 9050 ou porta > Habilita o Proxy DNS when using socks5

*Isso muda seu IP e esconde sua localização

----

# Pastebin e Trello

Outros meios conhecidos utilizados para difusão de leaks são o **Pastebin** e **Trello** onde muitas vezes podemos encontrar várias informações importantes referentes a empresa alvo.

## Pastebin

[www.pastebin.com](http://www.pastebin.com)

Existem duas formas muito interessantes de utilizar o pastebin para buscar informações sobre a empresa alvo, reforço que o pastebin muitas vezes é um repositório de informações coletadas por hackers sendo assim não deve ser descartado nas investigações

1. Acesso via GUI e pesquisa na barra search pelo dominio da empresa
2. Usando google hacking: site:`pastebin.com <dominio do alvo>`

## Trello

Muitas empresas por vezes usam o Trello para seus projetos e inserem itens de grande criticidade em projetos públicos.

Dessa forma é sempre bom revisar itens relacionados ao domínio da empresa no Trello com:

`site:trello.com <dominio do alvo>`


-----




# URLcrazy e Cache

## URLcrazy

Útil para criar um dominio parecido com o do alvo, isso pode ajudar ao enviar um e-mail para phishing entre outros casos

```bash
sudo apt install urlcrazy

urlcrazy <dominio do alvo>
```

## Pesquisando paginas em cache

Muitas vezes uma informação critica disponibilizada na internet pelo alvo foi em dado momento corrigida ou alterada.

Sendo assim podemos usar páginas que armazenam cache da internet para consultar essas informações fantasmas

Exemplo: **[web.archive.org](http://web.archive.org)**


-----


# Google Hacking

Google hacking é a principal forma de achar itens disponibilizados na internet pelo alvo que podem ser utilizados para exploração em pentest…

**Operadores:**

- `site:` Especificar pesquisa baseada em um site
- `“ ”` Pesquisar precisamente por um item
- `inurl:` Pesquisar por item presente na URL do alvo
- `filetype:` Pesquisar por tipo de arquivo disponibilizado
- `ext:` Pesquisar por tipo de extensão
- `-` Pesquisa por tudo menos o item após o -
- `intext:` Pesquisa por item que esteja presente no texto do site disponibilizado
- `cache:` Pesquisa por URL no cache do browser

Exemplos:

```bash
site:msb.com.br ext:.pdf --> Pesquisar por pdfs disponibilizados pelo site msb.com.br

site:msb.com.br filetype:docx

cache:msb.com.br/paginaquejanãoxiste --> Pesquisar pagina em cache do browser

site:msb.com.br "senha do mateus"

site:msb.com.br intitle:"Admin" -www --> Pesquisa por titulo Admin mas em paginas sem WWW

```

*Importante ressaltar que o arquivo `robots.txt` é padrão existir pois fornece a comunicação entre máquina controlada e controlador.

*Pesquisar por `Index of` também sempre pode ser útil pois é assim que uma página incompleta se parece

- **Palavras chave**: config, senha, admin, ftp, acesso, dados, backup
- **Extensões chave:** PHP,PDF,DOC,DOCX,TXT,XLS,JS,SQL e ASPX
- **Titulos chave:** index of, login, acesso, admin

### Como fazer pesquisar via CLI usando Google hacking?

`firefox “<https://google.com/search?q=”><pesquisa1+pesquisa2>”`

Exemplos:

```bash
firefox “<https://google.com/search?q=”site:msb.com.br+ext:.pdf”>

firefox “<https://google.com/search?q=”site:msb.com.br+filetype:docx”>

firefox “<https://google.com/search?q=”site:msb.com.br+intitle:"Admin"+-www”>
```


----

# Metadados

Como ultima verificação de arquivos disponibilizados ou não pelo alvo na internet sempre devemos investigar os metadados que muitas vezes podem conter versões, usuários, senhas e etc.

Para fazermos essa análise utilizaremos a ferramenta `exiftool`

```bash
sudo apt install exiftool

wget <URL do arquivo>

exiftool <arquivo baixado>
```

Para automatizar essa busca é extremamente interessante criar um script que faça a coleta desses arquivos e armazene todo o output do exiftool de cada um deles em um TXT a parte, abaixo está o script para isso comentado:

```bash
#!/bin/bash

arquivo="imagemnet.txt" //Só pra puxar imagem ferinha na entra
while read linha;
do
        echo $linha
done < $arquivo

if [ "$1" == "" ]
then
        echo "外国人歓迎1~外国人歓迎"
        echo " "
        echo "Para usar esse script você deve passar um paramêtro"
        echo "Exemplo: ./metaSearch <dominio>"

else
        echo "外国人歓迎1~外国人歓迎"
        echo "Choose the type of file to be search..."
        echo "1 - PDF"
        echo "2 - TXT"
        echo "3 - DOCX"
        echo "4 - PHP"
        read op
        case $op in //Opções acima 1-4
        "1")
                lynx --dump "<https://google.com/search?&q=site:$1+ext:pdf>" | grep pdf | cut -d "&" -f1 | cut -d " " -f5 > ARCHmetaSearch.txt 
                //URLs de download e envio a txt para ser baixado
                //Filtro por tipo necessário para organizar 
	              //Tratamento da URL com cut
	              
	                
                while read ler;
                do
                        wget "$ler" 2> /dev/null
                done < ARCHmetaSearch.txt
                //Download dos arquivos

                exiftool *.pdf > metaResultfor--"$1".txt
                //Coleta de metadados e envio a arquivo
                
                rm *?q*
								//Delete de lixo
        ;;
        
				        //Repetição
        "2")
                lynx --dump "<https://google.com/search?&q=site:$1+ext:txt>" | grep txt | cut -d "&" -f1 | cut -d " " -f5 > ARCHmetaSearch.txt
                while read ler;
                do
                        wget "$ler" 2> /dev/null
                done < ARCHmetaSearch.txt
                exiftool *.txt > metaResultfor--"$1".txt
                rm *?q*

        ;;
        "3")
                lynx --dump "<https://google.com/search?&q=site:$1+ext:docx>" | grep docx | grep txt | cut -d "&" -f1 | cut -d " " -f5 > ARCHmetaSearch.txt
                while read ler;
               ;;
        "4")
                lynx --dump "<https://google.com/search?&q=site:$1+ext:php>" | grep php | grep txt | cut -d "&" -f1 | cut -d " " -f5 > ARCHmetaSearch.txt
                while read ler;
                do
                        wget "$ler" 2> /dev/null
                done < ARCHmetaSearch.txt
                exiftool *.php > metaResultfor--"$1".txt
                rm *?q*

        ;;

        esac

        echo "Fim da coleta!!!!"
        cat metaResultfor--"$1".txt
        echo " "
        echo " "
        echo "Arquivos armazenados em: metaResultfor--"$1".txt"

fi

```


----

# Resolvendo Labs

*LAB1*

Acesso a pagina e no fim dela existe e-mails pra contato

Foi ali que peguei

*LAB2*

site:businesscorp.com.br ext:txt

*LAB3 Site Wayback Machine*

[http://businesscorp.com.br/robots.txt](http://businesscorp.com.br/robots.txt)

Dentro do Robots tem um path

Esse path tu pode por na wayback machine que la atras ele tem várias infos

Trello e Key

*LAB4*

Acessa o trello pego no ultimo lab

[https://trello.com/b/kaqiIGDl/businesscorpcombr](https://trello.com/b/kaqiIGDl/businesscorpcombr)

Pega infos

*LAB5*

Acessa pastebin, e pesquisa por [businesscorp.com.br](http://businesscorp.com.br)

*LAB6*

site:businesscorp.com.br ext:doc

*LAb7*

Baixa arquivo e usa o exiftool pra analisar metadados

exiftool RI.doc

*LAb8*

Pesquisa no linkedin para achar key

*LAB 9*

Pesquisa no linkedin para achar as vagas e ver o firewall