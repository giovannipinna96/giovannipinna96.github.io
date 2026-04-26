---
title: "HotCat: Selezione di Feature Green ed Efficace per la Tassonomia dei Bug Hotfix"
date: 2025-10-13
draft: false
tags: ["Tassonomia Bug", "NSGA-II", "Ottimizzazione Multi-Obiettivo", "Green AI", "Selezione Feature"]
categories: ["Ricerca"]
description: "Un approccio di ottimizzazione multi-obiettivo per la classificazione degli hotfix software che bilancia qualità della classificazione ed efficienza computazionale, dimostrando che i principi del Green AI possono essere applicati senza sacrificare l'efficacia."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
Classificare gli hotfix software in categorie di bug è impegnativo a causa di dati scarsi, grave sbilanciamento delle classi e alto costo computazionale dell'analisi basata su LLM. HotCat affronta queste sfide tramite l'ottimizzazione multi-obiettivo con NSGA-II, trattando la selezione delle feature come un problema di ricerca su 18 feature disponibili estratte dal dataset HotBugs (88 entry di hotfix su 17 categorie). Il framework ottimizza simultaneamente accuratezza della classificazione, Informazione Mutua Normalizzata e tempo computazionale di esecuzione. Una strategia di augmentazione dei dati a due stadi migliora la generalizzazione dal 55% al 72%. Il fronte di Pareto risultante rivela che qualità della classificazione ed efficienza non devono necessariamente essere in conflitto: una configurazione bilanciata raggiunge il 59% di accuratezza e 0.58 NMI in soli 129 secondi, mentre l'eliminazione selettiva delle feature migliora effettivamente i risultati rimuovendo feature che introducono rumore. Pubblicato a SSBSE 2025, Challenge Track on Hot Fixing Benchmark.
{{< /summary-box >}}

## Introduzione

Nell'ingegneria del software, non tutti i bug sono uguali. Mentre alcuni difetti possono essere messi in coda per la prossima release programmata, altri richiedono attenzione immediata. Queste patch urgenti — note come **hotfix** — affrontano problemi critici che necessitano di deployment rapido in produzione: vulnerabilità di sicurezza, malfunzionamenti nell'elaborazione dei pagamenti, interruzioni di servizio o bug di corruzione dati che colpiscono utenti in produzione.

Comprendere la natura e la distribuzione di questi hotfix è essenziale per i team di sviluppo software. Una **tassonomia dei bug** ben costruita — una classificazione sistematica dei tipi di bug — aiuta i team a prioritizzare le risorse, identificare pattern di fallimento ricorrenti e implementare misure preventive. Ma costruire tali tassonomie è impegnativo, in particolare per gli hotfix: i dati sono scarsi (gli hotfix sono una piccola frazione di tutte le patch), la distribuzione delle classi è gravemente sbilanciata (alcuni tipi di bug sono molto più rari di altri) e una classificazione accurata richiede un'analisi semantica sofisticata delle modifiche al codice.

Questo articolo, presentato a **SSBSE 2025** (il 17° Symposium on Search-Based Software Engineering) come parte del Challenge Track on Hot Fixing Benchmark, introduce **HotCat** — un framework che affronta queste sfide aderendo ai principi del **Green AI** minimizzando le spese computazionali non necessarie.

## La Motivazione del Green AI

Gli approcci moderni all'analisi del codice si affidano sempre più ai Large Language Models per la comprensione semantica — riassumere le modifiche al codice, generare embedding e classificare l'intento. Questi modelli sono potenti ma computazionalmente costosi: ogni inferenza LLM consuma energia, e quando si elaborano migliaia di patch attraverso decine di feature, il costo cumulativo diventa significativo.

HotCat pone una domanda diretta: **abbiamo bisogno di tutte le feature disponibili per ottenere una buona classificazione, o possiamo selettivamente ridurre lo spazio delle feature per diminuire il costo computazionale senza sacrificare la qualità?** Questa non è semplicemente una preoccupazione di efficienza — è un imperativo ambientale ed economico, man mano che gli strumenti di analisi basati su LLM diventano standard nei flussi di lavoro di sviluppo.

## La Pipeline di HotCat

### Fondamento dei Dati: Il Dataset HotBugs

HotCat opera sul dataset **HotBugs**, che contiene **88 entry di hotfix** distribuite su **17 categorie di bug** estratte da progetti software reali. Ogni entry è una patch di codice associata a metadati dal sistema di issue tracking Jira, fornendo sia le modifiche al codice grezze sia informazioni contestuali su ciascuna correzione.

### Ingegnerizzazione delle Feature

Partendo dai dati grezzi, HotCat arricchisce lo spazio delle feature integrando:

- **Feature a livello di codice**: Estratte dai diff effettivi — linee aggiunte, linee rimosse, file modificati, complessità sintattica
- **Metadati di progetto da Jira**: Tempo di risoluzione, numero di partecipanti (sviluppatori, revisori), livelli di priorità e altri segnali organizzativi
- **Riassunti generati da LLM**: Ogni hotfix viene riassunto utilizzando un LLM per produrre descrizioni concise in linguaggio naturale di cosa fa la patch

Questi riassunti vengono poi trasformati in rappresentazioni vettoriali dense usando gli **embedding di Sentence-BERT**, che catturano il contenuto semantico di ogni descrizione. I vettori vengono organizzati attraverso il **clustering K-Means** per produrre la classificazione effettiva.

In totale, sono disponibili **18 feature** per la pipeline di classificazione, creando uno spazio di ricerca di 2^18 (oltre 260.000) possibili combinazioni di feature.

### Selezione Multi-Obiettivo delle Feature con NSGA-II

Questa è l'innovazione centrale. Anziché utilizzare tutte e 18 le feature o selezionarne manualmente un sottoinsieme, HotCat formula la selezione delle feature come un **problema di ottimizzazione multi-obiettivo** e lo risolve con **NSGA-II** (Non-dominated Sorting Genetic Algorithm II), implementato usando la libreria **pymoo**.

Ogni soluzione candidata è rappresentata come una **maschera binaria** — un vettore di 0 e 1 che indica quali feature includere. NSGA-II ottimizza simultaneamente tre obiettivi:

1. **Massimizzare l'accuratezza della classificazione**: Quanto bene le feature selezionate consentono una corretta categorizzazione dei bug
2. **Massimizzare l'Informazione Mutua Normalizzata (NMI)**: Una misura dell'accordo tra il clustering predetto e le etichette ground truth, robusta allo sbilanciamento delle dimensioni dei cluster
3. **Minimizzare il tempo computazionale di esecuzione**: Quanto velocemente la pipeline di classificazione viene eseguita con le feature selezionate

La ricerca evolutiva usa una **popolazione di 20 individui** evoluta per **20 generazioni**, con operatori di crossover binario e mutazione bit-flip adatti alla codifica binaria.

Attraverso questo processo, NSGA-II scopre un **fronte di Pareto** di soluzioni non dominate — configurazioni in cui migliorare un obiettivo peggiora necessariamente un altro. Questo offre ai professionisti un menù di opzioni tra cui scegliere in base ai propri vincoli specifici.

### Augmentazione dei Dati per la Robustezza

Le dimensioni ridotte del dataset HotBugs (88 entry) e il grave sbilanciamento delle classi pongono sfide per qualsiasi approccio di classificazione. HotCat affronta questo con una **strategia di augmentazione a due stadi**:

1. **Bilanciamento delle categorie**: Vengono generati esempi sintetici per equalizzare la rappresentazione delle categorie di bug rare
2. **Generazione di record post-ottimizzazione**: Dati aggiuntivi vengono creati dopo la fase di selezione delle feature per migliorare la generalizzazione

Questa augmentazione si è rivelata cruciale: **la performance di generalizzazione è migliorata dal 55% al 72%** — un guadagno di 17 punti percentuali che dimostra l'importanza di affrontare la scarsità dei dati in questo dominio.

## Risultati

Il fronte di Pareto ha rivelato diversi punti operativi praticamente utili:

- **Configurazione bilanciata**: **59% di accuratezza** e **0.58 NMI** con un tempo di esecuzione di soli **129 secondi**
- **Configurazione a massima accuratezza**: **63% di accuratezza** in **132 secondi** — solo 3 secondi aggiuntivi per un miglioramento di 4 punti percentuali in accuratezza

Questi risultati dimostrano una scoperta chiave: **qualità della classificazione ed efficienza computazionale non devono necessariamente essere in conflitto**. Una maggiore accuratezza era raggiungibile senza aumenti drastici nel consumo di risorse.

L'analisi ha anche rivelato quali feature contano di più. Non tutti i campi di metadati contribuiscono equamente alla qualità della classificazione — alcune feature in realtà **degradano le prestazioni** introducendo rumore. Eliminando selettivamente queste feature, HotCat ottiene risultati migliori con meno computazione, incarnando il principio del Green AI di fare di più con meno.

## Implicazioni

HotCat dimostra una metodologia pratica per il **Green AI nell'ingegneria del software**. Man mano che gli strumenti di analisi basati su LLM diventano parte integrante dei flussi di lavoro di sviluppo, la loro impronta energetica cumulativa diventa una preoccupazione concreta. La selezione multi-obiettivo delle feature offre un approccio principiato per tenere sotto controllo questa impronta.

Il framework è progettato per essere replicabile e scalabile, offrendo ai team un metodo per automatizzare l'analisi degli hotfix all'interno di sistemi di issue tracking come Jira rispettando i vincoli computazionali. Il lavoro futuro espanderà l'approccio a dataset più ampi ed esplorerà l'incorporazione di metriche dirette di emissione di carbonio come obiettivi di ottimizzazione.

---

*Pubblicato al 17° Symposium on Search-Based Software Engineering (SSBSE 2025), Challenge Track on Hot Fixing Benchmark. Questa ricerca è stata condotta presso la University College London (UCL) e l'Università degli Studi di Trieste.*
