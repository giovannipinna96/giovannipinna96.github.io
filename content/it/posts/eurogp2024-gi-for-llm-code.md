---
title: "Migliorare la Generazione di Codice degli LLM con il Genetic Improvement"
date: 2024-04-03
draft: false
tags: ["Genetic Improvement", "LLM", "Generazione di Codice", "Evoluzione Grammaticale", "EuroGP"]
categories: ["Ricerca"]
description: "Come il Genetic Improvement e l'Evoluzione Grammaticale possono migliorare sistematicamente il codice generato dai Large Language Models, ottenendo miglioramenti statisticamente significativi su molteplici LLM e problemi di benchmark."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
I Large Language Models possono generare codice a partire da descrizioni in linguaggio naturale, ma i programmi risultanti contengono frequentemente errori. Anziché affidarsi all'LLM per correggere i propri errori tramite self-correction, proponiamo di trattare il codice generato come punto di partenza per il Genetic Improvement. La nostra pipeline a tre fasi estrae il codice dall'output dell'LLM, costruisce dinamicamente una grammatica BNF specializzata dal suo Abstract Syntax Tree e applica l'Evoluzione Grammaticale per cercare varianti migliorate. Valutato su 25 problemi del benchmark PSB2 attraverso cinque LLM — GPT-4, ChatGPT, LLaMA-2, Alpaca-13B e Alpaca-7B — con popolazioni di 1.000 individui per 100 generazioni, l'approccio ottiene miglioramenti statisticamente significativi (p < 0.001) per ogni modello testato e supera costantemente la self-correction dell'LLM. I maggiori guadagni relativi si osservano sui modelli più piccoli, suggerendo che il raffinamento evolutivo è particolarmente efficace nel compensare la capacità limitata del modello. Pubblicato a EuroGP 2024.
{{< /summary-box >}}

## Introduzione

I Large Language Models (LLM) come ChatGPT e GPT-4 hanno dimostrato capacità notevoli nella generazione di codice sorgente a partire da descrizioni in linguaggio naturale. Tuttavia, nonostante la loro impressionante fluenza, il codice che producono è tutt'altro che affidabile. Diversi studi hanno mostrato che il codice generato dagli LLM contiene spesso errori logici sottili, fallisce nei casi limite o semplicemente non soddisfa i requisiti funzionali specificati nel prompt.

L'approccio dominante per affrontare questo problema è stato la **self-correction** — fornire all'LLM il proprio output errato chiedendogli di riprovare. Sebbene intuitiva, questa strategia presenta limitazioni fondamentali: il modello tende a ripetere gli stessi tipi di errori e non esiste un meccanismo sistematico che guidi il processo di correzione verso soluzioni genuinamente migliori.

In questo lavoro, pubblicato a **EuroGP 2024** (parte della conferenza EvoStar), abbiamo proposto un approccio fondamentalmente diverso: anziché affidarci all'LLM per correggere il proprio codice, trattiamo il codice generato come punto di partenza e applichiamo il **Genetic Improvement (GI)** — una tecnica di ingegneria del software basata sulla ricerca — per evolverlo verso la correttezza.

## L'Idea Centrale: Il Codice come Materiale Evolvibile

Il Genetic Improvement si ispira all'evoluzione biologica. Così come la selezione naturale opera sulla variazione genetica per produrre organismi meglio adattati al loro ambiente, il GI opera su varianti di programmi per produrre software meglio adattato alle sue specifiche. L'intuizione chiave è che il codice generato dagli LLM, anche quando errato, è tipicamente *vicino* a una soluzione corretta — cattura la struttura generale, utilizza tipi di dati appropriati e spesso implementa l'approccio algoritmico giusto. Ciò che necessita è un raffinamento mirato, non una riscrittura completa.

Il nostro approccio combina il GI con l'**Evoluzione Grammaticale (GE)**, una tecnica evolutiva basata su grammatiche che vincola le mutazioni a produrre solo programmi sintatticamente validi. Questo è fondamentale: mutazioni casuali applicate al codice sorgente producono quasi sempre programmi che non vengono nemmeno parsati, tanto meno eseguiti correttamente. Definendo una grammatica formale che descrive lo spazio delle modifiche valide, garantiamo che ogni mutante nella ricerca evolutiva sia almeno sintatticamente ben formato.

## La Pipeline a Tre Fasi

La nostra metodologia consiste in tre fasi che trasformano l'output grezzo dell'LLM in codice migliorato:

### Fase 1: Estrazione e Analisi del Codice

La prima fase prende l'output testuale grezzo di un LLM ed estrae il codice Python eseguibile. Questo è meno banale di quanto sembri — gli LLM incorporano il codice all'interno di testo esplicativo, formattazione markdown, e talvolta producono blocchi di codice multipli. Abbiamo sviluppato routine di estrazione che isolano in modo affidabile il programma previsto dal testo circostante.

Una volta estratto, il codice viene sottoposto ad **analisi dell'Abstract Syntax Tree (AST)**. Il modulo `ast` di Python analizza il codice in una rappresentazione ad albero, permettendoci di identificare tutti gli elementi sintattici presenti: tipi di variabili, strutture di controllo, definizioni di funzioni, operatori e chiamate a librerie.

### Fase 2: Specializzazione Dinamica della Grammatica

Questa è una delle contribuzioni chiave del nostro lavoro. Anziché utilizzare una singola grammatica universale per tutti i programmi, **generiamo dinamicamente una grammatica BNF (Backus-Naur Form) specializzata** dall'analisi AST di ciascun programma generato dall'LLM.

La logica è semplice: se il codice dell'LLM utilizza solo variabili intere e cicli `for`, non c'è beneficio nell'esplorare mutazioni che introducono aritmetica in virgola mobile o cicli `while`. Restringendo la grammatica ai costrutti già presenti nel codice (o strettamente correlati), riduciamo drasticamente lo spazio di ricerca preservando la capacità di apportare miglioramenti significativi.

La grammatica specializzata viene costruita automaticamente mappando i tipi di nodi AST a regole di produzione grammaticale. Ogni programma ha quindi il proprio spazio di mutazione personalizzato — una forma di progettazione intelligente dello spazio di ricerca che sfrutta l'output dell'LLM come conoscenza pregressa.

### Fase 3: Ricerca Evolutiva

Con la grammatica definita, applichiamo l'Evoluzione Grammaticale utilizzando la libreria **PonyGE2**. Il processo evolutivo opera come segue:

- **Popolazione**: 1.000 individui, ciascuno rappresentante una variante del codice LLM originale
- **Generazioni**: fino a 100 generazioni evolutive
- **Selezione**: Selezione a torneo con dimensione 2
- **Crossover**: Crossover a un punto variabile (probabilità 0.75)
- **Mutazione**: Mutazione integer flip (probabilità 0.05)
- **Fitness**: La proporzione di casi di test del benchmark PSB2 che ciascuna variante risolve correttamente

Ogni individuo nella popolazione è una sequenza di interi (un genotipo) che viene mappata attraverso la grammatica per produrre un programma concreto (il fenotipo). Crossover e mutazione operano sulle sequenze di interi, mentre la grammatica garantisce che i programmi risultanti siano sempre sintatticamente validi.

## Setup Sperimentale

Abbiamo valutato il nostro approccio su **25 problemi di sintesi di programmi** dalla suite **PSB2 (Program Synthesis Benchmark 2)**. Ogni problema specifica una trasformazione input-output — ad esempio, calcolare il massimo comun divisore, ordinare una lista o elaborare stringhe — e viene fornito con 1.000 casi di test per la valutazione del fitness.

Abbiamo testato cinque LLM che coprono una gamma di capacità:

- **GPT-4**: Il modello più capace di OpenAI all'epoca
- **ChatGPT (GPT-3.5-turbo)**: Il modello conversazionale ampiamente diffuso
- **LLaMA-2 13B**: Il modello open-source di Meta
- **Alpaca-13B**: Una variante fine-tuned di LLaMA
- **Alpaca-7B**: Un modello fine-tuned più piccolo

Per ciascun LLM e ciascun problema, abbiamo generato il codice iniziale usando un prompt standardizzato, poi applicato la nostra pipeline GI. Ogni esperimento è stato ripetuto 30 volte per garantire robustezza statistica, e abbiamo utilizzato il test di Wilcoxon dei ranghi con segno per valutare la significatività.

## Risultati

I risultati sono stati chiari e consistenti:

**Il GI ha migliorato il codice di ogni LLM testato**, con miglioramenti statisticamente significativi (p < 0.001) su tutta la linea. L'entità del miglioramento variava per modello — i modelli più piccoli e meno capaci come Alpaca-7B hanno mostrato i maggiori guadagni relativi, suggerendo che il GI è particolarmente efficace nel compensare la capacità limitata del modello.

Risultati specifici includono:

- **Alpaca-7B**: Il performer inizialmente più debole ha mostrato un miglioramento drammatico attraverso il GI, con molti problemi che passavano da zero casi di test superati a correttezza parziale o completa.
- **GPT-4**: Anche il modello più forte ha beneficiato del raffinamento GI, sebbene i guadagni fossero minori in termini assoluti (poiché il codice iniziale di GPT-4 era già relativamente buono).
- **Consistenza cross-model**: Il miglioramento non era limitato a tipi specifici di problemi — il GI ha aiutato attraverso categorie aritmetiche, manipolazione di stringhe, elaborazione di liste e altre.

**È importante sottolineare che il GI ha costantemente superato la self-correction dell'LLM.** Confrontando il nostro approccio con il semplice richiedere all'LLM di correggere i propri errori (con accesso alle informazioni sui casi di test falliti), l'approccio evolutivo ha prodotto codice finale migliore. Questo ha senso: la self-correction è vincolata dai bias e dai punti ciechi del modello stesso, mentre il GI esplora uno spazio molto più ampio di possibili modifiche.

## Perché Funziona

L'efficacia del nostro approccio si basa su tre fattori complementari:

1. **Gli LLM forniscono buoni punti di partenza.** Il codice generato è solitamente strutturalmente solido — vengono chiamate le funzioni giuste, vengono usate le strutture dati corrette, l'algoritmo generale è ragionevole. Questo significa che lo spazio di ricerca attorno all'output dell'LLM contiene molti miglioramenti praticabili.

2. **La specializzazione della grammatica focalizza la ricerca.** Personalizzando la grammatica di mutazione per ciascun programma specifico, evitiamo di sprecare sforzo computazionale esplorando regioni irrilevanti dello spazio del codice. La ricerca è concentrata dove è più probabile trovare miglioramenti.

3. **La ricerca evolutiva è robusta.** A differenza delle strategie di riparazione greedy che possono rimanere bloccate in ottimi locali, la ricerca basata su popolazione della GE mantiene la diversità e può combinare miglioramenti parziali da diversi individui attraverso il crossover.

## Implicazioni

Questo lavoro apre diverse direzioni importanti:

Per i **professionisti**, suggerisce che le pipeline di generazione di codice con LLM possono essere significativamente migliorate aggiungendo una fase di ottimizzazione post-processing. Anziché accettare l'output dell'LLM così com'è o affidarsi a costosi re-prompting, una fase GI può migliorare sistematicamente la qualità del codice.

Per i **ricercatori**, la tecnica di specializzazione dinamica della grammatica è applicabile oltre questo specifico contesto — qualsiasi scenario in cui il GI viene applicato a programmi con caratteristiche strutturali note potrebbe beneficiare di un approccio simile alla riduzione dello spazio di ricerca.

Per il **campo più ampio**, questo lavoro dimostra che i punti di forza degli approcci neurali ed evolutivi sono complementari: gli LLM eccellono nel generare codice plausibile rapidamente, mentre i metodi evolutivi eccellono nel raffinamento sistematico verso specifiche precise.

---

*Pubblicato alla 27ª Conferenza Europea sulla Programmazione Genetica (EuroGP 2024), parte di EvoStar 2024, 3-5 aprile, Aberystwyth, UK. Questa ricerca è stata condotta presso l'Università degli Studi di Trieste e la NOVA Information Management School (NOVA IMS), Universidade Nova de Lisboa.*
