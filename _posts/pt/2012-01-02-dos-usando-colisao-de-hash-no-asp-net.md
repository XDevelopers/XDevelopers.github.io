---
layout: post
title: DoS usando colisão de hash no ASP .net
uname: dos-using-hash-collisins-in-asp.net
date: 2012-01-02 05:49
author: Sergio Garcia
comments: true
categories: pt, ASP .net, asp .net, C#, C#, Desenvolvimento, DoS, hash collision attack, performance, Sem categoria, Tecnologia, vulnerability, zero day flow, zero day vulnerability
---
<p>Este fim de semana na <a href="http://events.ccc.de/congress/2011/Fahrplan/" title="28th Chaos Communication Congress" target="_blank">28C3</a> Alexander "alech" Klink and Julian "zeri" Wälde apresentaram <a href="http://events.ccc.de/congress/2011/Fahrplan/events/4680.en.html" title="DoS efetivo contra plataformas de aplicações web" target="_blank">Effective Denial of Service attacks against web application platforms</a> que pode fazer um server ficar com 99% de CPU ocupada usando poucos recursos.</p>

<!--more-->

<p>Nos meus testes, um simples post HTTP, com cerca de 100k hashes colididos (cerca de 1,2MB) deixou uma thread do servidor por horas usando 99% de CPU, conforme a imagem abaixo:</p>

<a href="http://blog.ginx.com.br/wp-content/uploads/2012/01/hash-collision.png"><img src="http://blog.ginx.com.br/wp-content/uploads/2012/01/hash-collision-300x176.png" alt="" title="hash-collision" width="300" height="176" class="alignnone size-medium wp-image-87" /></a>

<p>Se mais thread forem usadas, o servidor ficará facilmente sobrecarregado. No estudo, alech e zeri concluiram que com uma conexão de 30 kbits/s é suficiente para manter uma thread de um Core2 usando 99% de CPU indefinidamente, em um servidor rodando ASP .net. E uma conexão de 1 Gbit/s conseguirá ocupar cerca de ~30k thread do Core2. Um atacante por facilmente derrubar um pequeno datacenter usando uma conexão banda larga comum.</p>

<h3>O Ataque</h3>

<p>O problema ocorre pela forma como o ASP .net gerencia Forms, Cookies, Session e Query Strings. Todas essas coleções são armazenadas usando a <a href="http://msdn.microsoft.com/en-us/library/system.collections.specialized.namevaluecollection.aspx" title="NameValueCollection" target="_blank">NameValueCollection</a>, que internamente usa <a href="http://msdn.microsoft.com/en-us/library/system.stringcomparer.ordinalignorecase.aspx" title="StringComparer.OrdinalIgnoreCase" target="_blank">StringComparer.OrdinalIgnoreCase</a> para obter o hash das strings.</p>

<p>A função <a href="http://msdn.microsoft.com/en-us/library/a04941bd.aspx" title="StringComparer.GetHashCode Method " target="_blank">GetHashCode</a> pode retornar valores iguais para strings diferentes e o método usado para calcular o hash é conhecido e pode ser adivinhado. O atacante pode gerar uma grande coleção de hashes colididos para postar em qualquer página ASP .net. O servidor tentará construir o objeto <a href="http://msdn.microsoft.com/en-us/library/system.web.httpcontext.request.aspx" title="HttpContext.Request Property" target="_blank">Request</a>, e por causa das chaves colididas do Form, a chaves do <a href="http://msdn.microsoft.com/en-us/library/system.collections.specialized.namevaluecollection.aspx" title="NameValueCollection" target="_blank">NameValueCollection</a> levarão muito tempo para serem criadas e o servidor utilizará muita CPU para comparar todos os valores.</p>

<p>Nos meus testes, usando IIS 7.5 e ASP .net 4.0, o servidor levou mais de uma hora usando 99% na thread afetada. O servidor não pode parar o processamento no tempo padrão (90s) porque a requisição foi afetada no começo do Pipeline do ASP .net.</p>

<h3>A solução</h3>

<p>Um módulo HTTP que verifique a requisição antes dela ser passada ao ASP .net e que pode para-lá caso um ataque seja detectado.</p>

<p>Eu comecei a criar este módulo, que é funcional mas não está pronto para uso em produção. Por enquanto, é para estudos, visto que a descoberta da falha ainda é recente.</p>

<p>O módulo HTTP e seu código fonte pode ser baixado do <a href="https://github.com/ginx/HashCollisionDetector" title="HashCollisionDetector" target="_blank">GitHub</a>.

<p>Qualquer comentário para ajudar a melhorar e garantir qualidade de produção ao módulo são bem vindos.</p>
