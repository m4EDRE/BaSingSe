
Para esse módulo é importantíssimo que já tenha sido feita uma base sobre Assembly no módulo anterior…

Aqui exploraremos conceitos semelhantes aplicados para Linux

# Introdução

O primeiro conceito importante de se entender no Linux é que ao invés de termos as chamadas de APIs ou endereços de funções utilizaremos as chamadas **`syscalls` como será visto na sequência.**

**Base para syscalls e suas chamadas:**

- `/usr/include/x86_64-linux-gnu/asm/unistd_64.h`
- `/usr/include/x86_64-linux-gnu/asm/unistd_32.h`
- `https://syscalls.w3challs.com/`
- `man syscalls`
- `man 2 <syscall>`

**Compiladores:**

- nasm: `nasm -f elf32 <arquivo.asm>`
- GoLink:`ld_entry_main <file.o> -o <file>`
    - `ld_entry_main -M <arquitetura> <file.o> -o <file> elf_i386`

**Debuggers**:

- **GDB e GDB TUI**

**`int 0x80`**; Chamada da syscall em linux



----





# Criando Script em Assembly

A seguir entenderemos qual o processo para criar um script Assembly básico….

1. Primeiro passo básico de um script em Assembly é criar a estrutura básica que revisamos na página anterior
    
    ```bash
    global _main
    
    section .text
    _main:
    ```
    
2. Com a estrutura mais básica possível criada agora precisamos escolher qual syscall usaremos, para esse exemplo será usada a write
    
    1. Acha syscall nos arquivos vistos na introdução
    2. Entendimento de argumentos necessários da função com `man <syscall>`
    3. Revisão do número referente a syscall em `unistd.h` (aqui =4)
    
    ```bash
    global _main
    
    section .text
    _main:
        mov eax, 4 ;4 é o numero da syscall
        mov ebx, 1 ; De acordo com a doc em ebx deve ir stdin,stdout padrão
        mov ecx, curso ; Passa em ecx a variável de valor a ser escrito
        mov edx, 15 ; De acordo com a doc aqui passamos o tamanho da variável posta em ecx
        int 0x80 ; Chamada da syscall em linux
    ```
    
3. Criação de variáveis necessárias na sessão de .data
    
    ```bash
    global _main
    
    section .text
    section .data
        curso db "Desec Security",0xa
    _main:
        mov eax, 4 ;4 é o numero da syscall
        mov ebx, 1 ; De acordo com a doc em ebx deve ir stdin,stdout padrão
        mov ecx, curso ; Passa em ecx a variável de valor a ser escrito
        mov edx, 15 ; De acordo com a doc aqui passamos o tamanho da variável posta em ecx
        int 0x80 ; Chamada da syscall em linux
    ```
    
    *PERCEBA que aqui Desec Security tem 14 caracteres, + 1 para pular a linha (0xa) temos os 15 passados no main
    
4. Adiciona a função exit no final do código (Padrão)
    
    ```bash
    global _main
    
    section .text
    section .data
        curso db "Desec Security",0xa
    _main:
        mov eax, 4 ;4 é o numero da syscall
        mov ebx, 1 ; De acordo com a doc em ebx deve ir stdin,stdout padrão
        mov ecx, curso ; Passa em ecx a variável de valor a ser escrito
        mov edx, 15 ; De acordo com a doc aqui passamos o tamanho da variável posta em ecx
        int 0x80 ; Chamada da syscall em linux
        
        mov eax, 1
        mov ebx, 0
        int 0x80
    ```
    
5. Compiladores
    
    ```bash
    nasm -f elf32 <código>.asm
    ld --entry _main -m elf_i386 <código>.o -o <código>
    ```
    
    NOTA:
    
    - Perceba que `eax`recebe o número da função e `ebx` recebe seu conteúdo



----



# Debbugers

A seguir temos a sessão de debuggers para aprofundarmos alguns conceitos e comandos utéis na utilização desses itens no Linux

### Comandos base:

- `gdb` : Inicializa GDB
    - -q : Inicializa sem infos
- `gdbtui` : Inicializa interface do GDB
- `break _main` : Ao rodar põe break point na main
- `info registers` : Mostra informações dos registradores
    - `i r`
- `dissamble` : Mostra sintaxe do código que você está
    - `dissas`
- `set dissambly-flavor intel` : Mostra contéudo no forma intel (hexadecimal)
- `stepi` : Roda até o próximo passo
    - `si`
- `x/s <endereço de memória>` : Mostra conteúdo que está alocado no endereço de memória
- `run` : Roda programa

### Comandos especiais TUI

- `gdb -q <programa> -tui` : Abre interface do GDB
- `layout asm` : Passa a tela do layout o código analisado
- `layout reg` : Passa a tela os registradores do código analisado

### Interfaces GDB:

- GDB:
    
    ![[Pasted image 20251119223618.png]]
    
- GDB TUI:
    
![[Pasted image 20251119223624.png]]
    



----




# Criando Script x64 em Assembly

*Lembre-se que para pegar informações do x64 é em: `/usr/include/x86_64-linux-gnu/asm/unistd_64.h`

1. Inicie como anteriormente entendendo a syscall que usaremos verificando na documentação presente no Linux.
    
2. A seguir crie a estrutura básica para o script Assembly
    
    ```bash
    global _main
    
    section .text
    _main:
    ```
    
3. Crie as variáveis necessárias de acordo com a função desejada.
    
    ```bash
    global _main
    
    section .text
    section .data
        curso: db "<texto>",0xa
    
    _main:
    ```
    
4. Desenvolva o código Assembly dentro do main mas utilizando os registradores corretos da **arquitetura x64**, em caso de dúvida é possível visitar as libs para encontrar essas informações.
    
    ```bash
    global _main
    
    section .text
    section .data
        curso: db "<texto>",0xa
    
    _main:
        mov rax, 1
        mov rdi, 1
        mov rsi, curso
        mov rdx, 15
        syscall
    ```
    
5. Adicione a função exit no fim do código.
    
    ```bash
    global _main
    
    section .text
    section .data
        curso: db "<texto>",0xa
    
    _main:
        mov rax, 1
        mov rdi, 1
        mov rsi, curso
        mov rdx, 15
        syscall
        
        mov rax, 60
        mov rdi, 0
        syscall
    ```
    
6. Adicione os compiladores e finalize o código
    



-----





# Monitoramento de Syscalls

Como ultima ferramenta a ser explorada nesse módulo entenderemos como monitorar syscalls utilizando o strace

`strace ./<programa>` : Intercepta, registra e monitora syscalls no sistema

Com ele é possível ver via interface gráfica cada uma das syscalls sendo chamadas na execução do programa.

**LEMBRE-SE** também —> `objdump -d -M intel <programa>`

Mostra código em Assembly do programa.



----





# Testando a criação de um código baseado em nova Syscall

### Passo a passo

1. Selecione a syscall

```bash
Visualize a syscalls utilizando --> man syscalls

Abra a syscall utilizando: man <syscall> (A selecionada para esse caso foi a creat, para criar arquivos) <--- LER BEM DOC
	int creat(const char *pathname, mode_t mode);
	*pathname --> ebx vai receber o path pra abrir arquivo
	mode_t --> ecx vai receber o tipo de permissão do arquivo (O_RDWR)
	

Descubra número da syscall com: cat /usr/include/x86_64-linux-gnu/asm/unistd_32.h 
	#define __NR_creat 8
```

1. Construção de código padrão
    
    ```bash
    global _main
    
    section .text
    
    section .data
    
    _main:
    ```
    
2. Crie váriaveis necessárias para o main acima do mesmo
    
    ```bash
    	section .data
        path: db "/home/mytmsh/Scripts/Assembly"
    ```
    
3. Crie o conteúdo do main e a chamada exit
    
    ```bash
    global _main
    
    section .text
    
    section .data
        path: db "/home/mytmsh/Scripts/Assembly/criacao",0
        modo: db "O_RDWR",0
    
    _main:
    
        mov eax, 8 ;Envio do número da syscall pra eax
        mov ebx, path ;stdin,stdout padrão da função de acordo com a doc
        mov ecx, modo ;Envio de váriavel de texto para ecx
        int 0x80 ;Call da função
    
        mov eax, 1 ;Chamada do número da syscall do exit em eax
        mov ebx, 0 ;Inserção de item padrão em ebx pra exit
        int 0x80   ;Call da função
    ```