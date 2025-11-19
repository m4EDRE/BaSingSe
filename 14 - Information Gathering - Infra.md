
Palavras chave: DIG

# Código de mapeamento rápido

### Python

Código que mapeia IPs na rede

```python
 python3 -c "import os; [print(f'172.16.2.{i}') for i in range(1, 255) if os.system(f'ping -c 1 -W 1 172.16.2.{i} > /dev/null 2>&1') == 0]"
```

`python -c` : Executa python com 1 comando

`import os` : Importa lib necessária para execução de comandos na shell

`[ ]` : Mantém código em 1 linha

`f'172.16.2.{i}') for i in range(1, 255)` : Cria loop na rede /24 do 1 até o 255 de IPs para percorrermos


----


# Introdução

Nesse módulo observaremos como fazer um mapeamento relacionado aos itens de infraestrutura de um alvo

### Iana (Internacional Assign Numbers Authority):

é a organização mundial que supervisiona a atribuição global dos números na [Internet](https://pt.wikipedia.org/wiki/Internet), entre os quais estão os números das [portas](https://pt.wikipedia.org/wiki/Porta_\(inform%C3%A1tica\)), os [endereços IP](https://pt.wikipedia.org/wiki/Endere%C3%A7o_IP), [sistemas autônomos](https://pt.wikipedia.org/wiki/Sistema_aut%C3%B4nomo_\(Internet\)), [servidores-raiz](https://pt.wikipedia.org/wiki/Servidor-raiz) de números de domínio [DNS](https://pt.wikipedia.org/wiki/DNS) e outros recursos relativos aos protocolos de Internet.

Pode ser de grande ajuda para encontrar os provedores que contém **netblocks** e **ASNs**

Fluxo: Iana —> Países —> Provedores —> Alvos

### Whois

Protocolo que permite encontrar informações de domínio ou IP, utiliza porta 43

O papel do whois da Iana é mostrar autoridade responsável, daí você faz o whois nessa autoridade para pegar infos como **netblocks** e **ASNs**

*Protocolo que pode substituir o Whois é o RDAP


-----




# Mapeando infra (Pesquisa por IP)

### Como descobrir ASN e Netblock?

1. Verifica IP do domínio
    1. `host <domínio>`
2. Aplica whois nesse IP
    1. `whois <IP do domínio>`
3. Observa as sessões:
    1. `inetnum:` e `aut-num:`

**IMPORTANTE**: Muitas empresas mascaram isso com proxys em seus domínios primários, vale a pena sempre conferir nos subdomínios também para ter certeza.

### Mapeando BGP

Para aprofundar ainda mais o que você achou é possível usar os sites [bgpview.io](http://bgpview.io) e [bgp.he.net](http://bgp.he.net) para pegar mais informações dos ranges via browser.

### Como achar hosts relacionados ao um domínio do AD?

`crackmapexec smb <lista de hosts possíveis>.txt`


-----




# Shodan e pesquisa DNS

Outra ferramenta extremamente útil no mapeando de infraestruturas é o chamado google dos hackers ou _Shodan_, nele é possível pesquisar por devices expostos na internet e fazer uma série de invasões direcionadas a partir dos operadores;

Operadores do Shodan:

- `ip:192.168.1.1`
- `net:192.168.1.0/24`
- `port:443`
- `city:"São Paulo”`
- `hostname:example.com`

Pesquisa DNS: Para os próximos itens é importante sabermos o básico sobre DNS para que as buscas sejam mais completas e tenhamos conhecimentos sobre as execuções sendo feitas.

Itens Básicos do DNS:

- A: IPv4
- AAAA: IPv6
- CNAME: URL ou domínio (alias do address)
- TXT: Texto que contém informações úteis do NS
- NS: Nameserver ou servidor que resolve os nomes para o device

Fazendo pesquisa baseado em item com host:

`host -t CNAME 183.123.94.101`

`host -t ptr 183.123.94.101` :Usado para retornar o dominio

_**MAIS IMPORTANTE**_

`host -a <dominio>`

**Equivalente com DIG** : Utilize apenas a nomenclatura após do dominio para fazer a pesquisa

- `dig -x <IP> +short` : Consulta dominio vinculado ao IP pra usar os outros comandos
- `dig <dominio> NS +short`
- `dig <dominio> A +short`
- `dig <dominio> AAAA +short`
- `dig <dominio> CNAME +short`
- `dig <dominio> TXT +short`
- `dig <dominio> ANY +noall +answer` : Tenta consultar tudo
- `dig @<IP do DNS> <dominio> +short` : Direciona a pesquisa para um NS especifico que você achou

-----

# Transferência de Zona e BruteForce

### Transferência de Zona

Uma exploração muito famosa de DNS que ocorre quando os NSs do ambiente estão mal **protegidos** ou **desatualizados** é a transferência de zona.

Basicamente a transferência de zona se baseia no AXFR que é o protocolo que sincroniza as tabelas entre Nameservers em cluster. Com um comando podemos simular a requisição do AXFR e dessa forma pegar todas as informações listadas em um Nameserver muito fácil.

Comando usado: `host -l -a <dominio> <nameserver1, nameserver 2....>`

Tenho um script criado que já tenta forçar essa ação nos NSs encontrados de um domínio.

### Bruteforce

Outra maneira muito usada de verificar subdomínios é utilizar um txt contendo diversas possibilidades e rodar em cima do domínio escolhido buscando obter retorno positivo.

Para isso baixei um TXT de [https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/shubs-subdomains.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/shubs-subdomains.txt) entre outras listas e adicionei ao meu script em bash. Dessa forma são testados diversos domínios e os que respondem corretamente aparecem na tela.

O principal nessa sessão é ter uma boa wordlist para verificação.

Também já foi criado um script para essa sessão em BADFISH.

[https://github.com/m4EDRE/WARISPEACE/blob/main/bruteforcedns.sh](https://github.com/m4EDRE/WARISPEACE/blob/main/bruteforcedns.sh)

Uma vez com os **`DNS coletados`** podemos fazer o **`Equivalente com DIG` :**

- `dig axfr @<IP do DNS> <dominio>`
- `dig axfr @<Endereço NameServer> <dominio>`


----


# Reverse DNS e SPF

Para fazer uma busca por domínios pode descobrir o range de IPs do alvo como já visto no módulo **Mapeando infra** e a partir desse range faremos pesquisas reversas para identificar novos sub domínios

Passo a passo do script:

1. Encontre range de IPs utilizando host e whois
2. Utilize o comando `host -t ptr <IP>` para fazer as pesquisas
3. Trate o retorno para que apenas os novos subdomínios apareçam

**`Equivalente com DIG` :**

- `dig -x <endereço IP> +short`

### SPF

A função principal do SPF é realizar a verificação de emails que entram na caixa de entrada dos funcionários de uma empresa, dependendo da sua configuração ele pode mandar um email de dominio estranho para caixa de entrada, SPAM ou direto para lixeira….

Classificação:

- `?all` = Configuração de SPF vulnerável
- `~all` = Configuração de SPF semi vulnerável
- `-all` = SPF não vulnerável

Caso o SPF esteja vulnerável ou semi vulnerável podemos usar ferramentas como o [emkei.cz](http://emkei.cz) para forjar emails vindo domínios extremamente parecidos para os usuários do alvo. Emails que poderiam conter phishing, malwares, informações adulteradas e muito mais.

Comando para verificar SPF de host: `host -t txt <dominio>`

-----


# Subdomain Takeover e Outras ferramentas

### O que é Subdomain Takeover?

A ideia da técnica de Subdomain Takeover basicamente é achar endereços relacionados ao alvo que tenham CNAME mas esse redirecionamento de em um site desativado, sites como AWS permitem que você registre esse CNAME caso seja relacionado a eles, e dessa forma, você assumiria controle para o CNAME do site do alvo.

Isso permite que você tenha um sub dominio ativo vinculado ao dominio do alvo liberando você a manipular esse site como quiser, aplicando malwares, fake sites para coletar informações e muitos outros casos.

### Como achar um domínio que permite exploração?

1. Pegue os subdomínios mapeados até então e aplique o seguinte comando:
    1. `host -t cname <subdominio1>`
2. Caso o subdomínio tenha um CNAME registrado a ele o próximo passo é utilizar o comando host novamente para ver se esse endereço que tem CNAME está ativo. Caso retorne (NXDOMAIN) seguimos as verificações. Outro ponto é se esse dominio estiver **atrelado** a um site como **WordPress ou Amazon**, nesse caso ele vai mostrar quando acessado que não está mais disponível e muitas vezes você pode registrar outro site com esse mesmo nome e tomar controle **mesmo não aparecend**o o NXDOMAIN
    1. `host <subdominio1>` ou `www.subodminio1.com`
3. Feito esse processo certifique-se que verificou bem todos os alias que você encontrou para que tenhamos certeza de que tudo foi validado. Certifique-se de validar o subdomínio pesquisado na wordlist e também o alias para qual foi resolvido.

### Tomando controle de um subdomínio

Um exemplo que temos de tomar controle seria pegar um subdomínio que tem cname para um bucket na Amazon do nosso alvo mas está desativado.

Depois disso acesse a Amazon e registre um bucket com esse subdomínio do alvo.

Feito! Agora você pode fazer o que quiser com o controle de um subdomínio do alvo.

![[Pasted image 20251109204900.png]]


----



# Análise via certificados WEB

Nessa sessão utilizaremos os certificados das páginas alvo para poder explorar um pouco a mais sobre a infra do nosso alvo.

O site [https://crt.sh/](https://crt.sh/) permite validar os certificados emitidos para um dominio a que queremos revisar seja ele qual for. Basta por o dominio selecionado na barra de pesquisa

Essa tarefa é **EXTREMAMENTE** útil na pesquisa de subdomínios pois o site faz uma lista dos mesmos que possuem certificado HTTPs

Sendo assim essa é uma tarefa que não pode de forma alguma ser evitada no passo a passo de levantar informações de INFRA


![[Pasted image 20251109204918.png]]


---


# Resolvendo Labs

*Lab 1*

Primeiramente faz um: `host businesscorp.com.br`

Achou o IP manda pro `whois <IP>`

Ele vai retornar tando o inetnum quanto o ASN

*Lab 2*

Mesmo processo do 1, só vai pegar o ASN agora

*LAB 3*

Para resolver esse usei a técnica de transferência de zona

host -l [businesscorp.com.br](http://businesscorp.com.br) [ns2.businesscorp.com.br](http://ns2.businesscorp.com.br)

*LAB 4*

basta usar comando host pra resolver

*LAB 5*

Utilizei script de dns reverso que pega os IPs no range e resolve eles pra dominio

*LAB 6*

Apenas resolvi os subdominios encontrados para o IP com o comando host

*LAB 7 (MAIS DIFERENTE)*

Ele quer mais infos sobre o DNS do host, pra isso usei

host -a [businesscorp.com.br](http://businesscorp.com.br)

*LAB 8*

Resolvi com host -t txt [businesscorp.com.b](http://businesscorp.com.br/)r

*LAB 9*

Pra resolver esse eu passei a meu txt bruteforce mas dessa vez resolvendo pra cname

Existia um dominio [admin.businesscorp.com.br](http://admin.businesscorp.com.br) que quando você resolve o cname dele aparece a key