---
name: hld-documentation
description: creare, rivedere, completare o migliorare High Level Design di sistemi software per sviluppo, test, UAT, staging, produzione e disaster recovery. usare per HLD, descrizioni architetturali, viste C4, deployment, integrazioni, dati, identità, sicurezza, rete, observability, resilienza, DR, CI/CD, requisiti non funzionali, rischi, debito tecnico e decisioni. non scendere al livello di LLD.
---

# HLD Documentation

## Obiettivo

Produrre un HLD revisionabile che spieghi il sistema, le preoccupazioni degli stakeholder, le viste, le decisioni, gli ambienti, i controlli e il modello operativo. Coprire esplicitamente DEV e PROD; aggiungere TEST, UAT, STAGING e DR quando pertinenti.

## Riferimenti standard

Usare i concetti dello standard come mappa di completezza, senza applicarli integralmente in ogni documento:

- **ISO/IEC/IEEE 42010**: stakeholder, preoccupazioni, punti di vista, viste, decisioni e motivazioni.
- **arc42**: checklist di copertura (obiettivi, vincoli, contesto, strategia, blocchi, runtime, deployment, concetti trasversali, decisioni, qualità, rischi, glossario).
- **C4**: Context e Container sempre; Deployment per ambiente; Dynamic per i flussi critici; Component solo se utile; nessuna vista Code.
- **Security by design**: sicurezza in tutto il ciclo di vita, non come checklist finale.

Se l'organizzazione ha un template approvato, usarlo e mappare la copertura al suo interno senza duplicazioni.

## Regole non negoziabili

- Preservare scope, nomi, tecnologie, vincoli, responsabilità e comportamenti dichiarati.
- Separare fatti, assunzioni, raccomandazioni, decisioni, rischi, debito e punti aperti (TBD).
- Non inventare topologia, porte, sizing, SLA/SLO, RTO/RPO, volumi, credenziali, conformità o approvazioni.
- Ogni TBD ha owner, scadenza, impatto bloccante e decisione richiesta.
- Ogni decisione rilevante è collegata a preoccupazioni, requisiti, vincoli, rischi o scenari di qualità.
- Terminologia e relazioni coerenti tra testo, diagrammi, cataloghi e ambienti.
- Non dichiarare un sistema sicuro, conforme, scalabile, altamente disponibile o production-ready senza evidenze e criteri.

## Input da raccogliere

Raccogliere o derivare, ed esporre le lacune:

- obiettivo, capacità, utenti, owner, fase del ciclo di vita, criticità, criteri di successo;
- stakeholder e preoccupazioni: business, utenti, architettura, sicurezza, privacy, dati, infrastruttura, operation, supporto, audit;
- scope, esclusioni, stato attuale e target, vincoli, assunzioni, dipendenze, rischi e debito;
- hosting, regioni, tenant o subscription, reti, ambienti, servizi condivisi, confini di ownership;
- componenti, responsabilità, tecnologie, stato, scalabilità, stato di supporto;
- interfacce: direzione, protocollo, identità, dati, frequenza, volumi, timeout, retry, idempotenza, comportamento in caso di errore;
- domini dati, sistemi di riferimento, classificazione, residenza, retention, cancellazione, backup;
- identità, ruoli, identità dei workload, accessi privilegiati, segreti, certificati, chiavi;
- carico, latenza, throughput, concorrenza, crescita, disponibilità, RTO/RPO, orari di supporto;
- delivery, rollback, observability, incidenti, costi e licenze, normative, approvazioni, standard, ADR.

## Workflow

1. Identificare se si tratta di creazione, revisione o aggiornamento. Stabilire pubblico, fase, formato e confine del sistema.
2. Inventariare le evidenze. Segnalare contraddizioni, affermazioni obsolete, affermazioni senza supporto e lacune.
3. Mappare stakeholder su preoccupazioni e preoccupazioni su viste.
4. Definire scope, contesto, attori, sistemi esterni, confini di fiducia e ownership.
5. Costruire l'architettura logica e i flussi runtime critici prima del deployment fisico.
6. Descrivere integrazioni, dati, concetti trasversali, identità, sicurezza, privacy e operatività.
7. Modellare ogni ambiente mantenendo una sola architettura logica di riferimento.
8. Aggiungere scenari di qualità misurabili, base di capacità, resilienza, DR, supporto, delivery e costi.
9. Registrare decisioni (opzioni e motivazioni), assunzioni, dipendenze, rischi, debito e punti aperti.
10. Applicare i review gate e consegnare HLD, registro delle lacune, decisioni richieste ed evidenze mancanti.

## Struttura dell'HLD

Adattare la profondità al rischio e alla fase. Non eliminare mai una sezione in silenzio: se non applicabile, dichiararlo.

### 1. Controllo documento e sintesi

Sistema, scopo, stato, versione, data, autori, revisori, approvatori, distribuzione, storico modifiche, riservatezza, riferimenti. Sintesi di problema, soluzione, utenti, hosting, integrazioni, benefici, rischi principali e approvazione richiesta.

### 2. Stakeholder, preoccupazioni, scope e driver

Per ogni stakeholder: ruolo, preoccupazione, informazione richiesta, vista di riferimento, autorità di accettazione. Definire in scope e fuori scope, capacità, confine, prerequisiti, stato attuale e target, principi, vincoli e principali obiettivi di qualità.

### 3. Contesto e strategia di soluzione

Contesto di business e tecnico, attori, sistemi esterni, ownership e interfacce direzionali. Scelte fondamentali con alternative considerate, motivazione e conseguenze. Includere la vista C4 di contesto.

### 4. Architettura logica, runtime e integrazioni

- Vista C4 dei container o logica, con narrativa del flusso end-to-end.
- Per ogni blocco: responsabilità, tecnologia, owner, stato, scalabilità, ownership dei dati, dipendenze, stato di supporto.
- Scenari runtime significativi: flusso normale, autenticazione e autorizzazione, lavoro asincrono, errore o degrado, ripristino.
- Catalogo delle interfacce: origine, destinazione, scopo, direzione, protocollo e porta confermati, formato o riferimento allo schema, autenticazione, classificazione dei dati, frequenza e volume, timeout, retry e backoff, idempotenza, gestione errori e dead letter, versionamento, criticità della dipendenza, owner.
- Distinguere flussi sincroni, eventi, code, batch, file, interazioni umane e flussi amministrativi.

### 5. Dati, sicurezza, privacy e identità

- Domini ed entità, sistema di riferimento, ownership di lettura e scrittura, movimentazione, sincronizzazione, consistenza, lineage, qualità, residenza, retention, cancellazione.
- Classificazione, minimizzazione, mascheramento o dati sintetici in non-PROD, cifratura e ownership delle chiavi, perimetro dei backup, accessi, audit.
- Identità umane e dei workload, provider e federazione, flussi di autenticazione, RBAC o ABAC, least privilege, accessi privilegiati, segregazione dei compiti.
- Confini di fiducia, minacce, superficie di attacco, ingressi e uscite, segmentazione, gestione di segreti, certificati e chiavi, patching e supply chain.
- Obblighi applicabili mappati a controlli, evidenze e owner. Eccezioni e rischio residuo accettato registrati.
- Per i dati personali: finalità, flussi, minimizzazione, retention, gestione dei diritti degli interessati, trasferimenti.

Il dettaglio dei controlli e dei template di sicurezza è in `references/`. Il dettaglio di sicurezza operativa resta in `tiziano-security-review`.

### 6. Deployment, rete e ambienti

- Zone, reti, subnet, routing, firewall, WAF, gateway, load balancer, DNS, proxy, endpoint pubblici e privati, NAT, accessi amministrativi, connettività esterna.
- Matrice di connettività: origine, destinazione, protocollo, porta, direzione, scopo, ambiente, controllo, owner, stato.
- Vista C4 di deployment per DEV e PROD, più TEST, UAT, STAGING e DR quando pertinenti. Indicare nodi, istanze, alta disponibilità e domini di guasto.
- Per ogni ambiente: scopo, tenancy, base di sizing, classe di dati, identità, segreti, integrazioni, esposizione, scalabilità, telemetria, backup, alta disponibilità, accessi, costo.
- Promozione: artefatti e configurazioni immutabili, parità tra ambienti, deviazioni approvate, prevenzione del drift.
- Dati di produzione fuori da PROD solo con autorizzazione esplicita; preferire dati sintetici o mascherati.

Piani IP volatili e valori di build restano nei documenti di implementazione, salvo che siano approvati e stabili.

### 7. Qualità, operatività, resilienza, DR e delivery

- **Scenari di qualità** nel formato: sorgente, stimolo, artefatto, ambiente, risposta, misura. Ogni scenario ha target, metodo di misura, owner, metodo di validazione ed evidenza.
- **Capacità e costi**: assunzioni sul carico, metodo di sizing, margine, soglie, quote, licenze, driver di costo, budget e allarmi.
- **Operatività**: concetti trasversali (configurazione, segreti, certificati, health, feature flag, versionamento API), log, metriche, trace, audit, correlation ID, retention, RACI, orari di supporto, runbook, escalation, manutenzione, patching.
- **Resilienza e DR**: single point of failure, modalità di guasto delle dipendenze, ridondanza, zone e regioni, failover, degrado, riconciliazione, ripristino. Dichiarare SLI, SLO, SLA, RTO, RPO, frequenza e retention dei backup, restore test, procedura di invocazione e failback. Se non approvati: TBD.
- **Delivery e supply chain**: repository, branching, gate di qualità e sicurezza in CI, SBOM, scansione di dipendenze e immagini, firma e provenienza degli artefatti, registry, IaC, migrazioni, promozione, approvazioni, segregazione dei compiti, rollback o roll-forward, conservazione delle evidenze. Codice, configurazione e segreti separati.

Il dettaglio operativo di delivery resta in `tiziano-delivery`.

### 8. Decisioni, rischi, debito, assunzioni, punti aperti, glossario

- **Decisione**: ID, preoccupazione, opzioni, scelta, motivazione, scenari di qualità coinvolti, conseguenze, owner, data, stato, link all'ADR.
- **Rischio**: causa, evento, impatto, probabilità, esposizione, mitigazione, piano di contingenza, rischio residuo, owner, stato.
- **Debito**: impatto, trigger di rimedio, data di scadenza o revisione, owner.
- **Assunzione**: metodo e data di validazione.
- **Dipendenza**: owner, SLA, fallback.
- **Punto aperto**: owner, scadenza, impatto bloccante.
- **Glossario e riferimenti**: termini di dominio, acronimi, termini ambigui, notazione, standard, policy, ADR, specifiche di interfaccia, LLD, runbook, documenti sorgente.

## Policy dei diagrammi

Set minimo: contesto, container o logica, flusso dinamico critico, deployment DEV, deployment PROD e, quando i confini di fiducia contano, sicurezza e rete.

- Un solo livello di astrazione per vista.
- Indicare scope e ambiente, direzione e ownership, includere una legenda.
- Nessun diagramma decorativo né icone non spiegate.
- Ogni elemento e relazione è nominato e descritto.
- Ogni vista risponde a una preoccupazione dichiarata.
- Usare Mermaid, PlantUML o Structurizr se supportati; altrimenti fornire una specifica pronta per Visio o draw.io.

## HLD e LLD

**Resta nell'HLD**: sistemi e container, responsabilità, interfacce, protocolli, modelli, nodi di deployment, pattern, ownership, modalità di guasto, scenari di qualità, scelte tecnologiche.

**Va nell'LLD**: classi e funzioni, struttura dei package, algoritmi, dettagli dei payload, DDL e query, mapping dei campi, Dockerfile, codice di pipeline o IaC, configurazione esatta, passi di build.

Un dettaglio di basso livello resta nell'HLD solo se dimostra un vincolo, un contratto, un rischio, la base di sizing o un controllo.

## Review gate

Verificare:

- tracciabilità tra preoccupazioni, scope, requisiti, componenti, interfacce, dati, controlli, decisioni e scenari di qualità;
- ogni componente ha owner, deployment, dipendenze, stato, scalabilità e stato di ciclo di vita;
- ogni flusso ha direzione, classificazione, controllo di identità, telemetria e comportamento in caso di errore;
- differenze tra ambienti, promozione, segregazione dei dati, accessi, percorsi di rete, alta disponibilità, restore, RTO/RPO, supporto, rollback, capacità, licenze e costi;
- viste e cataloghi sono coerenti; rischi, debito, assunzioni, eccezioni, rischio residuo e TBD hanno owner e criteri di chiusura;
- non restano affermazioni senza supporto, decisioni obsolete, dettagli LLD superflui o preoccupazioni degli stakeholder non coperte.

Ogni area riceve un punteggio: `0` mancante, `1` parziale, `2` sufficiente. Un'area critica con punteggio `0` blocca l'approvazione.

## Regole di output

- **Creazione**: produrre la bozza più completa possibile, poi registro delle lacune e domande mirate.
- **Revisione**: valutazione complessiva, blocchi, punteggi per area, incoerenze, evidenze mancanti, modifiche compatte e, se richiesto, documento revisionato.
- Etichettare `FATTO`, `ASSUNZIONE`, `RACCOMANDAZIONE`, `DECISIONE`, `RISCHIO`, `DEBITO`, `TBD` dove l'ambiguità conta.
- Chiudere con: decisioni richieste, rischi da accettare, evidenze mancanti, prossime azioni di architecture review.
