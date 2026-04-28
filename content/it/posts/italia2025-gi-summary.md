---
title: "Migliorare il Codice Generato dagli LLM tramite Genetic Improvement: Un Riepilogo dei Recenti Avanzamenti"
date: 2025-06-23
draft: false
tags: ["Genetic Improvement", "LLM", "Generazione di Codice", "Evoluzione Grammaticale", "Ital-IA"]
categories: ["Ricerca"]
description: "Un riepilogo completo del nostro programma di ricerca sull'applicazione del Genetic Improvement al codice generato dagli LLM, presentato alla conferenza nazionale italiana sull'IA Ital-IA 2025."
ShowToc: true
TocOpen: false
cover:
  image: "/images/post-placeholder.svg"
  alt: "Genetic Improvement workflow for LLM-generated code"
  hiddenInList: false
---

{{< summary-box title="Abstract" >}}
Questo articolo, presentato a Ital-IA 2025, fornisce un riepilogo completo del nostro programma di ricerca sull'applicazione del Genetic Improvement al codice generato dagli LLM, estendendo due studi pubblicati (EuroGP 2024 e SN Computer Science 2025). La scoperta centrale attraverso entrambi i lavori è che gli approcci neurali ed evolutivi sono complementari: gli LLM eccellono nella generazione rapida di codice plausibile, mentre il Genetic Improvement eccelle nel raffinamento sistematico verso specifiche precise. L'effetto "amplificatore di capacità" — dove i modelli più piccoli e open-source beneficiano in modo più drammatico dal raffinamento evolutivo — suggerisce che il GI può contribuire a ridurre il divario tra modelli piccoli e grandi. Discutiamo inoltre le limitazioni chiave, tra cui la dipendenza dall'oracolo, la scalabilità a progetti multi-file, il bias della grammatica ereditato dall'output dell'LLM e la mancanza di garanzie formali inerente alla ricerca stocastica. Pubblicato a Ital-IA 2025, Roma.
{{< /summary-box >}}

## Introduzione

L'intersezione tra Large Language Models e calcolo evolutivo rappresenta una delle frontiere più promettenti nell'ingegneria del software automatizzata. Negli ultimi due anni, il nostro gruppo di ricerca ha sviluppato e raffinato una metodologia per migliorare sistematicamente il codice generato dagli LLM utilizzando tecniche di Genetic Improvement (GI). Questo articolo, presentato a **Ital-IA 2025** (la 5ª Conferenza Nazionale sull'Intelligenza Artificiale, organizzata dal CINI), fornisce un riepilogo completo di questo programma di ricerca e delle sue scoperte chiave.

## Il Programma di Ricerca

Il nostro lavoro si basa su un'osservazione semplice ma potente: il codice generato dagli LLM, anche quando non soddisfa completamente le specifiche, contiene tipicamente informazioni strutturali preziose. I programmi generati utilizzano tipi di dati appropriati, implementano algoritmi ragionevoli e catturano la forma generale delle soluzioni corrette. Quello che manca loro è la precisione — le condizioni esatte, la gestione dei casi limite e i dettagli algoritmici che separano "approssimativamente corretto" da "completamente corretto".

Il Genetic Improvement sfrutta questa osservazione trattando gli output degli LLM come punti di partenza per l'ottimizzazione evolutiva. Piuttosto che scartare il codice errato e ricominciare da zero, il GI lo evolve verso la correttezza attraverso un processo di variazione guidata e selezione.

## Contributi Chiave Attraverso Due Studi

### Studio 1: GI con Evoluzione Grammaticale (EuroGP 2024)

Il nostro primo studio ha introdotto la pipeline a tre fasi di estrazione del codice, specializzazione dinamica della grammatica e ricerca evolutiva. L'innovazione tecnica chiave è stata la generazione automatica di grammatiche BNF personalizzate per ciascun programma generato dall'LLM, garantendo che le mutazioni rimangano sintatticamente valide e focalizzate sui costrutti di codice rilevanti.

Valutato su **25 problemi PSB2** attraverso **5 LLM** (GPT-4, ChatGPT, LLaMA-2 13B, Alpaca-13B, Alpaca-7B), l'approccio ha raggiunto miglioramenti statisticamente significativi (p < 0.001) per ogni modello testato, superando costantemente la self-correction dell'LLM.

### Studio 2: GI Migliorato con Selezione Lexicase (SN Computer Science 2025)

Il nostro secondo studio ha raffinato le componenti evolutive. Abbiamo introdotto la **selezione lexicase** — una strategia che valuta gli individui sui casi di test in sequenza, preservando soluzioni specialiste che eccellono su sottoinsiemi specifici. Combinata con il **10% di down-sampling** per l'efficienza computazionale e una **funzione di fitness raffinata F_E** che fornisce credito parziale anziché un verdetto binario, la pipeline migliorata ha raggiunto miglioramenti in **11 su 12 combinazioni modello-problema**.

La lista aggiornata dei modelli includeva Code Llama 7B e LLaMA 3 8B, con risultati che confermano che il GI fornisce il maggior beneficio relativo ai modelli più piccoli e meno capaci — amplificandone efficacemente le capacità.

## Risultati Trasversali

Diverse scoperte sono emerse in modo consistente attraverso entrambi gli studi:

### La Complementarietà degli Approcci Neurali ed Evolutivi

LLM e algoritmi evolutivi hanno punti di forza fondamentalmente diversi. Gli LLM eccellono nella generazione rapida di soluzioni plausibili sfruttando pattern appresi da vasti corpus di codice. Gli algoritmi evolutivi eccellono nel raffinamento sistematico guidato dalle specifiche. La loro combinazione produce risultati che nessuno dei due approcci raggiunge da solo.

### L'Effetto "Amplificatore di Capacità"

Il beneficio relativo del GI è inversamente proporzionale alla capacità iniziale dell'LLM. I modelli più deboli (Alpaca-7B, Code Llama 7B) mostrano i miglioramenti più drammatici, mentre i modelli più forti (GPT-4) mostrano guadagni minori ma comunque significativi. Questo ha importanti implicazioni pratiche: le organizzazioni che utilizzano modelli più piccoli e open-source per ragioni di costo o privacy possono usare il GI per ridurre il divario con i modelli proprietari più grandi.

### La Specializzazione Grammaticale come Progettazione dello Spazio di Ricerca

La generazione dinamica di grammatiche specifiche per problema si è rivelata più di una comodità tecnica — è una forma di progettazione intelligente dello spazio di ricerca. Sfruttando le scelte strutturali dell'LLM come conoscenza pregressa, la ricerca evolutiva opera in una regione molto più piccola e produttiva dello spazio dei programmi.

## Limitazioni e Sfide Aperte

Abbiamo identificato diverse limitazioni che definiscono i confini dell'approccio attuale:

1. **Dipendenza dall'oracolo**: La funzione di fitness richiede casi di test o un oracolo di riferimento per la valutazione. I problemi senza specifiche chiare o suite di test non possono essere affrontati con la metodologia attuale.

2. **Vincoli di scalabilità**: La nostra valutazione si è concentrata su programmi piccoli e autonomi. L'ingegneria del software nel mondo reale coinvolge progetti multi-file con dipendenze complesse, che l'approccio basato su grammatica attuale non gestisce.

3. **Propagazione del bias dell'LLM**: Poiché la grammatica è derivata dall'output dell'LLM, i bias strutturali nel codice generato — come la preferenza per certi costrutti di loop o strutture dati — vengono ereditati dallo spazio di ricerca. Questo potrebbe impedire la scoperta di soluzioni che richiedono scelte architetturali fondamentalmente diverse.

4. **Mancanza di garanzie**: Gli algoritmi evolutivi sono stocastici per natura. Sebbene i nostri risultati mostrino miglioramenti consistenti in media, non c'è garanzia di miglioramento per ogni singola esecuzione o istanza di problema.

## Direzioni Future

Il programma di ricerca continua su diversi fronti: esplorazione di approcci GI senza grammatica che possano operare direttamente sugli AST senza l'intermediario BNF; sviluppo di tecniche di approssimazione del fitness per ridurre la dipendenza dalla valutazione esaustiva dei casi di test; e indagine sull'integrazione del GI in flussi di lavoro di ingegneria del software su scala più ampia dove sono necessarie modifiche multi-file.

---

*Presentato alla 5ª Conferenza Nazionale sull'Intelligenza Artificiale (Ital-IA 2025), organizzata dal CINI, 23-24 giugno 2025, Roma, Italia. Questa ricerca è stata condotta presso l'Università degli Studi di Trieste e la NOVA Information Management School (NOVA IMS), Universidade Nova de Lisboa.*
