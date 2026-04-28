---
title: "Il campo del Text-to-SQL ha un problema di misurazione"
date: 2025-07-02
draft: false
tags: ["Text-to-SQL", "Metriche di Valutazione", "Similarità Semantica", "SQL", "LLM"]
categories: ["Ricerca"]
description: "Tutti i benchmark text-to-SQL di oggi valutano le query come perfette o sbagliate. È un lancio di moneta vestito da metrica. Ne abbiamo costruita una che ti dice davvero quanto ti sei avvicinato."
ShowToc: true
TocOpen: false
cover:
  image: "/images/post-placeholder.svg"
  alt: "QAS — Query Accuracy Score combining semantic and table similarity"
  hiddenInList: false
---

{{< summary-box title="TL;DR" >}}
Il text-to-SQL è ovunque, ma lo misuriamo male. **Exact Match** ti penalizza se scrivi `users AS u`. **Execution Accuracy** non si interessa se hai azzeccato 99 righe su 100 — sbagliato è sbagliato. Abbiamo costruito **QAS (Query Accuracy Score)**: un punteggio continuo che combina similarità semantica code-aware (quanto è vicino il SQL?) con similarità di tabella basata su edit-distance (quanto è vicina la risposta?). Testato su 11 modelli su BIRD, QAS rivela differenze *enormi* che le metriche binarie schiacciano nello stesso numero.
{{< /summary-box >}}

## Un campo costruito su lanci di moneta

Il text-to-SQL è una di quelle aree dove le demo sembrano magiche. Scrivi una domanda in italiano, ottieni una query SQL, ottieni una risposta dal tuo database. Niente DBA. La promessa è enorme.

Il progresso è reale. La *misurazione* del progresso non lo è.

Guarda una qualunque leaderboard text-to-SQL e vedrai due metriche che fanno tutto il lavoro: **Exact Match** e **Execution Accuracy**. Entrambe binarie. Una query è corretta o non lo è. Non c'è una via di mezzo.

Questo è un problema. La maggior parte delle query che "falliscono" non è davvero spazzatura casuale — sono giuste all'80%, con un filtro sbagliato, una colonna mancante, una riga in più. Trattarle come identiche a "query completamente sbagliata sulla tabella sbagliata" butta via esattamente l'informazione che ci serve per migliorare i modelli.

## Perché Exact Match è una pessima battuta

Exact Match confronta due stringhe SQL carattere per carattere. Stringa uguale, score 1. Stringa diversa, score 0.

```sql
-- Reference
SELECT name FROM users WHERE age > 25

-- Generato
SELECT u.name FROM users AS u WHERE u.age > 25
```

Restituiscono risultati identici su ogni riga di ogni database dell'universo. Exact Match valuta la seconda come fallimento totale.

SQL è un linguaggio *dichiarativo*. Ci sono dozzine di modi validi per scrivere la stessa query. Alias, ordine delle JOIN, subquery contro JOIN, WHERE contro HAVING — tutta flessibilità sintattica, tutto semanticamente equivalente, tutto silurato da una metrica che confronta stringhe.

## Perché Execution Accuracy è meglio ma comunque sbagliata

Execution Accuracy almeno esegue le due query e confronta le tabelle dei risultati. Se le righe combaciano, score 1. Altrimenti, 0.

Risolve elegantemente il problema degli alias. E ne crea uno diverso: **niente credito parziale**.

Una query che restituisce 99 righe corrette su 100 prende zero. Una query che seleziona le colonne giuste dalle tabelle giuste ma con un filtro leggermente sbagliato prende zero. Una query contro tabelle completamente sbagliate — anch'essa zero.

Non sono lo stesso tipo di fallimento. Trattarli come uno solo distrugge segnale di cui ricercatori e professionisti hanno disperato bisogno.

## QAS: un punteggio continuo con due occhi

Abbiamo costruito **QAS — il Query Accuracy Score** — per sistemare la cosa. È un numero tra 0 e 1, e ha due componenti che misurano cose diverse:

![Pipeline QAS: similarità semantica incontra similarità di tabella](/images/scireports2025-text-to-sql-metrics/Figure2.png)

**S_C — Similarità Semantica.** Quanto sono vicine le *query stesse*? Embeddiamo entrambe le query con **UAE-Code-Large-V1**, un modello addestrato specificamente sul codice. Gli embedding di testo general-purpose non capiscono il SQL — non sanno che LEFT JOIN e RIGHT JOIN non sono sinonimi, o che le subquery possono essere funzionalmente identiche alle JOIN. Gli embedding code-specialized sì. Prendiamo la cosine similarity tra gli embedding. Quella è S_C.

**S_T — Similarità di Tabella.** Quanto sono vicini i *risultati*? Eseguiamo entrambe le query e confrontiamo le tabelle risultanti con edit distance — il numero minimo di inserimenti, cancellazioni e sostituzioni per trasformare una tabella nell'altra, normalizzato per dimensione. Sbagliata di una riga? S_T alto. Sbagliata su ogni valore? S_T basso.

Il punteggio finale è semplicemente:

> QAS = w · S_T + (1 − w) · S_C

Abbiamo testato quanto sia sensibile il ranking a *w* usando la distanza di Kendall e la risposta è: poco. I ranking sono stabili per w ∈ [0.25, 0.75]. Abbiamo scelto **w = 0.5** perché volevamo che intento e risultato pesassero ugualmente.

Un sotto-risultato piccolo ma importante: proxy semplici come "le tabelle dei risultati hanno lo stesso numero di righe?" non funzionano. Abbiamo misurato la correlazione tra similarità di forma e similarità di contenuto effettivo ed era essenzialmente zero. **Due tabelle con forma identica possono contenere dati completamente diversi.** L'edit distance sul contenuto effettivo è necessario.

## Cosa mostra QAS che le metriche binarie nascondono

Abbiamo applicato QAS a 11 modelli text-to-SQL sul benchmark BIRD — specialisti fine-tuned, sistemi general-purpose tipo GPT-4, modelli open-source di varie dimensioni.

Due risultati spiccano.

**Differenze nascoste tra modelli "equivalenti".** Due modelli con la stessa Execution Accuracy del ~65% possono avere *forme* di fallimento completamente diverse. Uno era un mediocre affidabile — sempre vicino, raramente perfetto. L'altro era bipolare — perfetto o catastrofico, niente in mezzo. Le metriche binarie li chiamano "lo stesso modello". Non lo sono.

**Capacità diagnostica.** Le due componenti di QAS funzionano come un piccolo kit diagnostico:

- **S_C alto, S_T basso** → il modello ha capito la query ma ha sbagliato i valori (filtro sbagliato, condizione mancante). Parla SQL, ma non sa fare i conti.
- **S_C basso, S_T alto** → query strutturalmente diversa, output simile. O una riformulazione intelligente o un match accidentale.
- **S_C basso, S_T basso** → il modello non ha capito la domanda.

Quella distinzione conta. Il primo caso si sistema con grounding migliore sul contenuto del database. Il terzo serve comprensione migliore dell'intento. Le metriche binarie non ti danno nessuna delle due — solo "sbagliato".

## Cosa abilita

Per i ricercatori: smettete di riportare un numero solo. Riportate una *distribuzione*. Mostrare che il vostro modello ha QAS medio più alto sui fallimenti rispetto alla baseline è un'affermazione significativa, anche quando l'accuratezza binaria è identica.

Per i professionisti: un modello accurato al 70% con QAS medio alto sui fallimenti è *molto diverso* da un modello accurato al 70% con QAS medio basso sui fallimenti. Il primo è "vicino la maggior parte delle volte, deploya con cautela". Il secondo è "successo binario, prepara il fallback". Le decisioni di deployment dipendono da questo.

Per il training: QAS è continuo e differenziabile in spirito. Potrebbe essere usato come segnale di reward più ricco rispetto ai segnali pass/fail su cui i modelli si allenano oggi. Non dobbiamo accontentarci della supervisione binaria quando finalmente la nostra metrica di valutazione ha trama.

Il TL;DR è semplice. **Non puoi ottimizzare quello che non vedi.** Le metriche binarie non vedono il gradiente di "quasi giusto". QAS sì.

---

### Reference

Questo post è una sintesi divulgativa di:

> Pinna, G., Manzoni, L., De Lorenzo, A., Castelli, M. (2025). *Beyond Exact Set and Execution Matches: Redefining Text-to-SQL Metrics with Semantic and Structural Similarity*. **Scientific Reports**, 15(1): 22357.
>
> [Leggi il paper originale (PDF)](/images/scireports2025-text-to-sql-metrics/2025_Scientific_Reports_Beyond_Exact_Set_and_Execution_Matches__Redefining_Text_to_SQL_Metrics_with_Semantic_and_Structural_Similarity.pdf) — [Codice su GitHub](https://github.com/giovannipinna96/sql_metric)

*Ricerca condotta presso l'Università degli Studi di Trieste e la NOVA Information Management School (NOVA IMS), Universidade Nova de Lisboa.*
