# Gerenciando Datasets - CTE Simples

Navegação:
[Transact-SQL do ZERO ao MASTER](/README.md) | [T-SQL Avançado](/T-SQL_Avancado/README.md)

```sql

------------------------------------------------------------
-- CTE - Common Table Expression: Intro & CTE Simples
------------------------------------------------------------
/*
	Características

		Assim como subquerys e tabelas derivadas, facilita:

			Reutilização de código
			Leitura e depuração de código

	Tipos de CTE:

		Simples
			Funciona como uma tabela derivada (select from select / Join).
			Aplicável ao merge

			Lembrar prova: Exclusão dados duplicados

		Recursiva
			Substitui cursores

			Lembrar prova: Gera hierarquias
*/

-- Banco de testes

USE MASTER
IF DB_ID('Curso') is not null
	DROP DATABASE CURSO
CREATE DATABASE CURSO
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
INSERT INTO colaboradores
VALUES
	(00,NULL,'Marcelino Pinga','CEO',40000,0),
	(01,00,'Email Suarez','Gerente de Vendas',25000,0),
	(02,01,'Feliciano Triste','Assistente de Vendas',7000,20),
	(03,01,'Erondina Damiana','Assistente de Vendas',8000,20),
	(04,01,'Aventureiro das Neves','Assistente de Vendas',9000,20),
	(05,00,'Vespertino Cedo','Gerente Administrativo',15000,0),
	(06,05,'Aviador da Silva','Caixa',7000,2),
	(07,05,'Navegador da Luz','Assistente Administrativo',7000,2),
	(08,00,'José Cracha','Aspone Senior',15000,10),
	(09,00,'Salvador das Dores Fortes','Gerente de IT',30000,0),
	(10,09,'Vitor Festeiro','DBA',30000,0),
	(11,09,'Saturnino Besta','Coordenador de IT',25000,0),
	(12,11,'Rolando da Rocha','Analista de IT',20000,0),
	(13,11,'Rodrigo Rego Penteado','Analista de IT',20000,0)

CREATE TABLE clientes (
	id SMALLINT NOT NULL IDENTITY(1,1) CONSTRAINT pk_clientes PRIMARY KEY,
	nm VARCHAR(100) NOT NULL,
	ic_sexo CHAR(1) NOT NULL CONSTRAINT ck_colaboradores_ic_sexo CHECK (ic_sexo IN ('M', 'F'))
)
INSERT INTO clientes (nm, ic_sexo)
VALUES
	('Ari Tuba','M'),
	('Dolores Fuertes','F'),
	('José João Maria Antunes','M'),
	('José Maria Manguaça','M'),
	('Jacinto Pinto da Luz','M'),
	('Mestrualina Periódica Oliveira','F'),
	('Orlando Modesto Pinto','M'),
	('Oscarito Majela Godinho','M'),
	('Paula Tejando','F'),
	('Souza da Silva','M'),
	('Vitor Festeiro','M') -- também é colaborador

CREATE SEQUENCE sProdutos AS SMALLINT START WITH 1
CREATE TABLE produtos (
	id SMALLINT CONSTRAINT df_produtos_id DEFAULT (NEXT VALUE FOR sProdutos),
	nm VARCHAR(100),
	vl DECIMAL(10,2),
	ic_ativo BIT NOT NULL CONSTRAINT df_produtos_ic_ativo DEFAULT (1),
	CONSTRAINT pk_produtos PRIMARY KEY (id)
)
INSERT produtos (nm, vl)
VALUES
	('Mouse Gammer 25 botões',200),
	('Teclado Gammer 350 teclas',300),
	('Monitor 32 Pol Full HD',1000),
	('RAM DDR4 4GB Powerturbo',500),
	('CPU Nasa 10Ghz',2000),
	('HD SSD 1TB',2000)

CREATE TABLE vendas (
	id INT NOT NULL IDENTITY(1, 1),
	dt DATE NOT NULL,
	id_produto SMALLINT NOT NULL CONSTRAINT fk_vendas_produtos FOREIGN KEY REFERENCES produtos(id),
	id_vendedor SMALLINT CONSTRAINT fk_vendas_colaboradores FOREIGN KEY REFERENCES colaboradores(id),
	id_cliente SMALLINT CONSTRAINT fk_vendas_clientes FOREIGN KEY REFERENCES clientes(id),
	qt INT CONSTRAINT ck_vendas_qt CHECK (qt > 0),
	vl_unitario DECIMAL (10,2),
	vl_venda AS qt * vl_unitario,
	ds_MesVenda AS CHOOSE(MONTH(dt), 'Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'),
	CONSTRAINT pk_vendas PRIMARY KEY (id)
)
INSERT INTO dbo.vendas (dt, id_produto, id_vendedor, id_cliente, qt, vl_unitario)
VALUES
	('2010-01-02',1,4,3,9,200.00),
	('2010-01-04',4,6,8,7,500.00),
	('2010-01-06',1,7,8,6,200.00),
	('2010-01-10',2,1,5,1,300.00),
	('2010-02-03',4,3,4,6,500.00),
	('2010-02-04',2,8,2,4,300.00),
	('2010-02-07',2,8,8,7,300.00),
	('2010-03-07',2,8,8,5,300.00),
	('2010-03-08',2,3,4,8,300.00),
	('2010-03-08',3,1,3,9,1000.00),
	('2010-04-09',3,1,3,1,1000.00),
	('2010-04-10',2,8,5,1,300.00),
	('2010-05-10',2,8,5,1,300.00),
	('2010-05-11',2,8,8,5,300.00),
	('2010-06-13',4,3,4,6,500.00),
	('2010-07-02',1,4,3,9,200.00),
	('2010-08-04',4,6,8,7,500.00),
	('2010-09-06',1,7,8,6,200.00),
	('2010-10-10',2,1,5,1,300.00),
	('2010-11-03',4,3,4,6,500.00),
	('2010-12-04',2,8,2,4,300.00),
	('2011-02-07',2,8,8,7,300.00),
	('2011-03-07',2,8,8,5,300.00),
	('2011-03-08',2,3,4,8,300.00),
	('2011-03-08',3,1,3,9,1000.00),
	('2011-04-09',3,1,3,1,1000.00),
	('2011-04-10',2,8,5,1,300.00),
	('2011-05-10',2,8,5,1,300.00),
	('2011-05-11',2,8,8,5,300.00),
	('2011-06-13',4,3,4,6,500.00)

------------------------------------------------------------
-- CTE Simples
------------------------------------------------------------

-- vendas por vendedor (CTE no lugar de tabela derivada)

;WITH cte_vendas_por_vendedor AS (
	SELECT
		id_vendedor AS id,
		SUM(vl_venda) AS vl_vendas,
		COUNT(1) AS nr_vendas
	FROM vendas
	GROUP BY id_vendedor
)
SELECT
	c.*,
	v.*
FROM colaboradores c
LEFT JOIN cte_vendas_por_vendedor v ON v.id = c.id

/* -- Exemplo utilizando tabela derivada
LEFT JOIN (
	SELECT
		id_vendedor AS id,
		sum(vl_venda) AS vl_vendas,
		count(1) AS nr_vendas
	FROM vendas
	GROUP BY id_vendedor
) v ON v.id = c.id
*/



-- Volume de compras vs. Volume de vendas

;WITH
	-- CTE com colunas declaradas
	cte_vendas_colaboradores (id, nome, vl_venda) AS
	(
		SELECT
			c.id,
			nm,
			SUM(vl_venda)
		FROM colaboradores c
		INNER JOIN vendas v ON v.id_vendedor = c.id
		GROUP BY c.id, nm
	),

	-- CTE sem colunas declaradas
	cte_compras_cliente AS
	(
		SELECT
			SUM(vl_venda) vl_venda,
			id_cliente
		FROM vendas
		GROUP BY id_cliente
	)
SELECT
	v.dt,
	c.nm AS cliente,
	v.vl_venda,
	cte_cc.vl_venda AS compras_Totais_cliente,
	-- Percentual de cada compra em relação ao total de compras do cliente
	v.vl_Venda / cte_cc.vl_venda AS pc_venda_cliente,
	cte_vc.nome AS Vendedor,
	cte_vc.vl_venda AS Vendas_totais_vendedor,
	-- Percentual de cada venda listada em relação to total de vendas do vendedor
	v.vl_Venda / cte_vc.vl_venda pc_venda_vendedor
FROM vendas v
LEFT JOIN clientes c ON c.id = v.id_cliente
LEFT JOIN cte_compras_cliente cte_cc ON cte_cc.id_cliente = v.id_cliente
LEFT JOIN cte_vendas_colaboradores cte_vc ON cte_vc.id = v.id_vendedor



-- CTE Simples: Exemplo para excluir dados duplicados

-- Vendas atuais
SELECT * FROM vendas

-- Duplicar as vendas
INSERT INTO vendas (dt, id_produto, id_vendedor, id_cliente, qt, vl_unitario)
	SELECT dt, id_produto, id_vendedor, id_cliente, qt, vl_unitario FROM vendas

-- Adicionar venda "não duplicada"
INSERT INTO vendas (dt, id_produto, id_vendedor, id_cliente, qt, vl_unitario)
VALUES
	('2017-01-01', 2, 3, 1, 10, 300)

-- Remover os registros duplicados:
;WITH cte_vendas_duplicadas AS
(
	-- Dica: Vendas com dados iguais e ID maior
	SELECT
		MAX(id) id
		--,dt, id_produto, id_vendedor, id_cliente, qt, vl_unitario
	FROM vendas
	GROUP BY dt, id_produto, id_vendedor, id_cliente, qt, vl_unitario
	-- having abaixo lista somente vendas duplicadas
	HAVING COUNT(1) > 1
)
--DELETE v
SELECT *
FROM vendas v
INNER JOIN cte_vendas_duplicadas cte ON cte.id = v.id

-- Vendas originais
SELECT * FROM vendas


```
