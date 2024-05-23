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

Neste passo, é onde são carregadas as [configurações](./9023.md#configurações-da-rotina-9827) que impactam diretamente as atribuições de ordens de serviço e também as lista de ruas que estão super lotadas e o total de funcionários nas ruas.

### Total de ordens de serviço nas ruas

São analisadas apenas ordens do tipo 98 na posição `P`, e totalizadas pelo número da rua do endereço da ordem de serviço.

### Total de funcionários nas ruas

São analisadas ordens dos tipos 61 e 98, em que o [status](./status_senha.md) da senha esteja como `E` ou `R`, e totalizadas pelo número da rua do endereço da ordem de serviço.

No caso do tipo 61, consideramos o endereço de origem.

### Ruas super lotadas

São analisadas apenas ordens do tipo 98 na posição `P`, e totalizadas pelo número da rua do endereço da ordem de serviço.

As ruas que excederem a [configuração 249](./9023.md#configurações-da-rotina-9827) serão consideradas como **SUPER LOTADAS**.

### Ruas ignoradas

As ruas que serão ignoradas nas pesquisas são:

- Ruas onde o total de funcionários excede as [configurações 251 e 252](./9023.md#configurações-da-rotina-9827)
- Ruas definidas na [configuração 248](./9023.md#configurações-da-rotina-9827)

---

> **Busca pela Ordem de Serviço**
>
> A partir desse ponto é que são iniciadas as pesquisas por ordens de serviço para cada solicitação pendente carrega na critério 2.
>
> Em todos os critérios serão consideradas apenas ordens de serviço:
>
> - Dos últimos 30 dias;
> - Na posição `P`;
> - Não estornadas;
> - Que já não estejam em alguma solicitação com [status](./status_senha.md) `E` ou `R`
> - E que não estejam associadas com algum outro usuário (`PCMOVENDPEND.CODFUNCOS`);
>
> Cada critério irá adicionar suas próprias restrições além dessas descritas acima.

## Critério 5 - Ruas super lotadas antes

**Este critério é destinado apenas para operadores de empilhadeira**

O primeiro passo nesse critério é analisar os dados da solicitação anterior finalizada do solicitante, se ela foi atendida e marcada como **SUPER LOTADA** (`BOFILAOS.FLAGSL`), iremos pesquisar as ordens de serviço nesse critério. Caso contrário, esse critério será pulado.

O objetivo é manter os funcionários neste local até que todas as ordens de serviço dali sejam finalizadas.

As ordens de serviço pesquisadas nesse critério são:

- Apenas do tipo 98;
- Apenas onde o campo `PCMOVENDPEND.CODROTINA` seja 1709 ou 1721;
- Apenas aquelas em que a ordem de serviço do tipo 97 do mesmo código U.M.A. e núm. trans. wms esteja finalizada;
- Da mesma rua da última solicitação finalizada pelo operador;

A ordenação do resultado é:

- Data da ordem de serviço, mais antigas primeiro (ASC)
- Estoque disponível do produto no estoque gerencial, os menores primeiro (ASC)
- O giro/dia do produto, os maiores primeiro (DESC)

Se forem encontrados resultados, a primeira ordem de serviço é atribuída a senha com [tipo de serviço](./tipos_servico.md) `SL`, e além do tipo de serviço a senha também é marcada como **SUPER LOTADA** que fará que a próxima solicitação desse mesmo funcionário também entre nesse critério.

Se não forem encontrados resultados, seguimos para o próximo critério.

---