## <a name="bofilaos"></a>BOFILAOS

Tabela que armazena as senhas (solicitações) dos operadores de paleteiras e empilhadeiras

```sql

CREATE TABLE BOFILAOS
   (
	SENHA NUMBER(30,0), -- Definido pela sequence SEQ_BOFILAOS
	MATRICULA NUMBER(8,0) NOT NULL, -- Matrícula do solicitante no Winthor
	DTSOLICITACAO DATE NOT NULL,  -- Data e hora da solicitação
	RUARANGEINICIO NUMBER(3,0) NOT NULL, -- Rua inicial para filtragem das ordens de serviços
	RUARANGEFIM NUMBER(3,0) NOT NULL, -- Rua final para filtragem das ordens de serviços
	STATUS CHAR(1) NOT NULL, -- * Descrito mais abaixo
	NUMOS NUMBER(10,0), -- Número da ordem de serviço encontrada
	CODIGOUMA NUMBER(14,0), -- Código UMA associado à ordem de serviço
	TIPOOS NUMBER(4,0), -- Tipo da ordem de serviço
	DTATENDIMENTO DATE, -- Data e hora que a solicitação foi atentida
	CODENDERECOORIG NUMBER(10,0), -- Código de endereço de origem na ordem de serviço
	CODENDERECO NUMBER(10,0), -- Código de endereço destino na ordem de serviço
	FLAGSL NUMBER, -- Indica se a rua da ordem de serviço estava "SUPER LOTADA" no momento da atribuição
	DTONDA DATE, -- Data da onda associada à ordem de serviço
	NRONDA NUMBER, -- Número da onda associada à ordem de serviço
	TIPOSERVICO CHAR(2), -- ** Tipos de serviço
	DTRESERVA DATE, -- Preenchida com o mesmo valor do campo DTATENDIMENTO
	DTATRIBUIDA DATE, -- Data e hora em que o solicitante iniciou a ordem de serviço
	DTFINALIZADO DATE, -- Data e hora em que o solicitante finalizou a ordem de serviço
	TIPOOPERADOR VARCHAR2(1), -- P (Paleteiro) / E (Empilhador)
	CRITERIO NUMBER(4,1), -- Número do critério utilizado para a localização da ordem ser serviço
	ARMAZEMTODO VARCHAR2(1) -- (S/N) Indica se a pesquisa pela ordem de serviço foi o não para o armazém todo
   );



   CREATE SEQUENCE SEQ_BOFILAOS; -- Sequence que define o campo BOFILAOS.SENHA

```

\* Os status possíveis podem ser encontrados [aqui](./status_senha.md)

\*\* Os tipos de serviços podem ser encontrados [aqui](./tipos_servico.md)

---

## <a name="bofilaosr"></a>BOFILAOSR

Tabela que contém as demais ordens de serviço associadas a ordem de serviço atribuída ao usuário por ter os mesmos endereços de origem e destino

```sql
   CREATE TABLE BOFILAOSR
	(
	SENHA NUMBER(30,0) NOT NULL, -- A mesma senha da BOFILAOS.SENHA
	NUMOS NUMBER(10,0) NOT NULL -- Número da ordem de serviço associada
	);
```
