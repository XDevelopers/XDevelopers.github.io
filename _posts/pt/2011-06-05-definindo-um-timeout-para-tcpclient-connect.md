---
layout: post
title: Definindo um Timeout para TcpClient.Connect
date: 2011-06-05 11:39
author: Sergio Garcia
comments: true
categories: [C#, C#, Desenvolvimento, network, performance, socket, TcpClient]
---
Algum tempo atrás, fiz uma aplicação cliente servidor que quando não
encontrava o cliente no IP previamente configurado, ela efetuava uma busca
pela rede para encontrar o servidor.

Porém, eu só podia efetuar a busca pela rede, caso eu não conseguisse conectar
usando o metodo Connect da classe TcpClient, o que demorava cerca de 30
segundos gerar uma exception informando que não conseguiu conexão com o
servidor.

Usando está dica, você poderá definir um tempo menor para o timeout, pois
aplicação que usam muitas conexões não podem ficar esperando muito tempo a
resposta de servidores que não existem.

O tempo de conexão, usando o método Connect, compreende ao tempo de buscar o
nome do host, mais ao tempo de resposta deste servidor.

Os tempos de resposta normalmente são muito baixos, cerca de 10ms para um
servidor na rede local, e muito raramente passa dos 500ms para um servidor do
outro lado do mundo.

Sabendo desta informação, podemos utilizar um timeout menor, caso isso seja
necessário.

Para isso, utilizaremos o construtor padrão da classe TcpClient.

```csharp
var client = new TcpClient();
```

E, ao invés de usarmos o método bloqueante, Connect, usaremos sua alternativa
não bloqueante, o BeginConnect. O BeginConnect recebe uma função de callback
que é executada quando o sistema obtem o resultado da conexão, seja ele
sucesso ou não.

Quando chamamos o Close, antes do sistema terminar a conexão, uma
*NullReferenceException* é gerada, por isso nosso método deve trata-lá, apesar
de não precisar fazer nada com ela.

```csharp
var asyncResult = client.BeginConnect(host, port,
    ar =>
    {
        try
        {
            client.EndConnect(ar);
        }
        catch (NullReferenceException)
        {
            // Está exception é esperada, quando
            // usamos Close, então não faça nada
        }
    }, client);
```

Como este método não é bloqueante, precisamos bloquear a execuxão pelo tempo
que será a nossa definição de timeout, para isso, utilizaremos o
*ManualResetHandler* que está contido dentro de **asyncResult** (que é uma
interface chamada *IAsyncResult*, usada para todos os métodos não bloqueantes
do .net).

Eu estou utilizando um timeout de 1000ms, ou seja, um segundo. Caso você
utilize o nome do computador ou dominio, este valor deve ser maior, pois irá
compreender a busca de DNS (que pode demorar) mas o tempo de resposta do
destino.

Uma busca de DNS, normalmente leva até um segundo, mas há casos em que pode
demorar mais, pois é feita a consulta de vários servidores de DNS até que o
destino responda com o endereço da máquina, o que não é incomum levar uns 5
segundos.

```csharp
asyncResult.AsyncWaitHandle.WaitOne(1000);
```

Caso a conexão seja efetuada, o próprio .net setará o Handler, continuando a
execução do sistema.

O próximo passo, é checar se estamos conectados, caso não conseguirmos
conexão, devemos fechar o TcpClient, para liberar os recursos utilizados.

```csharp
if (!client.Connected)
{
    client.Close();
    return;
}
```

Notem que somente estou dando um return, a maneira como você deve tratar o
erro na conexão irá variar.

Está dica é válida para a classe Socket também, com pouquíssimas alterações.

Abaixo segue o exemplo completo.

```csharp
const string host = "127.0.0.1";
const int port = 8099;

var client = new TcpClient();

//
// Tenta iniciar a conexão
//
var asyncResult = client.BeginConnect(host, port,
    ar =>
    {
        try
        {
            client.EndConnect(ar);
        }
        catch (NullReferenceException)
        {
            // Está exception é esperada, quando
            // usamos Close, então não faça nada
        }
    }, client);

//
// Procure sempre usar o IP do destino,
// se você usar nomes, o tempo de timeout
// deve contemplar a busca pelo DNS
//
asyncResult.AsyncWaitHandle.WaitOne(1000);

//
// Aqui checamos se estamos conectados, se
// não estivermos conectados, fechamos a
// conexão e tratamos o erro
//
if (!client.Connected)
{
    client.Close();
    return;
}
```
