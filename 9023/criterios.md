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

## Critério 5 - Ruas super lotadas onde o operador está

**Este critério é destinado apenas para operadores de empilhadeira**

O primeiro passo nesse critério é analisar os dados da solicitação anterior finalizada do solicitante, se ela foi atendida e marcada como **SUPER LOTADA** (`BOFILAOS.FLAGSL`), iremos pesquisar as ordens de serviço nesse critério. Caso contrário, esse critério será pulado.

O objetivo é manter os funcionários neste local até que todas as ordens de serviço dali sejam finalizadas.

As ordens de serviço pesquisadas nesse critério são:

- Apenas do tipo 98;
- Apenas onde o campo `PCMOVENDPEND.CODROTINA` seja 1709 ou 1721;
- Apenas aquelas em que a ordem de serviço do tipo 97 do mesmo código U.M.A. e núm. trans. wms esteja finalizada;
- Da mesma rua da última solicitação finalizada pelo operador.

A ordenação do resultado é:

- Data da ordem de serviço, mais antigas primeiro (ASC);
- Estoque disponível do produto no estoque gerencial, os menores primeiro (ASC);
- O giro/dia do produto, os maiores primeiro (DESC).

Se forem encontrados resultados, a primeira ordem de serviço é atribuída a senha com [tipo de serviço](./tipos_servico.md) `SL`, e além do tipo de serviço a senha também é marcada como **SUPER LOTADA** que fará que a próxima solicitação desse mesmo funcionário também entre nesse critério.

Se não forem encontrados resultados, seguimos para o próximo critério.

---

## Critério 6 - Esvaziar ruas super lotadas

**Este critério é destinado apenas para operadores de empilhadeira**

Se não houverem ruas super lotadas esse critério é pulado.

O objetivo é priorizar as ruas com maior quantidade de ordens de serviço pendentes.

É bastante semelhante ao critério anterior, a principal diferença é dar prioridade para as ruas com maior quantidade de ordens de serviço pendentes, mesmo que isso signifique mudar o operador de rua ao contrário do critério 5 que buscava manter o operador na mesma rua.

As ordens de serviço pesquisadas nesse critério são:

- Apenas do tipo 98;
- Apenas onde o campo `PCMOVENDPEND.CODROTINA` seja 1709 ou 1721;
- Apenas aquelas em que a ordem de serviço do tipo 97 do mesmo código U.M.A. e núm. trans. wms esteja finalizada;
- Dentro do intervalo de ruas estabelecido;
- Apenas de ruas consideradas super lotadas;
- Que não possuam registro de pendência (`BOOSCOMPENDENCIA`) não resolvido.

A ordenação do resultado é:

- Ordens de serviço da mesma rua que da senha anterior finalizada pelo solicitante;
- Total de ordens de serviço pendentes na rua;
- Número da rua.

Se forem encontrados resultados, a primeira ordem de serviço é atribuída a senha com [tipo de serviço](./tipos_servico.md) `SL`, e além do tipo de serviço a senha também é marcada como **SUPER LOTADA** que fará que a próxima solicitação desse mesmo funcionário seja analisada pelo critério 5.

Se não forem encontrados resultados, seguimos para o próximo critério.

---

## <a name="criterio6_5"></a>Critério 6.5 - Separação Pallet Box

**Este critério é destinado apenas para operadores de paleteira**

Se a [configuração 264](./9023.md#configurações-da-rotina-9827) estiver diferente de `S`, esse critério é pulado.

O objetivo desse critério é realizar a separação chamada de pallet box, onde um pallet fechado é direcionado de um endereço do tipo pulmão diretamente para a doca (box).

As ordens de serviço pesquisadas nesse critério são:

- Apenas do tipo 17;
- Apenas onde o campo `PCMOVENDPEND.CODROTINA` **não** seja 1709 ou 1721;
- Apenas aquelas em que a ordem de serviço do tipo 23 do mesmo código U.M.A. e núm. trans. wms esteja com separação finalizada;
- Que não possuam registro de pendência (`BOOSCOMPENDENCIA`) não resolvido;
- Em que o percentual de finalização de outras ordens de serviço do mesmo carregamento, mas de outros tipos seja igual ou superior ao da [configuração 263](./9023.md#configurações-da-rotina-9827).

A ordenação do resultado é:

- Data da onda do carregando da ordem de serviço;
- Número da onda do carregando da ordem de serviço;
- Menor rua dentre o intervalo de ruas definido;
- Mais próxima da rua da senha anterior do operador;
- Número da ordem de serviço.

Se forem encontrados resultados, a primeira ordem de serviço é atribuída a senha com [tipo de serviço](./tipos_servico.md) `PB`.

Se não forem encontrados resultados, seguimos para o próximo critério.

---

## Critério 7 - Abastecimento corretivo mas com pendências de conferência ou separação

O objetivo é liberar o trabalho que está parado por conta de pendências encontradas durante a conferência ou separação.

As ordens de serviço pesquisadas nesse critério são:

- No caso de operador de empilhadeira, ordens de serviço do tipo 58;
- No caso de operador de paleteira, ordens de serviço do tipo 61;
- Dentro do intervalo de ruas estabelecido;
- Que não possuam registro de pendência (`BOOSCOMPENDENCIA`) não resolvido;
- Que possuam pendências de conferência em aberto (`BOPENDENCIACONF`) **ou** que possuem etiquetas de separação marcadas como pendentes e não resolvidas (`BOETIQUETAS.PENDENTE`).

A ordenação do resultado é:

- Data da onda do carregando da ordem de serviço;
- Número da onda do carregando da ordem de serviço;
- Número da ordem do carregando da ordem de serviço;

Se forem encontrados resultados, a primeira ordem de serviço é atribuída a senha com [tipo de serviço](./tipos_servico.md) `SL`, e além do tipo de serviço a senha também é marcada como **SUPER LOTADA** que fará que a próxima solicitação desse mesmo funcionário seja analisada pelo critério 5.

Mas o processo também pesquisa pela existência de outras ordens de serviço pendentes com a mesma origem e destino, caso sejam encontradas, o tipo de serviço será definida como `OC`, e essas outras ordens de serviço serão associadas nesta mesma senha (`BOFILAOSR`).

Se não forem encontrados resultados, seguimos para o próximo critério.

---

## Criério 8 - Ordens de serviço sem super lotação, priorizando onda e rua que da última senha

O objetivo é a realização de abastecimentos corretivos comuns, e se possível mantendo o operador na mesma rua que estava antes.

As ordens de serviço pesquisadas nesse critério são:

- No caso de operador de empilhadeira, ordens de serviço do tipo 58;
- No caso de operador de paleteira, ordens de serviço do tipo 61;
- Apenas onde o campo `PCMOVENDPEND.CODROTINA` **não** seja 1709 ou 1721;
- Que não possuam registro de pendência (`BOOSCOMPENDENCIA`) não resolvido;
- No caso de solicitação pelo operador de paleteira, é verificado se as ordens de serviço do tipo 58, da mesmo núm. trans. wms e código U.M.A estão em uma posição diferente de `P`.

A ordenação do resultado é:

- Data da onda do carregando da ordem de serviço;
- Número da onda do carregando da ordem de serviço;
- Menor rua dentre o intervalo de ruas definido;
- Mais próxima da rua da senha anterior do operador;
- Númeração da rua da ordem de serviço;
- Número da ordem de serviço.

Se forem encontrados resultados, a primeira ordem de serviço é atribuída a senha com [tipo de serviço](./tipos_servico.md) `MR` se o operador permanecer na mesma rua ou `TR` caso a ordem de serviço seja de outra rua em relação a última senha finalizada.

Mas o processo também pesquisa pela existência de outras ordens de serviço pendentes com a mesma origem e destino, caso sejam encontradas, o tipo de serviço será definida como `OC`, e essas outras ordens de serviço serão associadas nesta mesma senha (`BOFILAOSR`).

Se não forem encontrados resultados, seguimos para o próximo critério.

---

## Criério 8.1 (DESABILITADO) - Pallet box

**Este critério deixou de ser o 8.1 e passou a ser o [6.5](#criterio6_5)**

---

## Criério 8.2 (DESABILITADO) - Abastecimento preventivo com demanda

O objetivo é a realização de abastecimentos preventivos dando prioridade às ruas que possuem separação pendentes.

As ordens de serviço pesquisadas nesse critério são:

- No caso de operador de empilhadeira, ordens de serviço do tipo 58;
- No caso de operador de paleteira, ordens de serviço do tipo 61;
- No caso de solicitação pelo operador de paleteira, é verificado se as ordens de serviço do tipo 58, da mesmo núm. trans. wms e código U.M.A estão em uma posição diferente de `P`.
- Apenas onde o campo `PCMOVENDPEND.CODROTINA` seja 1723;
- Em endereços que possuem ordens de serviços dos tipos 10 e 22 com posição igual a `P`;
- Que não possuam registro de pendência (`BOOSCOMPENDENCIA`) não resolvido;
- Não tenha o valor do campo `PCMOVENDPEND.NUMTRANSWMS` como registro na tabela `PCWMS` (`PCWMS.NUMTRANSWMS`).

A ordenação do resultado é:

- Data da onda do carregando da ordem de serviço;
- Número da onda do carregando da ordem de serviço;
- Menor rua dentre o intervalo de ruas definido;
- Mais próxima da rua da senha anterior do operador;

Se forem encontrados resultados, a primeira ordem de serviço é atribuída a senha com [tipo de serviço](./tipos_servico.md) `AP`.

Mas o processo também pesquisa pela existência de outras ordens de serviço pendentes com a mesma origem e destino, caso sejam encontradas, o tipo de serviço será definida como `OC`, e essas outras ordens de serviço serão associadas nesta mesma senha (`BOFILAOSR`).

Se não forem encontrados resultados, seguimos para o próximo critério.

---

## Critério 9.5 - Ordens de serviço preventivas baseadas em pedidos

O objetivo é a realização de abastecimentos preventivos comuns, e se possível mantendo o operador na mesma rua que estava antes.

As ordens de serviço pesquisadas nesse critério são:

- No caso de operador de empilhadeira, ordens de serviço do tipo 58;
- No caso de operador de paleteira, ordens de serviço do tipo 61;
- Apenas onde o campo `PCMOVENDPEND.CODROTINA` seja 1752;
- Que não possuam registro de pendência (`BOOSCOMPENDENCIA`) não resolvido;
- No caso de solicitação pelo operador de paleteira, é verificado se as ordens de serviço do tipo 58, da mesmo núm. trans. wms e código U.M.A estão em uma posição diferente de `P`.
- Não tenha o valor do campo `PCMOVENDPEND.NUMTRANSWMS` como registro na tabela `PCWMS` (`PCWMS.NUMTRANSWMS`).

**Este critério tem uma diferença em relação aos demais**, pois ao contrário dos demais ele não ignora as ruas de exceção e as ruas com excesso de operadores.

A ordenação do resultado é:

- Mais próxima da rua da senha anterior do operador;
- Menor rua dentre o intervalo de ruas definido.

Se forem encontrados resultados, a primeira ordem de serviço é atribuída a senha com [tipo de serviço](./tipos_servico.md) `PP`.

Mas o processo também pesquisa pela existência de outras ordens de serviço pendentes com a mesma origem e destino, caso sejam encontradas, o tipo de serviço será definida como `OC`, e essas outras ordens de serviço serão associadas nesta mesma senha (`BOFILAOSR`).

Se não forem encontrados resultados, seguimos para o próximo critério.

---

## Critério 10 - Armazenamento comum

**Este critério é destinado apenas para operadores de empilhadeira**

O objetivo é a realização de ordens de serviço de armazenamento comuns.

As ordens de serviço pesquisadas nesse critério são:

- Apenas do tipo 98;
- Apenas onde o campo `PCMOVENDPEND.CODROTINA` **não** seja 1709 ou 1721;
- Apenas aquelas em que a ordem de serviço do tipo 97 do mesmo código U.M.A. e núm. trans. wms esteja finalizada;
- Que não possuam registro de pendência (`BOOSCOMPENDENCIA`) não resolvido;

A ordenação do resultado é:

- Data da ordem de serviço (ASC);
- Menor estoque gerencial disponível (ASC)
- O giro/dia do produto, os maiores primeiro (DESC).

Se forem encontrados resultados, a primeira ordem de serviço é atribuída a senha com [tipo de serviço](./tipos_servico.md) `AC`.

Se não forem encontrados resultados, seguimos para o próximo critério.

---

## Critério 11 - Abastecimento preventivo comum

O objetivo é a realização de abastecimentos preventivos comuns, mas observando o giro/dia dos produtos.

As ordens de serviço pesquisadas nesse critério são:

- No caso de operador de empilhadeira, ordens de serviço do tipo 58;
- No caso de operador de paleteira, ordens de serviço do tipo 61;
- Apenas onde o campo `PCMOVENDPEND.CODROTINA` **não** seja 1709 ou 1721;
- Que não possuam registro de pendência (`BOOSCOMPENDENCIA`) não resolvido;
- No caso de solicitação pelo operador de paleteira, é verificado se as ordens de serviço do tipo 58, da mesmo núm. trans. wms e código U.M.A estão em uma posição diferente de `P`.

A ordenação do resultado é:

- Menor rua dentre o intervalo de ruas definido (ASC);
- O giro/dia do produto, os maiores primeiro (DESC).

Se forem encontrados resultados, a primeira ordem de serviço é atribuída a senha com [tipo de serviço](./tipos_servico.md) `PV`.

Mas o processo também pesquisa pela existência de outras ordens de serviço pendentes com a mesma origem e destino, caso sejam encontradas, o tipo de serviço será definida como `OC`, e essas outras ordens de serviço serão associadas nesta mesma senha (`BOFILAOSR`).

---
