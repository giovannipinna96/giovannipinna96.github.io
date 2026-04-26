---
title: "Ridefinire le Metriche Text-to-SQL: Oltre la Valutazione Binaria"
date: 2025-07-02
draft: false
tags: ["Text-to-SQL", "Metriche di Valutazione", "Similarità Semantica", "SQL", "LLM"]
categories: ["Ricerca"]
description: "Presentazione del Query Accuracy Score (QAS), una metrica di valutazione continua per sistemi text-to-SQL che cattura l'intero spettro tra 'perfettamente corretto' e 'completamente sbagliato' combinando similarità semantica e strutturale."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
Le metriche di valutazione dominanti per i sistemi text-to-SQL — Exact Match e Execution Accuracy — sono entrambe binarie, valutando ogni query generata come completamente corretta o completamente sbagliata. Questo cancella distinzioni critiche tra query quasi corrette e quelle fondamentalmente errate. Introduciamo il Query Accuracy Score (QAS), una metrica continua che combina la similarità semantica (usando embedding di codice UAE-Code-Large-V1 e distanza coseno) con la similarità delle tabelle (usando il confronto basato sulla distanza di edit delle tabelle risultanti). Valutato su 11 modelli text-to-SQL attraverso il benchmark BIRD, il QAS rivela differenze di qualità nascoste invisibili alle metriche binarie: modelli con execution accuracy simile presentano profili di errore sorprendentemente diversi. La struttura a due componenti abilita inoltre la diagnosi differenziale — distinguendo tra fallimenti di intento (struttura della query sbagliata) e fallimenti di esecuzione (struttura corretta, valori sbagliati). Pubblicato su Scientific Reports, 2025.
{{< /summary-box >}}

## Introduzione

Il Text-to-SQL — il compito di tradurre automaticamente domande in linguaggio naturale in query SQL per database — ha visto enormi progressi grazie ai Large Language Models. Sistemi costruiti su GPT-4, modelli specializzati fine-tuned e varie alternative open-source possono ora gestire query sempre più complesse attraverso schemi di database diversificati. La promessa è trasformativa: permettere a chiunque di interrogare database senza competenze SQL.

Ma il progresso in qualsiasi campo è affidabile solo quanto le metriche utilizzate per misurarlo. E il campo del text-to-SQL ha un problema di misurazione. Le due metriche di valutazione dominanti — **Exact Match (EM)** e **Execution Accuracy (EX)** — sono entrambe binarie: assegnano a ogni query generata un punteggio di completamente corretta (1) o completamente sbagliata (0). Questa natura binaria crea un sostanziale punto cieco che può fuorviare sia ricercatori che professionisti.

Questo articolo, pubblicato su **Scientific Reports (2025)**, introduce il **Query Accuracy Score (QAS)** — una metrica continua che cattura il ricco spettro tra correttezza perfetta e fallimento totale, consentendo una valutazione più sfumata e un confronto tra modelli più informato.

## Il Problema delle Metriche Binarie

### Exact Match: Troppo Restrittivo

L'Exact Match confronta la stringa SQL generata carattere per carattere con una query di riferimento. Se sono testualmente identiche, il punteggio è 1; altrimenti, 0. Il problema fondamentale è che SQL è un linguaggio dichiarativo con ampia flessibilità sintattica. Consideriamo queste due query:

```sql
-- Query A
SELECT name FROM users WHERE age > 25

-- Query B
SELECT u.name FROM users AS u WHERE u.age > 25
```

Queste query sono semanticamente identiche — restituiscono esattamente gli stessi risultati su qualsiasi database. Ma l'Exact Match valuta la Query B come fallimento se la Query A è il riferimento. Alias di tabella, ordine dei JOIN, riordinamento delle clausole, riformulazioni subquery-versus-JOIN — tutti questi producono query funzionalmente equivalenti che EM tratta come errate.

### Execution Accuracy: Troppo Grossolana

L'Execution Accuracy migliora l'EM eseguendo effettivamente entrambe le query e confrontando le tabelle risultanti. Se gli output corrispondono, il punteggio è 1; altrimenti, 0. Questo gestisce elegantemente la variazione sintattica, ma introduce un problema diverso: **nessun credito parziale**.

Una query che restituisce 99 su 100 righe corrette riceve lo stesso punteggio (0) di una che restituisce dati completamente irrilevanti. Una query che seleziona le colonne giuste dalle tabelle giuste ma con una condizione di filtro leggermente sbagliata è trattata identicamente a una che interroga tabelle completamente diverse. Queste distinzioni sono critiche per comprendere le capacità dei modelli e guidare i miglioramenti, eppure l'execution accuracy binaria le cancella completamente.

### L'Impatto Pratico

Per i ricercatori, le metriche binarie rendono impossibile distinguere tra modelli "quasi arrivati" e modelli fondamentalmente fuori strada. Due modelli con il 70% di execution accuracy potrebbero avere profili di errore molto diversi — uno che commette consistentemente piccoli errori, l'altro che alterna output perfetti e fallimenti completi. Le metriche binarie non possono distinguere questi casi.

Per i professionisti che valutano la prontezza per il deployment, la differenza tra "di solito vicino al corretto" e "o perfetto o inutile" è enorme — ma invisibile sotto le metriche attuali.

## Il Query Accuracy Score (QAS)

Il QAS fornisce un valore continuo tra 0 e 1 combinando due misure di similarità complementari:

### Similarità Semantica (S_C)

Misuriamo la similarità strutturale tra query generate e di riferimento usando **modelli di embedding specializzati per il codice**. In particolare, impieghiamo **UAE-Code-Large-V1**, un modello addestrato per produrre rappresentazioni vettoriali significative del codice, incluso SQL.

La scelta progettuale chiave è l'uso di embedding specifici per il codice anziché di uso generale. I modelli linguistici standard non comprendono pienamente i costrutti specifici di SQL: l'equivalenza funzionale di diverse sintassi JOIN, il ruolo semantico delle clausole WHERE rispetto alle clausole HAVING, o il significato dell'annidamento di subquery. Gli embedding specializzati per il codice catturano queste sfumature, producendo punteggi di similarità che correlano con l'effettiva similarità funzionale.

La similarità coseno tra i vettori di embedding delle query generate e di riferimento ci fornisce S_C — una misura di quanto le due query siano simili in termini di intento strutturale.

### Similarità delle Tabelle (S_T)

Mentre la similarità semantica cattura l'intento, la similarità delle tabelle cattura i risultati. Eseguiamo entrambe le query sul database e confrontiamo le tabelle risultanti usando un **algoritmo basato sulla distanza di edit**.

Questo va ben oltre il confronto binario. L'algoritmo calcola il numero minimo di operazioni di modifica (inserimenti, cancellazioni, sostituzioni) necessarie per trasformare una tabella risultante nell'altra, normalizzato per la dimensione della tabella. Una tabella a cui manca una riga ottiene un alto punteggio di similarità; una tabella con dati completamente diversi ottiene un punteggio basso.

Abbiamo anche dimostrato che semplici proxy strutturali — come confrontare il numero di righe o colonne — non sono affidabili. La nostra analisi ha mostrato essenzialmente **nessuna correlazione tra differenze nelle dimensioni delle tabelle e l'effettiva similarità del contenuto**. Tabelle con forme identiche possono contenere dati completamente diversi, confermando la necessità di un confronto a livello di contenuto.

### Combinare le Componenti

Il QAS finale è una combinazione pesata:

> QAS = w × S_T + (1 − w) × S_C

Abbiamo analizzato la sensibilità del parametro di peso w usando la **distanza di Kendall** tra le classifiche dei modelli a diverse impostazioni. Le classifiche erano stabili attraverso valori intermedi (w = 0.25, 0.5, 0.75), indicando che il QAS è robusto alla specifica scelta del peso. Abbiamo selezionato **w = 0.5** per pesare equamente entrambe le componenti.

## Valutazione Sperimentale

Abbiamo valutato il QAS sul **benchmark BIRD**, un dataset impegnativo di query per database del mondo reale attraverso domini diversi. Abbiamo valutato **11 modelli text-to-SQL**, includendo:

- Modelli specialistici fine-tuned progettati specificamente per il text-to-SQL
- LLM general-purpose (GPT-4 e varianti)
- Varie alternative open-source di diverse dimensioni

### Risultati Chiave

**Distinzioni nascoste rivelate.** Modelli che apparivano equivalenti sotto metriche binarie mostravano differenze significative sotto QAS. Due modelli con punteggi EX simili intorno al 65% risultavano avere profili di errore sorprendentemente diversi: uno produceva query consistentemente mediocri (punteggi QAS moderati su tutta la linea), mentre l'altro era più "tutto-o-niente" (QAS alto sui successi, QAS molto basso sui fallimenti).

**Capacità diagnostica.** La struttura a due componenti del QAS abilita la diagnosi differenziale:
- **Alto S_C, basso S_T**: Il modello comprende l'intento della query ma commette errori a livello di esecuzione (valori di filtro sbagliati, condizioni mancanti).
- **Basso S_C, alto S_T**: Query strutturalmente diverse che per caso producono risultati simili.
- **Basso S_C, basso S_T**: Incomprensione fondamentale dei requisiti della query.

Questa informazione diagnostica è inestimabile per il miglioramento mirato dei modelli — una capacità che le metriche binarie semplicemente non possono fornire.

**Classifiche stabili.** Le classifiche dei modelli prodotte dal QAS erano consistenti attraverso diverse configurazioni di peso, suggerendo che la metrica cattura differenze di qualità sottostanti robuste.

## Implicazioni Più Ampie

Per la comunità di ricerca, il QAS abilita un benchmarking più informativo. Invece di riportare un singolo numero di accuratezza binaria, i ricercatori possono caratterizzare la distribuzione completa della qualità delle query, permettendo confronti tra modelli più sfumati e miglioramenti architetturali più mirati.

Per i professionisti, il QAS fornisce una valutazione più onesta delle capacità dei modelli. Un sistema con 70% di accuratezza binaria e alto QAS medio sui fallimenti è fondamentalmente diverso da uno con 70% di accuratezza e basso QAS medio sui fallimenti — e le decisioni di deployment dovrebbero riflettere questa differenza.

Guardando al futuro, il QAS potrebbe potenzialmente servire come **obiettivo di addestramento**, fornendo feedback continuo e differenziabile durante il training del modello anziché segnali binari passa/fallisce. Questo potrebbe cambiare radicalmente il modo in cui i modelli text-to-SQL apprendono, abilitando un'ottimizzazione basata sul gradiente verso la qualità delle query invece di affidarsi esclusivamente alla supervisione binaria.

---

*Pubblicato su Scientific Reports, 15.1: 22357, 2025. Questa ricerca è stata condotta presso l'Università degli Studi di Trieste e la NOVA Information Management School (NOVA IMS), Universidade Nova de Lisboa. Codice disponibile su [github.com/giovannipinna96/sql_metric](https://github.com/giovannipinna96/sql_metric).*
