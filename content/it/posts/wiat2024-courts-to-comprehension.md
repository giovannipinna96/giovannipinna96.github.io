---
title: "GPT-4 può rendere le sentenze più leggibili. Può anche raccontartele in modo sbagliato, con sicurezza."
date: 2024-12-10
draft: false
tags: ["LLM", "NLP Giuridico", "Riassunto Testuale", "Fine-Tuning", "Accessibilità"]
categories: ["Ricerca"]
description: "Abbiamo chiesto a 75 persone di leggere riassunti di sentenze della Corte Costituzionale italiana — scritti da esperti, da GPT-4o, da un LLaMA fine-tuned, e le sentenze grezze stesse. I risultati dicono più sugli LLM che sui tribunali."
ShowToc: true
TocOpen: false
cover:
  image: "/images/post-placeholder.svg"
  alt: "Comprehension rates across text types and educational background"
  hiddenInList: false
---

{{< summary-box title="TL;DR" >}}
Le sentenze della Corte Costituzionale italiana sono scritte per gli avvocati. La maggior parte dei cittadini non riesce a seguirle. Un LLM può sistemare la cosa? Abbiamo condotto uno studio umano con 75 persone confrontando quattro versioni dello stesso contenuto giuridico: sentenze originali, massime di esperti, riassunti di GPT-4o, e un LLaMA 2 7B fine-tuned. Tassi di comprensione: **massime di esperti 45%**, **GPT-4o 38%**, **sentenze grezze 33%**, **LLaMA fine-tuned 30%**. GPT-4o rende davvero più leggibile il testo giuridico. Produce anche un pattern preoccupante: **sicuro, fluente, sbagliato** — i lettori se ne vanno con comprensioni errate ma fortemente possedute. Usa gli LLM per il riassunto giuridico. Non usarli senza supervisione umana.
{{< /summary-box >}}

## Un problema democratico travestito da problema tecnico

Le sentenze della Corte Costituzionale sono tra i documenti più importanti che un paese produce. Definiscono cosa il tuo governo può e non può fare. Modellano quali sono i tuoi diritti. Sono anche scritte da giuristi, per giuristi, in una densa prosa giuridica italiana che il cittadino medio non ha alcuna possibilità di comprendere.

La Corte Costituzionale italiana cerca già di risolvere la cosa. Per ogni sentenza pubblica una *massima* — un riassunto condensato scritto da esperti giuridici il cui mestiere intero è rendere accessibile la giurisprudenza. Le massime sono meglio delle sentenze complete, ma assumono comunque un'alfabetizzazione giuridica che la maggior parte delle persone non ha.

Quindi la domanda è semplice: **un LLM può aiutare a chiudere il divario?** Non sostituendo i giudici o gli esperti giuridici, ma producendo riassunti che un cittadino normale possa davvero leggere?

Abbiamo fatto l'esperimento.

## Cosa abbiamo testato

Quattro versioni dello stesso contenuto giuridico:

1. **Sentenze originali** — il testo grezzo dalla Corte.
2. **Massime di esperti** — i riassunti scritti da umani, il nostro tetto di qualità.
3. **Riassunti di GPT-4o** — generati promptando GPT-4o di OpenAI su ciascuna sentenza.
4. **LLaMA 2 7B fine-tuned** — un modello open-source più piccolo addestrato su **10.000 coppie sentenza-massima** estratte dagli archivi della Corte.

Abbiamo provato anche Gemma 2B/7B e LLaMantino 7B (un LLaMA italianizzato) nella pipeline di fine-tuning; LLaMA 2 7B è risultato il migliore, quindi rappresenta il lato open-source nello studio umano.

## Come abbiamo misurato la comprensione

75 partecipanti. Circa il **25% con conoscenza giuridica** (studenti di legge, professionisti), il **75% senza** (pubblico generale). Ogni persona ha letto riassunti su tutti i tipi di testo e ha risposto a domande di comprensione sul contenuto effettivo.

L'abbiamo eseguito come disegno between-subjects — ogni caso sottostante è stato visto da ciascun partecipante in uno solo dei quattro formati — per eliminare effetti di apprendimento. Le differenze tra formati sono state testate con il chi-quadro.

![Comprensione per titolo di studio sui vari tipi di testo](/images/wiat2024-courts-to-comprehension/percentuale_delle_risposte_corrette_per_titolo_di_studio.png)

## Cosa abbiamo trovato

I numeri principali:

- **Massime di esperti: 45%** di comprensione. Il nostro tetto, come previsto.
- **GPT-4o: 38%** di comprensione. Significativamente meglio delle sentenze grezze.
- **Sentenze originali: 33%** di comprensione. Lo status quo.
- **LLaMA 2 7B fine-tuned: 30%** di comprensione. Leggermente *peggio* della sentenza grezza.

Quest'ultimo merita una pausa. Fare il fine-tuning di un piccolo modello open su 10.000 riassunti di esperti non ha aiutato. Ha peggiorato. La capacità conta; per questo task, 7B parametri sembra essere troppo poco per interiorizzare la comprensione strutturale che rende buona una massima.

GPT-4o, dall'altra parte, dà un significativo +5 punti rispetto a leggere la sentenza da soli. È reale.

## E poi diventa scomodo

Ecco la parte che dovrebbe darti da pensare.

Quando abbiamo guardato *quali tipi di risposte sbagliate* davano le persone, i lettori di GPT-4o mostravano un tasso molto più alto di **errori sicuri**. Non solo capivano male — uscivano con comprensioni forti, definitive e *sbagliate* di cosa avesse stabilito la Corte.

Il testo era fluente. Autorevole. Liscio. Si leggeva come se l'avesse scritto un esperto. E nei casi in cui era sbagliato, quella fluenza rendeva i lettori più sicuri, non meno, di aver capito.

Non è unico al riassunto giuridico. È il noto pattern degli LLM di *confabulazione fluente*. Ma la posta in gioco cambia radicalmente quando il tema è "cosa ha detto la Corte Costituzionale sui tuoi diritti". Un lettore sicuro di sé ma sbagliato di una sentenza è peggio di un lettore confuso. La confusione ti spinge a chiedere. La sicurezza no.

## Cosa ha fatto il titolo di studio sul quadro

I partecipanti con conoscenza giuridica avevano una comprensione più uniforme su tutti i tipi di testo — leggevano la sentenza originale all'incirca come il riassunto. Il formato contava meno perché portavano il proprio grounding.

I partecipanti senza conoscenza giuridica erano enormemente dipendenti dal formato. Hanno beneficiato di più di un buon riassunto — ed erano i più vulnerabili a un riassunto sicuro ma sbagliato.

In altre parole: **le persone che il riassunto LLM dovrebbe aiutare sono anche le persone più esposte alle sue modalità di fallimento**. È il vincolo di design che chiunque deployi questo tipo di strumento deve prendere sul serio.

## Cosa farne davvero

Tre conclusioni concrete.

**Per chi costruisce legal-tech:** i riassunti LLM di testo giuridico sono una vera vincita sull'accessibilità, ma il deployment grezzo è pericoloso. Il pattern giusto è **bozze LLM, revisione esperta** — usa il modello per la scala, l'umano per l'accuratezza. Il risparmio è nel *revisionare* una bozza invece di scriverla da zero.

**Per i ricercatori AI:** le metriche di valutazione che premiano fluenza e coerenza *mancano completamente questa classe di fallimenti*. Servono metodi di valutazione che cerchino specificamente l'errore sicuro. Un riassunto che si legge magnificamente ma ti dice la cosa sbagliata è un fallimento peggiore di uno goffo ma corretto.

**Per tutti gli altri:** quando leggi un documento giuridico riassunto da un'AI — o qualunque documento ad alta posta in gioco — calibra. La sicurezza nella prosa non è prova della verità della prosa. La fluenza è la confezione, non il contenuto.

## Il quadro più ampio

L'accessibilità dell'informazione giuridica è, alla fine, una questione di partecipazione democratica. Quando i cittadini non possono leggere le sentenze che governano le loro vite, i principi di trasparenza e responsabilità si erodono. Gli LLM possono aiutare a chiudere quel divario. Possono anche, se deployati senza cura, allargare un divario diverso — quello tra ciò che le persone *pensano* di aver capito e ciò che hanno effettivamente capito.

La tecnologia è pronta ad assistere. Non è pronta a essere lasciata da sola.

---

### Reference

Questo post è una sintesi divulgativa di:

> Pinna, G., Manzoni, L., De Lorenzo, A., Castelli, M. (2024). *From Courts to Comprehension: Can LLMs Make Judgments More Accessible?*. In: **Proceedings of the 23rd IEEE/WIC International Conference on Web Intelligence and Intelligent Agent Technology (WI-IAT 2024)**, dicembre 2024.
>
> [Leggi il paper originale (PDF)](/images/wiat2024-courts-to-comprehension/2024_WI_IAT_From_Courts_to_Comprehension__Can_LLMs_Make_Judgments_More_Accessible_.pdf)

*Ricerca condotta presso l'Università degli Studi di Trieste e la NOVA Information Management School (NOVA IMS), Universidade Nova de Lisboa.*
