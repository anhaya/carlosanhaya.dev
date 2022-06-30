---
title: "[Go] Struct e Memória"
date: 2022-06-30
tags: ["golang"]
author: "Carlos Anhaya"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A ordem em que você declara seus atributos dentro de uma struct importa e eu vou te mostrar."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
editPost:
    URL: "https://github.com/anhaya/carlosanhaya.dev/tree/main/carlosanhaya.dev/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
CPU lê a memória em &quot;palavras&quot;, 4 bytes em sistema de 32 bits, e 8 bytes em um sistema de 64 bits. Então...?

Vamos considerar a seguinte código

![image info](/struct_bad.png)

Quando executado, iremos perceber que o total é 88, **porém se somarmos o total de cada campo não dá 88…**

Como disse, sistemas de 32bits = blocos de 4 Bytes, sistemas de 64bits, blocos de 8 bytes. Isso significa que no nosso caso o bool que tem 1 byte e que está entre as strings, passará a ter 7 bytes &quot;vazios&quot;, isso porque ele está entre tipos de variáveis que são maiores do que bool. O mesmo vale para o int32 que está na última posição.

Para resolver esse &quot;problema&quot;, precisamos refatorar da seguinte maneira:

![image info](/struct_good.png)

Neste caso, teremos o total igual 72, pois as strings estão declaradas primeiro, bool e int32 por último, o que significa que eles ocuparão uma palavra de leitura uma vez que teremos _HaveDog; IsProgrammer e Idade_ no mesmo bloco, o que resulta em 6 bytes e consequentemente 2 bytes vazios.

Ou seja, a ordem em que estão declarados os atributos dentro da struct importa, importa pois isso evitará consumo de memória e ciclos de leitura da CPU.

Eu ainda acho que um dia o Go irá implementar isso em tempo de compile, mas até lá, vale a dica =)

**Referências**

[https://www.geeksforgeeks.org/data-structure-alignment-how-data-is-arranged-and-accessed-in-computer-memory/](https://www.geeksforgeeks.org/data-structure-alignment-how-data-is-arranged-and-accessed-in-computer-memory/)

[https://towardsdev.com/golang-writing-memory-efficient-and-cpu-optimized-go-structs-62fcef4dbfd0](https://towardsdev.com/golang-writing-memory-efficient-and-cpu-optimized-go-structs-62fcef4dbfd0)

