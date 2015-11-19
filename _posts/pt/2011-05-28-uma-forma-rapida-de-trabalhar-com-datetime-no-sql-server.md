---
layout: post
title: Uma forma rápida de trabalhar com datetime no SQL Server
date: 2011-05-28 21:53
author: Sergio Garcia
comments: true
categories: [datetime, Desenvolvimento, performance, SQL, sql, sqlserver]
---
Existem diversas formas de armazenar datas, em diversos sistemas diferentes.
Se nós conhecermos a forma como estes dados são armazenados, podemos utilizar
formas mais eficientes de fazer as coisas.

Alguns sistemas armazenam a diferença de tempo entre a data a ser representada
e uma data comum previamente estabelecida chamada *epoch*, como o Unix Time,
que representa as datas como número de segundos desde 01/01/1970.

De posse deste conhecimento sobre o Unix Time, é facil saber que para
incrementar um dia na data, é mais rápido incrementar o número que representa
aquela data por 86.400 (24 * 60 * 60) do que utilizar as funções de data para
isso.

As funções de data, não levam apenas em consideração a diferença númerica em
sua representação, mas sim outras formalidades como anos bissextos,
[segundos bissextos](http://pt.wikipedia.org/wiki/Segundo_bissexto) e o
horário de verão, assim a idéia apresentada acima nem sempre vai estar
correta, apesar de ser util e correta na maioria dos casos.

No SQL Server, as datas  são representadas através da diferença em dias desde
seu *epoch*, que é 01/01/1900. Execute o exemplo abaixo no seu SQL Server.

```sql
SELECT CAST(0 AS DATETIME)
SELECT CAST(1 AS DATETIME)
```

Se você for um bom conhecedor de SQL Server, deve estar se perguntando sobre a
data de 01/01/1753. Se ela é ou não o *epoch* do SQL Server. Bem, essa não é o
epoch do SQL Server como já disse, o que ainda não disse é que o SQL Server
aceita representações negativas para uma data. Assim tente o exemplo abaixo.

```sql
SELECT CAST(-53690 AS DATETIME)
```

Achou a data que procurava?

Você ainda deve estar se pergundo sobre onde deve estar o tempo, já que o SQL
Server guarda a diferença em dias. Bom, eu ainda não tinha falado a diferença
em dias não é gravada em um número inteiro, mas sim em um número de ponto
flutuante, mas espeficicamente um FLOAT.

Assim, temos o valor de 0.5 para 12:00, 0.25 para 06:00, 0.125 para 03:00 e
1/24 para 01:00.

```sql
SELECT CAST(0.5 AS DATETIME)
SELECT CAST(0.25 AS DATETIME)
SELECT CAST(0.125 AS DATETIME)
SELECT CAST(0.041666667 AS DATETIME)
```

Notém que 1/24 é uma dizima periodica, e se você tivesse colocado no seu select
isso, teria obtido 00:59:59.940 e não 01:00. Isso ocorre porque a precisão do
tipo float e consequentemente do datetime é limitada. Assim, a precisão do
tipo datetime é arredondada para 0, 3 ou 7 milisegundos, o que ainda é muito
melhor que apenas segundos.

```sql
SELECT CAST(1.0/24 AS DATETIME)
SELECT CAST(0.0000001 AS DATETIME)
SELECT CAST(0.00000005 AS DATETIME)
```

Depois de toda essa teoria, vamos um exemplo bem prática da utilização disso.
Temos uma tabela em nosso banco e precisamos agrupar os registros por dia,
podem ser pedidos por dia, acessos por dia, etc. O importante é o por dia.

Para isso, vamos criar uma tabela com muitos registros. E quando digo muitos,
eu digo muitos mesmo, para teste. Se você quiser, diminua a quantidade de
items a serem gerados, mudando o valor da váriavel @genRowCount, porque a
geração destes itens leva mais de meia hora.

Neste exemplo, vamos apenas colocar o campo de data e não vamos colocar
indexes, pois iremos trabalhar com todas as linhas, tornando um index aqui
desnecessário.

```sql
CREATE TABLE [DateSample]
(
    [DateColumn]    DATETIME
);

DECLARE @dateValue DATETIME
DECLARE @counter INT
DECLARE @genRowCount INT

SET @counter = 0;
-- Change here the quantity of rows to be generated
SET @genRowCount = 10000000
-- The greatest day in the history of humankind
SET @dateValue = '1984-09-13';

SET NOCOUNT ON;

WHILE @counter < 10000000
BEGIN
    SET @dateValue = DATEADD(SS, CAST(RAND() * 5 as INT), @dateValue)
    SET @counter = @counter + 1
    INSERT INTO [DateSample] VALUES (@dateValue);
    -- Only to display progress
    IF (@counter % 10000 = 0) PRINT @counter
END
```

De forma tradicional, executariamos uma consulta convertendo a data para um
formato de texto em que a data não utilize o tempo, convertendo novamente para
data em sequida. Aqui teriamos todo o overhead que falamos no começo.

```sql
SELECT
    CONVERT(DATETIME,CONVERT(VARCHAR, [DateColumn], 111),111) AS DateColumnWithoutTime,
    COUNT(1)
FROM
    [DateSample]
GROUP BY
    CONVERT(DATETIME,CONVERT(VARCHAR, [DateColumn], 111),111)
ORDER BY
    DateColumnWithoutTime
```

Como não estamos fazendo operações de soma e subtração na data, não estamos
alterando seu valor, apenas removendo a parte do tempo. Neste caso, é
perfeitamente seguro tratar as datas como float e remover a parte de ponto
flutuante usando floor.

```sql
SELECT
    CAST(FLOOR(CAST([DateColumn] AS FLOAT)) AS DATETIME) AS DateColumnWithoutTime,
    COUNT(1)
FROM
    [DateSample]
GROUP BY
    CAST(FLOOR(CAST([DateColumn] AS FLOAT)) AS DATETIME)
ORDER BY
    DateColumnWithoutTime
```

A segunda versão desta consulta consumiu aproximadamente 30% do tempo da
primeira versão, ambas retornando os mesmos dados.
