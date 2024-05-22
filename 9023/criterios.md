# Critérios

> **ATENÇÃO**
>
> Nessa primeira parte, estamos apresentando os critérios em ordem numérica. Porém na segunda parte iremos mostrar a ordem de execução, onde será possível verificar que alguns critérios são executados mais de uma vez mudando apenas o range de ruas a serem consideradas.

---

## Critério 1: Cancelar senhas atribuídas e não iniciadas

De acordo com a [configuração 262](./9023.md#configurações-da-rotina-9827), há um tempo limite onde o solicitante pode iniciar a ordem de serviço depois de atribuída a ele. Caso esse tempo seja ultrapassado, esse critério faz o cancelamento da senha para que a ordem de serviço seja liberada para os critérios seguintes.

---

## Critério 2: Listando solicitações pendentes

São consideradas solicitações pendentes todas aquelas com status igual a `'A'`, e para análise elas são listadas por ordem de senha, como a senha é definida por uma [sequence](./banco_dados.md#bofilaos) isso nos garante que iremos seguir a ordem de solicitação.

---

## Critério 3: Retornando a ordem de serviço atual

Caso haja uma senha anterior não finalizada para o solicitante, a senha anterior é cancelada, e a mesma ordem de serviço é atribuída nessa nova senha.

O [tipo de serviço](./tipos_servico.md) nesse caso fica como `RT`.

---

## Critério 4: Carregar totais e configurações

Neste passo, é onde são carregadas as configurações que impactam diretamente as atribuições de ordens de serviço e também as lista de ruas que estão super lotadas e o total de funcionários nas ruas.

### Total de ordens de serviço nas ruas

São analisadas apenas ordens do tipo 98 na posição `P`, e totalizadas pelo número da rua do endereço da ordem de serviço.

### Total de funcionários nas ruas

São analisadas ordens dos tipos 61 e 98, em que o [status](./status_senha.md) da senha esteja como `E` ou `R`, e totalizadas pelo número da rua do endereço da ordem de serviço.

No caso do tipo 61, consideramos o endereço de origem.

### Ruas super lotadas

São analisadas apenas ordens do tipo 98 na posição `P`, e totalizadas pelo número da rua do endereço da ordem de serviço.

As ruas que excederem a [configuração 249](./9023.md#configurações-da-rotina-9827) serão consideradas como **SUPER LOTADAS**.

---
