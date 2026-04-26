---
title: "Esplorare l'Effetto del Genetic Improvement sul Codice Generato dagli LLM"
date: 2025-07-01
draft: false
tags: ["Genetic Improvement", "LLM", "Selezione Lexicase", "Down-Sampling", "Generazione di Codice"]
categories: ["Ricerca"]
description: "Uno studio esteso sull'uso del Genetic Improvement con strategie di selezione avanzate per migliorare il codice generato dagli LLM, dimostrando miglioramenti in 11 su 12 combinazioni modello-problema."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
Questo articolo estende il nostro lavoro di EuroGP 2024 sul miglioramento del codice generato dagli LLM tramite Genetic Improvement, introducendo tre avanzamenti chiave: la selezione lexicase per preservare soluzioni specialiste, il down-sampling al 10% per l'efficienza computazionale e una funzione di fitness raffinata (F_E) con punteggio a credito parziale. Valutato su GPT-4, ChatGPT, Code Llama 7B e LLaMA 3 8B attraverso tre problemi PSB2 con popolazioni di 200 individui per 100 generazioni, la pipeline migliorata ottiene miglioramenti in 11 su 12 combinazioni modello-problema. I guadagni più forti si osservano sui modelli più piccoli, rafforzando la scoperta che il Genetic Improvement agisce come "amplificatore di capacità" — compensando la capacità limitata del modello in modo più efficace di quanto non migliori modelli già forti. Pubblicato su SN Computer Science, 2025.
{{< /summary-box >}}

## Introduzione

Nel nostro precedente lavoro a EuroGP 2024, abbiamo dimostrato che il Genetic Improvement (GI) combinato con l'Evoluzione Grammaticale può migliorare sistematicamente il codice generato dai Large Language Models. Quello studio ha stabilito la fattibilità fondamentale dell'approccio. Ma diverse domande rimanevano aperte: la ricerca evolutiva potrebbe essere resa più efficace con migliori strategie di selezione? L'approccio funzionerebbe anche con modelli più recenti e capaci? E potremmo sviluppare una funzione di fitness più informativa che fornisca un feedback più granulare al processo evolutivo?

Questo articolo, pubblicato su **SN Computer Science (2025)**, affronta tutte e tre le domande. Abbiamo esteso il nostro framework originale con la **selezione lexicase**, il **down-sampling** e una **funzione di fitness raffinata**, valutando poi la pipeline migliorata su LLM aggiornati tra cui Code Llama e LLaMA 3.

## Avanzamento della Strategia di Selezione

### I Limiti della Selezione a Torneo

Il nostro approccio originale utilizzava la selezione a torneo — una strategia semplice in cui gli individui competono in piccoli gruppi casuali e il migliore viene selezionato per la riproduzione. Sebbene efficace e computazionalmente economica, la selezione a torneo ha una limitazione ben documentata: tende a favorire i generalisti rispetto agli specialisti. Un individuo che performa moderatamente bene su tutti i casi di test verrà preferito rispetto a uno che risolve perfettamente un sottoinsieme di casi ma fallisce su altri.

Nella sintesi di programmi, questo è uno svantaggio significativo. Una variante di programma che gestisce perfettamente tutti i casi limite con interi ma fallisce sui numeri negativi contiene soluzioni parziali preziose — ma la selezione a torneo potrebbe scartarla a favore di un mediocre generalista.

### Selezione Lexicase

La **selezione lexicase** affronta questo problema valutando gli individui sui casi di test uno alla volta in ordine casuale. Il processo di selezione parte dall'intera popolazione, poi filtra iterativamente gli individui che non sono tra i migliori su ciascun caso di test successivo. Questo preserva naturalmente gli specialisti — individui che eccellono su sottoinsiemi specifici di casi di test sopravvivono anche se la loro performance complessiva non è eccezionale.

I vantaggi teorici della selezione lexicase per la sintesi di programmi sono ben stabiliti nella letteratura sul calcolo evolutivo. Il nostro contributo è dimostrarne l'efficacia nel contesto specifico del miglioramento del codice generato dagli LLM.

### Down-Sampling per l'Efficienza

La selezione lexicase diventa computazionalmente costosa quando il numero di casi di test è grande (nel nostro caso, fino a 1.000 per problema). Il **down-sampling** affronta questo selezionando casualmente un sottoinsieme di casi di test per ogni generazione. Questo riduce il costo computazionale per generazione mantenendo la pressione selettiva sull'intera suite di test nel tempo, poiché sottoinsiemi diversi vengono campionati in ogni generazione.

Abbiamo sperimentato un tasso di down-sampling del 10% — utilizzando solo 100 dei 1.000 casi di test disponibili per generazione — e abbiamo trovato che questo forniva un eccellente equilibrio tra efficienza e qualità della selezione.

## Una Funzione di Fitness Più Informativa

La nostra funzione di fitness originale era la semplice proporzione di casi di test superati. Sebbene intuitiva, questa valutazione binaria per caso di test scarta informazioni utili. Consideriamo un caso di test che si aspetta l'output `[1, 2, 3, 4, 5]`: un programma che produce `[1, 2, 3, 4, 6]` (un elemento sbagliato) ottiene lo stesso punteggio di uno che produce `"hello"` (completamente sbagliato).

Abbiamo sviluppato una funzione di fitness raffinata **F_E** che incorpora il credito parziale. Invece di verificare solo se ogni output corrisponde esattamente, F_E misura quanto ogni output è *vicino* al risultato atteso. Per output numerici, questo potrebbe usare la differenza assoluta; per sequenze, potrebbe considerare il confronto elemento per elemento. Questo feedback più granulare aiuta il processo evolutivo a distinguere tra varianti "quasi corrette" e "completamente sbagliate", fornendo migliori informazioni di gradiente per la ricerca.

## Setup Sperimentale

Abbiamo valutato la pipeline migliorata su **tre problemi PSB2** selezionati per i loro diversi livelli di difficoltà, utilizzando **quattro LLM**:

- **GPT-4**: Il modello di punta di OpenAI
- **ChatGPT (GPT-3.5-turbo)**: Il modello conversazionale ampiamente utilizzato
- **Code Llama 7B**: Il modello specializzato per il codice di Meta
- **LLaMA 3 8B**: Il modello open-source di ultima generazione di Meta

I parametri evolutivi sono stati aggiustati rispetto al nostro studio originale:

- **Dimensione della popolazione**: 200 (ridotta da 1.000, riflettendo la strategia di selezione più efficiente)
- **Generazioni**: fino a 100
- **Selezione lexicase** con **10% di down-sampling**
- Ogni esperimento ripetuto 30 volte per robustezza statistica

## Risultati

La pipeline migliorata ha ottenuto miglioramenti in **11 su 12 combinazioni modello-problema** — un risultato notevolmente consistente che valida sia la strategia di selezione lexicase sia la funzione di fitness raffinata.

I risultati chiave includono:

**Guadagni più forti sui modelli più piccoli.** Coerentemente con il nostro lavoro precedente, i maggiori miglioramenti relativi sono stati osservati sui modelli meno capaci. Code Llama 7B e LLaMA 3 8B — entrambi significativamente più piccoli di GPT-4 — hanno mostrato i guadagni più drammatici dal GI. Questo rinforza la scoperta che il GI è particolarmente prezioso come "amplificatore di capacità" per modelli open-source o con risorse limitate.

**La selezione lexicase preserva diversità utile.** Le dinamiche di popolazione sotto selezione lexicase erano qualitativamente diverse dalla selezione a torneo. Abbiamo osservato una maggiore diversità fenotipica mantenuta durante l'intera esecuzione evolutiva, con la popolazione contenente specialisti per diversi sottoinsiemi di casi di test. Questa diversità si è tradotta in una migliore esplorazione dello spazio delle soluzioni.

**Il down-sampling è efficace.** Utilizzare solo il 10% dei casi di test per generazione non ha degradato significativamente la qualità delle soluzioni rispetto all'uso dell'intera suite di test, riducendo sostanzialmente il costo computazionale. Questo rende l'approccio più pratico per il deployment nel mondo reale dove i budget di valutazione sono limitati.

**La funzione di fitness raffinata aiuta di più sui problemi più difficili.** Per problemi facili dove il codice iniziale dell'LLM è già vicino alla correttezza, la funzione di fitness binaria e F_E producono risultati simili. Ma per problemi più difficili dove il codice iniziale è lontano dalla correttezza, F_E fornisce informazioni di gradiente significativamente migliori, aiutando la ricerca a navigare verso le soluzioni in modo più efficiente.

## Confronto con la Self-Correction

Abbiamo nuovamente confrontato il nostro approccio GI con la self-correction dell'LLM, confermando e rafforzando i nostri risultati precedenti. L'approccio evolutivo ha costantemente superato la self-correction, in particolare sui problemi dove il codice iniziale aveva problemi strutturali fondamentali che il meccanismo di correzione dell'LLM stesso non riusciva a superare.

Questa è un'importante scoperta pratica: significa che anche per le organizzazioni che già utilizzano la self-correction nelle loro pipeline di generazione di codice LLM, l'aggiunta di una fase GI fornisce miglioramenti aggiuntivi e complementari.

## Limitazioni e Direzioni Future

Abbiamo identificato diverse limitazioni importanti che guidano la ricerca futura:

1. **Dipendenza dall'oracolo**: Il nostro approccio richiede casi di test (o un oracolo) per valutare il fitness. I problemi senza suite di test chiare sono più difficili da affrontare.
2. **Scalabilità**: La valutazione attuale riguarda programmi relativamente piccoli. Scalare a task di ingegneria del software più grandi e multi-file rimane una sfida aperta.
3. **Bias dell'LLM nella grammatica**: La grammatica generata dinamicamente eredita bias strutturali dall'output dell'LLM, potenzialmente limitando lo spazio dei miglioramenti scopribili.

Queste limitazioni motivano il lavoro in corso su approcci GI senza grammatica, tecniche di approssimazione del fitness e integrazione con flussi di lavoro di ingegneria del software su scala più ampia.

---

*Pubblicato su SN Computer Science, Volume 6, Numero 7, 2025. Questa ricerca è stata condotta presso l'Università degli Studi di Trieste e la NOVA Information Management School (NOVA IMS), Universidade Nova de Lisboa.*
