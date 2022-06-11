# Gerenciando Datasets - CTE Recursiva

Navegação:
[Transact-SQL do ZERO ao MASTER](/README.md) | [T-SQL Avançado](/T-SQL_Avancado/README.md)

```sql

------------------------------------------------------------
-- CTE - Common Table Expression: CTE Recursiva
------------------------------------------------------------

/*
Tipos de CTE:
	Simples
	Recursiva
		Realiza loops de código (recursividade)
		Exemplos: Gerar dados de teste | Gerar hierarquia

*/

-- Banco de testes

USE MASTER

IF DB_ID('Curso') IS NOT NULL
	DROP DATABASE Curso

CREATE DATABASE Curso
GO

USE Curso

CREATE SEQUENCE sColaboradores AS SMALLINT START WITH 0

CREATE TABLE colaboradores (
	id SMALLINT NOT NULL CONSTRAINT df_colaboradores_id DEFAULT (NEXT VALUE FOR scolaboradores),
	id_gerente SMALLINT NULL CONSTRAINT fk_colaboradores_gerentes FOREIGN KEY REFERENCES colaboradores(id),
	nm VARCHAR(100) NOT NULL,
	ds_cargo VARCHAR(100) NOT NULL,
	vl_salario DECIMAL(8,2) NOT NULL,
	pc_comissao DECIMAL(4, 2) NOT NULL CONSTRAINT df_colaboradores_comissao DEFAULT (0),
	CONSTRAINT pk_colaboradores PRIMARY KEY (id),
)
INSERT INTO colaboradores VALUES
	(00, NULL, 'Marcelino Pinga', 'CEO', 40000, 0),
	(01, 00, 'Email Suarez', 'Gerente de Vendas', 25000, 0),
	(02, 01, 'Feliciano Triste', 'Assistente de Vendas', 7000, 20),
	(03, 01, 'Erondina Damiana', 'Assistente de Vendas', 8000, 20),
	(04, 01, 'Aventureiro das Neves', 'Assistente de Vendas', 9000, 20),
	(05, 00, 'Vespertino Cedo', 'Gerente Administrativo', 15000, 0),
	(06, 05, 'Aviador da Silva', 'Caixa', 7000, 2),
	(07, 05, 'Navegador da Luz', 'Assistente Administrativo', 7000, 2),
	(08, 00, 'José Cracha', 'Aspone Senior', 15000, 10),
	(09, 00, 'Salvador das Dores Fortes', 'Gerente de IT', 30000, 0),
	(10, 09, 'Vitor Festeiro', 'DBA', 30000, 0),
	(11, 09, 'Saturnino Besta', 'Coordenador de IT', 25000, 0),
	(12, 11, 'Rolando da Rocha', 'Analista de IT', 20000, 0),
	(13, 11, 'Rodrigo Rego Penteado', 'Analista de IT', 20000, 0)



------------------------------------------------------------
-- CTE Recursiva
------------------------------------------------------------

-- Sintaxe CTE recursiva
-- Exemplo: Gerar um contador de 1 a 200

;WITH cte_seq AS
(
	-- query ancora (linha inicial)
	SELECT 1 AS nr_cont
	-- union all: Une a query ancora à query recursiva
	UNION ALL
	-- query recursiva (demais linhas)
	SELECT nr_cont + 1
	FROM cte_seq
	-- condição de saída do loop
	WHERE nr_cont < 200
)
SELECT *
FROM cte_seq
OPTION (maxrecursion 0) -- sem "maxrecursion" a execução termina após 100 loops

-- maxrecursion:
--		0 - infinito
--		n - quantidade de recursões

-- Msg 530, Level 16, State 1, Line 63
-- The statement terminated. The maximum recursion 100 has been exhausted before statement completion



-- Gerar "dados de teste" (exemplo 2)

;WITH dados_sequenciais AS
(
	-- query ancora (linha inicial)
	SELECT
		0 AS numeros_positivos,
		0 AS numeros_negativos,
		CONVERT(DATETIME, '2000-01-01') AS datas_futuras,
		CONVERT(DATETIME, '2000-01-01') AS datas_passadas
	UNION ALL
	-- query recursiva (demais linhas)
	SELECT
		dados_sequenciais.numeros_positivos + 1,
		dados_sequenciais.numeros_negativos - 1,
		dados_sequenciais.datas_futuras + 1,
		dados_sequenciais.datas_passadas - 1
	FROM dados_sequenciais
	-- condição de saída
	WHERE dados_sequenciais.numeros_positivos + 1 <= 365
)
SELECT *
FROM dados_sequenciais
OPTION (maxrecursion 1000)



-- Gerar "dados aleatórios" (mundo real)



-- CTE Recursiva: Gerar relatório de hierarquia (exemplo prova!)
SELECT * FROM colaboradores

-- Relatório da hierarquia
;WITH cte_hierarquia AS (
	-- Query ancora: Primeiro nível - CEO
	SELECT
		id,
		CAST(nm AS VARCHAR(1000)) AS nm_colaborador,
		ds_cargo,
		1 AS nr_nivel_hierarquico,
		CAST(nm AS VARCHAR(1000)) ds_nivel_hierarquico
	FROM colaboradores WHERE id_gerente IS NULL

	UNION ALL

	-- query recursiva: todos que não são CEO
	SELECT
		c.id,
		CAST(REPLICATE ('|    ' , g.nr_nivel_hierarquico) + c.nm AS VARCHAR(1000)),
		c.ds_cargo,
		g.nr_nivel_hierarquico + 1,
		CAST(ds_nivel_hierarquico  + ' | ' + c.nm AS VARCHAR(1000))
	FROM colaboradores c
	INNER JOIN cte_hierarquia g ON g.id = c.id_gerente -- condição de saída
)
SELECT * FROM cte_hierarquia ORDER BY ds_nivel_hierarquico

-- empresa cresceu e colocaram mais gente
INSERT INTO colaboradores
VALUES
	(14, null, 'Nair Queijo Branco', 'Chairwoman', 30000, 0),
	(15, 14, 'Free William Madeira', 'INED', 20000, 0),
	(16, 14, 'Hypotenusa Pereira', 'INED', 20000, 0)
UPDATE colaboradores SET id_gerente = 14 WHERE id = 0

-- INED = Independent non-executive director :-)

```
