# Introdução

Nessa página retiraremos um tempo para explorar os conceitos vistos de forma superficial DEP e ASLR. Esses conceitos basicamente são responsáveis pela proteção dos endereços de memória com a finalidade de não fazer eles serem exploráveis

### DEP - Data Execution Prevention

`Função`: O DEP foi criado com o intuito de realizar verificações na unidade de memória ESP e realizar bloqueios caso ocorra a tentativa de execução de comandos em cenários como Buffer Overflow

`Limitação`: O DEP necessita que para ele seja aplicado em sistemas não originais do windows seja habilitado uma flag específica para seu funcionamento. Caso a flag não esteja ativa dentro do serviço, isso pois o DEP não é habilitado por padrão na aplicação

`OBS`:

- É possível habilitar para todos os sistemas independente da flag alterando as configurações Windows
- Programa quando bloqueia algo por causa do DEP retorna Access Violation

### ASLR

`Função`: Simplificando a funcionalidade do ASLR basicamente ele se encarrega de tornar os endereços de memória sempre aleatórios a cada execução, tornando impossível comprometer o sistema utilizando apontamentos no EIP

`Limitação`: O ASLR necessita que para ele seja aplicado em sistemas não originais do sistema seja habilitada a flag `dynamic base` para seu funcionamento. Caso a flag não esteja ativa isso o ASLR não é habilitado por padrão na aplicação.

`OBS`: É possível habilitar para todos os sistemas independente da `dynamic base` alterando as configurações

_Via Task Manager é possível ir details e habilitar o suporte a DEP no que está sendo visto vinculado a aplicação_

_Process Explorer mostra as aplicações rodando e se elas tem ou não DEP e ASLR habilitado_