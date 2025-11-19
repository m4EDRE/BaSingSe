# Introdução ao terminal (Commands)

Principais comandos a serem utilizados pelo controlador do Linux

- `pwd`: Mostra caminho em que você está
- `man` : Mostra manual do comando
- `ls` : Lista o que tem dentro da folder que você está
- `cd` : Muda o diretório que você está
- `mkdir` : Cria diretório(Folder)
- `touch` : Cria arquivo
- `echo` : Comando que retorna item no terminal
- `>` : Envia algo a um arquivo
- `>>` : Envia algo a um arquivo adicionando no final sem sobrepor
- `cat` : Mostra conteúdo do arquivo
- `tac` : Mostra conteúdo de trás pra frente
- `cp` : Copia arquivo para um diretório `<arq> <diretório/arq2>`
- `mv` : Move arquivo para outro lugar `<arq> <diretório>`
- `head/tail` : Mostra começo/final do arquivo
    - `tail -f` : Mostra últimas linhas em tempo real
- `lscpu` : Informações sobre CPU
- `lshw` : 🔧 **Informações gerais do sistema**
- `lsblk` : 💽 **Informações sobre disco**
- `inxi -Fxz` : Resumo do sistema






# Gerenciando usuários

Sessão que explica como gerenciar usuários no sistema

- `adduser` : Adiciona usuário ao sistema, nesse comando ele solicita as informações necessárias como nome e senha
- `deluser` : Deleta usuário
- `su` : Muda de usuário

***/var/log/auth.log** —> Logs de autenticações no sistema



# Gerenciando a Rede no Linux (IP estático)

Abaixo está o processo para alteração do IP de uma interface utilizando os métodos com _ifupdown net-tools_

**—> apt install ifupdown net-tools**

Comandos para verificação de IP

- `ifconfig`
- `ip addr`

Comando que altera IP de interface de forma não persistente

```
ifconfig <interface> <IP> <Mask>
```

Comandos que alteram IP de interface de forma persistente,

```
nano /etc/network/interfaces

auto <interface>
iface <interface> inet static
address <IP>
netmask <máscara>
gateway <gw>
```

Após aplicar as mudanças utilize um dos comandos abaixo para reiniciar os serviços

```
/etc/init.d/networking restart

service network restart
```

Comandos adicionais
# Introdução ao VIM

Abaixo está um resumo e introdução as principais funcionalidades dos editor de texto **VIM.**

**Modos**

- Normal: Modo primário na entrada do editor, que permite aplicar comandos
- Edição: Modo que permite editar o arquivo selecionado —> i ou o
- Command: Modo de aplicação de comandos —> :
- Visual: Apenas visualização —> ctrl + v

**Principais comandos utilizados no editor:**

- `wq` - Salva e sai
- `ZZ` - Zalva e Zai
- `ctrl+q` - destrava terminal
- `G` ou `GG` - Ultima linha/Primeira linha
- `O` - Abre inserção no começo da linha
- `$` - Vai ao final da linha
- `u` - Desfaz ultima ação
- `dd` - Deleta linha
- `yy` - Copia linha
- `p` - Cola linha
- `x` - Deleta caracter
- `/` - Busca palavra
- `:set number` - Enumera linhas

**Substituição de arquivos:**

`%s/joao/jose/gc` - Comando que substitui João por José


# SED

Sessão alternativa para falar e rever itens do comando SED uteis no dia a dia

## Removendo linhas vazias de um arquivo

```bash
sed -i '/^$/d' arquivo
```

### Explicação:

- `/^$/`: Esta expressão regular corresponde a linhas vazias.
    - `^` representa o início da linha.
    - `$` representa o final da linha.
    - Juntos, `^$` significa "uma linha sem nenhum conteúdo entre o início e o fim".
- `d`: O comando `d` diz ao `sed` para **deletar** as linhas que correspondem ao padrão.