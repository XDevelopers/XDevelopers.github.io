---
layout: post
title: Detectando a mudança de abas do navegador
date: 2011-07-11 13:29
author: Sergio Garcia
comments: true
categories: [abas, Desenvolvimento, Javascript, javascript, jquery]
---
As vezes precisamos efetuar notificações ou exibir propagandas para o usuário, criando um efeito visual para o usuário. Caso o usuário não esteja visualizando a página no momento, este efeito pode ser perdido. Ou então, você pode querer saber o tempo que o usuário passou efetivamente visualizando a sua página.

<!--more-->

Você pode detectar se o usuário mudou a aba do navegador, minimizou a janela ou mudou de janela, através do evento <strong>blur</strong> da janela e se ele voltar a página pelo evento <strong>focus</strong>.

Abaixo segue um exemplo de como podemos fazer isso usando jQuery.
<pre lang="javascript">    var activePage = true;

    function showInformation() {
        if (activePage) {
            $("body").append("Page active.");
        } else {
            $("body").append("Page inactive.");
        }

        setTimeout("showInformation();", 1000);
    }

    $(function() {
        $(window)
            .focus(function() { activePage = true; })
            .blur(function() { activePage = false; });

        showInformation();
    });</pre>
Você pode ver isso funcionando <a href="/samples/2011/07/detectando-mudanca-de-abas.html">aqui</a>.
