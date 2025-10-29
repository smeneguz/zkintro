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

Anche gli esseri umani amano nascondersi nella folla. Un esempio concreto è quando più persone agiscono sotto uno pseudonimo collettivo. È il caso dei _Federalist Papers_, che contribuirono alla ratifica della Costituzione degli Stati Uniti: più autori scrissero saggi firmati da un solo pseudonimo, “Publius”. [^2] Un altro esempio è Bourbaki, pseudonimo collettivo di un gruppo di matematici francesi degli anni ’30. Il loro lavoro portò a una riscrittura radicale di intere aree della matematica moderna, con un’enfasi su rigore e metodo assiomatico. [^3]

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

Partiremo impostando l’ambiente di sviluppo, scrivendo un programma semplice, eseguendo un cosiddetto _trusted setup_ (fase iniziale fidata per generare parametri di sistema), e generando/verificando prove il prima possibile. Poi vedremo come migliorare il nostro programma, implementeremo questi miglioramenti e li testeremo. Durante il percorso costruiremo un modello mentale più chiaro di cosa significa davvero programmare con le ZKPs. Alla fine, avrai familiarità con (un modo di) scrivere ZKPs da zero.

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


### Pre-requisites

### Prerequisiti

We assume you are a software engineer with working experience in more than one programming language, who has basic familiar with using Unix-style command line interfaces. We also assume you have a passing familiarity with concepts like _digital signatures_, _public-key cryptography_ and _hash functions_. Nonetheless, we'll introduce their relevant properties as they become relevant.

Diamo per scontato che tu sia un software engineer con esperienza pratica in più di un linguaggio di programmazione e con una conoscenza di base dell’uso di interfacce a riga di comando in stile Unix. Supponiamo inoltre che tu abbia almeno una conoscenza superficiale di concetti come _digital signatures_ (firme digitali), _public-key cryptography_ (crittografia a chiave pubblica) e _hash functions_ (funzioni di hash). In ogni caso, introdurremo le proprietà rilevanti man mano che saranno necessarie.

When it comes to _Zero Knowledge Proofs_, we assume you've read my previous post, [_A Friendly Introduction to Zero Knowledge_](https://zkintro.com/articles/friendly-introduction-to-zero-knowledge). If you haven't read this article, we'll quickly recap the most important things here. For better understanding, we recommend reading the above article first. If you have already read it, you can safely skip the below.

Per quanto riguarda le _Zero Knowledge Proofs_ (ZKPs), assumiamo che tu abbia già letto il mio articolo precedente, [_Un’introduzione amichevole alla conoscenza zero_](https://zkintro.com/articles/friendly-introduction-to-zero-knowledge). Se non l’hai ancora letto, qui faremo un rapido riepilogo dei concetti fondamentali. Per una comprensione migliore ti consigliamo di leggere prima quell’articolo. Se invece lo hai già letto, puoi tranquillamente saltare la sezione seguente.

### Recap of ZKPs

### Riepilogo sulle ZKPs

Zero Knowledge Proofs (ZKPs) are a fairly new form of cryptography that have seen more practical applications lately. While traditional cryptography allows us to do things like signatures and encryption, ZKPs allows us to prove arbitrary statements in a general-purpose way.

Le Zero Knowledge Proofs (ZKPs) sono una forma relativamente nuova di crittografia che negli ultimi anni ha trovato applicazioni pratiche sempre più numerose. Se la crittografia tradizionale ci permette di realizzare cose come firme ed encryption, le ZKPs consentono di dimostrare asserzioni arbitrarie in modo general-purpose.

Outside of proving arbitrary statements, ZKPs give us two key properties: privacy and compression. These are also known as zero knowledge and succinctness, respectively. Privacy means we can prove something without revealing anything else. Compression means the proof of an arbitrary statement stays roughly the same size regardless of how complex the computation we are proving is. ZKPs are also general-purpose. Roughly speaking, this is the difference between a calculator, made for a specific task, and a computer, that can compute anything.

Oltre alla possibilità di dimostrare asserzioni arbitrarie, le ZKPs ci offrono due proprietà fondamentali: privacy e compressione. Queste sono note anche come _zero knowledge_ e _succinctness_. Privacy significa poter dimostrare qualcosa senza rivelare nient’altro. Compressione significa che la prova di un’asserzione rimane all’incirca della stessa dimensione indipendentemente dalla complessità del calcolo che stiamo dimostrando. Inoltre, le ZKPs sono general-purpose. In termini semplici, è la differenza tra una calcolatrice, progettata per un compito specifico, e un computer, capace di eseguire qualsiasi calcolo.

Two concrete examples of ZKPs:

Due esempi concreti di ZKPs:

- We can take a digital identity card and prove that we are over 18 years old

  - Without revealing anything else, like your full name or address

- Possiamo usare una carta d’identità digitale per dimostrare di avere più di 18 anni

  - Senza rivelare nient’altro, come nome completo o indirizzo

- We can prove that all state transitions have been executed correctly

  - Such as in a public blockchain, with the resulting proof being very small

- Possiamo dimostrare che tutte le transizioni di stato sono state eseguite correttamente

  - Ad esempio in una blockchain pubblica, con la prova risultante molto compatta

We can program many common types of ZKPs by writing special programs known as circuits. This allows one party, a prover, to create a proof of some statement. Another party, known as a verifier, can then verify this proof. Like a normal program, this program can take input and produce output. For these special programs, we can specify if the input is private or public. If it is private, it means only the prover can see this input. We program circuits by specifying constraints. One example of a constraint is "in a Sudoku puzzle all numbers 1 through 9 must be used exactly once in a row".

Possiamo programmare molti tipi comuni di ZKPs scrivendo programmi speciali chiamati _circuits_. Questo consente a una parte, il _prover_, di creare una prova di una certa asserzione. Un’altra parte, il _verifier_, può poi verificarla. Come un normale programma, anche questi circuit possono ricevere input e produrre output. Possiamo inoltre specificare se l’input è pubblico o privato. Se è privato, significa che solo il prover può vederlo. Programmiamo i circuit definendo _constraints_ (vincoli). Un esempio di vincolo è: “in un Sudoku, i numeri da 1 a 9 devono comparire esattamente una volta in ogni riga”.

ZKPs are fairly new but they are already used a lot in public blockchains, for example, to allow private payments with fungible money, or to allow more transactions to be processed faster.

Le ZKPs sono piuttosto nuove ma già largamente utilizzate nelle blockchain pubbliche. Ad esempio, permettono pagamenti privati con moneta fungibile oppure consentono di elaborare un numero maggiore di transazioni più rapidamente.

More and more applications are being discovered and developed every day. There are also a lot of different flavors of ZKPs, all with their own set of trade-offs, and it is a very active area of research. These different flavors are being developed rapidly, and allow for increased efficiency and other affordances.

Ogni giorno vengono scoperte e sviluppate nuove applicazioni. Esistono inoltre molte varianti di ZKPs, ciascuna con i propri compromessi, e si tratta di un campo di ricerca estremamente attivo. Queste diverse varianti stanno evolvendo rapidamente e consentono maggiore efficienza e nuove possibilità d’uso.

## Overview

## Panoramica

We are going to use Circom and Groth16. Circom is a domain-specific language (DSL) for writing ZKP circuits. Groth16 is a common and popular proving system. Roughly speaking, a proving system is just one way that you can program ZKPs. Other DSLs and proving systems also exists.

Utilizzeremo Circom e Groth16 Circom è un _domain-specific language_ (DSL) per scrivere circuiti ZKP. Groth16 è un sistema di proving tra i più diffusi. In termini semplici, un proving system è un metodo per programmare ZKPs. Esistono anche altri DSL e proving system.

We'll start by installing some tools and dependencies. After that, we'll proceed in the following rough steps:

Inizieremo installando strumenti e dipendenze. Dopodiché procederemo grosso modo nei seguenti passaggi:

- Write (write circuit)
- Scrivere (scrivere il circuito)
- Build (build circuit)
- Compilare (costruire il circuito)
- Setup (trusted setup)
- Setup (trusted setup)
- Prove (generate proof)
- Generare (creare la prova)
- Verify (verify proof)
- Verificare (verificare la prova)

After having gone through this flow once, we'll look at some problems with the current approach. We'll then make several incremental improvements, building up to the signature scheme above. Along the way, we'll explain necessary concepts and syntax.

Dopo aver seguito una prima volta questo flusso, vedremo quali sono i limiti dell’approccio iniziale. Poi introdurremo miglioramenti graduali fino ad arrivare allo schema di firma presentato sopra. Nel frattempo spiegheremo i concetti e la sintassi necessari.

At the end of each section, we'll also include some simple exercises that will check your understanding. These exercises are recommended. At the very end of the article we'll also include a list of problems. Problems are optional and require a lot more effort.

Alla fine di ogni sezione proporremo anche alcuni esercizi semplici per verificare la comprensione. Questi esercizi sono consigliati. Alla fine dell’articolo troverai inoltre una lista di problemi opzionali, che richiedono più impegno.

### Preparation

### Preparazione

First up, we have to install some tools and dependencies. We have prepared a [git repo](https://github.com/oskarth/zkintro-tutorial) that makes it easier for you to get started without getting lost in the weeds with details. If you prefer not to install any software, see the end of this section.

Per prima cosa dobbiamo installare alcuni strumenti e dipendenze. Abbiamo preparato un [git repo](https://github.com/oskarth/zkintro-tutorial) per semplificare l’avvio ed evitarti di perderti nei dettagli. Se preferisci non installare alcun software, vedi la fine di questa sezione.

The pre-requisites we require are:

I prerequisiti richiesti sono:

- `rust` (the programming language)
- `rust` (il linguaggio di programmazione)
- `just` (a modern `make`)
- `just` (un moderno sostituto di `make`)
- `npm` (package manager for JavaScript)
- `npm` (gestore di pacchetti per JavaScript)

The ZKP tools we will actually use are:

Gli strumenti ZKP che useremo effettivamente sono:

- `circom` (for building our special program, or _circuit_)
- `circom` (per costruire il nostro programma speciale, ovvero il _circuit_)
- `snarkjs` (for setup, and generating/verifying proofs)
- `snarkjs` (per il setup e per generare/verificare prove)
- `just` tasks (to simplify common operations related to above)
- task di `just` (per semplificare le operazioni comuni)

To install the above as well as make building and running things easier you can clone and use the [git repo](https://github.com/oskarth/zkintro-tutorial). This should work on any Unix-like system like MacOS and Linux. If you use Windows we suggest using a Linux VM, Windows Subsystem for Linux (WSL), or similar for development.

Per installare tutto e semplificare build ed esecuzione, puoi clonare e usare il [git repo](https://github.com/oskarth/zkintro-tutorial). Questo dovrebbe funzionare su qualsiasi sistema Unix-like come MacOS e Linux. Se usi Windows, ti consigliamo una VM Linux, Windows Subsystem for Linux (WSL) o strumenti simili per lo sviluppo.

```shell
# Clone the repo and run the prepare script
git clone git@github.com:oskarth/zkintro-tutorial.git
cd zkintro-tutorial

# Skim the contents of this file before executing it
less ./scripts/prepare.sh
./scripts/prepare.sh
```

We recommend you skim the contents of `./scripts/prepare.sh` to see what this will install, or if you prefer to install things manually. Once executed you should see `Installation complete` and no errors.

Ti consigliamo di dare un’occhiata al contenuto di `./scripts/prepare.sh` per vedere cosa verrà installato, o se preferisci installare manualmente i pacchetti. Una volta eseguito lo script, dovresti vedere `Installation complete` e nessun errore.

If you get stuck, please see the latest official documentation [here](https://docs.circom.io/getting-started/installation/). Once done, you should have the following versions (or higher) installed:

Se incontri difficoltà, consulta la documentazione ufficiale aggiornata [qui](https://docs.circom.io/getting-started/installation/). Al termine, dovresti avere installato le seguenti versioni (o superiori):

```shell
> circom --version
circom compiler 2.1.8

> snarkjs | head -n 1
snarkjs@0.7.4
```

In the repo there is a `justfile` that defines a set of common commands. These `just` commands aim to simplify common operations on ZKPs, so you can focus on conceptual understanding of the actual steps involved. This makes the process much less error-prone when you are starting out.

Nel repository c’è un `justfile` che definisce una serie di comandi comuni. Questi comandi di `just` servono a semplificare le operazioni più frequenti sulle ZKPs, così puoi concentrarti sulla comprensione concettuale dei passaggi. Questo rende il processo molto meno soggetto a errori, soprattutto all’inizio.

If at any time you want to see in more detail what commands are being executed, we recommend you look at the `justfile` and the various scripts in the `scripts` folder.

Se in qualsiasi momento vuoi capire nel dettaglio quali comandi vengono eseguiti, ti consigliamo di guardare il `justfile` e i vari script nella cartella `scripts`.

We highly recommend installing the above software for following along the tutorial and building intuition. However, If you do not want to install any software, you can follow along in a limited capacity using an online REPL (Read-Eval-Print Loop) tool such [zkrepl.dev](https://zkrepl.dev). If you do not want to install `just` and prefer to execute all the commands yourself you can do so with a little extra effort by using the accompanying shell scripts.

Ti consigliamo vivamente di installare i software sopra indicati per seguire il tutorial e costruire intuizione pratica. Tuttavia, se non vuoi installare nulla, puoi comunque seguire (in forma limitata) usando uno strumento REPL online come [zkrepl.dev](https://zkrepl.dev). Se invece non vuoi installare `just` e preferisci eseguire manualmente tutti i comandi, puoi farlo con un po’ di lavoro extra utilizzando gli script shell forniti.

## First iteration

## Prima iterazione

We are now ready to start coding. To build up to the signature scheme mentioned above, we will start with a very simple program, the equivalent of a "Hello World" in other programming languages.

Siamo finalmente pronti per scrivere del codice. Per arrivare allo schema di firma menzionato in precedenza, cominceremo con un programma molto semplice, l’equivalente di un “Hello World” in altri linguaggi di programmazione.

In practical terms, we will write a special program that will help us prove knowledge of two secret numbers whose product is a public number, _without ever revealing the secret numbers themselves_. For example, the public number might be "33" and the secret numbers are "11" and "3". This is an important stepping stone towards digital signatures and will build help intuition for how ZKPs work. If you are familiar with public-key cryptography, you can - very loosely - think of the secret numbers as a "private key" and the public number as a "public key".

In termini pratici, scriveremo un programma che ci permetterà di dimostrare di conoscere due numeri segreti il cui prodotto è un numero pubblico, _senza mai rivelare i numeri segreti stessi_. Per esempio, il numero pubblico potrebbe essere “33”, mentre i numeri segreti sono “11” e “3”. Questo è un passo fondamentale verso la costruzione delle firme digitali e ci aiuterà a sviluppare l'intuizione su come funzionano le ZKPs. Se conosci la crittografia a chiave pubblica, puoi pensare in modo approssimativo ai numeri segreti come a una “chiave privata” e al numero pubblico come a una “chiave pubblica”.

Since this is a different way of programming involving many new concepts, don't worry if things don't make sense at first. You can always keep going, focusing on the code, generating proofs, etc and come back to a specific section later on.

Dato che si tratta di un approccio alla programmazione piuttosto diverso e ricco di concetti nuovi, non preoccuparti se all’inizio qualcosa non ti è chiaro. Puoi tranquillamente proseguire concentrandoti sul codice, sulla generazione delle prove e sul flusso pratico, per poi tornare in seguito a rivedere una sezione specifica.

### Write a special program

### Scrivere un programma

Unlike most other programming, writing these special programs, circuits, look a bit different. What we are interested in is proving a _set of constraints_. [^4] The simplest set of constraints we can prove consists of a single constraint. [^5] What we will constrain is that two numbers multiplied by each other equal a third one.

A differenza della programmazione tradizionale, scrivere questi programmi speciali (_circuits_) è leggermente diverso. Qui ci interessa dimostrare un _insieme di vincoli_ (constraints). [^4] L’insieme più semplice che possiamo dimostrare è composto da un solo vincolo. [^5] In questo caso, vogliamo imporre che il prodotto di due numeri sia uguale a un terzo.

Go to the `example1` folder in the `zkintro-tutorial` repository above. There's a skeleton program in `example1.circom`. Modify it to look like this:

Vai nella cartella `example1` del repository `zkintro-tutorial`. Troverai un programma di esempio in `example1.circom`. Modificalo in questo modo:

```javascript
pragma circom 2.0.0;

template Multiplier2 () {
  signal input a;
  signal input b;
  signal output c;
  c <== a * b;
}

component main = Multiplier2();
```

This is our special program, or _circuit_. [^6] Going line by line:

Questo è il nostro programma speciale, ovvero il nostro _circuit_. [^6] Analizziamolo riga per riga:

- `pragma circom 2.0.0;` - definisce la versione di Circom utilizzata
- `template Multiplier()` - i template sono l’equivalente degli oggetti nella maggior parte dei linguaggi di programmazione, una forma comune di astrazione
- `signal input a;` - il nostro primo input, `a`; come impostazione predefinita gli input sono privati
- `signal input b;` - il secondo input, `b`; anch’esso privato per default
- `signal output c;` - l’output `c`; gli output invece sono sempre pubblici
- `c <== a * b;` - fa due cose: assegna al segnale `c` un valore _e_ impone che `c` sia uguale al prodotto di `a` e `b`
- `component main = Multiplier2()` - istanzia il componente principale del circuito

The most important line is `c <== a * b;`. This is where we actually declare our constraint. This expression is actually a combination of two: `<--` (assignment) and `===` (equality constraint). [^7] A constraint in Circom can only use operations involving constants, addition or multiplication. It enforces that both sides of the equation must be equal. [^8]

La riga più importante è `c <== a * b;`. È qui che dichiariamo effettivamente il nostro vincolo. Questa espressione è in realtà la combinazione di due operazioni: `<--` (assegnazione) e `===` (vincolo di uguaglianza). [^7] Un vincolo in Circom può usare solo operazioni con costanti, somme o moltiplicazioni. In pratica impone che entrambi i lati dell’equazione siano uguali. [^8]

### On constraints

### Sui vincoli

How do constraints work? In the context of something like Sudoku, we might say a constraint is "a number between 1 and 9". In the context of Circom however, this is not a single constraint, but instead something we have to express using a set of simpler equality constraints (`===`). [^9]

Come funzionano i vincoli? In un contesto come quello del Sudoku, potremmo dire che un vincolo è “un numero compreso tra 1 e 9”. In Circom, però, questo non è un singolo vincolo, ma un insieme di vincoli di uguaglianza (`===`) che devono essere combinati per rappresentarlo. [^9]

Why is this the case? This has to do with what is mathematically going on under the hood. Fundamentally, most ZKPs use _arithmetic circuits_ which represents computation over _polynomials_. When dealing with polynomials, you can easily introduce constants, add them together, multiply them and check if they are equal to each other. [^10] Other operations have to be expressed in terms of these fundamental operations. You do not have to understand this in detail in order to write ZKPs, but it can be useful to have some intuitition of what is going on under the hood. [^11]

Perché accade questo? La ragione è matematica. La maggior parte delle ZKPs si basa su _arithmetic circuits_, che rappresentano calcoli su _polinomi_. Quando si lavora con polinomi, è facile introdurre costanti, sommarle, moltiplicarle e verificare se due risultati sono uguali. [^10] Tutte le altre operazioni devono essere espresse in termini di queste operazioni fondamentali. Non è necessario comprenderle nei dettagli per scrivere ZKPs, ma avere un minimo di intuizione su cosa succede “sotto il cofano” può essere molto utile. [^11]

We can visualize the circuit as follows:

Possiamo visualizzare il circuito in questo modo:

![example1 circuit](../assets/02_example1_circuit.png "example1 circuit")

### Building our circuit

### Compilare il nostro circuito

For your reference, the final file can be found in `example1-solution.circom`. For more details on the syntax, see the [official documentation](https://docs.circom.io/circom-language/signals/).

Per riferimento, il file finale è disponibile in `example1-solution.circom`. Per maggiori dettagli sulla sintassi puoi consultare la [documentazione ufficiale](https://docs.circom.io/circom-language/signals/).

We can compile our circuit by running:

Possiamo compilare il circuito eseguendo:

```shell
just build example1
```

![example1 build](../assets/02_example1_build.png "example1 build")

This is a thin wrapper for calling `circom` to create a `example1.r1cs` and `example1.wasm` file. You should see something like:

Questo comando è un semplice wrapper che richiama `circom` per creare i file `example1.r1cs` e `example1.wasm`. Dovresti vedere un output simile a questo:

```shell
template instances: 1
non-linear constraints: 1
linear constraints: 0
public inputs: 0
private inputs: 2
public outputs: 1
wires: 4
labels: 4
Written successfully: example/target/example1.r1cs
Written successfully: example/target/example1_js/example1.wasm
```

In this case, we have the following:

In questo caso, abbiamo:

- due input privati, `a` e `b`
- un output pubblico, `c`
- un vincolo (non lineare), `c <== a * b`

We will ignore other parts of the output above for now. [^12] Now we have two files: `example1.r1cs` and `example1.wasm`.

Per ora possiamo ignorare le altre parti dell’output. [^12] Ora abbiamo due file: `example1.r1cs` e `example1.wasm`.

`r1cs` stands for *Rank 1 Constraint System*. This file contains our circuit in binary form. and corresponds to how we define our constraints mathematically. [^13]

`r1cs` sta per *Rank 1 Constraint System*. Questo file contiene il circuito in forma binaria e rappresenta il modo in cui i nostri vincoli vengono definiti matematicamente. [^13]

The `.wasm` file contains WebAssembly, which is what we need to generate our *witness*. The witness is how we specify the inputs that we want to keep private while still using them to create a proof.

Il file `.wasm` contiene il codice WebAssembly, necessario per generare il nostro *witness* (testimone). Il witness è ciò che ci permette di specificare gli input che vogliamo mantenere privati, pur usandoli per creare una prova.

We are not quite ready to make proofs yet though. First we need to perform a *setup* to get our prover and verification key.

Non siamo ancora pronti per generare le prove. Prima dobbiamo eseguire un *setup* per ottenere le chiavi del *prover* e del *verifier*.

Don't worry if it all doesn't make sense yet. It is a new way of doing things and it takes a while to get used to.

Non preoccuparti se tutto questo non ti è ancora chiaro. È un nuovo modo di lavorare e richiede un po’ di tempo per abituarsi.
