---
layout: post
title: Resetando a senha do usuário sa no SQL Server
date: 2011-10-02 04:51
author: Sergio Garcia
comments: true
categories: [Desenvolvimento, recuperar senha, sa, senha, SQL, sql, sqlserver]
---

Pode acontecer com todo mundo de esquecer uma senha, neste pequeno tutorialvou explicar como alterar a senha de qualquer usuário do SQL Server sem ter a senha do administrador.

O SQL Server permite que seja executado no modo Single User, que permite acesso irrestrito a todas as tabelas e opções do banco de dados. Portanto, essa dica deve ser usada com cuidado.

Você precisará ter acesso de administrador local no computador em que o SQL Server estiver rodando.

Neste exemplo, foi usado o SQL Server 2008 R2 Express com uma instância nomeada chamada SQLExpress (o padrão). Caso sua instância tenha outro nome, mude onde aparece SQLExpress para está instância. Este deve funcionar em todas as versões do SQL Server.

Abra o prompt de comando e navegue até a pasta onde está instalado o SQL Server, abaixo exemplo para o SQL 2008 R2 Express:

```
cd \Program Files\Microsoft SQL Server\MSSQL10_50.SQLEXPRESS\MSSQL\Binn
```

Caso está não seja a sua pasta, procure pelo arquivo sqlservr.exe na pasta "Program Files" ou "Arquivos de Programas". Execute o comando abaixo para ter acesso as opções de inicialização (apenas por curiosidade).

```
sqlservr /?
usage: sqlservr
[-c] (not as a service)
[-d file] (alternative master data file)
[-l file] (alternative master log file)
[-e file] (alternate errorlog file)
[-f] (minimal configuration mode)
[-m] (single user admin mode)
[-g number] (stack MB to reserve)
[-k ] (checkpoint speed in MB/sec)
[-n] (do not use event logging)
[-s name] (alternate registry key name)
[-T ] (trace flag turned on at startup)
[-x] (no statistics tracking)
[-y number] (stack dump on this error)
[-B] (breakpoint on error (used with -y))
[-K] (force regeneration of service master key (if exists))

See documentation for details.
```

Abra o gerenciador de serviços e pare o serviço do SQL Server. Se você usar o SQL Express poderá para-lo com o seguinte comando.

```
net stop MSSQL$SQLEXPRESS
```

Lembre-se que você deve ser administrador, senão o comando acima irá retornar um erro. Em seguinda, execute o SQL Server usando a opção -m para que ele seja executado como single user.

```
sqlservr -m -s SQLExpress
```

Agora seu banco está rodando no modo Single User, você pode conectar nele usando o SQL Managment Studio com seu usuário do Windows e você terá acesso total ao sistema, podendo alterar qualquer configuração. O comando SQL abaixo altera a senha do usuário **sa** para ***123**.

```sql
ALTER LOGIN sa WITH PASSWORD = '123'
```

Se você quiser usar o modo rápido, o seguindo comando pode ser usado no prompt.

```
sqlcmd -S (local)\SQLExpress -E -Q "ALTER LOGIN sa WITH PASSWORD = '123'"
```

Após finalizado as suas alterações, pare o banco que está rodando como Single User (use Ctrl+C) e reinicie o serviço do SQL Server. Você pode user o comando abaixo para isso.

```
net start MSSQL$SQLEXPRESS
```
