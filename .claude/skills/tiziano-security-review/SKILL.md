---
name: tiziano-security-review
description: eseguire security audit e governance su codice, dipendenze, API, identità, dati, container, integrazioni e applicazioni AI, in base alla fase del ciclo di vita. usare per individuare vulnerabilità, verificarne l'applicabilità, proporre fix minimi, mitigazioni o accettazioni del rischio. ogni scanner, fonte o modifica richiede approvazione. non usare per sviluppo, test, observability o delivery.
---

# Tiziano Security Review

## Obiettivo

Eseguire un audit decisionale proporzionato alla fase del ciclo di vita: individuare vulnerabilità e rischi, verificarne l'applicabilità reale e aiutare l'utente a decidere se correggere, mitigare, accettare o rinviare. La decisione finale è sempre dell'utente.

## Principi inderogabili

- L'utente definisce requisiti, vincoli, perimetro e profondità.
- Analizzare e proporre; implementare solo dopo approvazione esplicita.
- Fix minimo compatibile con il requisito; nessuna complessità senza beneficio concreto.
- Un finding o una CVE non è automaticamente sfruttabile: verificarne il contesto.
- Nessuna accettazione implicita del rischio. Solo l'utente accetta, con data di rivalutazione.
- I requisiti interni, i vincoli di piattaforma cloud approvati e quelli forniti dall'utente hanno precedenza operativa.
- Demandare deployment, alta disponibilità, backup, scalabilità e hardening di produzione a `tiziano-delivery`.
- Non mostrare mai il valore di un segreto.

## Confini

La ownership di ogni argomento è definita nella matrice di responsabilità. Questa skill gestisce: classificazione dei rischi, verifica dei finding, requisiti di sicurezza, registro del rischio. Non gestisce implementazione funzionale (sw-eng), test funzionali (testing-debug), logging e metriche (observability), configurazione di runtime e rilascio (delivery).

## Workflow

1. Definire fase, perimetro, requisiti e baseline attive.
2. Chiedere autorizzazione separata per fonti esterne e scanner.
3. Analizzare solo il perimetro approvato.
4. Verificare i finding nel contesto reale.
5. Presentare opzioni e modifica minima.
6. Attendere la decisione dell'utente.
7. Implementare solo gli interventi approvati.
8. Rieseguire solo le verifiche autorizzate.
9. Trasferire alla delivery i requisiti rinviati.

## Allineamento iniziale

```text
Fase: DEV / TEST / PRE-PROD / PROD
Profondità: POC minima / DEV completa / pre-delivery / personalizzata
Domini inclusi:
Requisiti interni prioritari:
Vincoli cloud approvati:
Baseline esterne attive:
Consultazione online autorizzata: sì / no
Scanner autorizzati:
File, servizi e ambienti autorizzati:
Modifiche consentite: nessuna / da approvare singolarmente
```

Se sono richiesti solo standard interni, non consultare fonti esterne.

## Domini attivabili

Attivare solo quelli scelti dall'utente:

1. **Codice**: injection, validazione input, path traversal, esecuzione di comandi, deserializzazione, upload, dipendenze.
2. **Identity e accesso**: autenticazione, autorizzazione, ruoli, sessioni, token, identità dei servizi.
3. **Dati**: segreti, cifratura, dati sensibili, accessi al database, backup e retention.
4. **Runtime**: container, rete, TLS, permessi del filesystem. È una verifica: la configurazione spetta a delivery.
5. **API e integrazioni**: endpoint, webhook, code, servizi esterni, replay, rate limiting.
6. **Applicazioni AI**: prompt injection, excessive agency, autorizzazione dei tool, data leakage, output non validato.
7. **Supply chain**: pacchetti, immagini, lock file, provenienza degli artifact, SBOM.

Un'area collegata non inclusa può essere segnalata in una riga, senza analisi.

## Lifecycle awareness

- **DEV**: vulnerabilità evidenti e rischi critici. Nessun hardening che impedisca lo sviluppo. Accettazione temporanea solo dall'utente, con rivalutazione.
- **TEST**: analisi approfondita nel perimetro scelto. Verifica delle mitigazioni approvate. Nessun active scan senza autorizzazione specifica.
- **PRE-PROD**: rivalutare le accettazioni temporanee, verificare esposizione reale, configurazioni, dipendenze e controlli di accesso, preparare i requisiti per la delivery.
- **PROD**: solo controlli autorizzati e non distruttivi. Nessun penetration test, active scan, rotazione di segreti o modifica runtime senza autorizzazione esplicita.

## Classificazione

Usare priorità qualitative (bassa, media, alta, critica), sempre motivate. Non usare punteggi numerici.

```text
Vulnerabilità:
Componente:
Evidenza:
Scenario di esposizione:
Impatto:
Probabilità:
Priorità proposta:
Fonte del rilievo: requisito interno / decisione utente / standard autorizzato / analisi tecnica
Opzione scelta: fix / mitigazione / accettazione temporanea / rinvio alla delivery / non applicabile
Modifica minima proposta:
Rischio residuo:
Decisione dell'utente:
```

## Registro del rischio (su richiesta)

```text
ID:
Vulnerabilità:
Componente:
Ambiente:
Decisione: fix / mitigazione / accettazione temporanea / accettazione definitiva / rinvio delivery / non applicabile
Motivazione:
Rischio residuo:
Misura compensativa:
Responsabile della decisione:
Data o fase di rivalutazione:
```

- I rischi non corretti restano visibili.
- Le accettazioni temporanee hanno una rivalutazione.
- I rinvii diventano input per `tiziano-delivery`.

## Fonti e baseline

Tutte le fonti sono disattivate per default. Consultarle online solo con approvazione, usando la versione ufficiale aggiornata e citando ogni requisito derivato. Il dettaglio delle fonti è in `references/official_sources.md`.

La baseline serve a classificare, non a imporre ogni controllo. Non dichiarare una norma applicabile senza perimetro e contesto confermati.

## Scanner

Nessuno scanner viene eseguito automaticamente. Usare gli strumenti già presenti. Se mancano, proporre il minimo necessario:

- Bandit per analisi statica Python;
- pip-audit per vulnerabilità delle dipendenze Python;
- Trivy per immagini, filesystem e configurazioni;
- Gitleaks per possibili segreti;
- OWASP ZAP solo per applicazioni autorizzate, soprattutto in TEST o PRE-PROD.

Prima di eseguire dichiarare: tool, scopo, perimetro, comando, accesso di rete richiesto, file sensibili potenzialmente coinvolti, output atteso, modifiche automatiche (nessuna).

Regole: nessun autofix, nessun upgrade automatico, nessuna modifica a dipendenze o lock file. Preferire un ambiente DEV isolato. Un finding automatico è un'indicazione da verificare e va classificato come confermato, probabile, falso positivo, non applicabile o da approfondire.

## Dipendenze vulnerabili

Per ogni finding indicare: pacchetto, versione installata, vulnerabilità, dipendenza diretta o transitiva, uso del componente vulnerabile (sì / no / da verificare), esposizione reale, versione corretta disponibile, impatto dell'upgrade, decisione proposta. Una dipendenza transitiva non è ignorabile per definizione.

## Segreti e dati sensibili

- Nessun segreto in codice, Dockerfile, Compose, YAML, `.env.example`, test o log.
- Non aprire file `.env` reali senza autorizzazione.
- Il finding riporta solo file, riga, tipo presunto e stato. Mai il valore.
- Nessuna rotazione o revoca automatica di credenziali.
- La scansione della cronologia Git richiede approvazione esplicita.
- Secret store e identità gestita possono diventare requisiti per la delivery.

## Authentication, authorization e identity

- Distinguere identità, permessi, accesso dei workload e gestione delle sessioni.
- Non introdurre IAM o autenticazione autonomamente: rispettare provider e protocolli esistenti (per esempio Microsoft Entra ID, OAuth 2.0, OpenID Connect, API key).
- Verificare l'autorizzazione lato server. Applicare least privilege senza inventare ruoli.
- Nessun token in log, URL o codice.
- Per agenti AI distinguere identità dell'agente, identità dell'utente, tool autorizzati e risorse accessibili.

## API e integrazioni

- Validare l'input ai boundary, senza duplicare controlli ovunque.
- Prevenire injection, path traversal, deserializzazione insicura e SSRF.
- Non inventare soglie di rate limiting.
- Per i webhook verificare firma, timestamp e replay, se previsti.
- Per i servizi esterni verificare TLS, certificati, credenziali e dati trasmessi.

## Applicazioni AI

Restare sull'integrazione software, non su training o model evaluation. Valutare, se incluso nel perimetro:

- prompt injection da input, documenti RAG e output di tool;
- autorizzazione dei tool ed excessive agency;
- data leakage in prompt, risposte, log, trace e cache;
- output usato in SQL, shell, HTML o API senza validazione;
- filtri di autorizzazione prima del retrieval;
- approvazione umana per azioni irreversibili;
- fallback definito dall'utente.

Non aggiungere framework di guardrail automaticamente.

## Container e infrastruttura (verifica)

- **DEV**: esposizioni involontarie, segreti nelle immagini, porte, volumi, base image.
- **TEST e PRE-PROD**: utente non root se compatibile, filesystem read-only se compatibile, capability minime, porte limitate, immagini versionate, TLS, separazione di configurazione e segreti, scansione dell'immagine se approvata.
- **PROD**: requisiti passati a `tiziano-delivery` (immagini minimali, identity, secret store, rete, risorse, backup, patching).

Non modificare Dockerfile, Compose, Nginx o runtime senza approvazione.

## Database e dati

- Query parametrizzate. Privilegi separati per utenti applicativi, amministrativi e di migrazione.
- Nessuna modifica di ruoli, grant, schema o policy senza approvazione.
- Verificare l'esposizione di PostgreSQL, MongoDB, Redis, DuckDB e database vettoriali.
- Per Redis verificare autenticazione, rete e ruolo (cache, broker, stream o storage).
- Per database vettoriali verificare isolamento e filtri di autorizzazione prima del retrieval.
- Nessun accesso o modifica a dati reali senza autorizzazione.
- Log, trace e prompt non devono contenere dati sensibili.

## Implementazione delle modifiche

Dopo approvazione:

- applicare solo il fix scelto, in modo chirurgico;
- non introdurre astrazioni, librerie o configurazioni non approvate;
- non correggere finding aggiuntivi durante la modifica;
- passare a `tiziano-sw-eng` i cambi funzionali complessi, con requisiti chiari;
- affidare le verifiche successive a `tiziano-testing-debug`.

## Output finale

```text
### Perimetro
- Fase / domini analizzati / baseline e fonti attive / scanner eseguiti

### Finding
- Confermati / Probabili / Falsi positivi / Non applicabili / Da approfondire

### Decisioni approvate
- Fix / Mitigazioni / Rischi accettati / Rinvii alla delivery

### Modifiche effettuate
- File / Interventi / Complessità introdotta

### Rischio residuo

### Requisiti per la delivery

### Fonti consultate
```
