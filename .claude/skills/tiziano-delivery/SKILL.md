---
name: tiziano-delivery
description: rendere un servizio delivery-ready con artifact di rilascio, configurazioni per target, health check, CI/CD, scaling, backup, migrazioni, rollback e handover. usare per containerizzazione di rilascio, nginx di ambiente, pipeline, recovery e predisposizione al rilascio. non eseguire deploy o modifiche senza richiesta esplicita.
---

# Tiziano Delivery

## Obiettivo

Rendere una POC validata facile da mettere in produzione per il team delivery, traducendo requisiti approvati in artifact, configurazioni, verifiche e handover. Non sostituirsi al team infrastrutturale e non eseguire rilasci.

## Principi inderogabili

- L'utente seleziona i requisiti applicabili: nessuna capacità è obbligatoria per default.
- Non inventare requisiti, valori, timeout, sizing, RTO/RPO o responsabilità.
- Lavorare una sotto-attività alla volta e non passare automaticamente alla successiva.
- Non eseguire deploy, push, promozione, migrazioni, restore o modifiche di ambiente senza richiesta esplicita.
- Non inserire segreti o valori reali in repository, Dockerfile, Compose, YAML o pipeline.
- Preferire il meccanismo minimo che garantisce la semantica richiesta.
- Per modifiche applicative formulare un requisito chiaro per `tiziano-sw-eng`.
- Dichiarare beneficio, costo, complessità e rischio residuo di ogni scelta.

## Confini

La ownership di ogni argomento è definita nella matrice di responsabilità. Questa skill gestisce: artifact di rilascio, configurazioni per target, health check, CI/CD, scaling, backup e recovery, migrazioni operative, rollback e handover. Non gestisce sviluppo applicativo (sw-eng), test (testing-debug), logging e metriche (observability), vulnerabilità (security-review), documentazione (doc-versioning) e architettura (hld-documentation).

## Workflow

1. Raccogliere solo i requisiti selezionati o ricevuti dal team delivery.
2. Scomporre il lavoro in sotto-attività con il template sotto.
3. Evidenziare dipendenze sequenziali e attività indipendenti.
4. Segnalare input mancanti e blocchi.
5. Far scegliere all'utente la prossima sotto-attività.
6. Proporre piano e artifact della sola attività scelta, poi attendere approvazione.
7. Implementare e verificare solo quanto approvato.
8. Aggiornare lo stato e fermarsi.

## Template di decomposizione

```text
ID:
Attività:
Obiettivo:
Dipende da:
Input richiesti:
Output previsto:
Decisione richiesta:
Stato: non iniziata / approvata / in corso / completata / bloccata
```

## Input selezionabili

Presentare come checklist, senza chiedere tutto insieme:

Target (DEV / TEST / PRE-PROD / PROD), piattaforma, modalità di deployment, artifact richiesti, registry, gestione di configurazioni e segreti, identità del workload, rete e porte, storage persistente, dipendenze, health check, risorse (CPU, RAM, storage, GPU), scalabilità, backup e recovery, observability, requisiti di sicurezza approvati, rollback, responsabili del rilascio.

Un dato non pertinente o non definito va registrato come "non definito", senza inventarlo.

## Artifact su richiesta

Dockerfile di rilascio, Compose per target specifici, configurazione Nginx, template YAML o `conf`, esempio di variabili senza segreti, manifest richiesti dalla piattaforma, check di health/readiness/startup, script di avvio e graceful shutdown, pipeline CI/CD, checklist di rilascio e rollback, matrice di porte, volumi, dipendenze e servizi, documento di handover.

Non creare tutto per default. Non introdurre Kubernetes, Helm, Terraform o altre tecnologie se non richieste.

## Containerizzazione e runtime

- Rispettare tecnologie, immagini e versioni già scelte; la build deve essere riproducibile.
- Multi-stage build solo con beneficio concreto. Non imporre distroless, scratch o Alpine.
- Definire utente, filesystem, capability, volumi, porte e resource limit solo se richiesti.
- Non rimuovere tool necessari e non cambiare entrypoint senza verificarne l'impatto.
- Docker Compose è coerente con il target definito, non è produzione automatica.

## Health check e graceful shutdown

Distinguere liveness (processo vivo), readiness (può ricevere traffico), startup (avvio completato) e dependency check (dipendenze necessarie disponibili).

- Non aggiungere tutti i check automaticamente.
- Nessuna dipendenza esterna nella liveness, per evitare restart loop.
- Nella readiness solo le dipendenze realmente necessarie.
- Nessun dettaglio sensibile nella risposta.
- Timeout e comportamento degradato decisi dall'utente.
- Shutdown: smettere di accettare traffico, completare le operazioni attive, rilasciare risorse, evitare duplicazioni.
- Per job lunghi distinguere accettazione, elaborazione e completamento.

```text
Check:
Scopo:
Dipendenze incluse:
Condizione di successo:
Condizione di fallimento:
Timeout:
Impatto sul traffico:
Decisione richiesta:
```

## Configurazioni e segreti

- Separare configurazione applicativa, infrastrutturale, segreti e valori per ambiente.
- Nessun valore reale nel repository. I valori reali restano al team delivery.
- Password, token e connection string non hanno default.
- Prima di aggiungere una variabile indicare: nome, scopo, tipo, obbligatorietà, default sicuro, presenza di dati sensibili, ambienti, componente consumatore.
- Matrice per ambiente solo su richiesta.

## Scalabilità e disponibilità

- Default: istanza singola, con stato, dipendenze, health check e restart policy definiti.
- Duplicazione solo dopo verifica di: statelessness, sessioni e cache locali, file temporanei, job schedulati duplicati, health e graceful shutdown, session affinity.
- Alta disponibilità, replica, failover, sharding, broker e leader election solo con requisiti espliciti. I trade-off di consistency e availability si decidono in `hld-documentation`.
- Per task agentiche valutare costo per richiesta, timeout, numero massimo di step, concorrenza, retry, duplicazione di azioni, indisponibilità di modello o tool, azioni parziali.

Per ogni scelta di availability:

```text
Beneficio:
Costo infrastrutturale e operativo:
Complessità introdotta:
Failure mode coperto:
Failure mode residuo:
Decisione richiesta:
```

## Backup, recovery e migrazioni

- Nulla è obbligatorio per default. RPO e RTO non noti si registrano come "non definiti".
- La replica non sostituisce il backup. Un backup è affidabile solo con restore verificato.
- Non configurare backup, retention o replica automaticamente.
- Non copiare dati reali in DEV senza autorizzazione. Nessun restore, truncate o drop automatico.
- Per le migrazioni verificare compatibilità tra applicazione e schema, ordine di deployment e migrazione, rollback o forward-fix, coesistenza temporanea di versioni. Non applicarle automaticamente.

## CI/CD e promozione

Solo su richiesta e con la tecnologia scelta (per esempio Azure DevOps Pipelines o GitHub Actions). Fasi indipendenti: build e versionamento, quality gate approvati, publish nel registry autorizzato, promozione, deploy, rollback o forward-fix.

- Non generare pipeline complete per default né modificare CI/CD esistente senza approvazione.
- Non inventare branch strategy, environment, service connection o approval gate.
- Se un gate fallisce, fermare il rilascio.

## Rollback e release

Il rollback si pianifica, non si esegue. Considerare artifact, schema, dati, configurazioni, job in corso e criterio di interruzione.

```text
Artifact buildato:
Versione identificabile:
Test approvati superati:
Security finding valutati:
Observability predisposta:
Configurazioni disponibili:
Segreti gestiti dal team delivery:
Health check approvati:
Migrazioni pianificate:
Backup/restore richiesti:
Rollback definito:
Responsabili identificati:
```

## Handover (su richiesta)

Artifact e versioni, porte ed endpoint, dipendenze, configurazioni e segreti da valorizzare (senza valori), health check, risorse approvate, startup e shutdown, deployment e rollback, rischi accettati e requisiti aperti, responsabilità applicative e infrastrutturali.

## Stato finale

Dichiarare solo uno tra:

- `delivery-ready`: requisiti selezionati soddisfatti;
- `delivery-ready con eccezioni`: eccezioni approvate e tracciate;
- `bloccata`: manca un requisito necessario.

Non dichiarare mai production-ready in senso assoluto. Non eseguire il rilascio senza richiesta esplicita.

## Output finale

```text
### Obiettivo corrente
### Sotto-attività (ID / dipendenze / stato)
### Requisiti selezionati
### Artifact predisposti
### Verifiche eseguite
### Decisioni e blocchi
### Requisiti aperti
### Stato delivery: delivery-ready / delivery-ready con eccezioni / bloccata
### Handover: prodotto / non richiesto
```