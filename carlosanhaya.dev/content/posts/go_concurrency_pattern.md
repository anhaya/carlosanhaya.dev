---
title: "[Go] Patterns de Concorrência"
date: 2022-08-09
tags: ["golang", "concurrency"]
author: "Carlos Anhaya"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Patterns de concorrência seguindo abordagem do livro Concurrency in Go"
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
Hi There!

Estou estudando o livro *"Concurrency in Go",* e hoje vou compartilhar aqui algumas coisas que pra mim são importantes do capítulo sobre patterns de concorrência, e claro, com minhas palavras.

Este texto só faz sentido para quem já sabe um pouco de Go, conhece channel, mutex e entende a diferença entre concorrência e paralelismo. Alguns patterns que irei descrever são conhecidos no mercado e às vezes até utilizam outros nomes, outros na minha opinião nem vem a ser *patterns* mas sim um conceito para entender outros *patterns.*  Não fique preso a isso, entenda a usabilidade de cada um a fim de identificar um cenário que poderia ser resolvido com determinado pattern ou técnica.


**Confinement**

É sobre garantir que uma informação só é acessível por um processo concorrente, como quando usamos mutex.lock() / mutex.unlock(), mas aqui a idéia é garantir que seja *concurrent safe* sem precisar usar *memory synchronization (e.g mutex).* Existem dois tipos possíveis desse "confinamento", *ad hoc* e *lexical.*

*ad hoc,* basicamente aqui você tem um doc. (*best practices)* que diz que você (programador) não pode expor informação que não deve ser exposta.

*lexical,* basicamente aqui você expõe somente aquilo que é necessário expor, fazendo assim a informação ser *concurrent safe*

- <https://github.com/anhaya/concurrency-with-go/blob/main/confinement/main.go>.

**for-select**

Quando você tem um select (<https://go.dev/ref/spec#Select_statements>) sem um default case, isso é block até que entre em algum case. O *for* com *select* é utilizado normalmente quando você deseja que caia em mais de um case. Imagine o seguinte cenário,  Você precisa chamar 3 APIs de forma concorrente e aguardar o resultado delas, suponha que uma delas dá erro e você precise interromper essa espera. 

- <https://github.com/anhaya/concurrency-with-go/blob/main/forselect/main.go>

**Preventing Goroutine leaks**

Goroutine não são *garbage collected.* É difícil entender isso sem ter um pouco de base sobre *stack*, *heap* e *garbage collector*, para isso recomendo este vídeo: <https://www.youtube.com/watch?v=ZMZpH4yT7M0>, lembrando que a idéia de *stack, heap e garbage collector* é independente da linguagem, mas cada uma lida com isso de uma maneira diferente, pode ser que quando você estiver lendo este texto, o Go gerencie memória de goroutines de uma forma diferente da que é feita hoje na versão 1.18. Mas o que vem a ser goroutine leak?

Vamos supor que você tenha uma goroutine, essa goroutine chama uma API que faz um processamento de qualquer coisa e não retorna nada, e você como desenvolvedor não toma precaução de colocar algum timeout para essa chamada, percebeu o problema né? 	O que aconteceria é que sua goroutine iria ficar executando mais do que necessário e consequentemente seu programa iria consumir mais memória do que o necessário.

- <https://github.com/anhaya/concurrency-with-go/blob/main/goroutineleak/main.go>


**The Or-Channel**

Basicamente este *pattern* implementa um cenário onde você precisa chamar várias funções e parar todas de uma vez quando uma delas feche o próprio channel. Se você não usasse esse *pattern* você teria que fazer um select case para o channel de cada função, funcionaria, mas com esta implementação o código fica bem mais limpo.

O algoritmo basicamente é recursivo, olhe o código que é mais fácil do que eu descrever. Eu particularmente tentei pensar em vários cenários em que isso se encaixasse mas não consegui, para cada um que eu elaborava não iria fazer sentido seguir com esse *pattern*, talvez porque minha experiência está muito limitada com APIs. Se você encontrou um cenário onde isso parecia fazer sentido, por favor, faça um PR que eu atualizo o texto aqui.

- <https://github.com/anhaya/concurrency-with-go/blob/main/orchannel/main.go>.

**Error handling**

Você precisa chamar 2 APIs e verificar se as duas retornaram com sucesso, como você faria?
Essa abordagem de identificar erros que são tratados na goroutine é bem comum e você irá encontrar algo muito semelhante na internet. A ideia principal desta abordagem é, se sua goroutine tem a possibilidade de retornar um erro, então este erro deve estar no mesmo *channel* que você coloca o seu resultado com sucesso.

- <https://github.com/anhaya/concurrency-with-go/blob/main/errorhandling/main.go>

**Pipeline**

Pipeline nada mais é do que uma série de etapas onde a saída de cada etapa é a entrada da próxima, essas etapas é o que chamamos de stage. Exemplo, imagine uma *pipeline* da sua aplicação que tem o objetivo de fazer um deploy no k8s, para isso precisamos rodar a suite de testes, fazer o build, gerar imagem e fazer deploy (fui bem simplista mas é só pra passar a idéia), cada passo desse é o que chamamos de *stage* e cada passo seguinte depende do anterior.

Se uma pipeline consiste em um processo sequencial de várias etapas onde a saída de um é a entrada de outro, como isso se encaixa em um *pattern* de concorrência? O *pattern* de *pipeline* com concorrência no Go consiste basicamente de que cada *stage* terá como saída um channel cujo qual será usado como entrada no próximo *stage*. Cenário hipotético em que você poderia utilizar *pattern* de pipeline:

*1 - Você precisa ler um arquivo txt onde cada linha contém a informação de organização e repositório, separados por vírgula. Cada linha demora 1 segundo para ler.*

*2 - Para cada linha você deve transformá-los em upper case. Cada linha demora 1 segundo para fazer o upper case.*

*3 - Com os dados restantes você deve checar se a organização e repositório é válida, se não for você deve remover essa informação. Cada validação demora 1 segundo.*

O arquivo contém 15 linhas, como você faria esta implementação? Provavelmente seu primeiro pensamento foi criar uma função que lê o arquivo todo e armazena num slice, outra que transforma todo o slice em upper case e outra que valida se organização e repositório são válidos e executá-las de maneira sequencial. Vai funcionar, mas como cada linha leva 1 segundo para ser lida, 1 segundo para ser processada e 1 segundo para ser validada, provavelmente sua pipeline irá demorar 45 segundos.

Executando com *pattern* de *pipeline* a média é de 17 segundos, ou seja, como o *pattern* sua pipeline iria ser executada ~3 vezes mais rápida, interessante né?

Este *pattern de pipeline* não é a única maneira de implementar um processo sequencial com várias etapas, talvez o seu cenário seja preciso usar workers, fila ou qualquer outro pattern. Sempre avalie outras maneiras.

- <https://github.com/anhaya/concurrency-with-go/blob/main/pipeline/main.go>

**Fan-in/Fan-out**

Imagine a seguinte situação: você precisa gerar métrica de um ano

- Cada mês demora 1 segundo para obter os dados
- Cada mês demora 4 segundos para ser calculado
- Um mês não depende do outro

Como você implementaria?

Solução 1: Eu poderia obter os dados de todos os meses e em seguida faria o cálculo em cima dos resultados. Partindo das premissas acima eu levaria 1 min (12s + 48s) para concluir.

Solução 2: Eu poderia usar uma estratégia de Pipeline aqui, ou seja, ao obter o resultado de um mês eu já começo a calcular de maneira concorrente. Parece bom, melhor que a solução 1.

Solução 3: Eu poderia remodelar a solução 2, usando a estratégia de Pipeline junto com a de Fanin/Fanout. Ou seja, ao invés de usar somente um worker para fazer o cálculo eu irei usar 4 workers (número aleatório, pode ser mais ou pode ser menos), iniciar mais de um worker para fazer o trabalho é o que chamamos de FanOut, após isso eu junto o resultado disso em um channel só, isso é o que chamamos de FanIn. Em alguns lugares também vemos a mesma abordagem mas como o mesmo nome, coisas como "worker pool pattern".

- <https://github.com/anhaya/concurrency-with-go/blob/main/fainfanout/main.go>




**Referência**

<https://austburn.me/blog/a-better-fan-in-fan-out-example.html>