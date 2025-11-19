
# Introdução

Tipo de arquivo: `.c`

C é uma linguagem que precisa ser compilada previamente antes da execução, sendo assim após configurar seu arquivo .c é necessário utilizar o seguinte comando para compilar:

`gcc <arquivo.c> -o <nomedoarquivo>`

Com isso você conseguirá rodar o arquivo como `./nomedoarquivo`

## Itens básicos:

**Main** - Todo arquivo c deve ter uma função main para que o compilamento seja feito corretamente, abaixo está o main básico onde não existe parâmetros passados pelo usuário.

Perceba que no código também é passada biblioteca básica no C, a **<stdio.h>**

```c
#include <stdio.h>
int main(void){
	códigos....
}
```

**PrintF** - Com o comando **printf podemos printar itens com a linguagem c, perceba que de forma básica ele abre um () e a string está entre “ “.**

`printf("Primeiro print");`

***TODOS AS LINHAS QUE NÃO SÃO FUNÇÕES NECESSITAM DE ; NO FINAL**

## Variáveis

Em relação as variáveis é importante ressaltar a necessidade de tipagem para cada uma delas e a chamada na string ocorre de forma bem especifica que revisaremos abaixo

Para chamar na string:

- **%s - Char**
- **%i - Integer**
- **%f - Float**

Exemplo: printf(”Estou chamando uma variável aqui: %i”,ip);

No exemplo acima a variável IP foi referenciada previamente como int.

*Ponto de atenção: Criação de var Char:

Char nome = “mateus”;

Char lista [20]; *char vazio necessita de tamanho


---

# Entrada de dados (Parâmetros) e comandos no terminal

Nessa sessão primeiramente precisamos entender como solicitar ao usuário que envie para parâmetros que complete o código, para isso existe a função em C chamada de `scanf`
```c
char ip[20]; 

printf("Passe o IP a ser explorado:"); 
scanf("%s",%ip);
```

Para aplicar comandos no terminal utilizaremos uma função simples chamada de system, lembre-se sempre de que para executa-la é necessário utilizar a biblioteca

*<stdlib.h>*
```c
#include <stdlib.h> 

system("netstat -nlpt");
```



---


# Argumentos e repetições

Para utilizarmos argumentos passados na chamada do programa é necessário primeiramente alterar o método main para que seja possível tratar esses argumentos

Novo main: `int main(int argc, char *argv[]){ .... }`

Após alterar o main, a chamada de parâmetro fica extremamente simples pois basta colocar os itens —> argv[1],argv[2]… e assim por diante

Tendo isso em vista é uma boa ideia sempre colocar um manual de como utilizar o programa caso os parâmetros não tenham sido passados. Para isso utilizaremos o condicional `IF` para criar situação de condição:

```c
if(argc<2) 
{ 
	printf("Welcome!"); 
	printf("Para utilizar esse programa passe o argumento na chamada"); 
	printf("Exemplo: ./programa.c <IP>"); 
} 
else{ 
	printf("Código rodando...."); 
}
```

Para finalizar essa sessão aprenderemos a como utilizar repetições no C, para isso utilizaremos a função `for`

O for nesse caso é bem semelhante ao utilizado no Java, sendo um loop simples com a criação de variável na chamada, exemplo:
```c
int ip;
for(ip = 0;ip<255;ip++){
printf("192.168.1.%i sendo escaneado...\n",ip);
}
```




---



# Trabalhando com Sockets em C

Bibliotecas necessárias: `<sys/socket.c> e <netdb.h>`

Os passos a seguir devem ser feitos dentro do main.

1. Crie as variáveis necessárias para o código
2. Estruture o socket com o comando struct
3. Crie o seu socket com os referenciais corretos da função
4. Liste o IP e porta do Socket
5. Conecte com a função connect
6. Trate a conexão, caso ela seja bem sucedida o connect retornará 0!

```c
#include <sys/socket.h> 
#include <netdb.h>

int ip; //1
int porta; //1.2

struct sockaddr_in alvo; //2

meusocket = socket(AF_INET,SOCK_STREAM,0); //3
alvo.sin_family = AF_INET; //3.2
alvo.sin_port = (porta); //4
alvo.sin_addr.s_addr = inet_addr(ip); //4.2

conexao = connect(meusocket,(struct sockaddr *)&alvo, sizeof alvo); //5

if (conexao == 0){ //6
	printf("Conexão bem sucedida");
}
else{
	printf("Conexão não estabelecida");
}
```



---


# Criando portscan em C

Nessa sessão usaremos o socket criado no última sessão de referência para conexões do Portscan.

Para isso seguiremos o seguinte passo a passo:

1. Alterar script para que ele possa receber argumento (IP que será explorado)
2. Cria for com o range de portas
3. Põe socket dentro do for…
4. Fechar conexões se não você causa um DoS…

```c
#include <stdio.h>
#include <stdlib.h>
#include <netdb.h> 
#include <sys/socket.h>
#include <arpa/inet.h> 
#include <unistd.h> //Imports


int main(int argc, char *argv[]){ //Main que recebe argumentos

		int meusocket;
		int conecta;
		
		int porta;
		int inicio = 0; //Variáveis de inicio e fim das portas rastreadas
		int final = 65355;
		
		char *destino; //Variável que recebe argumento
		destino = argv[1];
		
		struct sockaddr_in alvo; //Estruturação do socket fora do for
		for (porta=inicio;porta<=final;porta++){ //For com porta de iniciio sendo a var inicio (0) e indo até a var final
		
		meusocket = socket(AF_INET,SOCK_STREAM,0);
		alvo.sin_family = AF_INET;
		alvo.sin_port = htons(porta);
		alvo.sin_addr.s_addr = inet_addr(destino);
		conecta = connect(meusocket, (struct sockaddr *)&alvo, sizeof alvo);
		
		if (conecta == 0){
		
				printf("Porta %i conectou \n",porta);
} //Caso o retorno do conecta seja 0 ele conseguiu achar uma porta aberta!
```