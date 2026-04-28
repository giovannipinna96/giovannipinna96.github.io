---
title: "Non esiste un \"miglior\" agente di codifica AI — ed è esattamente questo il punto"
date: 2026-04-14
draft: false
tags: ["Agenti di Codifica AI", "Pull Request", "Studio Empirico", "Ingegneria del Software", "MSR"]
categories: ["Ricerca"]
description: "Abbiamo analizzato 7.156 pull request di cinque agenti di codifica AI su progetti open-source reali. L'agente conta meno di quanto pensi. Il tipo di lavoro conta molto di più."
ShowToc: true
TocOpen: false
cover:
  image: "/images/post-placeholder.svg"
  alt: "Acceptance rates across AI coding agents and task types"
  hiddenInList: false
---

{{< summary-box title="TL;DR" >}}
Abbiamo raccolto **7.156 pull request reali** create da Codex, Claude Code, Cursor, Devin e Copilot, poi tagliato i dati per *tipo di task*. La sintesi: il divario tra la categoria di task migliore e peggiore è di **29 punti percentuali** — molto più grande del divario tra agenti all'interno di una singola categoria. Codex è il generalista affidabile. Claude Code vince sulla documentazione. Cursor vince sui bug fix. Smetti di chiedere *quale agente è il migliore*. Inizia a chiedere *migliore in cosa*.
{{< /summary-box >}}

## La domanda sbagliata

Entra in un qualunque Slack di engineering e troverai sempre lo stesso dibattito: Codex contro Claude Code contro Cursor contro Devin contro Copilot. I thread esplodono. Volano benchmark. Qualcuno screenshotta una leaderboard.

La risposta onesta a "qual è il migliore" è *dipende* — ma quella risposta sa di scappatoia. Quindi abbiamo provato a renderla concreta. Abbiamo preso 7.156 pull request che questi cinque agenti avevano davvero aperto contro repo open-source reali, e abbiamo misurato l'unica cosa che conta davvero: **un maintainer umano l'ha mergiata?**

Poi abbiamo fatto la parte che la maggior parte delle valutazioni salta. Abbiamo separato le PR per tipo di lavoro che svolgevano.

## Perché un singolo numero mente

Bug fix, sviluppo di feature, documentazione, refactoring, aggiornamento di dipendenze, test — non sono lo stesso task. Hanno profili di difficoltà drasticamente diversi per un agente AI. La documentazione è soprattutto generazione di linguaggio naturale contro un bersaglio morbido. L'implementazione di feature richiede di tenere in testa un modello dell'architettura. Il bug fixing chiede di navigare il codice di qualcun altro.

Se fai la media di tutto, ottieni un numero. E perdi l'intera storia.

Quindi, invece di un tasso di accettazione per agente, ne abbiamo calcolato uno **per agente, per tipo di task**.

![Tassi di accettazione per agente e per tipo di task](/images/msr2026-comparing-ai-agents/Figure1.png)

## Il risultato principale

Guarda il grafico. La cosa che dovrebbe colpirti è *verticale*, non orizzontale.

Il divario tra la migliore e la peggiore categoria di task è circa **29 punti percentuali**. Il divario tra agenti *all'interno* di una stessa categoria è molto più piccolo. **Il tipo di lavoro che dai a un agente predice l'accettazione molto più di quale agente hai scelto.**

Questo riformula tutta la conversazione. Chiedere "Cursor è meglio di Claude Code?" è come chiedere "uno chef è meglio di uno chef di sushi?" — alla domanda manca un complemento. *Meglio in cosa?*

## Ogni agente ha la sua specialità

Quando tagli i dati per bene, gli agenti smettono di confondersi. Sviluppano personalità.

![Punti di forza per agente sui tipi di task](/images/msr2026-comparing-ai-agents/Figure2.png)

**Codex** — il generalista. Niente picchi spettacolari, niente baratri imbarazzanti. Se devi scegliere un solo agente e non sai cosa ti aspetta, questa è probabilmente la scelta sicura.

**Claude Code** — documentazione. La sua fluenza linguistica si traduce direttamente in descrizioni di PR, riscritture di README e commenti inline che i maintainer vogliono davvero mergiare.

**Cursor** — bug fix. L'integrazione con l'IDE gli dà un contesto più profondo sul codice circostante, ed è esattamente il contesto di cui ha bisogno il bug fixing.

Devin e Copilot finiscono altrove su questa mappa — il dettaglio è nel paper. Il punto non è la classifica, è che **non c'è una singola classifica**. C'è una forma.

## Cosa farne davvero

Se metti in produzione codice con agenti AI, due cose ne conseguono:

**Indirizza per task, non per preferenza.** Un team che dà ogni tipo di lavoro al proprio agente preferito sta lasciando tasso di accettazione sul tavolo. PR di documentazione? Mandala a Claude Code. Bug sottile? Cursor. Pulizia del martedì qualunque? Codex. Tratta i tuoi agenti come una squadra con specialità diverse, perché è quello che sono.

**Calibra le aspettative.** Quando una PR di feature fallisce in review, potrebbe non essere colpa dell'agente — potrebbe essere colpa del *task*. Un tasso di accettazione del 40% sui task difficili non è un agente rotto. Un 70% sui task facili non è uno magico. Entrambi sono rumore intorno a una baseline imposta dal lavoro stesso.

## Cosa cambia per il campo

Per chi sviluppa agenti: *smetti di rincorrere punteggi aggregati nei benchmark*. Nascondono dove stai perdendo. La strada per un agente migliore passa dalla categoria di task in cui ora sei sotto.

Per i ricercatori: riportate sempre i numeri per task. Un singolo tasso di accettazione non è una misurazione, è una funzione di smoothing sopra la variazione più interessante.

E per tutti gli altri: quando qualcuno ti dice che il suo agente è il migliore, chiedigli *in cosa*. Se non sa rispondere, non lo sa nemmeno il suo agente.

---

### Reference

Questo post è una sintesi divulgativa di:

> Pinna, G., Sarro, F. (2026). *Comparing AI Coding Agents: A Task-Stratified Analysis of Pull Request Acceptance*. In: **Proceedings of the 23rd International Conference on Mining Software Repositories (MSR 2026)** — Mining Challenge Track.
>
> [Leggi il paper originale (PDF)](/images/msr2026-comparing-ai-agents/MSR'26%20Mining%20Challenge%20-%20Comparing%20AI%20Coding%20Agents_%20A%20Task-Stratified%20Analysis%20of%20Pull%20Request%20Acceptance.pdf)

*Ricerca condotta presso la University College London (UCL), CREST, con la Prof.ssa Federica Sarro.*
