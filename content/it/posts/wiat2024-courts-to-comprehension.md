---
title: "Dai Tribunali alla Comprensione: Gli LLM Possono Rendere le Sentenze Più Accessibili?"
date: 2024-12-10
draft: false
tags: ["LLM", "NLP Giuridico", "Riassunto Testuale", "Fine-Tuning", "Accessibilità"]
categories: ["Ricerca"]
description: "Uno studio di valutazione umana che esamina se i riassunti generati dagli LLM delle sentenze della Corte Costituzionale italiana possono eguagliare la comprensibilità delle massime scritte da esperti."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
Le sentenze giuridiche sono in gran parte inaccessibili ai non esperti a causa del linguaggio complesso e dei riferimenti legali impliciti. In questo studio indaghiamo se i riassunti generati dagli LLM delle decisioni della Corte Costituzionale italiana possano eguagliare la comprensibilità delle massime scritte da esperti. In uno studio di valutazione umana con 75 partecipanti (25% con background giuridico, 75% senza), abbiamo confrontato quattro tipi di testo: sentenze originali, massime degli esperti, riassunti di GPT-4o e riassunti di LLaMA 2 fine-tuned (addestrato su 10.000 coppie sentenza-massima). Le massime degli esperti hanno ottenuto il punteggio più alto di comprensione (45%), seguite dai riassunti di GPT-4o (38%), dalle sentenze originali (33%) e da LLaMA 2 fine-tuned (30%). Tuttavia, GPT-4o ha mostrato un preoccupante problema di "errori sicuri" — producendo riassunti fluenti ma imprecisi che davano ai lettori falsa sicurezza nella propria comprensione, un problema particolarmente grave nel dominio giuridico. Pubblicato a IEEE WI-IAT 2024.
{{< /summary-box >}}

## Introduzione

Il linguaggio giuridico è notoriamente difficile da comprendere. Le sentenze dei tribunali, in particolare, sono scritte da e per professionisti del diritto, impiegando vocabolario specializzato, strutture sintattiche complesse e riferimenti impliciti a dottrine legali che le rendono in gran parte opache ai non esperti. Questo crea una significativa barriera di accessibilità: i cittadini le cui vite sono influenzate dalle decisioni giudiziarie spesso non possono comprendere il ragionamento alla base di tali decisioni.

Nel sistema giuridico italiano, la Corte Costituzionale pubblica due tipi di documenti per ogni decisione: la sentenza completa (*sentenza*), che contiene il ragionamento giuridico integrale, e un riassunto condensato chiamato *massima* (plurale: *massime*), scritto da esperti giuridici per catturare le conclusioni chiave e la motivazione. Le massime sono progettate per essere più accessibili delle sentenze complete, ma richiedono comunque una considerevole alfabetizzazione giuridica per essere comprese.

Questo pone una domanda avvincente: **i Large Language Models possono generare riassunti delle sentenze che siano comprensibili ai non esperti quanto le massime scritte da esperti — o anche di più?**

Questo articolo, pubblicato a **IEEE WI-IAT 2024** (la 23ª Conferenza Internazionale IEEE/WIC su Web Intelligence e Intelligent Agent Technology), affronta questa domanda attraverso un rigoroso studio di valutazione umana.

## Progettazione dello Studio

### I Quattro Tipi di Testo

Abbiamo confrontato quattro diverse versioni dello stesso contenuto giuridico:

1. **Sentenze originali** (*sentenze*): Il testo completo così come pubblicato dalla Corte Costituzionale
2. **Massime degli esperti**: Riassunti scritti da professionisti del diritto
3. **Riassunti di GPT-4o**: Generati chiedendo al modello GPT-4o di OpenAI di riassumere le sentenze originali
4. **Riassunti di LLaMA 2 fine-tuned**: Generati da un modello LLaMA 2 7B specializzato specificamente su testo giuridico italiano

### Processo di Fine-Tuning

Il modello fine-tuned è stato addestrato su un corpus di **10.000 coppie sentenza-massima** dagli archivi della Corte Costituzionale. Questo dataset ha fornito al modello ampi esempi di come gli esperti legali distillano sentenze complesse in riassunti concisi. Abbiamo anche valutato modelli aggiuntivi nella pipeline di fine-tuning, tra cui **Gemma 2B e 7B** e **LLaMantino 7B** (una variante di LLaMA specializzata per l'italiano), selezionando infine il modello con le migliori prestazioni per la valutazione umana.

### Protocollo di Valutazione Umana

Abbiamo reclutato **75 partecipanti** per lo studio, divisi in due gruppi in base alle loro conoscenze giuridiche:

- **25% con conoscenze giuridiche** (studenti di giurisprudenza, professionisti legali)
- **75% senza conoscenze giuridiche** (pubblico generale)

Ogni partecipante ha letto e valutato riassunti attraverso i quattro tipi di testo, giudicandoli su molteplici dimensioni di comprensione. La valutazione è stata progettata come uno **studio between-subjects** per evitare effetti di apprendimento — i partecipanti vedevano ogni caso sottostante una sola volta, in uno dei quattro formati testuali.

## Risultati

### Valutazioni di Comprensione

Le massime degli esperti hanno raggiunto il punteggio di comprensione complessivo più alto (**45%**), confermando il loro valore come riassunti giuridici accessibili. Tuttavia, il divario con le alternative generate dall'IA era più piccolo di quanto ci si potesse aspettare:

- **Massime degli esperti**: 45% di comprensione
- **Riassunti di GPT-4o**: 38% di comprensione
- **Sentenze originali**: 33% di comprensione
- **LLaMA 2 fine-tuned**: 30% di comprensione

I riassunti di GPT-4o hanno significativamente superato le sentenze originali, dimostrando che la summarizzazione tramite LLM rende effettivamente il testo giuridico più accessibile, anche senza fine-tuning specifico per il dominio. Curiosamente, il modello LLaMA 2 fine-tuned ha performato leggermente al di sotto delle sentenze originali, suggerendo che il fine-tuning su un modello più piccolo con capacità limitata potrebbe non essere sufficiente per questo compito complesso.

### Il Problema degli Errori Sicuri

Una delle scoperte più significative riguardava la **tendenza di GPT-4o a produrre informazioni errate ma dichiarate con sicurezza**. I partecipanti che leggevano i riassunti di GPT-4o mostravano un tasso più alto di risposte sicure ma sbagliate alle domande di comprensione. Questo è particolarmente preoccupante nel dominio giuridico, dove una comprensione errata di una sentenza può avere conseguenze reali.

Lo stile di scrittura fluente e autorevole del modello crea un'illusione di affidabilità che può indurre i lettori ad accettare riassunti imprecisi. Questo pattern di "errore sicuro" è ben documentato nella ricerca sugli LLM in generale, ma le sue implicazioni per il testo giuridico sono particolarmente serie.

### Effetto delle Conoscenze Giuridiche

Il background giuridico dei partecipanti ha influenzato significativamente i pattern di comprensione. Coloro con conoscenze giuridiche mostravano una comprensione più uniforme tra i tipi di testo, mentre coloro senza conoscenze giuridiche erano più fortemente influenzati dal formato del testo. Questo suggerisce che i riassunti generati dagli LLM possono essere particolarmente preziosi — ma anche particolarmente rischiosi — per il loro pubblico previsto di non esperti.

### Analisi Statistica

Abbiamo impiegato **test del Chi-quadrato** per valutare la significatività statistica delle differenze tra i tipi di testo. L'analisi ha confermato che le differenze tra le massime degli esperti e i riassunti generati dagli LLM erano statisticamente significative, così come le differenze tra i riassunti di GPT-4o e quelli di LLaMA 2 fine-tuned.

## Implicazioni

### Per la Tecnologia Giuridica

I risultati suggeriscono che gli LLM attuali si stanno avvicinando ma non hanno ancora raggiunto la qualità della summarizzazione giuridica esperta. GPT-4o produce riassunti significativamente più accessibili delle sentenze grezze, il che ha un valore pratico immediato. Tuttavia, il problema degli errori sicuri significa che il deployment non supervisionato di riassunti giuridici generati dagli LLM comporta dei rischi.

Una strategia di deployment pratica potrebbe coinvolgere **bozze generate dagli LLM riviste da esperti giuridici** — combinando la scalabilità della summarizzazione automatizzata con l'accuratezza della supervisione umana. Questo approccio ibrido potrebbe rendere le informazioni giuridiche più accessibili gestendo al contempo il rischio di errori.

### Per la Ricerca sull'IA

Lo studio evidenzia il divario tra fluenza e accuratezza negli output degli LLM. I modelli possono produrre testo che si legge in modo convincente e appare autorevole pur contenendo errori sostanziali. Sviluppare metodologie di valutazione che rilevino in modo affidabile questo pattern — oltre le semplici metriche di fluenza o coerenza — è un importante problema di ricerca aperto.

### Per la Società

Migliorare l'accessibilità delle informazioni giuridiche è fondamentalmente una questione di partecipazione democratica. Quando i cittadini non possono comprendere le decisioni legali che influenzano le loro vite, i principi di trasparenza e responsabilità vengono compromessi. I nostri risultati suggeriscono che gli LLM possono contribuire a questo obiettivo, ma un deployment attento con appropriate salvaguardie è essenziale.

---

*Pubblicato alla 23ª Conferenza Internazionale IEEE/WIC su Web Intelligence e Intelligent Agent Technology (WI-IAT 2024), dicembre 2024. Questa ricerca è stata condotta presso l'Università degli Studi di Trieste e la NOVA Information Management School (NOVA IMS), Universidade Nova de Lisboa.*
