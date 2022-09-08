---
title: "[Go] Order by similarity"
date: 2022-09-08
tags: ["golang", "levenshtein", "daily"]
author: "Carlos Anhaya"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Order by similarity using Levenshtein algorithm"
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

These days I had a daily task that I basically needed to show a list of random names on the UI, but, It needed to be displayed not ordered by names, but by similarity of another name.

When searching on google, I didn't find anything "ready" in golang so I could use it as an example. Actually it is a simple task because you already have a lot of Levenshtein algorithms, what you just need to understand is the goal of the algorithm and how you can use it to order by similarity.

What is Levenshtein? Well, read this one https://www.golangprograms.com/golang-program-for-implementation-of-levenshtein-distance.html, also I have used exactly this algorithm, yes, copy and paste \o/. Basically it measures the distance between two strings, it means that **the smaller the number, the greater the similarity**.

I have pushed the whole code here https://github.com/anhaya/daily/tree/main/levenshtein. The code is self explanatory, I have just copied and pasted the Levenshtein algorithm and used sort lib from golang.

I hope this could help you in some way. Bye!