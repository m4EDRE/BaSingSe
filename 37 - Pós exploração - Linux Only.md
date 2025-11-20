



# Enumeração de Host-Linux

Em caso de obtenção de uma shell em host Linux o primeiro passo é fazer o mapeamento básico da máquina afim de ter mais informações e possibilidades para escalonamento de privilégios.

### Comandos básicos

- `hostname` : Hostname da máquina
- `uname -a` : Infos da máquina, versão
- `cat /etc/passwd` e `/shadow/` : Usuários e senhas
- `ip a` : Informações de rede
- `dpkg -l | grep <nome de programa>` : Verifica se a máquina tem certo programa instalado
- `netstat -nlpt ou -nlpu` : Verifica portas tcp e udp abertas
- `ps aux` : Verifica processos ativos
    - `ps aux | grep root` : SEMPRE DAR UMA OLHADA OS PROCESSOS ROOT
- `cat /etc/crontab` : Verifica processos agendados
- `sudo -l` : Mostra arquivos que executam sudo na máquina

### Enumeração automatica

Caso deseje fazer essa enumeração de forma automatizada use o programa: [https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)

ou

[https://github.com/The-Z-Labs/linux-exploit-suggester](https://github.com/The-Z-Labs/linux-exploit-suggester)

Bom vìdeo: [https://www.youtube.com/watch?v=9xtIfiQ63Hk](https://www.youtube.com/watch?v=9xtIfiQ63Hk)

### Kernel

Com `uname -a` ou `cat /etc/issue` podemos verificar a versão do kernel do OS e com isso é possível validar exploits de acordo com essa versão.

### Binários com possibilidade de exploração

Muitas vezes na máquina linux binários que são muito usados como

- `find`, `vim`, `nano`, `cp`, `mv`
- `python`, `perl`, `ruby`
- `nmap` (versões antigas)
- `tar`, `zip`, `unzip`
- `bash`, `sh`, `dash`

Podem estar sendo executado como root e ter permissão de execução. Observe no exemplo a seguir como escalonar privilégios em um caso que o find tem essa permissão:

1. Pesquisa nos processos, `ps aux` mostrou:
    
    `root 14119 find . -exec /bin/sh -p ; -quit` : Parece que tem um find sendo executado por root o que levanta suspeita
    
2. `ls -la /usr/bin/find` : Comando para verificar que o find realmente é do root mas que tem possibilidade de execução remota
    

`-rwsr-xr-x 1 root root 224848 Jan 8 2023 /usr/bin/find` : Confirmado a suspeita

1. `find . -exec /bin/sh -p \\; -quit` : Comando que via find executa uma shell na máquina, quando ela é executada acessamos o servidor

### Sudo Rights

Dentro do sudoers (arquivo do Linux) temos todos os usuários capazes da máquina que conseguem executar comandos como root!

Com o comando `sudo -l` você vê tudo o que aquele usuário pode fazer como root.

Isso pode ser critico em casos em que o sistema libera paths específicos como root achando que não teria problemas, mas na realidade tem

`EX`: Sistema permite que o user execute o path man(Leitura de manuais) de qualquer coisa como root. Dentro do man você pode dar `!<comando>` o que faria com que esse comando rodasse como root!!! —> `!/bin/bash` Cria uma shell admin por exemplo

**`*GTFOBins`** ← : Auxilia na pesquisa por binários que possam ser explorados com sudo, em um caso que não seja o man por exempo o GTFOBins pode te salvar

- Selecione sudo dentro do GTFOBins e veja como `escalar privilégio com cada um dos binários`EX: `cp` `arp` `awk`

### Suid Bit Misconfiguration (SET USER ID)

Em alguns casos temos arquivos específicos que contém a permissão `SUID` em que é possível que executemos um binário via esse arquivo com as permissões do dono do arquivo (Muitas vezes root)

- **Como identificar SUID?**
    
    `find / -perm -u=s 2>/dev/null` : Procura arquivos com permissão `s`
    
    `find / -perm -4000 2> /dev/null` : Mesma coisa
    
- **Como explorar o SUID?**
    
    Novamente pesquisando no **`*GTFOBins`** observamos diversas formas de escalar o priv com o binário, dentro da sessão do binário deve ter `SUID` e dessa forma podemos só rodar esses comandos pra explorar… `(MUITAS VEZES VOCÊ PRECISA DAR UMA LEVE ADAPTADA NO COMANDO DO GTFO)`
    
    Então resumindo ache o binário com SUID, pesquise ele no GTFOBins e adapte o comando do GTFO pro seu path certinho —> Procure apenas pela sessão SUID no arquivo aberto do GTFO
    
    —> Tem que ser dito que esse também é bom demais, cuida pra adaptar bem o seu path achado pro exemplo que ta no site
    
    _NÃO É PORQUE NAO TA NO GTFOBIns que não tem…_
    

### Kernel Exploits

Vale muito a pena sempre revisar a versão do Kernel do alvo, pois muitas vezes ele pode ter vuls como:

- Uso de Buffer Overflow, Libs vulneráveis, mudança de UID em exploit publico entre outros exploits encontráveis na internet
    
    ```
    —> `Pesquisa no google a versão do Kernel e vulnerabilidades para descobrir itens e exploits`
    ```
    

### Path Bad Configuration

No linux temos a variável `PATH` que podemos usar para validar todo o path em que estamos `echo $PATH` tudo o que está no PATH pode ser executado em qualquer pasta, temos que ter atenção a isso!

**Como explorar?**

1. `ECHO $PATH`

### Weak Reused and PlainTexts

Com as senhas que achamos até o momento, porque não tentar acessos com as senhas que já temos?? Para isso faça os itens abaixo:

- Reutilize senhas em usuários já encontrados e em outros da forma que puder
- Revire arquivos nos diretórios para encontrar senhas e acessos

**`Técnica`**

- Comece com `sudo -l` (Veja se pode ou não)
- Verifica `SUID`
- Verifica senhas em usuários de todas as formas possíveis
- `getcap -r / 2>/dev/null` : Pesquisa capabilities, pega elas e joga no google com escalation pra ver se tem algo interessante
- Verifica Kernel (Exploits)
- Pesquisa arquivos *.conf e *.db com find
- Faça outra pesquisa com itens interessantes do find EX: `find / -type -f -name “*.txt” 2>/dev/null`

***Organize de forma que sempre seja consultável de forma fácil cada item avaliado acima

### Capabilities

Nesse item explicarei como explorar um item importantíssimo, as capabilities

**O Que São Capabilities?**

Linux Capabilities dividem os poderes do root em pequenos pedaços. Em vez de dar poder total (root), você pode dar poderes específicos a binários.

**Exemplos**:

- `cap_net_raw` = Pode criar pacotes de rede raw
- `cap_net_bind_service` = Pode bindar portas < 1024
- `cap_setuid` = Pode mudar o User ID para qualquer usuário

**Como achar algo interessante?**

`getcap -r / 2>/dev/null` : Mostra binários que tem essas permissões quebradas do root

Exemplo de output:

```bash
/opt/perl cap_setuid=ep

cap_setuid = O binário pode mudar seu UID (User ID)
ep = Effective + Permitted (capability está ativa e pode ser usada)
Implicação: /opt/perl pode executar código como qualquer usuário, incluindo root (UID 0)
```

Como explorar?:

```bash
Comando achado no Capabilities do perl no GTFOBins:
- Lembre que adaptei pro path...
/opt/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/bash";'

Breakdown do comando:
-e = Executa código Perl inlineuse POSIX 
qw(setuid) = Importa função 
setuidPOSIX::setuid(0) = Muda UID para 0 
(root)exec "/bin/bash" = Executa bash (que agora roda como root)
```

**`* SENDO ASSIM É DEVEMOS TER ATENÇÃO QUANDO COMO CAPABILITIES TIVERMOS EP OU EPI** *`

**`USE O GTFO BINS NA AREA DE CAPABILITIES** *`

### Resumo:

### `Ordem de Prioridade:`

1. **`sudo -l`** ← Verificar o que pode executar com sudo
2. Verificar `.bash_history` de TODOS os users…
3. **`getcap -r / 2>/dev/null`** ← Capabilities (o que achamos aqui!)
4. **`find / -perm -u=s 2>/dev/null`** ← Binários SUID
5. **`cat /etc/crontab`** ← Cron jobs rodando como root
6. **`ps aux | grep root`** ← Processos rodando como root
7. **`find / -writable`** ← Arquivos que você pode modificar
8. **`netstat -tulpn`** ← Serviços rodando internamente
9. **Exploits de kernel** ← Última opção

![[Pasted image 20251119230628.png]]

**Perdido?**

**`Se você tem acesso ao dir ou arquivo interessante isso pode ser uma dica… tente dar um cat em um arquivo conhecido ou tente copiar o arquivo para um novo em que você possa manipular`**

### `Revisar CRONTAB SEMPRE e SEMPRE REVISAR EXPLOITS PRO KERNEL`

→ [https://www.youtube.com/watch?v=MjptlhAgaK0&list=LL](https://www.youtube.com/watch?v=MjptlhAgaK0&list=LL)

