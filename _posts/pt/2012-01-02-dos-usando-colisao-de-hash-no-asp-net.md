---
layout: post
title: DoS usando colisão de hash no ASP .net
uname: dos-using-hash-collisions-in-asp.net
date: 2012-01-02 05:49
author: Sergio Garcia
comments: true
categories: [ASP .net, asp .net, C#, C#, Desenvolvimento, DoS, hash collision attack, performance, Sem categoria, Tecnologia, vulnerability, zero day flow, zero day vulnerability]
---

Este fim de semana na [28C3][] Alexander "alech" Klink and Julian "zeri" Wälde apresentaram [DoS efetivo contra plataformas de aplicações web][paper] que pode fazer um server ficar com 99% de CPU ocupada usando poucos recursos.

Nos meus testes, um simples post HTTP, com cerca de 100k hashes colididos (cerca de 1,2MB) deixou uma thread do servidor por horas usando 99% de CPU, conforme a imagem abaixo:

![Hash Collision](http://blog.ginx.com.br/wp-content/uploads/2012/01/hash-collision-300x176.png)

Se mais thread forem usadas, o servidor ficará facilmente sobrecarregado. No estudo, alech e zeri concluiram que com uma conexão de 30 kbits/s é suficiente para manter uma thread de um Core2 usando 99% de CPU indefinidamente, em um servidor rodando ASP .net. E uma conexão de 1 Gbit/s conseguirá ocupar cerca de ~30k thread do Core2. Um atacante por facilmente derrubar um pequeno datacenter usando uma conexão banda larga comum.

### O Ataque

O problema ocorre pela forma como o ASP .net gerencia Forms, Cookies, Session e Query Strings. Todas essas coleções são armazenadas usando a [NameValueCollection][], que internamente usa [StringComparer.OrdinalIgnoreCase][StringComparer_OrdinalIgnoreCase] para obter o hash das strings.

A função [GetHashCode][StringComparer_GetHashCode] pode retornar valores iguais para strings diferentes e o método usado para calcular o hash é conhecido e pode ser adivinhado. O atacante pode gerar uma grande coleção de hashes colididos para postar em qualquer página ASP .net. O servidor tentará construir o objeto [Request][HttpContext_Request], e por causa das chaves colididas do Form, a chaves do [NameValueCollection][] levarão muito tempo para serem criadas e o servidor utilizará muita CPU para comparar todos os valores.

Nos meus testes, usando IIS 7.5 e ASP .net 4.0, o servidor levou mais de uma hora usando 99% na thread afetada. O servidor não pode parar o processamento no tempo padrão (90s) porque a requisição foi afetada no começo do Pipeline do ASP .net.

### A solução

Um módulo HTTP que verifique a requisição antes dela ser passada ao ASP .net e que pode para-lá caso um ataque seja detectado.

Eu comecei a criar este módulo, que é funcional mas não está pronto para uso em produção. Por enquanto, é para estudos, visto que a descoberta da falha ainda é recente.

O módulo HTTP e seu código fonte pode ser baixado do [GitHub][HashCollisionDetector].

Qualquer comentário para ajudar a melhorar e garantir qualidade de produção ao módulo são bem vindos.


[28C3]: http://events.ccc.de/congress/2011/Fahrplan/ "28th Chaos Communication Congress"
[paper]: http://events.ccc.de/congress/2011/Fahrplan/events/4680.en.html "Effective Denial of Service attacks against web application platforms"
[NameValueCollection]: http://msdn.microsoft.com/en-us/library/system.collections.specialized.namevaluecollection.aspx
[StringComparer_OrdinalIgnoreCase]: http://msdn.microsoft.com/en-us/library/system.stringcomparer.ordinalignorecase.aspx
[StringComparer_GetHashCode]: http://msdn.microsoft.com/en-us/library/a04941bd.aspx "StringComparer.GetHashCode Method"
[HttpContext_Request]: http://msdn.microsoft.com/en-us/library/system.web.httpcontext.request.aspx "HttpContext.Request Property"
[HashCollisionDetector]: https://github.com/sergio-garcia/HashCollisionDetector
