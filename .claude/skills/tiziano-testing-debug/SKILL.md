---
name: tiziano-testing-debug
description: progettare, scrivere ed eseguire test approvati e supportare il debugging collaborativo human-in-the-loop di sistemi software deterministici o integrati. usare per unit, integration, contract, end-to-end, UI, pipeline dati, database, API, eventi, container e load test. non usare per AI R&D, model evaluation, training, observability, security review o correzione automatica del codice.
---

# Tiziano Testing & Debug

## Obiettivo

Supportare test e debugging con approccio Human in the Loop, controllato dall'utente e proporzionato al ciclo di vita. Scrivere ed eseguire solo test approvati. Non correggere automaticamente codice, configurazioni o ground truth.

## Principi inderogabili

- L'utente definisce cosa testare, profondità, ambiente e criteri di successo.
- Nessun reverse engineering dei requisiti da codice, test o comportamento osservato.
- I test derivano solo da requisiti confermati, acceptance criteria, contratti pubblici o comportamenti che l'utente chiede esplicitamente di preservare.
- Se i requisiti non bastano, dichiarare cosa non è verificabile e chiedere.
- Dopo un fallimento non modificare codice, configurazioni, test data o test.
- Test data, fixture, snapshot, golden file ed expected output sono ground truth read-only.
- Rispettare il limite dei tentativi (vedi sezione dedicata).

## Confini

Questa skill copre unit, integration, contract ed end-to-end test; API, servizi, eventi e code; PostgreSQL, DuckDB, MongoDB, Redis e database vettoriali; filesystem e servizi esterni; pipeline dati ed ETL/ELT; UI e user journey; load test approvati; integrazione software di moduli AI già realizzati.

Non copre: training, fine-tuning, model evaluation, benchmark, observability permanente, security review, correzione automatica del sistema sotto test. Vedere la matrice di responsabilità.

## Workflow Human in the Loop

### 1. L'utente definisce la riproduzione

```text
Obiettivo della verifica:
Componente da testare:
Livello: unit / integration / contract / end-to-end / UI / load
Profondità:
Requisiti confermati:
Fase: DEV / TEST / PRE-PROD / PROD
Ambiente containerizzato disponibile:
Dipendenze reali o mock:
Test data autorizzati:
File coinvolti:
```

### 2. La skill propone il test

Prima di scrivere codice:

```text
Obiettivo:
Requisito verificato:
Input:
Risultato atteso:
Livello di test:
Dipendenze reali o mock:
File di test da creare o modificare:
Comando proposto:
Limiti della validazione:
```

Attendere approvazione.

### 3. La skill scrive il test

- Creare o modificare solo i file di test approvati.
- Non toccare il sistema sotto test né ampliare la suite.

### 4. Esecuzione collaborativa

- L'utente può eseguire il test e fornire comando, errore e stack trace.
- La skill può eseguire il comando approvato nel sistema DEV indicato, mai l'intera suite se è richiesto un test mirato.
- Mostrare il comando prima della prima esecuzione.

### 5. Diagnosi

```text
Evidenza: dato osservato direttamente.
Classificazione: test / codice applicativo / ambiente / configurazione / requisito / test data / concorrenza.
Ipotesi principale: più probabile, non presentata come certezza.
Alternative: massimo due.
Prossima prova minima: un solo comando o controllo mirato.
Modifiche: nessuna.
```

### 6. Convalida

- Solo l'utente decide se il comportamento è convalidato.
- La skill può rieseguire test approvati. I regression test solo su richiesta.
- Il fix applicativo appartiene a una fase successiva, di norma con `tiziano-sw-eng`.

## Contesto e perimetro

- Partire dal contesto e dai file forniti dall'utente.
- Usare eventuali code graph o projection solo per individuare moduli direttamente collegati.
- Espandere l'analisi solo quando test, stack trace o dipendenze indicano un possibile errore in un modulo collegato, indicando file e motivazione.
- Evitare scansioni complete del repository, letture esplorative e lettura di documentazione, cronologia Git o configurazioni non pertinenti.

## Profondità e lifecycle

Priorità delle fonti: requisiti confermati, acceptance criteria, contratti pubblici, comportamento esistente che l'utente chiede di preservare.

- **DEV**: test funzionali, unitari e di integrazione mirati.
- **TEST**: contract, integration, end-to-end e user journey approvati.
- **PRE-PROD**: load, performance, resilienza e user journey approvati.
- **PROD**: solo verifiche esplicitamente autorizzate e non distruttive.

## Mock, servizi reali e dati

- Preferire integrazioni reali quando l'ambiente è autorizzato.
- Usare mock per isolamento mirato, API esterne, errori controllati e dipendenze lente o costose.
- Non simulare comportamenti complessi se il mock può divergere materialmente dal sistema reale.
- Non collegarsi a database condivisi, ambienti aziendali o PROD senza parametri confermati.
- Non introdurre Testcontainers, Compose di test o fixture di database senza autorizzazione.
- Per dati sintetici, considerare solo ciò che serve: schema e tipi, cardinalità e volumi, null e duplicati, vincoli e relazioni, sequenze temporali.
- Dichiarare cosa è simulato e cosa no. Un comportamento che dipende da dati reali non è validato con dati sintetici.

## Ground truth read-only

Non aprire, modificare, rigenerare o sostituire test data, fixture, snapshot o expected output senza autorizzazione esplicita. Prima di richiedere accesso:

```text
Dato richiesto:
Operazione: lettura / modifica / rigenerazione
Motivo:
Evidenza riscontrata:
Impatto sul test:
Alternativa senza intervenire sui dati:
```

I test si adattano ai dati approvati, non il contrario.

## Framework e stile

- Rispettare il framework esistente. Per Python senza framework proporre `pytest`.
- Non introdurre `pytest-asyncio`, `pytest-cov`, `pytest-mock`, Hypothesis o Testcontainers senza autorizzazione.
- **UI ed E2E**: Playwright per progetti Python; Puppeteer per progetti JavaScript o TypeScript; Selenium solo se già adottato. Non cambiare framework senza approvazione. Se il progetto non è chiaro, chiedere.
- Selettori preferiti: `data-testid`, ruoli accessibili, label, nomi visibili. Nessun selettore CSS fragile.
- Attendere eventi o condizioni precise, mai `sleep` arbitrari.
- Struttura Arrange, Act, Assert; un requisito per test; nomi descrittivi.
- Expected result esplicito, mai calcolato replicando l'algoritmo testato.
- Fixture solo per eliminare duplicazione reale. Parametrizzazione solo per casi equivalenti.
- Screenshot, video e trace solo su richiesta o in caso di fallimento.

## Test su risultati non deterministici

Per output di sistemi AI integrati chiedere sempre all'utente:

- **No**: testare solo integrazione, contratti e parti deterministiche.
- **Parzialmente**: verificare invarianti e vincoli indicati.
- **Sì**: usare criteri, tolleranze e ripetizioni definiti dall'utente.
- **Rinviare**: demandare alla futura skill di AI evaluation.

Non scegliere metriche, soglie, ground truth, numero di esecuzioni o criteri qualitativi.

## Esecuzione sicura

- Eseguire solo comandi approvati e mirati, con timeout ragionevoli.
- Preferire un ambiente DEV isolato.
- Non avviare container, database, browser o servizi senza approvazione.
- Non eseguire migrazioni, reset, cleanup o teardown dei dati senza autorizzazione.
- Non modificare `.env`, credenziali, Dockerfile, Compose, CI/CD, dipendenze, lock file, migrazioni, schema, test data o codice applicativo.
- Se manca un prerequisito, fermarsi e indicarlo.
- Test più ampi solo dopo il successo della verifica mirata e una nuova approvazione.

## Coverage

- Misurare solo su richiesta. Nessuna percentuale universale.
- Non scrivere test inutili per aumentare la coverage.
- Distinguere line coverage, branch coverage e copertura funzionale reale.
- Non modificare configurazioni di coverage senza approvazione.

## Fallimenti, flaky test e limite dei tentativi

Ogni fallimento va classificato come uno di: bug applicativo, errore nel test, test data non rappresentativi, requisito ambiguo, ambiente o configurazione, concorrenza o asincronia. Non correggere il test per farlo passare: spiegare prima se è il test a essere errato o il codice a non rispettare il requisito.

**Flaky test:**

- classificarlo come flaky, mai come superato;
- nessun retry automatico, nessun aumento arbitrario dei timeout, nessun `sleep`, nessun `skip` o `xfail` senza approvazione;
- indagare distinguendo asincronia, race condition, dipendenze esterne, stato condiviso, ordine di esecuzione, dati variabili e risorse insufficienti;
- ripetizioni diagnostiche solo se decise dall'utente.

**Limite dei tentativi (unica regola):** per iterazione una sola ipotesi principale, massimo due alternative, una sola prova mirata. Dopo due iterazioni inconcludenti fermarsi e restituire:

```text
Evidenze raccolte:
Ipotesi ancora valide:
Limite incontrato:
Dato o file richiesto:
Motivazione:
```

## Output finale

```text
### Risultato
- Test raccolti / eseguiti / superati / falliti / non eseguiti
- Ambiente
- Mock o integrazioni reali

### Evidenze
- Errore osservato / punto del flusso / file coinvolti

### Classificazione
- Codice / test / ambiente / configurazione / requisito / dati

### Ipotesi
- Principale / Alternative

### Prossima azione proposta
- Nuova prova / possibile correzione / skill successiva

### Garanzie
- Nessuna modifica al codice applicativo, alla configurazione o ai test data

### Limiti della validazione
```
