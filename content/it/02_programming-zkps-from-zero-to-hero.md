---
title: 'Programmare le ZKP: From Zero to Hero'
date: '2024-08-30'
tags: ['zero-knowledge']
draft: false
layout: PostSimple
slug: "programming-zkps-from-zero-to-hero"
images: [../assets/02_combined.png']
summary: "Impara a scrivere e modificare Zero Knowledge Proofs (ZKP, dimostrazioni a conoscenza zero) da zero. Costruirai uno schema di firma digitale basato su commitment basati su funzioni di hash, acquisendo competenze pratiche e intuizione sulla programmazione ZKP strada facendo. Alla fine, avrai tutti gli strumenti necessari per implementare cose come le firme di gruppo."
translator: 'Silvio Meneguzzo'
---

_Questo libro è stato tradotto e adattato da Silvio Meneguzzo_

_Un'introduzione pratica e guidata per chi programma._

Sai perché le zebre hanno le strisce? Una teoria è che si tratti di un meccanismo di mimetizzazione. Quando sono in branco, diventa più difficile per il leone distinguere la preda. I leoni devono isolarla dal branco per poterla inseguire. [^1]

Anche gli esseri umani amano nascondersi nella folla. Un esempio concreto è quando più persone agiscono come una sola sotto un nome collettivo. È il caso dei Federalist Papers, che contribuirono alla ratifica della Costituzione degli Stati Uniti: più autori scrissero saggi firmati da un solo pseudonimo, "Publius". [^2] Un altro esempio è Bourbaki, pseudonimo collettivo di un gruppo di matematici francesi degli anni '30. Il loro lavoro portò a una riscrittura completa di grandi parti della matematica moderna, con la loro enfasi sul rigore e sul metodo assiomatico. [^3]

![Congresso Bourbaki](../assets/02_bourbaki.png "Congresso Bourbaki")

_Congresso Bourbaki nel 1938_

Nell'era digitale, immaginiamo tu sia in una chat di gruppo e voglia inviare un messaggio controverso. Vuoi dimostrare di essere uno dei suoi membri, senza rivelare quale. Come si può fare questo nel mondo digitale usando la crittografia? Si usano le cosiddette _firme di gruppo_ (group signatures).

Tradizionalmente, le firme di gruppo sono complesse dal punto di vista matematico e difficili da implementare. Ma con le Zero Knowledge Proofs, questo problema matematico si trasforma in un compito di programmazione semplice. Alla fine di questo articolo, sarai in grado di programmare da solo le firme di gruppo.

## Introduzione

In questo articolo vedremo come scrivere da zero prove a conoscenza zero basilari.

Quando si impara una nuova tecnologia, la prima cosa è prendere confidenza con il ciclo modifica-costruzione-esecuzione. Solo allora si inizia davvero a imparare dall'esperienza.

Partiremo impostando l'ambiente di sviluppo, scrivendo un programma semplice, eseguendo un cosiddetto trusted setup (setup fidato), e generando e verificando prove. Poi vedremo come migliorare il nostro programma, implementeremo questi miglioramenti e li testeremo. Durante il percorso costruiremo un modello mentale più chiaro di cosa significa programmare con le ZKP nella pratica. Alla fine, avrai familiarità con (un modo di) scrivere ZKP da zero.

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

Avrai anche a disposizione tutti gli strumenti e le tecniche necessarie per modificare questo schema e adattarlo a supportare la firma di gruppo (group signature) descritta sopra.

### Prerequisiti

Diamo per scontato che tu sia un software engineer con esperienza pratica in più di un linguaggio di programmazione e con una conoscenza di base dell'uso di interfacce a riga di comando in stile Unix. Supponiamo inoltre che tu abbia almeno una conoscenza superficiale di concetti come le _firme digitali_, la _crittografia a chiave pubblica_ e le _funzioni di hash_. In ogni caso, introdurremo le proprietà rilevanti man mano che saranno necessarie.

Per quanto riguarda le _Zero Knowledge Proofs_, assumiamo che tu abbia già letto il mio articolo precedente, [_Introduzione semplificata alla Zero Knowledge_](https://zkintro.com/it/articles/friendly-introduction-to-zero-knowledge). Se non l'hai ancora letto, qui faremo un rapido riepilogo dei concetti fondamentali. Per una comprensione migliore ti consiglio di leggere prima quell'articolo. Se invece lo hai già letto, puoi tranquillamente saltare la sezione seguente.

### Riepilogo sulle ZKP

Le Zero Knowledge Proofs sono una forma relativamente nuova di crittografia che negli ultimi anni ha trovato applicazioni pratiche sempre più numerose. Se la crittografia tradizionale ci permette di realizzare cose come firme e cifratura, le ZKP consentono di dimostrare asserzioni arbitrarie in modo general-purpose (generico).

Oltre alla possibilità di dimostrare asserzioni arbitrarie, le ZKP ci offrono due proprietà fondamentali: privacy e compressione. Queste sono note anche rispettivamente come _zero knowledge_ e _succinctness_. Privacy significa poter dimostrare qualcosa senza rivelare nient'altro. Compressione significa che la prova di un'asserzione arbitraria rimane all'incirca della stessa dimensione indipendentemente dalla complessità del calcolo che stiamo dimostrando. Le ZKP sono inoltre general-purpose. In termini semplici, è la differenza tra una calcolatrice, progettata per un compito specifico, e un computer, capace di eseguire qualsiasi calcolo.

Due esempi concreti di ZKP:

- Possiamo usare una carta d'identità digitale per dimostrare di avere più di 18 anni
  - Senza rivelare nient'altro, come il nome completo o l'indirizzo
- Possiamo dimostrare che tutte le transizioni di stato sono state eseguite correttamente
  - Ad esempio in una blockchain pubblica, con la prova risultante molto piccola

Possiamo programmare molti tipi comuni di ZKPs scrivendo programmi speciali chiamati circuiti. Questo consente a una parte, detta prover, di creare una prova di una certa asserzione. Un'altra parte, detta verifier, può poi verificarla. Come un normale programma, anche questi circuiti possono ricevere input e produrre output. Per questi programmi speciali possiamo specificare se l'input è pubblico o privato. Se è privato, significa che solo il prover può vederlo. Programmiamo i circuiti definendo dei vincoli (constraints). Un esempio di vincolo è: "in un Sudoku, i numeri da 1 a 9 devono comparire esattamente una volta in ogni riga".

Le ZKP sono piuttosto nuove ma già largamente utilizzate nelle blockchain pubbliche. Ad esempio, permettono pagamenti privati con moneta fungibile oppure consentono di elaborare un numero maggiore di transazioni più rapidamente.

Ogni giorno vengono scoperte e sviluppate nuove applicazioni. Esistono inoltre molte varianti di ZKP, ciascuna con i propri compromessi, ed è un campo di ricerca estremamente attivo. Queste diverse varianti si stanno sviluppando rapidamente e consentono maggiore efficienza e nuove possibilità d'uso.

## Panoramica

Utilizzeremo Circom e Groth16. Circom è un _domain-specific language_ (DSL, linguaggio specifico di dominio) per scrivere circuiti ZKP. Groth16 è un sistema di dimostrazione tra i più diffusi e utilizzati. In termini semplici, un sistema di dimostrazione non è altro che uno dei modi in cui è possibile programmare le ZKP. Esistono anche altri DSL e sistemi di dimostrazione.

Inizieremo installando strumenti e dipendenze. A grandi linee, seguiremo poi questi passaggi:

- Scrivere (scrivere il circuito)
- Costruire (costruire il circuito)
- Setup (trusted setup)
- Dimostrare (generare la prova)
- Verificare (verificare la prova)

Dopo aver seguito una prima volta questo flusso, vedremo quali sono i limiti dell'approccio iniziale. Poi introdurremo miglioramenti graduali fino ad arrivare allo schema di firma presentato sopra. Strada facendo, spiegheremo i concetti e la sintassi necessari.

Alla fine di ogni sezione proporremo anche alcuni esercizi semplici per verificare la comprensione. Questi esercizi sono consigliati. Alla fine dell'articolo troverai inoltre una lista di problemi opzionali, che richiedono molto più impegno.

### Preparazione

Per prima cosa dobbiamo installare alcuni strumenti e dipendenze. Abbiamo preparato un [git repo](https://github.com/oskarth/zkintro-tutorial) per semplificare l'avvio ed evitarti di perderti nei dettagli. Se preferisci non installare alcun software, vedi la fine di questa sezione.

I prerequisiti richiesti sono:

- `rust` (il linguaggio di programmazione)
- `just` (un moderno sostituto di `make`)
- `npm` (gestore di pacchetti per JavaScript)

Gli strumenti ZKP che utilizzeremo sono:

- `circom` (per costruire il nostro programma speciale, ovvero il _circuito_)
- `snarkjs` (per il setup e per generare/verificare prove)
- task di `just` (per semplificare le operazioni comuni legate a quanto sopra)

Per installare tutto e semplificare build ed esecuzione, puoi clonare e usare il [git repo](https://github.com/oskarth/zkintro-tutorial). Questo dovrebbe funzionare su qualsiasi sistema Unix-like come MacOS e Linux. Se usi Windows, ti consigliamo una VM Linux, Windows Subsystem for Linux (WSL) o strumenti simili per lo sviluppo.

```shell
# Clone the repo and run the prepare script
git clone git@github.com:oskarth/zkintro-tutorial.git
cd zkintro-tutorial

# Skim the contents of this file before executing it
less ./scripts/prepare.sh
./scripts/prepare.sh
```

Ti consigliamo di dare un'occhiata al contenuto di `./scripts/prepare.sh` per vedere cosa verrà installato, o se preferisci installare manualmente gli strumenti. Una volta eseguito lo script, dovresti vedere `Installation complete` e nessun errore.

Se incontri difficoltà, consulta la documentazione ufficiale aggiornata [qui](https://docs.circom.io/getting-started/installation/). Al termine, dovresti avere installato le seguenti versioni (o superiori):

```shell
> circom --version
circom compiler 2.1.8

> snarkjs | head -n 1
snarkjs@0.7.4
```

Nel repository c'è un `justfile` che definisce una serie di comandi comuni. Questi comandi di `just` servono a semplificare le operazioni più frequenti sulle ZKP, così puoi concentrarti sul comprendere i concetti alla base di ogni passaggio. Questo riduce notevolmente il rischio di errori, soprattutto all'inizio.

Se in qualsiasi momento vuoi capire nel dettaglio quali comandi vengono eseguiti, ti consigliamo di guardare il `justfile` e i vari script nella cartella `scripts`.

Ti consigliamo vivamente di installare i software sopra indicati per seguire il tutorial e sviluppare un'intuizione pratica. Tuttavia, se non vuoi installare nulla, puoi comunque seguire, seppur con alcune limitazioni, usando uno strumento REPL (Read-Eval-Print Loop) online come [zkrepl.dev](https://zkrepl.dev). Se invece non vuoi installare `just` e preferisci eseguire manualmente tutti i comandi, puoi farlo con un po' di lavoro extra utilizzando gli script shell forniti.

## Prima iterazione

Siamo ora pronti per scrivere del codice. Per arrivare allo schema di firma menzionato in precedenza, cominceremo con un programma molto semplice, l'equivalente di un "Hello World" in altri linguaggi di programmazione.

In termini pratici, scriveremo un programma speciale che ci permetterà di dimostrare di conoscere due numeri segreti il cui prodotto è un numero pubblico, _senza mai rivelare i numeri segreti stessi_. Per esempio, il numero pubblico potrebbe essere "33", mentre i numeri segreti sono "11" e "3". Questo è un passo fondamentale verso la costruzione delle firme digitali e ci aiuterà a sviluppare l'intuizione su come funzionano le ZKP. Se conosci la crittografia a chiave pubblica, puoi pensare grosso modo ai numeri segreti come a una "chiave privata" e al numero pubblico come a una "chiave pubblica".

Dato che si tratta di un approccio alla programmazione piuttosto diverso e ricco di concetti nuovi, non preoccuparti se all'inizio qualcosa non ti è chiaro. Puoi tranquillamente proseguire concentrandoti sul codice, sulla generazione delle prove, ecc., per poi tornare in seguito a rivedere una sezione specifica.

### Scrivere un programma speciale

A differenza della programmazione tradizionale, scrivere questi programmi speciali (i circuiti) è leggermente diverso. Qui ci interessa dimostrare un _insieme di vincoli_ (constraints). [^4] L'insieme più semplice che possiamo dimostrare è composto da un solo vincolo. [^5] In questo caso, vogliamo imporre che il prodotto di due numeri sia uguale a un terzo.

Vai nella cartella `example1` del repository `zkintro-tutorial`. Troverai un programma scheletro in `example1.circom`. Modificalo in questo modo:

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

Questo è il nostro programma speciale, ovvero il nostro _circuito_. [^6] Analizziamolo riga per riga:

- `pragma circom 2.0.0;` - definisce la versione di Circom utilizzata
- `template Multiplier()` - i template sono l'equivalente degli oggetti nella maggior parte dei linguaggi di programmazione, una forma comune di astrazione
- `signal input a;` - il nostro primo input, `a`; gli input sono privati per impostazione predefinita
- `signal input b;` - il secondo input, `b`; anch'esso privato per impostazione predefinita
- `signal output c;` - l'output `c`; gli output sono sempre pubblici
- `c <== a * b;` - fa due cose: assegna al segnale `c` un valore _e_ impone che `c` sia uguale al prodotto di `a` e `b`
- `component main = Multiplier2()` - istanzia il componente principale

La riga più importante è `c <== a * b;`. È qui che dichiariamo effettivamente il nostro vincolo. Questa espressione è in realtà la combinazione di due operazioni: `<--` (assegnazione) e `===` (vincolo di uguaglianza). [^7] Un vincolo in Circom può usare solo operazioni con costanti, addizioni o moltiplicazioni. In pratica impone che entrambi i lati dell'equazione siano uguali. [^8]

### Sui vincoli

Come funzionano i vincoli? In un contesto come quello del Sudoku, potremmo dire che un vincolo è "un numero compreso tra 1 e 9". In Circom, però, questo non è un singolo vincolo, ma qualcosa che dobbiamo esprimere usando un insieme di vincoli di uguaglianza (`===`) più semplici. [^9]

Perché accade questo? La ragione è matematica. La maggior parte delle ZKP si basa su _circuiti aritmetici_, che rappresentano calcoli su _polinomi_. Quando si lavora con polinomi, è facile introdurre costanti, sommarle, moltiplicarle e verificare se due risultati sono uguali tra loro. [^10] Tutte le altre operazioni devono essere espresse in termini di queste operazioni fondamentali. Non è necessario comprendere tutto questo nei dettagli per scrivere ZKP, ma avere un minimo di intuizione su cosa succede "sotto il cofano" può essere molto utile. [^11]

Possiamo visualizzare il circuito in questo modo:

![esempio1 circuito](../assets/02_example1_circuit.png 'example1 circuit')

### Compilare il nostro circuito

Per riferimento, il file finale è disponibile in `example1-solution.circom`. Per maggiori dettagli sulla sintassi, consulta la [documentazione ufficiale](https://docs.circom.io/circom-language/signals/).

Possiamo compilare il circuito eseguendo:

```shell
just build example1
```

![example1 build](../assets/02_example1_build.png 'example1 build')

Questo comando è un piccolo wrapper che si limita a richiamare `circom` per creare i file `example1.r1cs` e `example1.wasm`. Dovresti vedere un output simile a questo:

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

In questo caso, abbiamo:

- due input privati, `a` e `b`
- un output pubblico, `c`
- un vincolo (non lineare), `c <== a * b`

Per ora possiamo ignorare le altre parti dell'output. [^12] Ora abbiamo due file: `example1.r1cs` e `example1.wasm`.

`r1cs` sta per _Rank 1 Constraint System_ (sistema di vincoli di rango 1, R1CS). Questo file contiene il circuito in forma binaria e rappresenta il modo in cui i nostri vincoli vengono definiti matematicamente. [^13]

Il file `.wasm` contiene WASM (WebAssembly), necessario per generare il nostro _witness_ (testimone). Il witness è ciò che ci permette di specificare gli input che vogliamo mantenere privati, pur usandoli per creare una prova.

Detto ciò, non siamo ancora pronti per generare le prove. Prima dobbiamo eseguire un _setup_ per ottenere la proving key e la verification key.

Non preoccuparti se tutto questo non ti è ancora chiaro. È un nuovo modo di lavorare e richiede un po' di tempo per abituarsi.

### Trusted setup

Con gli artefatti generati in precedenza possiamo eseguire un _trusted setup_ (setup fidato).

Un trusted setup è un'operazione che eseguiamo una sola volta come fase di pre-elaborazione. Genera quello che è chiamato _Common Reference String_ (CRS), che consiste in una _proving key_ (chiave di dimostrazione) e una _verification key_ (chiave di verifica). Queste chiavi possono poi essere utilizzate ogni volta che vogliamo generare e verificare prove.

![Trusted setup](../assets/02_example1_setup1.png 'Trusted setup')

Perché abbiamo bisogno di queste chiavi e chi dovrebbe avervi accesso? La proving key incorpora tutte le informazioni necessarie per generare una prova a conoscenza zero per quel circuito specifico. Analogamente, la verification key incorpora tutte le informazioni necessarie per verificare che la prova sia effettivamente corretta. Non si tratta di chiavi private, ma di informazioni che possono e devono essere distribuite pubblicamente. Chiunque debba generare o verificare una prova dovrebbe potervi accedere. [^14]

Perché lo chiamiamo trusted setup? Eseguire un setup è un processo che coinvolge più partecipanti ed è talvolta chiamato _cerimonia_. [^15] Tutti i partecipanti cooperano per creare un "segreto" crittografico, che è alla base della costruzione delle proving key e verification key. Se questo processo viene manipolato, crittograficamente potrebbe essere possibile creare prove false o dichiarare valide prove non corrette. Di conseguenza, si presuppone un certo grado di fiducia: almeno alcuni dei partecipanti devono essere onesti nel processo di setup, da cui deriva il termine "trusted setup".

Come punto di partenza, eseguiremo il trusted setup da soli. Esegui il seguente comando:

`just trusted_setup example1`

![example1 trusted setup](../assets/02_example1_setup2.png 'example1 trusted setup')

Ti verrà chiesto di fornire del testo casuale (entropia) per due volte. [^16] Al termine dovresti vedere "Trusted setup completed." e la posizione delle chiavi. Il file con estensione `.zkey` è la nostra proving key. Sebbene approfondire i dettagli dei trusted setup esuli dall'obiettivo di questo articolo, ci sono alcune cose utili da sapere.

Prima di tutto: qual è il problema dell'approccio appena usato? Avendo un solo partecipante, chiunque altro utilizzi il materiale crittografico generato in quel setup si fida di quell'individuo e del suo ambiente informatico. Questo non funzionerebbe in scenari di produzione, dove vorremmo massimizzare il numero di partecipanti per rendere il setup più affidabile. Se partecipano 100 persone, grazie al modo in cui viene costruito questo segreto crittografico, è sufficiente che una singola persona sia onesta. [^17]

Vale anche la pena sapere che diversi sistemi di dimostrazione ZKP hanno proprietà diverse in termini di sicurezza, prestazioni e funzionalità. Sebbene tutti i sistemi ZKP richiedano una qualche forma di setup, non tutti richiedono un trusted setup. Tra quelli che lo richiedono, alcuni differiscono nei requisiti.

Con Circom utilizziamo il _sistema di dimostrazione Groth16_, che richiede un trusted setup. In particolare, il setup è suddiviso in due fasi: fase 1 e fase 2. La fase 1 è indipendente dal circuito e può essere utilizzata da qualsiasi programma ZKP fino a una certa dimensione, mentre la fase 2 è _specifica del circuito_. Quando abbiamo eseguito il comando precedente, abbiamo eseguito entrambe le fasi.

Potresti chiederti: perché usare un trusted setup quando si può evitarlo? Molti condividono questa visione. Eppure esistono buone ragioni per cui si ricorre ancora a questi sistemi,come ad esempio strumenti e un ecosistema più maturi, nonché costi di verifica ridotti. Questi ultimi sono tradizionalmente molto importanti, specialmente quando si verificano prove su una blockchain pubblica come Ethereum. A seconda del caso d'uso, la scelta sarà probabilmente diversa. In un altro articolo approfondiremo i trusted setup e i loro compromessi, oltre ai diversi sistemi di dimostrazione.

### Generare la prova

Con il trusted setup completato in precedenza, disponiamo di una proving key e di una verification key. Possiamo ora generare una prova che dimostra di conoscere due valori segreti il cui prodotto è un altro numero pubblico.

In concreto, dimostreremo che 33 può essere ottenuto moltiplicando 3 per 11. Ricorda che i nostri input privati sono i segnali `a` e `b`. Li specifichiamo nel file `example1/input.json` come segue:

```json
{
  "a": "3",
  "b": "11"
}
```

Cioè, specifichiamo l'input come una mappa JSON, dove la chiave è il nome del segnale e il valore è quello che vogliamo assegnargli. Nota che il valore è una stringa, anche se concettualmente è un numero. Questa è una particolarità di Circom e della sua API JavaScript. Per via della natura delle ZKP, spesso si lavora con numeri molto grandi che richiedono l'uso di _BigInt_. Il modo più semplice per specificare un numero così grande in un file JSON è come stringa, che verrà poi convertita in BigInt.

Possiamo creare una prova usando il nostro circuito compilato (in forma WASM), la nostra proving key e l'input, eseguendo:

`just generate_proof example1`

![example1 generate proof](../assets/02_example1_generate_proof.png 'example1 generate proof')

Sotto il cofano, questo comando prende l'input e genera un _witness_ per il nostro specifico circuito. [^18] Normalmente, con witness intendiamo semplicemente l'input privato che usiamo per generare una prova. Nel contesto di Circom, il witness è l'assegnazione completa di tutti i segnali, sia privati che pubblici, in una forma che il software prover può elaborare. Questa forma è una rappresentazione interna in formato binario. [^19]

Con questo witness generato, possiamo creare una prova usando `snarkjs`. Al termine, otteniamo una prova e un output pubblico.

La prova ha un aspetto simile a questo:

```json
{
  "pi_a": ["15932[...]3948", "66284[...]7222", "1"],
  "pi_b": [
    ["17667[...]0525", "13094[...]1600"],
    ["12020[...]5738", "10182[...]7650"],
    ["1", "0"]
  ],
  "pi_c": ["18501[...]3969", "13175[...]3552", "1"],
  "protocol": "groth16",
  "curve": "bn128"
}
```

Questa specifica la prova sotto forma di oggetti matematici (tre elementi di curve ellittiche), `pi_a`, `pi_b` e `pi_c`. [^20] Include anche alcuni metadati sul protocollo (`groth16`) e sulla _curva_ (`bn128`, un dettaglio di implementazione matematica che ignoreremo per ora). Questo permette al verifier di sapere come trattare questa prova per verificarla correttamente.

Nota quanto sia breve la prova; indipendentemente da quanto sia complesso il nostro programma speciale, la prova avrà sempre e solo questa dimensione. Questo illustra la proprietà di _sinteticità_ delle ZKP di cui abbiamo parlato nella nostra [_introduzione semplificata_](https://zkintro.com/it/articles/friendly-introduction-to-zero-knowledge#compression).

Il comando precedente produce anche il nostro _output pubblico_:

```json
["33"]
```

Questo è un elenco di tutti gli output pubblici corrispondenti al nostro witness e circuito. In questo caso, c'è un unico output pubblico che corrisponde a `c`: 33. [^21]

Cosa abbiamo dimostrato? Che conosciamo due valori segreti, `a` e `b`, il cui prodotto è 33. Questo illustra la proprietà di _privacy_ di cui abbiamo parlato nell'articolo precedente.

Nota che la prova da sola non è utile: richiede l'output pubblico che la accompagna.

### Verificare la prova

Passiamo ora a verificare questa prova. Esegui:

`just verify_proof example1`

![example1 verify proof](../assets/02_example1_verify_proof.png 'example1 verify proof')

Questo comando prende la verification key, l'output pubblico e la prova. Con questi elementi siamo in grado di verificare la prova. Dovrebbe stampare "Proof verified". Nota come il verifier non abbia mai accesso agli input privati.

Cosa succede se modifichiamo l'output? Apri `example1/target/public.json` e cambia il 33 in 34, poi riesegui il comando precedente.

Noterai che la prova non viene più verificata. Questo perché la nostra prova non dimostra che abbiamo due numeri il cui prodotto è 34.

Complimenti, hai scritto il tuo primo programma ZKP, eseguito un trusted setup, generato una prova e infine verificata!

### Esercizi

1. Quali sono le due proprietà fondamentali delle ZKP e cosa significano?
2. Qual è il ruolo del prover e di quale input ha bisogno? E il verifier?
3. Spiega cosa fa la riga `c <== a * b;`.
4. Perché dobbiamo eseguire un trusted setup? Come usiamo i suoi artefatti?
5. Codice: Completa `example1` fino a generare e verificare una prova.

## Seconda iterazione

Con il circuito precedente stiamo dimostrando di conoscere il prodotto di due numeri (segreti). Questo problema è strettamente legato alla _fattorizzazione in numeri primi_, che è alla base di molta parte della crittografia. [^22] L'idea è che, dato un numero molto grande, sia difficile trovare due numeri primi il cui prodotto sia quel numero. D'altra parte, verificare se il prodotto di due numeri è uguale a un altro numero è estremamente semplice. [^23]

Tuttavia, c'è un problema importante nel nostro circuito. Riesci a individuarlo?

Potremmo facilmente cambiare il nostro input in "1" e "33". In altre parole, un numero `c` è sempre il prodotto di 1 e `c`. Non è certo qualcosa di particolarmente notevole, vero?

Quello che vogliamo fare è aggiungere un altro _vincolo_ (constraint): né `a` né `b` possono essere uguali a 1. In questo modo siamo costretti a fare una vera fattorizzazione intera.

Come possiamo aggiungere questo vincolo e quali modifiche dobbiamo apportare?

### Aggiornare il nostro circuito

Lavoreremo con la cartella `example2` per queste modifiche. Purtroppo non possiamo semplicemente scrivere `a !== 1`, perché non si tratta di un vincolo valido. [^24] Non è composto esclusivamente da costanti, addizioni, moltiplicazioni e verifiche di uguaglianza. Come esprimiamo allora il concetto di "qualcosa che non è"?

Questo non è immediatamente intuitivo, ed è in questo tipo di problema che emerge gran parte dell'arte di scrivere circuiti. Sviluppare questa competenza richiede tempo e va oltre l'obiettivo di questo tutorial iniziale; fortunatamente esistono molte buone risorse a riguardo. [^25]

Esistono però alcuni costrutti tipici ricorrenti. L'idea di fondo è usare un template `IsZero()` che verifica se un'espressione è uguale a zero oppure no. Restituisce 1 se vera, 0 se falsa.

Spesso è utile ricorrere a una tabella di verità [^26] per mostrare i possibili valori. Ecco la tabella di verità per `IsZero()`:

| in  | out |
| --- | --- |
| 0   | 1   |
| n   | 0   |

Si tratta di un elemento base così utile da essere incluso nella libreria di Circom, `circomlib`. In `circomlib` sono presenti anche molti altri componenti utili. [^27]

Possiamo includerlo creando un progetto `npm` (JavaScript) e aggiungendolo come dipendenza. Nella cartella `example2` lo abbiamo già fatto per te. Per importare il modulo rilevante, aggiungiamo la seguente riga all'inizio di `example2.circom`:

`include "circomlib/circuits/comparators.circom";`

Usando `IsZero()`, possiamo verificare se `a` o `b` è uguale a 1. Modifica il file `example2.circom` in modo che contenga le seguenti righe:

```javascript
component isZeroCheck = IsZero();
isZeroCheck.in <== (a - 1) * (b - 1);
isZeroCheck.out === 0;
```

Nel frammento di codice qui sopra, creiamo un nuovo component `isZeroCheck`, istanziando il template `IsZero()`. Se `a` o `b` è uguale a 1, a `isZeroCheck.in` verrà assegnato 0 e `isZeroCheck.out` varrà 1. Poiché abbiamo un vincolo che afferma `isZeroCheck.out === 0`, questo vincolo fallirà. Ciò significa che non sarà più possibile fornire input in cui `a` o `b` sia uguale a 1.

Ti invito a convincerti, mentalmente o con carta e penna (magari usando una tabella di verità?), che sia effettivamente così. Se vuoi una sfida, puoi provare a capire come è implementato `IsZero()`: è composto da poche righe di codice. Puoi vedere il codice nel file `comparators.circom` di `circomlib`. [^28]

Per riferimento, il file finale è disponibile in `example2-solution.circom`. Con le modifiche di cui sopra, possiamo installare la dipendenza npm `circomlib` e compilare il circuito con:

`just build example2`

### Ripetere il trusted setup

Con Circom e Groth16, ogni volta che modifichiamo il nostro circuito dobbiamo ripetere il trusted setup. Questo significa che faresti meglio ad accertarti che il circuito sia solido prima di rilasciarlo, soprattutto se si organizza una cerimonia vera e propria con molti partecipanti.

Più precisamente, dobbiamo ripetere solo il trusted setup specifico per il circuito (fase 2). Questo perché la fase 1 è generica per _qualsiasi_ circuito Groth16 scritto in Circom, fino a una certa dimensione. Quando abbiamo eseguito il trusted setup in precedenza, abbiamo svolto sia la fase 1 sia la fase 2, omettendo i dettagli della fase 1 per semplicità. Ecco alcune informazioni aggiuntive sulla fase 1 per dare un quadro più completo.

![Trusted setup (entrambe le fasi)](../assets/02_example2_setup_both.png 'Trusted setup (entrambe le fasi)')

Il risultato del trusted setup di fase 1 è contenuto in un file `.ptau`, dove ptau sta per _powers of tau_ (potenze di tau). [^29] Dal punto di vista matematico, questo file contiene potenze di alcuni segreti casuali, ed è ciò che ci permette di "accomodare" un certo numero di vincoli. Non è necessario capire come funziona matematicamente, ma ci sono due fatti chiave utili da sapere: (a) il file `.ptau` è indipendente dal circuito; (b) la sua dimensione ne indica la capacità. La "capacità" di un dato file ptau è di `2^n - 1` vincoli, dove `n` è un certo numero. Ad esempio, `pot12.ptau` indica che il numero di vincoli che può accogliere è `2^12 - 1`, ovvero poco più di 4000 vincoli.

Poiché non vogliamo ripetere la fase 1, eseguiamo solo la fase 2. Questa utilizzerà come input il file `pot12.ptau` precedentemente generato (archiviato nella cartella `ptau`). Possiamo eseguire il trusted setup di fase 2 con:

```
just trusted_setup_phase2 example2
```

![example2 trusted setup](../assets/02_example2_setup2.png 'example2 trusted setup')

### Testare le nostre modifiche

A questo punto possiamo eseguire:

```
just generate_proof example2
just verify_proof example2
```

La prova viene ancora generata e verificata come previsto.

Se modifichiamo gli input in `example2/input.json` inserendo `1` e `33` e proviamo a eseguire i comandi precedenti, otterremo un errore di asserzione. In altre parole, Circom non ci permetterà nemmeno di generare una prova, perché l'input viola i nostri vincoli.

### Diagramma di flusso completo

Ora che abbiamo percorso l'intero flusso due volte, facciamo un passo indietro e vediamo come tutti i pezzi si incastrano tra loro.

![example2 flusso completo](../assets/02_example2_complete_flow.png 'example2 flusso completo')

A questo punto le cose dovrebbero cominciare a essere più chiare. Con questo, alziamo il livello e rendiamo il nostro circuito più utile.

### Esercizi

6. Perché dobbiamo eseguire la fase 2 ma non la fase 1 del trusted setup per `example2`?
7. Qual era il problema principale dell'esempio precedente e come lo abbiamo risolto?
8. Codice: Completa `example2` fino al punto in cui non riesci più a generare una prova.

## Terza iterazione

Con il circuito precedente abbiamo dimostrato di conoscere il prodotto di due valori segreti. Di per sé, questo non è molto utile. Qualcosa che invece ha un'applicazione concreta nel mondo reale è uno _schema di firma digitale_. Con esso, puoi dimostrare a qualcun altro di aver scritto un messaggio specifico. Come potremmo implementarlo usando le ZKP? Per arrivarci, dobbiamo prima introdurre alcuni concetti di base.

Questo è un buon momento per una breve pausa e una tazza della tua bevanda preferita.

### Firme digitali

Le firme digitali esistono già e sono ovunque nell'era digitale. L'Internet moderno non potrebbe funzionare senza di esse. Di norma, vengono implementate usando la _crittografia a chiave pubblica_. In questo sistema si dispone di una chiave privata e di una chiave pubblica. La chiave privata è riservata esclusivamente a te, mentre la chiave pubblica è condivisa apertamente e rappresenta la tua identità.

Uno schema di firma digitale si compone delle seguenti parti:

- **Generazione delle chiavi**: generare una chiave privata e la corrispondente chiave pubblica
- **Firma**: creare una firma utilizzando la chiave privata e il messaggio
- **Verifica della firma**: verificare che il messaggio sia stato firmato dalla chiave pubblica corrispondente

Pur differendo nei dettagli, il programma che abbiamo scritto e l'algoritmo di generazione delle chiavi descritto sopra condividono un elemento comune: entrambi utilizzano una _funzione unidirezionale_, e più precisamente una _trapdoor function_ (funzione botola). Una botola è qualcosa in cui è facile cadere ma da cui è difficile uscire (a meno che non si trovi una scala nascosta). [^30]

![example3 trapdoor](../assets/02_example3_trapdoor.png 'example3 trapdoor')

Nella crittografia a chiave pubblica, è semplice ricavare la chiave pubblica da quella privata, ma è molto difficile fare il percorso inverso. Lo stesso vale per il programma che abbiamo scritto in precedenza: se i due numeri segreti sono numeri primi molto grandi, è estremamente difficile risalire ai valori originali a partire dal loro prodotto. La crittografia a chiave pubblica moderna spesso utilizza la _crittografia su curve ellittiche_ come meccanismo sottostante.

Tradizionalmente, creare protocolli crittografici come questi schemi di firma digitale richiede un notevole sforzo e implica la progettazione di un protocollo specifico basato su matematica ingegnosa. Non è quello che vogliamo fare. Vogliamo invece scrivere un programma usando le ZKP che ottenga lo stesso risultato.

Invece di questo: [^31]

![Signature verification](../assets/02_example3_sigverify.png 'Signature verification')

Vogliamo semplicemente scrivere un programma, generare una prova di ciò che intendiamo dimostrare, e poi verificare questa prova.

### Funzioni di hash e commitment

Invece di ricorrere alla crittografia su curve ellittiche, utilizzeremo due strumenti molto più semplici: le _funzioni di hash_ e i _commitment_ (impegni crittografici).

Una funzione di hash è anch'essa una funzione unidirezionale. Ad esempio, dalla riga di comando possiamo usare la funzione di hash SHA-256 in questo modo:

```shell
echo -n "foo" | shasum -a 256
```

Per produrre l'hash di "foo": `0beec7[...]a8a33` (abbreviato). [^32]

Di per sé, una funzione di hash non è una trapdoor function. Non esiste alcuna conoscenza speciale che permetta di risalire al valore originale. Si comporta più come un tritacarne che come una botola con una scala nascosta.

E i commitment? Un _commitment_ è semplicemente un modo per impegnarsi ("promettere") riguardo a un valore segreto, in modo da non poterlo cambiare in seguito. Nel nostro caso, utilizzeremo un commitment per generare l'equivalente di una chiave pubblica a partire da un valore segreto. Possiamo farlo usando una funzione di hash.

Gli schemi di commitment sono una primitiva crittografica molto comune. [^33] Ci consentono di:

- **commit**: impegnarsi su un valore specifico mantenendolo nascosto
- **reveal**: rivelare quel valore in modo che possa essere verificato

Questo ci offre due proprietà fondamentali:

- **hiding** (occultamento): il valore rimane nascosto
- **binding** (vincolante): non è possibile cambiare idea sul valore

Un modo per immaginare un commitment è pensare di dare a un amico una cassetta chiusa a lucchetto. Non puoi modificare il contenuto della cassetta una volta chiusa, ma il tuo amico non può guardare dentro. Solo quando gli consegni la chiave potrà aprirla.

![example3 lockbox](../assets/02_example3_lockbox.png 'example3 lockbox')

Tornando al nostro schema di firma digitale, abbiamo:

- **Generazione delle chiavi**: creare una stringa segreta e applicarle una funzione di hash per ottenere un commitment
- **Firma**: creare una firma applicando una funzione di hash al segreto insieme al messaggio
- **Verifica**: verificare la prova usando il commitment, il messaggio e la firma (output pubblico)

In pseudo-codice, ecco cosa vogliamo fare nel nostro circuito:

```python
commitment = hash(some_secret)
signature = hash(some_secret, message)
```

A questo punto probabilmente avrai qualche domanda. Proviamo a rispondere alle più probabili.

Prima di tutto, perché funziona e perché abbiamo bisogno di una ZKP per questo? Quando qualcuno verifica la prova, ha accesso solo al commitment, al messaggio e alla firma. Non esiste un modo diretto per verificare che il commitment corrisponda al segreto senza rivelare il segreto stesso. In questo caso, "riveliamo" il segreto solo al momento della generazione della prova, così il segreto resta al sicuro.

In secondo luogo, perché usare queste funzioni di hash e commitment invece della crittografia a chiave pubblica all'interno della ZKP? Si potrebbe assolutamente fare crittografia a chiave pubblica all'interno di una ZKP, e ci sono ragioni valide per farlo. Tuttavia, è molto più costosa in termini di vincoli rispetto all'approccio descritto sopra, il che la rende molto più lenta rispetto a questo schema più semplice. Come vedremo nella sezione successiva, la scelta della funzione di hash si rivela molto importante.

Infine, perché usare una ZKP se abbiamo già la crittografia a chiave pubblica? In questo semplice esempio, una ZKP non è strettamente necessaria. Tuttavia, costituisce un elemento fondamentale per applicazioni più interessanti, come l'esempio della firma di gruppo menzionato all'inizio di questo articolo. Dopotutto, il nostro obiettivo è _programmare la crittografia_.

È stato molto! Per fortuna, abbiamo superato l'ostacolo principale. Passiamo al codice. Non preoccuparti se non tutto quanto detto sopra ti è risultato subito chiaro: ci vuole un po' di tempo per abituarsi a questo tipo di ragionamento.

### Torniamo al codice

Lavoreremo dalla directory `example3`.

Per implementare le firme digitali, la prima cosa da fare è generare le nostre chiavi. Queste corrispondono alla chiave privata e alla chiave pubblica della crittografia a chiave pubblica. Poiché le chiavi corrispondono a un'identità (quella del prover), le chiameremo rispettivamente `identity_secret` e `identity_commitment`. Insieme formano una coppia di identità.

Queste verranno usate come input del circuito, insieme al messaggio che stiamo firmando. Come output pubblico avremo la firma, il commitment e il messaggio. Questo permetterà a chiunque di verificare che la firma sia effettivamente corretta.

Poiché abbiamo bisogno della coppia di identità come input per il circuito, la generiamo separatamente:

`just generate_identity`

Il risultato sarà simile a questo:

```shell
identity_secret: 43047[...]2270
identity_commitment: 21618[...]0684
```

Per mantenere il segreto al sicuro, utilizziamo un numero grande e casuale. A differenza di quanto visto in precedenza, non stiamo usando una funzione di hash come SHA-256 per creare il commitment. Utilizziamo invece quella che viene chiamata una _funzione di hash ZK-friendly_. Si tratta di una funzione di hash speciale ottimizzata per l'uso nelle ZKP. Questo incide notevolmente sulle prestazioni quando si eseguono molte operazioni di hashing. La funzione di hash ZK-friendly che utilizziamo si chiama _funzione di hash Poseidon_. [^34]

Internamente, questa operazione usa la libreria `circomlibjs` come _wrapper_ (livello di interfaccia) di `circomlib`. Si tratta di una libreria JavaScript che ci permette di usare i circuiti Circom. Questo garantisce che il nostro `identity_commitment` venga generato esattamente nello stesso modo in JavaScript/dalla riga di comando e nel nostro circuito. Se vuoi leggere il codice sorgente dello script, lo trovi in `example3/generate_identity.js`.

Così come abbiamo fatto con `IsZero`, dobbiamo includere il template Poseidon. Lo facciamo con il seguente include:

```
include "circomlib/circuits/poseidon.circom";
```

Il template per la funzione di hash Poseidon si usa nel modo seguente:

```javascript
component hasher = Poseidon(2);
hasher.inputs[0] = foo;
hasher.inputs[1] = bar;
quux <== hasher.out
```

Specifichiamo che il component `hasher` si aspetta due argomenti, indicati nell'array `.inputs[]`. Il risultato viene poi assegnato al segnale di output `.out`. In questo esempio, il component prende `foo` e `bar` come input, li combina tramite hash e il risultato è `quux`. [^35]

Infine, introduciamo un nuovo elemento di sintassi:

```javascript
component main {public [identity_commitment, message]} = SignMessage();
```

Per impostazione predefinita, tutti gli input del nostro circuito sono privati. Con questa istruzione, contrassegniamo esplicitamente `identity_commitment` e `message` come pubblici. Ciò significa che faranno parte dell'output pubblico.

Con queste informazioni dovresti avere tutto il necessario per completare il circuito `example3.circom`. Se sei ancora bloccato, puoi consultare `example3-solution.circom` per il codice completo.

Come in precedenza, dobbiamo compilare il circuito ed eseguire la fase 2 del trusted setup:

```shell
just build example3
just trusted_setup_phase2 example3
```

Durante la compilazione del circuito, noterai come il numero di vincoli sia aumentato notevolmente rispetto a `example2`. Questo è dovuto principalmente all'uso di due funzioni di hash Poseidon. [^36]

### Testare il nostro circuito

Per riferimento, ecco un'illustrazione del circuito completato:

![example3 circuit](../assets/02_example3_circuit.png 'example3 circuit')

Possiamo ora generare una prova. Abbiamo il seguente input in `example3/input.json`:

```json
{
  "identity_secret": "21879[...]1709",
  "identity_commitment": "48269[...]7915",
  "message": "42"
}
```

Puoi tranquillamente sostituire la coppia di identità con quella che hai generato tu stesso con `just generate_identity`. Dopotutto, vuoi tenere l'identity secret per te!

Potresti notare che il messaggio è semplicemente un numero racchiuso tra virgolette come stringa (`"42"`). Purtroppo, a causa di come funzionano matematicamente i vincoli (usando l'algebra lineare e i _circuiti aritmetici_) possiamo usare solo numeri e non stringhe. Le uniche operazioni supportate all'interno dei circuiti sono quelle aritmetiche di base, come addizione e moltiplicazione. [^37]

Possiamo ora generare e verificare una prova:

```
just generate_proof example3
just verify_proof example3
```

Come in precedenza, la prova mantiene la stessa dimensione, anche se stiamo facendo molte più operazioni. L'output pubblico che si trova in `example3/target/public.json` è:

```json
["48968[...]5499", "48269[...]7915", "42"]
```

Corrisponde rispettivamente alla firma, al commitment e al messaggio.

Vediamo ora cosa può andare storto se non siamo attenti. [^38]

Prima di tutto, cosa succede se cambiamo l'identity commitment con uno casuale nell'`input.json`? Noterai che non saremo più in grado di generare prove. Questo perché nel circuito stesso stiamo verificando anche la relazione tra l'identity commitment e l'identity secret. È fondamentale che questa relazione venga mantenuta.

In secondo luogo, cosa succede se non includiamo il messaggio nell'output? Otterremo comunque una prova, e questa verrà verificata. Ma il messaggio potrebbe essere _qualsiasi cosa_, quindi non dimostra davvero che tu abbia inviato un messaggio specifico. Analogamente, cosa succederebbe se non includessimo l'identity commitment nell'output pubblico? Significherebbe che l'identity commitment potrebbe essere qualunque cosa, e quindi non sapremmo _chi_ ha firmato il messaggio.

Come esercizio di riflessione, convinciti (o verifica) di cosa potrebbe andare storto se omettessimo uno di questi due vincoli fondamentali:

- `identity_commitment === identityHasher.out`
- `signature <== signatureHasher.out`

Complimenti, ora sai come programmare la crittografia! [^39]

### Esercizi

9. Quali sono i tre componenti di uno schema di firma digitale?
10. Qual è lo scopo di usare una funzione di hash ZK-friendly come Poseidon?
11. Cosa sono i commitment? Come possiamo usarli per uno schema di firma digitale?
12. Perché contrassegniamo l'identity commitment e il messaggio come pubblici?
13. Perché abbiamo bisogno dei vincoli sull'identity commitment e sulla firma?
14. Codice: completa `example3` fino a generare e verificare una prova.

## Prossimi passi

Con lo schema di firma digitale descritto sopra, e i trucchi visti in precedenza nell'articolo, hai a disposizione tutti gli strumenti per implementare lo _schema di firma di gruppo_ (group signature) menzionato all'inizio dell'articolo. [^40]

Il codice scheletro si trova in `example4`. Servono solo 5-10 righe di codice. L'unica sintassi nuova è il ciclo `for`, che funziona come nella maggior parte degli altri linguaggi.[^41]

Questo circuito ti permetterà di:

- firmare un messaggio
- dimostrare di essere una di tre persone (commitment di identità)
- senza rivelare quale

Puoi considerarlo un rompicapo. L'intuizione chiave si riduce essenzialmente a una singola espressione aritmetica. Prova a risolverla su carta, se puoi. Se ti blocchi, puoi consultare la soluzione come negli esercizi precedenti.

Infine, se vuoi qualche sfida extra, ecco alcuni modi per estenderlo:

1. Consentire un numero arbitrario di persone nel gruppo
2. Implementare un nuovo circuito `reveal` che dimostri di aver firmato un messaggio specifico
3. Implementare un nuovo circuito `deny` che dimostri di non aver firmato un messaggio specifico

Creare un protocollo crittografico del genere con strumenti classici sarebbe un compito enorme, che richiederebbe molte competenze specialistiche. [^42] Con le ZKP puoi diventare produttivo e pericolosamente capace in un pomeriggio, affrontando questi problemi come compiti di programmazione. E questa è solo la punta dell'iceberg di ciò che possiamo fare.

### Esercizi

15. Cosa fanno le firme di gruppo rispetto alle firme normali? Come possono essere utilizzate?

## Problemi

Questi problemi sono opzionali e richiedono molto più sforzo.

1. Scopri come è implementato `IsZero()`.
2. Codice: completa lo schema di firma di gruppo qui sopra (vedi `example4`).
3. Codice: estendi l'esempio di firma di gruppo qui sopra: consenti più partecipanti e implementa i circuiti `reveal` e/o `deny`.
4. Come progetteresti un sistema di "ZK Identity" per dimostrare di avere più di 18 anni? Quali altre proprietà potresti voler dimostrare? Ad alto livello, come lo implementeresti e quali sfide intravedi? Studia le soluzioni esistenti per capire meglio come vengono realizzate.
5. Per le blockchain pubbliche come Ethereum, a volte si ricorre a un _Layer 2_ (L2) per consentire transazioni più veloci, economiche e numerose. Ad alto livello, come progetteresti un L2 utilizzando le ZKP? Descrivi alcune sfide che intravedi. Studia le soluzioni esistenti per capire meglio come vengono realizzate.

## Conclusione

In questa introduzione pratica, abbiamo preso confidenza con la scrittura e la modifica di ZKP di base partendo da zero. Abbiamo configurato l'ambiente di programmazione e scritto un circuito di base. Abbiamo poi eseguito un trusted setup, creato e verificato le prove. Abbiamo individuato alcuni problemi e migliorato il nostro circuito, assicurandoci di testare le modifiche. Successivamente, abbiamo implementato uno schema di firma digitale di base usando funzioni di hash e commitment.

Abbiamo inoltre acquisito le competenze e imparato a usare gli strumenti necessari per implementare le firme di gruppo (group signatures), qualcosa che sarebbe difficile da realizzare senza le ZKP.

Spero che tu abbia sviluppato un miglior modello mentale di ciò che comporta la scrittura di ZKP, e che tu abbia un'idea più chiara di come si presenta in pratica il ciclo modifica-esecuzione-debug. Tutto questo costituirà una buona base per qualsiasi altro programma ZKP tu voglia scrivere in futuro, indipendentemente dallo stack tecnologico che utilizzerai.

## Ringraziamenti

Grazie a Hanno Cornelius, Marc Köhlbrugge, Michelle Lai, lenilsonjr e Chih-Cheng Liang per aver letto le bozze e aver fornito preziosi suggerimenti.

### Immagini

- _Congresso Bourbaki 1938_ - Autore sconosciuto, Pubblico dominio, tramite [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Bourbaki_congress1938.png)
- _Zebre di Hartmann_ - J. Huber, CC BY-SA 2.0, tramite [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Hartmann_zebras_hobatereS.jpg)
- _Ragno della botola_ - P.S. Foresman, Pubblico dominio, tramite [Wikimedia Commons](<https://commons.wikimedia.org/wiki/File:Trapdoor_(PSF).png>)
- _Cassetta di sicurezza Kingsley_ - P.S. Foresman, Pubblico dominio, tramite [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Kingsley_lockbox.jpg)

## Riferimenti

[^1]: Sebbene sia illustrativa come metafora, quella descritta è solo una delle diverse teorie. Se sei curioso, consulta https://en.wikipedia.org/wiki/Zebra#Function.
[^2]: Vedi [Federalist Papers (Wikipedia)](https://en.wikipedia.org/wiki/The_Federalist_Papers#Authorship).
[^3]: Vedi [Bourbaki (Wikipedia)](https://en.wikipedia.org/wiki/Nicolas_Bourbaki#Membership).
[^4]: A meno che tu non abbia già avuto esperienza con qualche forma di programmazione dichiarativa (cioè non procedurale, come Prolog), questa è probabilmente un'idea nuova. In un certo senso la ritroviamo anche in SQL. Descriviamo _cosa_ vogliamo, non necessariamente _come_ vogliamo ottenerlo.
[^5]: Tecnicamente, anche zero vincoli costituiscono un insieme di vincoli. Detto per scherzo, ma i circuiti sotto-vincolati sono un problema grave che può portare a bug seri. Ne vedremo un esempio più avanti.
[^6]: Lo chiamiamo _circuito_, o più precisamente _circuito aritmetico_, perché collega ingressi e uscite in modo simile ai gate logici come NAND, AND, NOT, XOR, ecc. Da questi è possibile costruire un computer universale, o circuito universale.
[^7]: In generale, l'uso di `<--` non è raccomandato: è quasi sempre preferibile utilizzare `<==` al suo posto.
[^8]: Questo rende la scrittura dei vincoli piuttosto impegnativa, come si può immaginare. Vedi https://docs.circom.io/circom-language/constraint-generation/ per maggiori dettagli sui vincoli in Circom.
[^9]: Per affermare "questo numero è compreso tra 1 e 9" dobbiamo implementare un _range check_ (verifica di intervallo). Ciò implica decomporre il numero in bit ed eseguire verifiche di uguaglianza su di essi. Per fortuna, molti di questi vincoli tipici sono già stati scritti e possono essere riutilizzati, come vedremo più avanti con _circomlib_.
[^10]: Ad esempio, `p(x) = ax^2 + bx + c` può facilmente essere sommato, moltiplicato o confrontato con `q(x) = dx^2 + 2bx + e`. Vale la pena notare che nelle ZKP si opera su campi finiti, non su numeri reali. Questo argomento esula dall'ambito di questo articolo.
[^11]: Sebbene la maggior parte delle ZKP utilizzi _circuiti aritmetici_, esistono altri sistemi di dimostrazione basati su astrazioni diverse. Ad esempio, zkSTARKs e Bulletproofs.
[^12]: Un vincolo lineare è un vincolo che può essere espresso come combinazione lineare usando solo l'addizione. Ciò equivale a usare moltiplicazioni per costanti. La cosa principale da tenere a mente è che i vincoli lineari sono meno complessi di quelli non lineari. Vedi [constraint generation](https://docs.circom.io/circom-language/constraint-generation/) per maggiori dettagli. _Wires_ e _label_ si riferiscono all'aspetto visivo del _circuito aritmetico_. Non è qualcosa di cui ci si deve solitamente preoccupare. Vedi [arithmetic circuits](https://docs.circom.io/background/background/#arithmetic-circuits) per maggiori dettagli.
[^13]: Matematicamente, quello che facciamo è verificare che l'equazione `Az * Bz = Cz` sia soddisfatta, dove `Z=(W,x,1)`. `A`, `B` e `C` sono matrici, `W` è il witness (l'input privato) e `x` è l'input/output pubblico. Pur essendo utile da sapere, non è necessario comprendere questo per scrivere circuiti. Vedi [Rank-1 constraint system](https://docs.circom.io/background/background/#rank-1-constraint-system) per maggiori dettagli.
[^14]: Un termine più corretto, anche se meno diffuso, sarebbe "parametri di dimostrazione" e "parametri di verifica", rispettivamente. Sarebbe più intuitivo, poiché le chiavi sono solitamente concepite come private. Manterremo il termine "chiave" invece di "parametro" perché è quello che si incontra più spesso nel mondo reale.
[^15]: Come [menzionato](https://zkintro.com/it/articles/friendly-introduction-to-zero-knowledge#user-content-fn-33) nell'articolo _friendly introduction_, esiste un ottimo podcast divulgativo sulla cerimonia di Zcash del 2016, disponibile [qui](https://radiolab.org/podcast/ceremony). Da allora molto è cambiato riguardo ai trusted setup, che sono diventati molto più semplici da organizzare e a cui partecipare.
[^16]: Questo perché ci affidiamo alla casualità per rendere sicura la generazione delle proving key e delle verification key. In un trusted setup reale, disporre di un maggior numero di fonti di entropia è spesso auspicabile.
[^17]: Questo è detto modello di fiducia 1-su-N. Esistono molti altri modelli di fiducia; quello più familiare è probabilmente la regola della maggioranza, in cui ci si fida che la maggioranza prenda la decisione giusta. È sostanzialmente il principio su cui si basano la democrazia e la maggior parte dei sistemi di voto.
[^18]: Poiché generiamo sempre il witness insieme alla prova, il file binario risultante `witness.wtns` è per lo più un passaggio intermedio e un dettaglio implementativo. Lo utilizziamo immediatamente per generare la prova, motivo per cui è omesso dal diagramma.
[^19]: In letteratura, un witness è semplicemente la parte `W` del vettore `Z=(W,x,1)` usato nel R1CS, dove `x` rappresenta tutti i segnali pubblici/di input. In Circom, l'intero vettore viene chiamato witness. Vedi anche la nota 13.
[^20]: I numeri sono stati abbreviati per brevità con `[...]`. Dal punto di vista matematico, si tratta di punti su curva ellittica della curva _bn128_, con una dimensione del campo di 254 bit. Un numero a 254 bit può avere fino a 77 cifre nella sua rappresentazione decimale.
[^21]: L'output è un po' controintuitivo nel senso che non corrisponde direttamente al nome originale del segnale in questo modo: `{"c": "33"}`. Lo sviluppatore deve quindi riassegnare gli output in base all'ordine in cui sono stati definiti nel circuito. Ciò è dovuto all'implementazione di `snarkjs`, che perde le informazioni sulle variabili durante la generazione della prova.
[^22]: Nota anche come _assunzione di difficoltà crittografica_. Vedi [Computational hardness assumption (Wikipedia)](https://en.wikipedia.org/wiki/Computational_hardness_assumption#Common_cryptographic_hardness_assumptions).
[^23]: Vedi https://en.wikipedia.org/wiki/Integer_factorization per approfondire.
[^24]: Sebbene si possano aggiungere _assert_, questi non sono in realtà vincoli ma vengono usati solo per il controllo degli input. Vedi https://docs.circom.io/circom-language/code-quality/code-assertion/ per capire come funzionano e https://www.chainsecurity.com/blog/circom-assertions-misconceptions-and-deceptions per un esempio di come l'uso improprio degli assert può causare problemi. Per una maggiore comprensione di cosa sono i vincoli, vedi la sezione precedente _Sui vincoli_.
[^25]: Questa risorsa di 0xPARC è eccellente se vuoi approfondire l'arte di scrivere circuiti (Circom): https://learn.0xparc.org/materials/circom/learning-group-1/circom-1/ (in particolare i workshop su Circom). Anche l'esame della libreria standard può rivelarsi molto istruttivo; vedi nota 26.
[^26]: A causa della natura della scrittura dei vincoli, questo argomento ricorre spesso. Vedi https://en.wikipedia.org/wiki/Truth_table.
[^27]: Vedi https://github.com/iden3/circomlib per maggiori informazioni su circomlib.
[^28]: Vedi https://github.com/iden3/circomlib/blob/master/circuits/comparators.circom.
[^29]: Di solito i file `ptau` vengono condivisi tra progetti per aumentare la sicurezza. Vedi https://github.com/privacy-scaling-explorations/perpetualpowersoftau per i dettagli. Un elenco di questi file ptau, di varie dimensioni, è disponibile anche in https://github.com/iden3/snarkjs.
[^30]: Qui la scala rappresenta un valore che ci permette di percorrere il cammino opposto, quello "difficile". Un altro modo di pensarci è come un lucchetto: è facile chiuderlo, ma difficile aprirlo senza la chiave. Le trapdoor function hanno anche una definizione formale più rigorosa; vedi https://en.wikipedia.org/wiki/Trapdoor_function.
[^31]: Screenshot da Wikipedia. Vedi [ECDSA (Wikipedia)](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm#Signature_verification_algorithm).
[^32]: Questo comando dovrebbe funzionare sulla maggior parte dei sistemi di tipo Unix. Usiamo `-n` per specificare l'assenza del carattere di newline (`foo`, non `foo\n`), e `-a` per indicare che vogliamo usare SHA256.
[^33]: Vedi https://en.wikipedia.org/wiki/Commitment_scheme. Si noti che la proprietà di hiding (occultamento) non è sempre necessaria. Ad esempio, quando si usano le ZKP per rendere Ethereum più scalabile, si vuole soltanto una rivelazione efficiente di un sottoinsieme del trie di stato.
[^34]: Usiamo Poseidon, ma ne esistono molte altre. Perché è più veloce? Queste funzioni di hash ZK-friendly (ottimizzate per ZKP) vengono implementate usando operazioni aritmetiche su campi primi, non operazioni bit a bit come SHA256. Richiedono molti meno vincoli da implementare, il che si traduce in tempi di dimostrazione più rapidi. La differenza di prestazioni tra le due può arrivare a due ordini di grandezza. Di contro, una funzione di hash come SHA256 è stata studiata in modo molto più rigoroso rispetto alla maggior parte di queste nuove funzioni di hash ZK-friendly.
[^35]: Nelle ZKP, spesso vogliamo fare l'hash di più elementi insieme. A differenza di un contesto tradizionale, non possiamo semplicemente concatenare stringhe ("foo bar"), quindi specifichiamo invece quanti input passiamo alla nostra funzione di hash.
[^36]: Come indicato nella nota precedente, se si usasse SHA-256 o operazioni su curve ellittiche, il numero di vincoli sarebbe molto più elevato. Con più di 4000 vincoli, saremmo costretti a eseguire (o riutilizzare) un'altra fase 1 del trusted setup con un file ptau di capacità maggiore.
[^37]: Possiamo tuttavia codificare la stringa come array di byte, usando Unicode o ASCII. In un'applicazione reale si utilizzerebbe probabilmente l'hash del messaggio nella sua rappresentazione BigInt.
[^38]: In uno schema di firma digitale reale, in cui vengono scambiati più messaggi, si vorrebbe probabilmente introdurre anche un nonce crittografico. Questo per evitare i replay attack (attacco di riproduzione), in cui qualcuno potrebbe riutilizzare la stessa firma in un momento successivo. Vedi https://en.wikipedia.org/wiki/Replay_attack.
[^39]: Per applicazioni reali, cerca di riutilizzare il più possibile lavoro esistente e le best practice. Ci sono molte cose che possono andare storte se non si è prudenti. Per fortuna, la situazione sta migliorando man mano che l'ecosistema ZKP matura. A un certo punto del loro sviluppo, molte applicazioni ad alto rischio si sottopongono ad audit di sicurezza per verificare che siano sicure (o almeno non dimostrabilmente insicure).
[^40]: L'implementazione delle firme di gruppo (group signature) in ZKP è stata ispirata da 0xPARC; vedi https://0xparc.org/blog/zk-group-sigs.
[^41]: Vedi https://docs.circom.io/circom-language/control-flow/.
[^42]: A titolo di confronto, un articolo scientifico che implementa firme di gruppo come https://eprint.iacr.org/2015/043.pdf presenta una complessità crittografica e matematica ben maggiore.
