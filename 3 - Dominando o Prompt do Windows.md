# Introdução ao prompt (principais comandos)

Os comandos do terminal no Windows são parecidos com o Linux mas com algumas alterações. Abaixo estão os principais comandos a serem usados e seus respectivos comparativos no Linux

- `cd` : Mudança de diretório/folder
    
- `dir` : Mostra o que tem no diretório em que você está (ls)
    
- `mkdir/rmdir` : Cria/Remove diretório
    
- `>` : Envia output para um arquivo estabelecido
    
- `echo` : Imprime output na tela
    
- `cls` : Limpa o terminal (clear)
    
- `type` : Mostra conteúdo dentro de arquivo (clear)
    
- `copy` : Copia arquivos (cp)
    
- `move` : Movimenta arquivos para outros locais (mv)
    
- `del` : Deleta o arquivo selecionado

- `/? = —help`
    
    - Ex. `attrb /?`
- `atrrb +h <dir>` : Oculta arquivo (arquivos . no Linux)
    
    - Só pode ser visto com `dir /a`
    - Pode ser desfeito com `-h`

# Pesquisa de arquivos e Portas

Primeiramente a seguir revisaremos como realizar uma pesquisa por arquivo no prompt windows:

```jsx
		dir /b *<Nome a ser procurado>* --> Realiza pesquisa de qualquer coisa com o nome no diretório que voce está
		
		dir /s *<Nome a ser procurado>* --> Realiza a pesquisa de qualquer coisa em todo o disco
```

Para completar a introdução ao prompt é importante revisar alguns comandos que podem ser úteis no dia a dia que tem funcionalidades para manter o sistema monitorado e corretamente configurado, são eles:

- `netstat -n` : Demonstra portas e serviços abertos
    
    - `-p tcp` : Filtra pelas portas TCP
    - `-p udp` : Filtra por portas UDP
- `tasklist/taskkill -pid <pid>` : Mostra serviços e encerra eles
    
- `net user <nome> <senha> /add` : Adiciona usuário ao sistema