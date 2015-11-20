---
layout: post
title: Uma classe para ajudar a trabalhar com diversos idiomas
date: 2011-07-14 21:39
author: Sergio Garcia
comments: true
categories: [C#, C#, convert, culture, Desenvolvimento, format]
---

Muitas vezes quando estamos desenvolvendo uma aplicação, precisamos que apenas
um pequeno ponto da aplicação trabalhe em um idioma especifico, que muitas
vezes pode ser diferente do idioma atual do sistema. Um exemplo disso, é
consumirmos serviços REST ou JSON em que os números e datas estão em um
formato especifico, normalmente no idioma inglês, e nossa aplicação
normalmente está num sistema cujo idioma é português.

No exemplo abaixo, podemos ver bem o problema de qual estou falando. Em um
sistema em pt-BR teremos *3135* enquanto em um sistema en-US teremos *31.35*.

```csharp
Console.WriteLine(decimal.Parse("31.35"));
```

Quando consumimos serviços que retornam texto, normalmente o idioma em questão
é conhecido, assim podemos forçar o idioma do sistema definindo os valores do
idioma da *Thread* atual, como abaixo:

```csharp
var culture = new CultureInfo("en-US");

Thread.CurrentThread.CurrentCulture = culture;
Thread.CurrentThread.CurrentUICulture = culture;

Console.WriteLine(decimal.Parse("31.35"));
```

A desvantagem é que estamos mudando todo o idioma da aplicação, inclusive nos
textos que são mostrados para o usuário. Uma alternativa, é utilizarmos a
classe abaixo, que encapsula um pequeno bloco de código em um idioma
especifico, retornando ao idioma atual após o final do bloco.

```csharp
public class GinxCultureHelper : IDisposable
{
    private readonly CultureInfo _currentUiCulture;
    private readonly CultureInfo _currentCulture;

    public GinxCultureHelper(string cultureName)
    {
        _currentCulture = Thread.CurrentThread.CurrentCulture;
        _currentUiCulture = Thread.CurrentThread.CurrentUICulture;

        var culture = new CultureInfo(cultureName);

        Thread.CurrentThread.CurrentCulture = culture;
        Thread.CurrentThread.CurrentUICulture = culture;
    }

    public void Dispose()
    {
        Thread.CurrentThread.CurrentCulture = _currentCulture;
        Thread.CurrentThread.CurrentUICulture = _currentUiCulture;
    }
}
```

Abaixo segue um exemplo de utilização da classe.

```csharp
class Program
{
    static void Main(string[] args)
    {
        const decimal money = 3345.098M;
        DateTime now = DateTime.Now;

        using (new GinxCultureHelper("en-US"))
        {
            Console.WriteLine("Today is {0:D} and the number is {1:C}.", now, money);
        }

        using (new GinxCultureHelper("en-GB"))
        {
            Console.WriteLine("Today is {0:D} and the number is {1:C}.", now, money);
        }

        using (new GinxCultureHelper("fr-FR"))
        {
            Console.WriteLine("Today is {0:D} and the number is {1:C}.", now, money);
        }

        using (new GinxCultureHelper("ja-JP"))
        {
            Console.WriteLine("Today is {0:D} and the number is {1:C}.", now, money);
        }

        using (new GinxCultureHelper("pt-BR"))
        {
            Console.WriteLine("Today is {0:D} and the number is {1:C}.", now, money);
        }

        Console.ReadKey();
    }
}
```
