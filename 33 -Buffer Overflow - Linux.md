

Nesse módulo encerraremos o assunto sobre BufferOverflow realizando os processos e um desafio via Linux. Dessa forma conseguimos confirmar esse processo em todos os meios usuais em que ele pode ser explorado.

# Debug do Programa

Para esse módulo usaremos de parametro o programa ~protegido para testes.

### Passo a passo de debug

1. Rodar o programa, para iniciar sempre um pentester deve medir o funcionamento da aplicação. Com isso entendemos que:
    
    1. A aplicação pede senha ao user e relata senha incorreta
    2. Enviando muitos dados temos o erro: Segmentation Fault
2. Debuggaremos o programa com o GDB
    
    1. `Info Function`: Mostra funções presentes no programa
    2. `Disas main`: Mostra código Assembly do programa
3. Verificando a `main` do programa já conseguimos identificar que ele tem a função verifica, que está sendo usada provavelmente para comparação de senha.
    
    1. `disas verifica` : Abre a função verifica para ver seu funcionamento
    2. Abrindo verifica é possível confirmar que o programa solicita input do user com `gets` e compara com algum valor. Caso seja igual ele faz uma ação, caso seja diferente ele toma outro caminho
    
    *Itens para se ter atenção
    
    - Se consegue quase tudo analisando as funções
    - `run <arquivo.txt>` : Permite rodar aplicação no debugger com argumento dentro do txt
    - `/x<quantidade de bytes>xb $<endereço de memória>` : Equivalente ao Follow in Dump do Linux

----





# Tomando controle de um programa

Para esse módulo avançaremos após as analises para entender como tomar o controle do programa, para isso realizaremos um processo já visto para descobrir os bytes de EIP e ESP.

### Passo a passo

1. Põe breakpoint no gets e avança debug até lá
    1. `b* <endereço de memória>`
2. Usa o Follow in Dump para validar os a sobreposição dos bytes em ESP e EIP (lembre da lógica de A’s e B’s para identificar bytes)
    1. `/x<quantidade de bytes>xb $<endereço de memória>`
3. Encontra os bytes fazendo vários envios e descobrindo exatamente onde EIP está
4. Manda ao contrário para EIP o endereço da função `acessa()`para dessa forma triggarmos o que ocorre quando o usuário acerta a senha de forma manual.

_Passo 4 importantíssimo para pentest_

`disas main` :

![[Pasted image 20251119224528.png]]

Perceba como dá pra ver que se ele verifica com sucesso ele chama a função acessa, é isso que queremos entrar de forma forçada!!!




-----





# Truques no Debugger

A seguir entenderemos dois itens interessantes para se saber no GDB, são eles:

### Como direcionar fluxo para um endereço especifico?

1. Abre programa e põe `break main`
2. Encontra endereço a ser chamado de acordo com o que você precisa
3. `set $eip=<endereço>` : Manipula valor do registrador, não precisa ser apenas EIP

### Como achar a senha do programa que estamos revisando no buffer do programa?

*Perceba que isso é possível pois para comparar com a senha passada o programa precisa consultar a senha correta em si mesmo

1. `disas verifica` : Abre função que verifica a senha
2. Põe breakpoint na verificação de senha (JNE)
3. Examina memória com `i r`
4. Usa o Follow in Dump no EIP para verificar seus conteúdos
    1. `x/20s $eip` : Examina 20 bytes de EIP
5. Acha senha usando o follow in dump, conforme esperado!!!



------






# Exploração do binário do Linux

Para finalizar essa sessão entenderemos em um último exemplo como realizar a análise de um binário vindo de um programa a ser analisado. Para isso realizaremos o seguinte passo a passo

### Passo a passo

1. Comece realizando o padrão em todos os casos de pentest. Analise o funcionamento do programa desafio e observe que ele solicita senha ao user.
2. Entendido o funcionamento da aplicação para o usuário agora abriremos o GDB para realizar os primeiros passos, observe:
    1. `info functions`: Mostra todas as funções presentes na aplicação
    2. `disas main`: Mostra código main em Assembly da aplicação
3. Realiza o envio de muitos “A’s” e usa o Follow in Dump no GDB para entender como a aplicação recebe, se tem estouro no buffer, como ficam os registradores. Aproveita e já põe um breakpoint logo após a função `gets()`
4. Acha o EIP utilizando o método por padrões com a ajuda do metasploit, como já visto em módulos passados.
5. Identificando as funções que ocorrem quando a senha correta é passada, pegaremos esses endereços das funções (acessa() por exemplo) e direcionaremos o GDB para esse endereço. Com isso é possível pegar a chave do desafio após investigar buffers na aplicação.
    1. `set $eip=<endereço>`