**Ponto de atenção:**

- **Flags PUSH(PSH)** : Enviam comandos ou output na comunicação
- **Flags FIN(F ou F.)** : Demonstram a finalização da comunicação e denotam que essa porta estava aberta

# Wireshark

Nessa sessão faremos uma exploração sobre a ferramenta Wireshark de análise de tráfego via GUI.

## Ferramentas do Wireshark

Clicando com o botão direito em um pacote é possível fazer duas ações extremamente úteis no dia a dia das análises.

- Com o botão direito selecione a opção de fazer essa coluna como filtro. Isso permite que você use a coluna para pesquisar outros itens com o referencial das linhas de filtro
- **Follow TCP/Outro Protocolo Stream:** Permite ver fluxo daquela conexão em outra aba detalhadamente
    - **Importante ressaltar o quanto essa sessão é importante para seguir e entender o fluxo de um pacote ou transmissão seja ela qual for, mais pra frente isso será importante em um LAB**
- **tcp contains `<algo>`:** Pesquisa nos itens TCP por uma string `<algo>`
- **http.request** : Solicitação HTTPS
- **ip.addr** = <> : Filtro por IP
- **tcp contains*** `<algo>` : Permite pesquisar no payload por algum output ou flag interessante

Importante ressaltar que o segredo da análise no Wireshark é usar os filtros mas manter a análise sequencialmente entendendo pacote por pacote.

Ajuda MUITO ir anotando as conexões, IPs e movimentos que o atacante foi fazendo pelo percurso.

Filtrando por flags no Wireshark:

`tcp.flags.push == 1` ou _ws.col.info contains "PSH”

**Segredo para análise:** Analisar sequencialmente e se apegar aos PSHs, que mostram o que o atacante mandou de comando. É interessante sempre botar o mouse em cima dos bytes e ver o que eles representam nos headers.

Para entender os headers sempre olhe a parte de Data (payload)

# TCPDump

O TCPDump tem funcionalidade de permitir uma análise de tráfego via **CLI**, permitindo a utilização de funcionalidade como **cut,awk, grep e etc.**

### Lendo arquivos com TCPDump:
`tcpdump -r <arquivo>`

```
tcpdump -r <arquivo> 

tcpdump -vr <arquivo> :Verbose 

tcpdump -vnr <arquivo> :Vernose e numerical 
tcpdump -venr <arquivo> 
tcpdump -vAnr <arquivo> :Decodifica o ASCCI 
tcpdump -vAXr <arquivo> :Mostra output 
tcpdump -r <arquivo> | <parametros linux> <argumento> 
tcpdump -r <arquivo> src host <ip> --> Pesquisa por IP Source especifico 
tcpdump -r <arquivo> udp --> Pesquisa por pacotes UDP 
tcpdump -r <arquivo> port <porta> 
tcpdump tcp[tcpflags] & tcp-ack != 0' --> Pesquisa por flag ACK 
tcpdump -r <arquivo> | grep -w :Pega exatamewnte o que estiver dentro do [] 
tcpdump -r <arquivo> | grep -u :Pega tudo que não tiver o que estiver no []

```

### Lendo arquivos com TCPDump:
`tcpdump -i <interface> -w <arquivo>`

### Analisando ataque Portscan:

1. Comece normalmente analisando o IP do atacante
    
2. Verifica que esse atacante manda Flag SYN para todas as portas, o que classifica como Portscan
    
3. Pesquise por conexões que partam do IP malicioso que retornaram uma Flag [F.] (FIN) o que confirma que houve comunicação que começou e encerrou com a porta
    
    1. **`tcpdump -vnr <arquivo> | grep <IP do atacante> | cut -d “ “ -f<coluna> | grep -w [F.]`**
    
    *Legal ressaltar que o grep -w pega exatamente a linha especificada, e o -u pega todas as linhas que não tem aquela linha





# Laboratórios Explicados:

## Scan Interno:

1. Qual o endereço fisico do atacante?
    1. Basta identificar o atacante, da pra ver aqui pq é o IP privado e pegar o MAC dele
    2. Os primeiros bytes de tudo sempre são os MACs de destino e source
2. Qual fabricante dele?
    1. Pega o MAC e joga no MAC Vendors
3. Qual endereço lógico?
    1. IP
4. Repetido

## Portscan:

1. Quais portas estavam abertas no alvo?
    1. Para identificar isso primeiro filtra por pacotes com a FLAG [F.] que indica que a porta estava aberta
    2. Dai pega as portas relacionadas ao IP do alvo mas nos pacotes quando quem inicia é o atacante

## Acesso:

1. Qual versão do serviço o atacante se autenticou?
    1. Filtra pelas Flags de PSH, que são as referentes a conversa do atacante com o alvo
    2. Eventualmente tu ve ele iniciando o FTP
2. Quais credenciais foram usadas?
    1. Só ir vendo os payloads traduzidos no wireshark para ver ele tentando as credenciais
3. Qual mensagem tem no serviço ao se conectar?
    1. Só ver no payload do wireshark e seguir o fluxo pra entender
4. Qual código, user e senha do usuário usado pelo atacante?
    1. Só ver no payload do wireshark e seguir o fluxo pra entender

## Análise de Bytes:

1. Qual Socket de origem?
    1. O começo desse é meio complexo, vamos começar por Socket = IP:Porta
    2. Os primeiros 12 Bytes são os endereços MAC
    3. Os próximos 2 são referentes ao tipo do Ethernet
    4. O 15 byte nesse caso é 45, basta multiplicar esses dois números para observar quantos bytes o header IP terá (20)
    5. Os ultimos do header são os IPs de source e destino
    6. Achando os IPs basta seguir pois os próximos bytes depois do IP são as source e destinations Ports
2. Quais Flags estão ativas nos bytes?
    1. Aqui o mais facil é usar o wireshark de exemplo pra entender qual byte é o das Flags
    2. Logo após use o Wireshark novamente de exemplo para comparar as flags dos pacotes com os bytes que você está vendo
3. Mesma lógica do socket
4. Mesma lógica das flags
5. Só buscar os primeiros bytes logo após passar o header IP
6. Qual referer foi usado na requisição
    1. Qual URL foi usada no acesso? Basta ver nos payloads traduzidos
7. Qual navegador foi usado?
    1. Mesma lógica do 6
8. Mesma lógica do 6 (cuidar a criptografia do %)
9. Acesse o site e ponha as credenciais pra pegar a key

## Invasão

1. Buscar o endereço fisico ja foi visto acima
2. MAC Vendors
3. IP do alvo
4. Como pegar o serviço já foi visto
5. Quais credenciais foram usadas para se autenticar?
    1. Aqui são as credenciais que após ele por aparece Login Sucessful
6. Qual arquivo foi roubado?
    1. Procurando apenas pelos PSH você vê que ele faz um RETR para o arquivo manual.pdf
7. O MAIS dificil
    1. Aqui procura pelos PSH
    2. Você vai ver que ele pede o RETR do arquivo, marca esse pacote e tira o fitro
    3. Vai seguindo o fluxo até chegar no pacote que tem no payload o binário do PDF, vai dar pra ver PDF-1.4 no payload
    4. Clica nesse pacote e Follow TCP String
    5. Abre ela em modo Raw e salva o arquivo
    6. Muda esse arquivo pra PDF e abre ele, feito dai

