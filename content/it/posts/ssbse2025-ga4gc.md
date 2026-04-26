---
title: "GA4GC: Agente Più Green per Codice Più Green"
date: 2025-10-13
draft: false
tags: ["Green AI", "Agenti di Codifica AI", "Ottimizzazione Multi-Obiettivo", "NSGA-II", "Sostenibilità"]
categories: ["Ricerca"]
description: "Utilizzo dell'ottimizzazione multi-obiettivo per configurare agenti di codifica AI per un funzionamento energeticamente efficiente, ottenendo fino al 37.7% di riduzione del runtime migliorando contemporaneamente la correttezza del codice."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
Gli agenti di codifica AI consumano risorse computazionali sostanziali — spesso oltre 100.000 token per esecuzione — eppure vengono distribuiti con configurazioni default che sono lontane dall'essere ottimali. GA4GC (Greener Agent for Greener Code) applica l'ottimizzazione multi-obiettivo NSGA-II per regolare sistematicamente le configurazioni degli agenti, bilanciando correttezza del codice, miglioramento delle prestazioni e tempo di esecuzione dell'agente. Valutato su un'architettura mini-SWE-agent alimentata da Gemini 2.5 Pro sul benchmark SWE-Perf, il framework ottiene fino al 37.7% di riduzione del runtime e un miglioramento dell'ipervolume di 135 volte rispetto alle impostazioni default, migliorando contemporaneamente la correttezza del codice. L'analisi Random Forest rivela che la temperatura è il parametro singolarmente più influente, e che gli iperparametri dell'LLM guidano principalmente l'efficacia del task mentre i vincoli dell'agente guidano principalmente il consumo di risorse — una separazione che consente la regolazione indipendente di qualità ed efficienza. Pubblicato a SSBSE 2025, Challenge Track on Green SBSE.
{{< /summary-box >}}

## Introduzione

Gli agenti di codifica AI — strumenti come GitHub Copilot, Claude Code, Devin e OpenAI Codex — rappresentano un'evoluzione significativa rispetto al semplice completamento del codice. Questi sistemi operano attraverso pipeline di ragionamento complesse e multi-step: analizzano la struttura del repository, pianificano soluzioni, generano codice, lo eseguono in ambienti sandbox, diagnosticano i fallimenti e iterano attraverso multipli cicli di raffinamento. Possono affrontare task di ingegneria del software reali che nessuna singola chiamata LLM potrebbe gestire.

Ma questa potenza ha un costo sostanziale. Una singola esecuzione di un agente su un task di ingegneria del software moderatamente complesso può consumare oltre **100.000 token**, traducendosi in spese monetarie e consumo energetico significativi. E qui risiede un paradosso critico: quando un agente AI è incaricato di *ottimizzare le prestazioni del codice*, l'energia consumata dall'agente stesso durante il processo di ottimizzazione può superare vastamente l'energia risparmiata dai miglioramenti del codice risultanti. Senza un'attenta regolazione della configurazione, un agente potrebbe dover produrre codice che viene eseguito **centinaia di migliaia di volte** prima che il risparmio energetico compensi il costo dell'ottimizzazione. Alcune "ottimizzazioni" sono in realtà una perdita netta di energia.

Questo articolo, presentato a **SSBSE 2025** (il 17° Symposium on Search-Based Software Engineering) come parte del Challenge Track on Green SBSE, introduce **GA4GC** (Greener Agent for Greener Code) — un framework che applica l'ottimizzazione multi-obiettivo per trovare configurazioni degli agenti che bilancino efficacia ed efficienza delle risorse.

## Il Problema dello Spazio di Configurazione

Un agente di codifica AI ha uno spazio di configurazione sorprendentemente ampio. I parametri chiave includono:

- **Temperatura dell'LLM**: Controlla la casualità della generazione del testo (0 = deterministico, più alto = più creativo/casuale)
- **Campionamento top_p**: Un altro parametro di controllo della diversità che limita la selezione dei token alle opzioni più probabili
- **Limiti massimi di token**: Quanti token l'agente può generare per step
- **Limiti di step**: Quanti cicli ragionamento-azione l'agente può eseguire
- **Varianti del template di prompt**: Diverse istruzioni e prompt di sistema che modellano il comportamento dell'agente

Ogni parametro influenza sia la qualità dell'output dell'agente sia il suo consumo di risorse. Le interazioni tra parametri sono complesse e spesso controintuitive — una temperatura più alta potrebbe migliorare la qualità del codice su task creativi ma sprecare risorse su task semplici. Lo spazio di configurazione combinato è troppo grande per un'esplorazione manuale efficace.

## Come Funziona GA4GC

### L'Architettura dell'Agente

GA4GC opera su un'architettura **mini-SWE-agent** alimentata da **Gemini 2.5 Pro** come LLM backbone. L'agente segue il flusso di lavoro standard SWE-agent: lettura dei file del repository, pianificazione delle modifiche, generazione di patch, esecuzione dei test e iterazione sui fallimenti. Questa architettura rappresenta uno scenario di deployment realistico per gli agenti di codifica AI nella pratica.

### Il Framework di Ottimizzazione

GA4GC inquadra la regolazione della configurazione come un **problema di ottimizzazione multi-obiettivo** e lo risolve usando **NSGA-II** (Non-dominated Sorting Genetic Algorithm II). I tre obiettivi in competizione sono:

1. **Minimizzare le patch errate**: Massimizzare la probabilità che l'agente produca codice corretto e funzionale
2. **Massimizzare il miglioramento delle prestazioni del codice**: Assicurare che il codice ottimizzato venga effettivamente eseguito più velocemente — questo è lo scopo primario del task di ottimizzazione
3. **Minimizzare il tempo di esecuzione dell'agente**: Ridurre le risorse computazionali (tempo, token, energia) consumate dall'agente stesso

Lo spazio di ricerca è eterogeneo: parametri continui (temperatura, top_p), vincoli interi (token massimi, limiti di step) e variabili categoriche (template di prompt). NSGA-II è particolarmente adatto per questo tipo di problema a variabili miste, applicando operatori di crossover e mutazione appropriati per ogni tipo di parametro.

### Valutazione su SWE-Perf

Ogni configurazione candidata viene valutata su task dal **benchmark SWE-Perf**, che fornisce task autentici di ottimizzazione delle prestazioni a livello di repository dal progetto **astropy** (una libreria Python ampiamente utilizzata per l'astronomia). L'agente genera patch che vengono validate in **ambienti Docker isolati**, garantendo misurazioni riproducibili sia della correttezza del codice sia dei guadagni di prestazione.

### Processo Evolutivo

Partendo da una popolazione di configurazioni casuali, NSGA-II evolve soluzioni migliori su più generazioni. Attraverso selezione, crossover e mutazione, l'algoritmo converge verso un **fronte di Pareto** di configurazioni non dominate — soluzioni in cui migliorare un obiettivo peggiora necessariamente un altro. Questo fronte di Pareto offre ai professionisti un menù di opzioni di trade-off tra cui scegliere.

## Risultati Chiave

Con un budget vincolato di sole **25 valutazioni di configurazione**, GA4GC ha ottenuto risultati notevoli:

### Guadagni di Efficienza

Le configurazioni non dominate hanno raggiunto fino al **37.7% di riduzione del runtime** (943 secondi vs. 1.513 secondi per la configurazione default) migliorando *contemporaneamente* la correttezza del codice. Il **miglioramento dell'ipervolume** ha raggiunto fino a **135 volte** rispetto al baseline default.

Questa è una scoperta critica: significa che le configurazioni default fornite con gli agenti di codifica AI sono lontane dall'essere ottimali. Miglioramenti significativi in termini sia di qualità che di efficienza sono disponibili attraverso una regolazione sistematica.

### Analisi dell'Importanza dei Parametri

Usando l'**analisi di regressione Random Forest**, abbiamo identificato quali parametri contano di più:

**La temperatura è il parametro più influente in assoluto.** Questo ha senso intuitivo: la temperatura controlla la casualità fondamentale degli output dell'LLM, influenzando tutto, dalla creatività del codice all'ampiezza dell'esplorazione. Piccoli cambiamenti di temperatura possono alterare drasticamente il comportamento dell'agente.

L'analisi ha rivelato una scoperta strutturale importante — **due categorie di parametri svolgono ruoli diversi**:

- **Iperparametri dell'LLM** (temperatura, top_p) impattano principalmente sull'**efficacia del task** — se l'agente produce codice corretto e performante
- **Vincoli dell'agente** (limiti di token, conteggio degli step) impattano principalmente sul **consumo di risorse** — quanto tempo e computazione l'agente utilizza

Questa separazione è altamente azionabile: i professionisti possono regolare il consumo di risorse senza necessariamente influenzare la qualità del codice, e viceversa. Significa che i miglioramenti dell'efficienza e i miglioramenti della qualità possono spesso essere perseguiti in modo indipendente.

## Strategie Pratiche di Deployment

GA4GC traduce i risultati dell'ottimizzazione in tre scenari concreti di deployment:

### 1. Ambienti Runtime-Critical

Usare **temperatura bassa** con **top_p restrittivo**. L'agente genera codice meno diversificato ma più veloce, completando i task con un overhead computazionale minimo. Ideale quando il tempo di completamento è la priorità e i task sono relativamente semplici.

### 2. Scenari Performance-Critical

Usare **temperatura moderata** (0.65–0.73) con **top_p bilanciato**. Questo dà all'agente sufficiente libertà esplorativa per scoprire soluzioni genuinamente migliori, al costo di un maggiore consumo di risorse. Appropriato quando i guadagni di prestazione del codice giustificano il runtime aggiuntivo dell'agente.

### 3. Ottimizzazione Context-Specific

Eseguire GA4GC stesso sul proprio specifico codebase e distribuzione di task. Il fronte di Pareto che scopre sarà personalizzato per le vostre particolari esigenze, fornendo i migliori trade-off possibili per il vostro ambiente. Questo rappresenta l'approccio più rigoroso per le organizzazioni in cui l'uso degli agenti è frequente e l'ottimizzazione conta.

## Perché Questo È Importante

Man mano che gli agenti di codifica AI passano da strumenti sperimentali a infrastruttura di sviluppo standard, la loro impronta computazionale cumulativa diventa una preoccupazione di sostenibilità. Un'organizzazione che esegue centinaia di task di agenti al giorno genera costi energetici significativi — sia finanziariamente che ambientalmente.

GA4GC dimostra che la **regolazione della configurazione è una leva di sostenibilità** che complementa gli approcci tradizionali come i miglioramenti dell'architettura dei modelli o l'ottimizzazione hardware. Anziché accettare le configurazioni default e assorbire il costo, i professionisti possono scoprire sistematicamente configurazioni che offrono le prestazioni necessarie minimizzando gli sprechi.

Il framework fornisce un template metodologico per il deployment responsabile dell'AI nell'ingegneria del software: misurare, ottimizzare e deployare con consapevolezza del quadro completo costi-benefici.

---

*Pubblicato al 17° Symposium on Search-Based Software Engineering (SSBSE 2025), Challenge Track on Green SBSE. Questa ricerca è stata condotta presso la University College London (UCL) e l'Università degli Studi di Trieste. Codice e risultati sono pubblicamente disponibili.*
