## Análise de Logs like a Pro

Nessa sessão exploraremos um item de grande importancia para um Pentester que é o entendimento na análise de logs

Para podermos ter um referencial nessa sessão baixaremos o arquivo de logs do site Business Corp para podermos entender se ele está correndo algum perigo.

`wget [www.businesscorp.com.br](<http://www.businesscorp.com.br>)/access.log` —> Baixando arquivo do site

`cat access.log` —> Abrindo ele no nosso Linux

A seguir está o passo a passo primário para fazer uma análise de logs

1. O primeiro passo é entender a lógica com que os logs estão dispostos. No cenário em que estamos testando primeiramente aparece o IP que tentou a conexão, depois a data e logo depois outras informações.
    
    Outro ponto muito importante do primeiro passo é entender o que divide as informações no arquivo. No nosso exemplo existe um espaço entre o IP e a data, esse espaço é delimitador das logs
    
2. Agora entendendo a lógica e o delimitador começaremos a usar o cat (comando que mostra informações) e o cut (comando que delimita saída) para mostrar os IPs que estão fazendo acesso em nosso site.
    
    Como o IP é a primeira coluna do nosso arquivo escreveremos o filtro dividindo as colunas pelo delimitador e filtrando apenas pela primeira coluna
    
    `cat access.log | cut -d “ “ -f1`
    
    Perceba que -**d** de delimitador e **-f1** retorna apenas a primeira coluna
    
    *Você pode usar o `| uniq -c` para ver a quantidade de aparições e o `| sort -un` para organizar sem repetições
    
3. Achando o IP suspeito é valido entender quando foi o primeiro e ultimo acesso dele.
    
    Para isso faremos o seguinte, faremos uma busca apenas pelos arquivos com esse IP e utilizaremos head e tail para visualizar primeiro e ultimo.
    
    `cat access.log | grep <IP> | head -n1`
    
    `cat access.log | grep <IP> | tail -n1`
    
    Isso permite que consigamos entender o range do ataque
    
4. Para confirmar que é um ataque precisamos ter o crivo da recorrência desse acesso, caso seja um acesso repetido muitas vezes no mesmo segundo por exemplo conseguimos confirmar que está sendo feito por uma máquina consequentemente um ataque
    
    Para isso análise as primeiras 500 linhas relacionadas ao IP
    
    `cat access.log | grep <IP> | head -n500`
    
    Tendo feito isso finalizamos uma análise de ataque DoS em um servidor WEB

## Laboratórios (Análise de Logs 1, 2 e 3)

Laboratório 1

Quais IPs estão envolvidos no logs?

`cat access.log | cut -d “ “ -f1 | uniq -c`

Laboratório 2

Qual IP do atacante?

IP que mais aparece nos logs.

Pra confirmar basta ver as requisições desse mesmo IP, se for a maioria no mesmo sec é bot

Laboratório 3

Quando começou o ataque e quando acabou?

`cat access.log | grep <IP do atacante> | head -n1`

`cat access.log | grep <IP do atacante> | tail -n1`

## Laboratórios (Investigando ataque 1, 2, 3, 4, 5, 6, 7, 8, e 9)

Laboratório 1

Examine o atacante de outro arquivo

`cat lab.log | cut -d " " -f1 | uniq -c | sort -un`

Laboratório 2

Examine o arquivo que o atacante conseguiu a informação sensível

Pra isso tu precisar pesquisar pelas requisições bem sucedidas (código 200) e ver qual foi o primeiro diretório a ser visto. Tem que ter o crivo de achar entre os outros dir qual faz mais sentido

`cat lab.log | grep 205.251.150.186 | grep 200`

Laboratório 3

Mesma lógica mas já que tu descobriu o dir sensível adiciona ele na pesquisa

`cat lab.log | grep 205.251.150.186 | grep 200 | grep configuracoes`

Laboratório 4

Mesmo comando mas analisando a mesma linha mais a fundo

`cat lab.log | grep 205.251.150.186 | grep 200 | grep configuracoes`

Laboratório 5

Mesmo comando mas analisando a mesma linha mais a fundo

`cat lab.log | grep 205.251.150.186 | grep 200 | grep configuracoes`

Laboratório 6

Novamente pesquisar pelas bem sucedidas mas agora ver o que vem logo na sequência do 200

`cat lab.log | grep 205.251.150.186 | grep 200`

Laboratório 7

Mesmo comando mas analisando a mesma linha mais a fundo

`cat lab.log | grep 205.251.150.186 | grep 200 | grep intranet`

Laboratório 8

Mesmo comando mas analisando a mesma linha mais a fundo

`cat lab.log | grep 205.251.150.186 | grep 200 | grep intranet`

Laboratório 9

Mesmo comando mas analisando a mesma linha mais a fundo

`cat lab.log | grep 205.251.150.186 | grep 200 | grep intranet`


## Resumo da metodologia utilizada para passar nos LABs e realizar análises boas de logs

Abaixo está resumidamente o passo a passo utilizado para aplicar a análise de logs. Importante reforçar que esses comandos devem ser usados de forma fluída de forma a se adaptar com a situação.

1. Abrir o log e entender padrão
2. Identificar IP do atacante observando a quantidade de requisições
3. Validar ataque vendo o tempo da requisição e as também as ferramentas que ele usou como NMAP, tentativas de exploração de vulnerabilidades e etc
4. Identificar o primeiro momento do acesso desse IP e ultimo momento com head e tail -n1
5. Para identificar diretório infectado buscar pelo código de acesso (200)
6. Revisar diretórios e observar todos os itens das linhas observadas.
7. Buscar especificamente a informação que você deseja saber