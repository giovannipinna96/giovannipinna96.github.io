---
title: "Confrontare gli Agenti di Codifica AI: Un'Analisi Stratificata per Task dell'Accettazione delle Pull Request"
date: 2026-04-14
draft: false
tags: ["Agenti di Codifica AI", "Pull Request", "Studio Empirico", "Ingegneria del Software", "MSR"]
categories: ["Ricerca"]
description: "Uno studio empirico su larga scala di 7.156 pull request attraverso cinque agenti di codifica AI, che rivela come il tipo di task sia il fattore dominante nell'accettazione delle PR — più della scelta dell'agente — e che nessun singolo agente vince su tutte le categorie."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
Presentiamo uno studio empirico su larga scala di 7.156 pull request create da cinque agenti di codifica AI — OpenAI Codex, Claude Code, Cursor, Devin e GitHub Copilot — in repository open-source reali. Stratificando i risultati per tipo di task (bug fixing, implementazione di feature, documentazione, refactoring, aggiornamento dipendenze e testing), scopriamo che il tipo di task è il fattore dominante nell'accettazione delle PR, con un divario di 29 punti percentuali tra le migliori e le peggiori categorie — di gran lunga superiore alla varianza tra agenti all'interno di qualsiasi singola categoria. Nessun singolo agente supera tutti gli altri in tutti i tipi di task: Codex ottiene i risultati general-purpose più consistenti, Claude Code eccelle nei task di documentazione e Cursor primeggia nel bug fixing. Questi risultati suggeriscono che i team di sviluppo dovrebbero adottare un approccio a "portfolio di agenti", abbinando ciascun agente ai task dove performa meglio. Pubblicato a MSR 2026, Mining Challenge.
{{< /summary-box >}}

## Introduzione

Gli agenti di codifica AI sono passati da prototipi di ricerca a strumenti di produzione con una velocità notevole. GitHub Copilot, OpenAI Codex, Devin, Cursor, Claude Code — questi sistemi non sono più limitati alla scrittura di frammenti di codice o al completamento di funzioni. Creano pull request intere, implementano feature end-to-end, correggono bug attraverso file multipli e aggiornano la documentazione con supervisione umana minima.

Questa rapida adozione ha prodotto una domanda inevitabile: **come si confrontano effettivamente questi agenti nella pratica reale?** La maggior parte delle valutazioni esistenti risponde attraverso benchmark controllati — problemi sintetici progettati per testare capacità specifiche in condizioni standardizzate. Sebbene preziosi, i benchmark hanno limitazioni ben note: potrebbero non riflettere la diversità, la complessità e le complicazioni dei task reali di ingegneria del software.

Questo articolo, pubblicato a **MSR 2026** (la 23ª Conferenza Internazionale su Mining Software Repositories), adotta un approccio diverso. Anziché valutare gli agenti su task sintetici, abbiamo studiato le loro prestazioni su **pull request reali** in **repository open-source reali**, usando i tassi di accettazione come segnale ultimo di utilità pratica.

## Progettazione dello Studio

### Dataset

Abbiamo analizzato **7.156 pull request** create da **cinque agenti di codifica AI** attraverso un insieme diversificato di repository open-source. Ogni pull request rappresenta un'unità di lavoro completa — modifiche al codice, messaggi di commit e descrizioni della PR — sottomessa a un progetto reale con maintainer reali che prendono decisioni di accettazione reali.

### Stratificazione per Task

Il contributo metodologico chiave di questo studio è la **stratificazione per task**. Anziché calcolare un singolo tasso di accettazione complessivo per agente (che oscura variazioni importanti), abbiamo categorizzato ogni pull request per tipo di task:

- **Bug fixing**: Correzione di difetti nel codice esistente
- **Implementazione di feature**: Aggiunta di nuove funzionalità
- **Documentazione**: Aggiornamento o creazione di documentazione
- **Refactoring**: Ristrutturazione del codice senza cambiarne il comportamento
- **Aggiornamento dipendenze**: Upgrade di librerie e dipendenze
- **Testing**: Aggiunta o miglioramento della copertura dei test

Questa stratificazione consente un confronto molto più ricco: anziché chiedere "quale agente è il migliore in assoluto?", possiamo chiedere "quale agente è il migliore per ogni tipo di lavoro?"

## La Scoperta Chiave: Il Tipo di Task Domina

Il nostro risultato più significativo è che **il tipo di task è il fattore dominante che influenza se una pull request generata dall'AI viene accettata**. Il divario nel tasso di accettazione tra le categorie con le migliori e le peggiori prestazioni ha raggiunto **29 punti percentuali** — superando sostanzialmente la varianza tra diversi agenti all'interno di qualsiasi singola categoria.

Questa scoperta ha profonde implicazioni su come pensiamo alla valutazione degli agenti AI. L'inquadramento comune di "quale agente è il migliore?" risulta essere in qualche modo fuorviante. La domanda più utile è: "quale agente è il migliore per *questo specifico tipo di task*?"

Diverse categorie di task hanno livelli di difficoltà intrinsecamente diversi per gli agenti AI. I task di documentazione, che coinvolgono principalmente la generazione e la modifica di linguaggio naturale con struttura relativamente ben definita, tendono ad avere tassi di accettazione più alti. I task di implementazione di feature, che richiedono la comprensione di architetture di sistema complesse e decisioni su trade-off di design, tendono ad avere tassi di accettazione più bassi. Queste differenze nella difficoltà intrinseca sovrastano le differenze tra gli agenti.

## Punti di Forza Specifici degli Agenti

La nostra analisi stratificata per task ha rivelato che **nessun singolo agente supera tutti gli altri in tutte le categorie di task**. Ogni agente ha genuine aree di forza:

### OpenAI Codex

Codex ha raggiunto tassi di accettazione costantemente alti attraverso la maggior parte delle categorie, rendendolo una solida **scelta general-purpose**. La sua forza risiede nella versatilità — non presenta picchi o valli drammatiche tra i tipi di task, performando in modo affidabile indipendentemente dal tipo di lavoro.

### Claude Code

Claude Code ha mostrato particolare forza nei **task di documentazione**, dove le sue sofisticate capacità di generazione linguistica si traducono direttamente in prosa di alta qualità. Aggiornamenti della documentazione, miglioramenti dei README e generazione di commenti inline hanno tutti beneficiato della fluenza nel linguaggio naturale di Claude Code.

### Cursor

Cursor ha eccelluto specificamente negli **scenari di bug-fixing**. Il suo flusso di lavoro integrato nell'IDE, che fornisce contesto approfondito sul codebase circostante durante il processo di correzione, sembra dargli un vantaggio quando il task richiede di navigare e comprendere il codice esistente per identificare e correggere i difetti.

### Il Panorama Eterogeneo

Questi risultati dipingono un panorama genuinamente eterogeneo. Diversi agenti hanno profili cognitivi diversi — molto come gli sviluppatori umani che si specializzano in diversi aspetti dell'ingegneria del software. Uno sviluppatore che eccelle nel debugging potrebbe non essere la scelta migliore per scrivere documentazione, e viceversa. Lo stesso vale per gli agenti AI.

## Implicazioni Pratiche

### Per i Team di Sviluppo

Il messaggio pratico è duplice:

1. **Abbinare l'agente al task.** Anziché utilizzare un singolo agente per tutto il lavoro, i team di sviluppo possono ottenere risultati migliori instradando diversi tipi di lavoro verso gli agenti più adatti. I bug fix vanno a un agente, gli aggiornamenti della documentazione a un altro, le implementazioni di feature a un terzo. Questo approccio a "portfolio di agenti" può migliorare significativamente i tassi di accettazione complessivi.

2. **Stabilire aspettative realistiche per tipo di task.** Alcune categorie di lavoro sono intrinsecamente più difficili per gli agenti AI di altre. Comprendere queste baseline aiuta i team a calibrare i processi di revisione ed evitare frustrazioni quando gli agenti hanno difficoltà con certi tipi di task. Se le PR di implementazione di feature hanno un tasso di accettazione del 40% mentre quelle di documentazione hanno un tasso del 69%, la differenza riflette la difficoltà del task, non la qualità dell'agente.

### Per gli Sviluppatori di Agenti

L'analisi stratificata per task fornisce indicazioni specifiche per il miglioramento degli agenti. Anziché ottimizzare per benchmark aggregati, gli sviluppatori di agenti possono concentrarsi sulle categorie di task in cui il loro agente ha prestazioni inferiori rispetto ai concorrenti. Questo approccio di miglioramento mirato è più efficiente e più propenso a produrre guadagni significativi nelle capacità.

### Per i Ricercatori

I nostri risultati evidenziano l'importanza critica della **valutazione stratificata per task** nella ricerca sugli agenti AI. Le metriche aggregate — tasso di accettazione complessivo, punteggio medio dei benchmark — possono mascherare variazioni importanti delle prestazioni che diventano visibili solo quando i risultati vengono suddivisi per tipo di task. Le future valutazioni degli agenti di codifica AI dovrebbero routinariamente riportare le prestazioni a livello di task per fornire un quadro completo e onesto.

## Il Quadro Più Ampio

Man mano che gli agenti di codifica AI vengono sempre più integrati nei flussi di lavoro di sviluppo software, comprendere le loro effettive capacità e limitazioni — non come misurate da benchmark sintetici, ma come osservate nel deployment reale — è essenziale. Il nostro studio contribuisce con una base empirica su larga scala a questa comprensione, dimostrando che la domanda "quale agente dovrei usare?" ha una risposta più sfumata di quanto la maggior parte dei professionisti si renda conto.

---

*Pubblicato alla 23ª Conferenza Internazionale su Mining Software Repositories (MSR 2026) — Mining Challenge. Questa ricerca è stata condotta presso la University College London (UCL).*
