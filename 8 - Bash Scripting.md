# Introdução a Bash Scripting

A seguir revisaremos o básico para começar a aprender a programar com Bash Scripting.

- `#!/bin/bash` **(Shebang) :** A Shebang tem a função de invocar o interpretador bash no arquivo .sh que estamos programando para que os comandos possam funcionar corretamente
- `#` : Comando utilizado para comentar uma linha, item muito necessário para explicar o código por observações necessárias
- `echo` : Imprime item no terminal ao rodar código, funciona como **print** no python
- `a=<variável>` : Declaração de variável no código, necessita ser **exatamente assim**, sem por espaços entre os itens
- `$<variável>` : O item **$** funciona para chamar variáveis já postas no código ou impostas na chamada do programa, sempre necessário ao invocar os itens postos nas variáveis
    - `$1,$2,$3` : Invoca as entradas postas pelo usuário que rodou o programa, exemplo: ./programa.sh <entrada 1> <entrada 2> <entrada 3>
- `read` : Item que permite leitura via prompt, ao escrever `read <variável>` você necessita que o usuário escreva no prompt um item que será incorporado na variável declarada após o comando
- Armazenando conteúdo de comando em variável: `ipv4=$(host -t A $1 | cut -d " " -f4)`

*Não se esqueça que para rodar o script você necessita dar permissão de execução para ele: **chmod +x <[arquivo.sh](http://arquivo.sh)>**












# Trabalhando com condições

Para utilizar funções de condição no bash scripting temos 2 opções

- if
- case

A seguir revisaremos como usar cada uma delas

### IF

Como no python essa função solicita uma condição para algo acontecer. Interessante em **todo código** fazer um if para caso o usuário rode o código sem parâmetros ele receba um retorno com um manual do código.

```bash
#!/bin/bash

if [ "$1" = "" ]
then 
	echo "Explicação do código"
elif [ "$2" = "" ]
then 
	echo "Outra explicação caso o segundo parâmetro não apareça"
else
	echo "Comandos do código normal"

fi
```

***Importante: [ $1 = “” ] : Variáveis sempre com esse espaço dos []**

### Case

Mesma função do IF, mas permite uma visualização mais simples em casos que o elif seria necessário:
```bash
#!/bin/bash 

case $1 in 
"1")
	echo "Passa o que aconteceria com $1 como 1"
;; 
"2") 
	echo "Passa tudo o que aconteceria se fosse = 2" 
;; 
esac
```









# Trabalhando com argumentos

Nesse caso argumentos são os parâmetros que o usuário envia logo após a chamada do script, esses comandos podem ser invocados no script a partir de `$1,$2,$3,$4…`

Tabela de condicionais úteis em IF e outros operadores:


| Comando | Conteúdo         | Equivalente |
| ------- | ---------------- | ----------- |
| -lt     | Less than        | <           |
| -gt     | Greater than     | >           |
| -le     | Less or equal    | <=          |
| -ge     | Greater or equal | >=          |
| -eq     | Equal            | =           |
| -ne     | Not Equal        | !=          |





# Trabalhando com repetições

Antes de começarmos a trabalhar é importante ressaltar que o `echo` é capaz de criar array que cria repetição, exemplo:

```bash
echo {1..123}

#Cria sequência de 1 até 123 sendo exposta no prompt!
```

Sabendo disso, existem dois tipos de repetição no bash script:

### While (mais usado para leitura de arquivos)

While funciona da mesma forma que em python, a diferença essencial está na forma com que é disposto no código:

```jsx
#!/bin/bash

while true
do
	echo "teste";
done 

ou 

while [ $contador -le 5 ]
do
  echo "Contador: $contador"  # Exibe o valor da variável;
  ((contador++))  # Incrementa o contador em 1;
done

ou 
arquivo="imagemnet.txt"
while read linha;
do
        echo $linha
done < $arquivo
```

### For (mais usado para sequência de números)

For funciona da mesma forma que em python, a diferença essencial está na forma com que é disposto no código:

```jsx
#!/bin/bash

for variavel in {1..123}
do
	echo "$variavel";
done

#Aqui ele vai imprimir na tela números de 1 a 123
```

Fazendo mesmo for contendo variáveis no lugar de números:
```bash
for num in $(seq $primeiro $ultimo); 
do 
	echo "$subnet1.$subnet2.$subnet3.$num" 
	host "$subnet1.$subnet2.$subnet3.$num" | grep -v NXDOMAIN; 
done
```
***Leitura de arquivos:**
```bash
arquivo="imagemnet.txt" 
while read linha; 
do 
	echo $linha 
done < $arquivo
```








# Criando script PortScan

Para testarmos nossos conhecimentos abordados nos tópicos acima criaremos um script que faz a varredura de uma porta especifica nos hosts de uma rede, retornando apenas as portas que estão abertas na rede mapeada.

```bash
#!/bin/bash

if [ "$1" = "" ] "#Caso o user não passe argumento porque não sabe usar Script"
then 
	echo " Explicação de como usar programa "

else
while ip in {1..254} #"Looping para passar por todos os IPs do /24"
do
	hping3 -S -p 80 -c 1 $1.$ip; #"Ping usando a rede passada como argumento para todos IPs do /24"
done

fi
```

Acima o script cumpre o estipulado, pingando todos os itens da rede, o problema é que o output disso ainda não está filtrando os hosts com a porta 80 aberta, a seguir usaremos o grep para realizar esse filtro.

```bash
#!/bin/bash

if [ "$1" = "" ] 
then 
	echo " Explicação de como usar programa ";

else
while ip in {1..254} 
do
	hping3 -S -p 80 -c 1 $1.$i 2> /dev/null | grep sa;
	

fi
```

***Leitura de arquivos:**
```bash
arquivo="imagemnet.txt" 
while read linha; 
do 
	echo $linha 
done < $arquivo
```








# Revisão laboratórios

1. HTML Parcing —> Roda o parcing e pega os subdominios, só observar os que tem que tem um ali que é o certo
2. HTML Parcing 2 —> So pegar o IP do subdominio com o script
3. Malware no Servidor —> Só pegar o endereço do alvo
4. Malware no Servidor 2 —> Só comparar esse IP com os subdominios que tu encontrou
5. Malware no Servidor 3 —> Só ver as portas que o alvo recebe os SYN únicos antes da conexão que se fecha
6. Malware no Servidor 4 —> Pegar IP e porta que o cara usa pra conexão que dá certo (a invasão)
7. Malware no Servidor 5 —> Usando o IP público visto na captura cria um script que o envio de um SYN para cada uma das portas do PortKnoking.
    1. Aqui é fera pq o script pode mandar apenas 1 SYN com `nc -zv -w1 $1 13` e depois eu fiz ele dar um telnet para o IP do server.
    2. Caso o telnet feche é porque a porta ta aberta, daí é so acessar ela via browser com o Socket
8. Malware no Servidor 6 —> Acessando via Web o server pela porta certa da pra preg

