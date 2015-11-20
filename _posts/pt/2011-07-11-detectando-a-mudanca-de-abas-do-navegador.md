---
layout: post
title: Detectando a mudança de abas do navegador
date: 2011-07-11 13:29
author: Sergio Garcia
comments: true
categories: [abas, Desenvolvimento, Javascript, javascript, jquery]
---

As vezes precisamos efetuar notificações ou exibir propagandas para o usuário,
criando um efeito visual para o usuário. Caso o usuário não esteja
visualizando a página no momento, este efeito pode ser perdido. Ou então, você
pode querer saber o tempo que o usuário passou efetivamente visualizando a sua
página.

Você pode detectar se o usuário mudou a aba do navegador, minimizou a janela
ou mudou de janela, através do evento **blur** da janela e se ele voltar a
página pelo evento **focus**.

Abaixo segue um exemplo de como podemos fazer isso usando jQuery.

```javascript
var activePage = true;

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
});
```
