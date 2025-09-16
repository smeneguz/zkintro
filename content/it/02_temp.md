---

title: 'Programmare le ZKP: da Zero a Esperto'
date: '2024-08-30'
tags: ['zero-knowledge']
draft: false
layout: PostSimple
slug: "programming-zkps-from-zero-to-hero"
images: [../assets/02\_combined.png']
summary: "Impara a scrivere e modificare Zero Knowledge Proofs da zero. Costruirai uno schema di firma digitale basato su commitment con hash, acquisendo competenze pratiche e intuizioni durante la lettura. Alla fine, avrai tutti gli strumenti per implementare schemi come le firme di gruppo."
---

_A tutorial introduction for the working programmer._

_Un’introduzione pratica per chi programma._

Do you know why zebras have stripes? One theory is that it is a form of camouflage. When zebras are in a herd together, it makes it harder for the lion to distinguish their prey. Lions have to isolate their prey from the flock to be able to go after it. [^1]

Sai perché le zebre hanno le strisce? Una teoria è che si tratti di un meccanismo di mimetizzazione. Quando sono in branco, diventa più difficile per i leoni distinguere la preda. Per attaccare, i leoni devono isolarla dal gruppo. [^1]

Humans like to hide in a crowd too. One specific example of this is when multiple people act as one under a collective name. This was done for the Federalist Papers which led to the ratification of the United States Constitution. Multiple individuals wrote essays under the single Pseudonym "Publius". [^2] Another example is Bourbaki, a collective pseudonym for a group of French mathematicians in the 1930s. This lead to a complete re-write of large parts of modern mathematics with their focus on rigor and the axiomatic method. [^3]

Anche gli esseri umani amano nascondersi nella folla. Un esempio concreto è quando più persone agiscono sotto uno pseudonimo collettivo. È il caso dei *Federalist Papers*, che contribuirono alla ratifica della Costituzione degli Stati Uniti: più autori scrissero saggi firmati da un solo pseudonimo, “Publius”. [^2] Un altro esempio è Bourbaki, pseudonimo collettivo di un gruppo di matematici francesi degli anni ’30. Il loro lavoro portò a una riscrittura radicale di intere aree della matematica moderna, con un’enfasi su rigore e metodo assiomatico. [^3]

![Congresso Bourbaki](../assets/02_bourbaki.png "Congresso Bourbaki")

_Bourbaki congress in 1938_

_Congresso Bourbaki nel 1938_

In the digital age, let's say you are in a group chat and want to send a controversial message. You want to prove that you are one of its members, without revealing which one. How can we do this in the digital realm using cryptography? We can use something called _group signatures_.

Nell’era digitale, immaginiamo tu sia in una chat di gruppo e voglia inviare un messaggio controverso. Vuoi dimostrare di essere parte del gruppo, senza dire chi sei. Come si fa, crittograficamente, nel mondo digitale? Si usano le cosiddette _group signatures_.

Traditionally speaking, group signatures are quite mathematically involved and hard to implement. However, with Zero Knowledge Proofs (ZKPs), this math problem becomes a straightforward programming task. By the end of this article, you'll be able to program group signatures yourself.

Tradizionalmente, le firme di gruppo sono complesse dal punto di vista matematico e difficili da implementare. Ma con le Zero Knowledge Proofs (ZKP), questo problema matematico si trasforma in un compito di programmazione diretto. Alla fine di questo articolo, sarai in grado di programmare da solo uno schema di firme di gruppo.

## Introduction

## Introduzione

This post will show you how to write basic Zero Knowledge Proofs (ZKPs) from scratch.

In questo articolo vedremo come scrivere da zero prove a conoscenza zero (ZKPs) di base.

When learning a new tech stack, we want to get a hang of the edit-build-run cycle as soon as possible. Only then can we start to learn from our own experience.

Quando si impara una nuova tecnologia, la prima cosa è prendere confidenza con il ciclo modifica-compila-esegui. Solo allora si inizia davvero a imparare dall’esperienza.

We will start by getting you to setup your environment, write a simple program, perform a so-called trusted setup, and then generate and verify proofs as quickly as possible. After that, we'll identify some ways to improve our program, implement these improvements and test them. Along the way, we'll build up a better mental model of the pieces involved in programming ZKPs in practice. At the end of, you'll be familiar with (one way of) writing ZKPs from scratch.

Partiremo impostando l’ambiente di sviluppo, scrivendo un programma semplice, eseguendo un cosiddetto *trusted setup* (fase iniziale fidata per generare parametri di sistema), e generando/verificando prove il prima possibile. Poi vedremo come migliorare il nostro programma, implementeremo questi miglioramenti e li testeremo. Durante il percorso costruiremo un modello mentale più chiaro di cosa significa davvero programmare con le ZKPs. Alla fine, avrai familiarità con (un modo di) scrivere ZKPs da zero.

We will build up step by step to a simple signature scheme where you can prove that you sent a specific message. You'll be able to understand what this piece of code is doing and why:

Costruiremo passo dopo passo uno schema di firma semplice, in cui puoi dimostrare di aver inviato un certo messaggio. Sarai in grado di capire cosa fa questo frammento di codice e perché:

```javascript
template SignMessage () {
  signal input identity_secret;
  signal input identity_commitment;
  signal input message;
  signal output signature;

  component identityHasher = Poseidon(1);
  identityHasher.inputs[0] <== identity_secret;
  identity_commitment === identityHasher.out;

  component signatureHasher = Poseidon(2);
  signatureHasher.inputs[0] <== identity_secret;
  signatureHasher.inputs[1] <== message;
  signature <== signatureHasher.out;
}

component main {public [identity_commitment, message]} = SignMessage();
```

You'll also have been given all the tools and techniques necessary to modify this to support the group signature scheme mentioned above.

Avrai anche a disposizione tutti gli strumenti e le tecniche per modificare questo schema e adattarlo a supportare una firma di gruppo come quella descritta sopra.
