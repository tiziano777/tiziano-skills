---
name: tiziano-observability
description: analizzare, pianificare e implementare, dopo approvazione, error handling, regole di logging, metriche, tracing e telemetria applicativa, di sistema, dati, AI e GPU secondo il ciclo di vita di tiziano. usare per observability, instrumentation, OpenTelemetry, OpenObserve, Grafana, Arize Phoenix, failure policy, middleware e logging. non usare per debugging di un singolo bug, testing, sviluppo funzionale, security review o delivery.
---

# Tiziano Observability

## Obiettivo e confini

Progettare una observability proporzionata alla fase del ciclo di vita: individuare il minimum viable telemetry set, proporre gli interventi e implementarli solo dopo approvazione esplicita.

Gestisce: error handling, regole di logging, metriche, tracing, telemetria applicativa, di sistema, dati, AI e GPU, middleware e backend di osservabilità.

Non gestisce: diagnosi di un bug (testing-debug), sviluppo funzionale (sw-eng), security review, delivery. Vedere la matrice di responsabilità.

## Principi inderogabili

- Progettare prima, implementare solo quanto approvato.
- Non ampliare il perimetro: ogni intervento è un elemento distinto, da approvare.
- Adeguarsi allo stack esistente. Non introdurre SDK, collector, agent, server o librerie senza approvazione.
- Telemetria minima e proporzionata alla fase: più dettaglio in DEV, meno in PROD.
- Nessun segreto, credenziale o dato sensibile nei segnali, in nessuna fase.

## Allineamento iniziale

```text
Fase: DEV / TEST / PRE-PROD / PROD
Paradigma: standard / event-driven / microservizi / ibrido
Livello di telemetria desiderato:
Stack esistente:
Backend preferito:
Dati esclusi dalla raccolta:
Perimetro richiesto:
```

Se manca uno stack esistente, chiedere quale adottare prima di introdurre dipendenze.

## Lifecycle

- **DEV**: telemetria dettagliata, sampling fino al 100%. Payload, prompt e output intermedi solo se richiesti e confinati. Mai credenziali.
- **TEST**: dati sufficienti a riprodurre errori e verificare il comportamento. Correlation ID, request ID, job ID o trace ID dove utili.
- **PRE-PROD**: verificare volumi, cardinalità, overhead, costi e affidabilità. Proporre masking, minimizzazione, sampling, retention e accessi. Coordinarsi con security-review.
- **PROD**: solo ciò che serve a operatività, diagnosi, audit e SLO. Niente payload completi. Sampling, retention e accessi concordati. Solo alert e dashboard essenziali e approvati.

## Regole di logging

Le regole si applicano a ogni codice nuovo o modificato. La skill sw-eng le rispetta, ma le definisce qui.

**Libreria e strumenti**
- Usare la libreria di logging già adottata; in Python, il modulo `logging` standard se non ne esiste una.
- Non usare `print()` come logging.
- Non cambiare libreria né configurazione senza richiesta.

**Livelli**
- `DEBUG`: dettagli tecnici, solo per diagnosi mirata.
- `INFO`: eventi rilevanti (avvio, completamento di un job, esito di un'operazione).
- `WARNING`: anomalie recuperabili.
- `ERROR`: operazione fallita.
- `CRITICAL`: il servizio non può proseguire.

**Posizione**
- Solo ai boundary significativi: entrypoint, endpoint, consumer, job, integrazioni esterne, error mapper.
- Non in ogni funzione né per ogni record elaborato, salvo debug mirato.

**Unicità**
- Un errore si registra una sola volta, nel punto che lo gestisce. I livelli che lo propagano non lo ripetono.

**Stack trace**
- Solo quando hanno valore diagnostico (errori imprevisti o difetti di programmazione).
- Conservare la causa originale con exception chaining.

**Contenuto**
- Messaggio stabile e leggibile. Contesto in campi strutturati (request, job, trace ID) solo se utili.
- Formato strutturato (JSON o simili) solo se richiesto o già adottato.

**Cosa non registrare mai**
- Password, token, chiavi, connection string, dati personali, contenuti di documenti o dataset.
- Prompt e output completi, salvo `DEBUG` in DEV autorizzato e confinato.

**Configurazione**
- Il livello di log viene dalla configurazione esistente, mai hardcoded.
- Nessun nuovo handler o sink senza approvazione.

**Logging temporaneo**
- Il logging di diagnosi va marcato come temporaneo e rimosso a fine attività.

**Prima di implementare** proporre: evento, livello, posizione, campi, motivazione e durata (permanente o temporanea).

## Error handling

Classificare ogni errore come: input o validazione, dominio, integrazione, infrastruttura, transitorio, difetto di programmazione.

Catturare un'eccezione solo quando si può:
- recuperare;
- tradurre in errore di dominio;
- aggiungere contesto;
- rilasciare risorse;
- definire la risposta verso API o consumer.

Vietato:
- `except Exception` generici, salvo boundary approvati;
- catturare solo per registrare e rilanciare senza valore;
- fallback silenziosi, `pass` o valori neutri che nascondono l'errore.

**Retry, timeout, backoff, circuit breaker, DLQ**: la decisione spetta a `hld-documentation` (strategia), l'implementazione a sw-eng e la configurazione a delivery. Observability definisce solo cosa osservare di questi meccanismi.

**Classi dedicate** (per esempio `ApiExceptionMapper` o `MessageFailureHandler`): solo per middleware, consumer o pipeline complesse e integrazioni critiche. Per script e casi semplici, gestione locale. Una classe dedicata dichiara: responsabilità, errori gestiti, punto d'invocazione, recovery, dipendenze e vantaggio rispetto alla gestione locale.

## Observer

Usare un observer solo quando disaccoppia realmente produttori e consumatori di eventi applicativi, con almeno due consumatori o un vincolo di disaccoppiamento esplicito. Altrimenti usare il logging diretto. Non deve diventare un contenitore generico né duplicare il logging.

## Metriche

Quattro categorie, da attivare solo se approvate:

1. **Applicative**: richieste, job, esiti, errori, durata, code e backlog.
2. **Infrastruttura**: CPU, RAM, disco, rete, container, connessioni e pool del database.
3. **Data pipeline**: record letti, validi, scartati, duplicati; throughput; latenza; freshness.
4. **AI e GPU**: token, latenza, throughput, costi, GPU, VRAM, temperatura, batch, checkpoint e metriche di evaluation confermate.

Definire prima il minimum viable telemetry set. Evitare label ad alta cardinalità e dati personali nelle label. Soglie, dashboard e alert solo se richiesti e giustificati.

Per ogni metrica indicare: nome, motivo, tipo e unità, punto di emissione, label essenziali, rischio di cardinalità.

Progressione tipica: POC (esito, durata, errore principale), stabilizzazione (metriche sui problemi osservati), validation (metriche funzionali, data quality e AI evaluation confermate), delivery (health, capacity, alerting, dashboard), ottimizzazione (profiling, GPU, memoria, costi).

## Tracing

Una trace rappresenta il percorso di una richiesta o di un job; una span è una singola operazione significativa, con inizio, fine, durata, esito e attributi tecnici.

```text
Trace: answer_question
├── Span: load_document
├── Span: create_embedding
├── Span: vector_search
├── Span: reranking
└── Span: model_inference
```

- Span solo per operazioni significative, non per ogni funzione.
- Propagare il contesto attraverso i boundary reali.
- Per AI e RAG distinguere retrieval, reranking, costruzione del prompt, inferenza, invocazione di tool, evaluation.
- Mai credenziali o segreti negli attributi.
- Sampling e retention definitivi in PRE-PROD e PROD, concordati con delivery.

## Standard e backend

- OpenTelemetry come standard vendor-neutral quando compatibile, non obbligatorio.
- Separare instrumentation ed exporter.
- Backend da concordare: OpenObserve, Grafana o compatibili, Arize Phoenix, strumenti specializzati per GPU e runtime.
- Non legarsi a un framework specifico.

## Sicurezza e privacy

Questa skill progetta la raccolta tecnica. Minimizzazione, masking, accessi, retention e cifratura dei segnali sono verificati da `tiziano-security-review`.

## Output del piano

```text
### Perimetro
- Fase / stack esistente / obiettivo

### Interventi proposti
1. ...

### Segnali
- Logging: ...
- Metriche: ...
- Trace: ...
- Error handling: ...

### Impatto
- File coinvolti / nuove dipendenze / overhead previsto / dati raccolti

### Decisioni richieste
```

Dopo l'approvazione, implementare solo gli elementi accettati e chiudere con changelog, verifiche e punti aperti.
