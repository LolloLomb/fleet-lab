# Thales Gaia K8s

Questo repository contiene i manifest e le configurazioni Kubernetes utilizzate per l'ambiente **Thales Gaia**.

Il presente documento descrive:

* i componenti applicativi presenti nel cluster;
* la funzione di ciascun componente;
* l'ordine corretto di installazione;
* le principali dipendenze tra i componenti.

L'installazione è organizzata in **quattro fasi principali**. La prima fase contiene i componenti infrastrutturali necessari al corretto funzionamento dello stack e deve essere completata prima di procedere con le fasi successive.

---

## 1. Ordine di installazione

L'ordine consigliato è il seguente:

```text
FASE 1 - Componenti infrastrutturali obbligatori
│
├── envoy-gateway
├── envoy-gateway-redis
├── envoy-ai-gateway-crd
├── envoy-ai-gateway
├── postgres-operator
└── valkey
        │
        ▼
FASE 2 - Monitoring e discovery (opzionale)
│
├── kube-prometheus-stack
└── Cluster Discovery Dashboard
        │
        ▼
FASE 3 - Logging e ricerca
│
├── opensearch
└── opensearch-dashboard
        │
        ▼
FASE 4 - Applicativi e GPU
│
├── keycloak
├── nvidia-operator
└── openwebui
```

> **Nota:** la **FASE 1 è propedeutica** alle fasi successive e deve essere installata per prima.
> Le fasi 2, 3 e 4 possono essere installate successivamente, verificando di volta in volta le eventuali dipendenze specifiche delle configurazioni utilizzate.

---

# 2. FASE 1 — Componenti infrastrutturali obbligatori

Questa fase costituisce la base dello stack **Thales Gaia**.

Prima di installare gli applicativi di livello superiore devono essere installati e configurati i componenti relativi a **Envoy**, i relativi componenti AI Gateway, il **Postgres Operator** e **Valkey**.

## 2.1 `envoy-gateway`

**Envoy Gateway** è il componente utilizzato come **API Gateway Controller** del cluster.

Il suo compito principale è gestire il traffico in ingresso verso i servizi Kubernetes attraverso le risorse della **Gateway API**.

In particolare permette di:

* esporre servizi attraverso Gateway e HTTPRoute;
* gestire il routing del traffico;
* applicare policy al traffico;
* centralizzare la gestione degli endpoint esposti;
* fornire il layer di ingresso utilizzato dai servizi AI e dagli altri workload.

**Installazione:** primo componente dello stack Envoy.

---

## 2.2 `envoy-gateway-redis`

Questo componente fornisce un'istanza Redis utilizzata come componente di supporto per le funzionalità di **rate limiting**.

Redis permette di mantenere lo stato necessario per applicare limiti alle richieste anche quando il traffico viene gestito da più istanze/pod.

Viene utilizzato quindi come componente di supporto alle funzionalità di controllo del traffico di Envoy.

**Installazione:** dopo `envoy-gateway` e prima delle configurazioni che utilizzano le funzionalità che dipendono da Redis.

---

## 2.3 `envoy-ai-gateway-crd`

Questo componente installa le **Custom Resource Definitions (CRD)** necessarie all'AI Gateway.

Le CRD estendono l'API Kubernetes introducendo le risorse necessarie per configurare i servizi e i workload relativi al traffico AI.

Le CRD devono essere disponibili **prima dell'installazione/configurazione di `envoy-ai-gateway`**, perché quest'ultimo utilizza tali risorse Kubernetes.

**Installazione:** dopo la disponibilità di Envoy Gateway e prima di `envoy-ai-gateway`.

---

## 2.4 `envoy-ai-gateway`

L'**Envoy AI Gateway** costituisce il layer dedicato alla gestione del traffico verso i servizi e i modelli di intelligenza artificiale.

Utilizza le funzionalità di Envoy Gateway e le CRD precedentemente installate per permettere di configurare:

* routing verso servizi AI;
* accesso ai modelli;
* gestione centralizzata delle richieste AI;
* policy e controllo del traffico AI;
* integrazione con i backend AI presenti nel cluster.

**Installazione:** dopo `envoy-gateway` e `envoy-ai-gateway-crd`.

---

## 2.5 `postgres-operator`

Il **Postgres Operator** permette di gestire database PostgreSQL direttamente tramite Kubernetes.

L'operator automatizza e semplifica attività come:

* provisioning dei database PostgreSQL;
* gestione dei cluster PostgreSQL;
* configurazione delle istanze;
* gestione del ciclo di vita del database;
* operazioni di manutenzione e gestione secondo le funzionalità offerte dall'operator utilizzato.

La sua installazione deve precedere gli applicativi che necessitano di database PostgreSQL gestiti tramite Kubernetes.

**Installazione:** nella prima fase, insieme ai componenti infrastrutturali.

---

## 2.6 `valkey`

**Valkey** è un datastore in-memory compatibile con il modello operativo di Redis.

Viene utilizzato come componente infrastrutturale per applicazioni che necessitano di:

* caching;
* storage temporaneo;
* sessioni;
* code o meccanismi di coordinamento;
* dati a bassa latenza.

La sua disponibilità deve precedere l'installazione degli applicativi che lo utilizzano come backend.

**Installazione:** nella prima fase, insieme agli altri componenti infrastrutturali.

---

# 3. FASE 2 — Monitoring e Cluster Discovery

Una volta completata la prima fase, è possibile installare lo stack di **monitoring**.

Questa fase è **opzionale** e può essere installata quando sono richieste funzionalità di osservabilità, monitoring e discovery dell'ambiente Kubernetes.

## 3.1 `kube-prometheus-stack`

Il **kube-prometheus-stack** fornisce lo stack principale per il monitoring del cluster Kubernetes.

Include i componenti necessari per raccogliere e visualizzare metriche relative a:

* nodi Kubernetes;
* pod e container;
* workload;
* risorse CPU e memoria;
* componenti del cluster;
* applicazioni che espongono metriche Prometheus.

Lo stack consente di avere una visione dello stato e dell'utilizzo delle risorse del cluster.

**Installazione:** dopo il completamento della FASE 1.

---

## 3.2 Cluster Discovery Dashboard

La **Cluster Discovery Dashboard** è la dashboard messa a disposizione per la **scoperta e la visualizzazione dei cluster** e per le funzionalità di monitoring previste dall'ambiente.

Può essere installata insieme allo stack di monitoring quando sono richieste funzionalità centralizzate di:

* discovery dei cluster;
* visualizzazione delle informazioni sui cluster;
* accesso alle informazioni di monitoring;
* consultazione dello stato dell'infrastruttura.

**Installazione:** dopo `kube-prometheus-stack` e dopo la disponibilità dei componenti necessari al monitoring.

> Se le funzionalità di discovery/monitoring non sono richieste nell'ambiente specifico, questa fase può essere omessa.

---

# 4. FASE 3 — Logging e Search

Dopo aver predisposto l'infrastruttura di base e, se necessario, il monitoring, è possibile installare lo stack **OpenSearch**.

## 4.1 `opensearch`

**OpenSearch** è il motore utilizzato per la ricerca, l'indicizzazione e la gestione di grandi quantità di dati, in particolare dati provenienti da log ed eventi.

Può essere utilizzato per:

* indicizzazione dei log;
* ricerca e analisi dei dati;
* aggregazione di eventi;
* conservazione dei dati necessari alle funzionalità di osservabilità;
* supporto alle attività di troubleshooting.

**Installazione:** nella FASE 3.

---

## 4.2 `opensearch-dashboard`

**OpenSearch Dashboards** fornisce l'interfaccia web per interagire con OpenSearch.

Permette di:

* effettuare ricerche sui dati indicizzati;
* visualizzare e analizzare i log;
* creare dashboard;
* creare visualizzazioni;
* effettuare analisi dei dati raccolti da OpenSearch.

Il componente dipende dalla disponibilità di OpenSearch.

**Installazione:** dopo `opensearch`.

---

# 5. FASE 4 — Identity, GPU e AI User Interface

L'ultima fase comprende i componenti applicativi e quelli necessari per l'esecuzione e l'accesso ai workload AI.

## 5.1 `keycloak`

**Keycloak** è il componente utilizzato per la gestione centralizzata dell'**Identity and Access Management (IAM)**.

Permette di gestire:

* utenti;
* gruppi;
* ruoli;
* autenticazione;
* autorizzazione;
* Single Sign-On (SSO);
* integrazione con applicazioni che supportano protocolli standard di autenticazione.

Può essere utilizzato come componente di autenticazione per gli applicativi della piattaforma.

**Installazione:** nella FASE 4.

---

## 5.2 `nvidia-operator`

Il **NVIDIA GPU Operator** automatizza la gestione dello stack software necessario per utilizzare le **GPU NVIDIA** all'interno del cluster Kubernetes.

Permette di predisporre i componenti necessari per rendere le GPU disponibili ai workload Kubernetes, inclusi, in base alla configurazione adottata:

* NVIDIA Driver;
* NVIDIA Container Toolkit;
* Kubernetes device plugin;
* componenti per la gestione e l'esposizione delle GPU.

È il componente da utilizzare quando il cluster deve eseguire workload che richiedono accelerazione GPU NVIDIA.

**Installazione:** nella FASE 4, prima dei workload AI che richiedono GPU.

---

## 5.3 `openwebui`

**Open WebUI** è l'interfaccia utente web per interagire con i modelli di intelligenza artificiale e con i relativi backend.

Fornisce un'interfaccia centralizzata attraverso la quale gli utenti possono accedere alle funzionalità AI configurate nell'ambiente.

Può essere integrato con i componenti AI Gateway e con i backend AI disponibili nel cluster.

**Installazione:** al termine della procedura, dopo che i componenti infrastrutturali e i servizi necessari sono disponibili.

---

# 6. Sequenza operativa completa

La sequenza consigliata per l'installazione è quindi:

### FASE 1 — Infrastruttura

1. **`envoy-gateway`**
2. **`envoy-gateway-redis`**
3. **`envoy-ai-gateway-crd`**
4. **`envoy-ai-gateway`**
5. **`postgres-operator`**
6. **`valkey`**

Questa fase deve essere completata prima di procedere con gli altri componenti.

### FASE 2 — Monitoring e Discovery — opzionale

7. **`kube-prometheus-stack`**
8. **Cluster Discovery Dashboard**

### FASE 3 — Logging e Search

9. **`opensearch`**
10. **`opensearch-dashboard`**

### FASE 4 — Identity, GPU e AI

11. **`keycloak`**
12. **`nvidia-operator`**
13. **`openwebui`**

---

# 7. Riepilogo dei componenti

| Componente                  | Funzione                                   | Fase |
| --------------------------- | ------------------------------------------ | ---: |
| `envoy-gateway`             | API Gateway Controller / gestione traffico |    1 |
| `envoy-gateway-redis`       | Supporto al rate limiting                  |    1 |
| `envoy-ai-gateway-crd`      | CRD per AI Gateway                         |    1 |
| `envoy-ai-gateway`          | Routing e gestione traffico AI             |    1 |
| `postgres-operator`         | Gestione PostgreSQL su Kubernetes          |    1 |
| `valkey`                    | Datastore/cache in-memory                  |    1 |
| `kube-prometheus-stack`     | Monitoring e metriche Kubernetes           |    2 |
| Cluster Discovery Dashboard | Discovery e visualizzazione dei cluster    |    2 |
| `opensearch`                | Indicizzazione, ricerca e analisi dati/log |    3 |
| `opensearch-dashboard`      | Dashboard e interfaccia per OpenSearch     |    3 |
| `keycloak`                  | Identity e Access Management               |    4 |
| `nvidia-operator`           | Gestione GPU NVIDIA nel cluster            |    4 |
| `openwebui`                 | Interfaccia web per i servizi/modelli AI   |    4 |

---

# 8. Considerazioni sulle dipendenze

L'ordine sopra riportato deve essere considerato come **ordine logico di installazione dello stack**.

In particolare:

* le **CRD di Envoy AI Gateway** devono essere disponibili prima delle risorse che le utilizzano;
* **Envoy Gateway** deve essere disponibile prima delle configurazioni che dipendono dal Gateway API;
* **Redis** deve essere disponibile prima delle funzionalità che lo utilizzano per il rate limiting;
* **Postgres Operator** deve essere disponibile prima degli eventuali database PostgreSQL gestiti tramite operator;
* **Valkey** deve essere disponibile prima degli applicativi che lo utilizzano come datastore;
* **OpenSearch Dashboards** deve essere installato dopo OpenSearch;
* **NVIDIA Operator** deve essere configurato prima dei workload che richiedono GPU NVIDIA;
* **OpenWebUI** deve essere installato quando i servizi/backend AI a cui deve collegarsi sono disponibili e raggiungibili.

Prima di procedere alla fase successiva è consigliato verificare che i componenti della fase precedente siano **correttamente installati, in stato `Ready` e senza errori nei relativi controller/operator**.

---

# 9. Architettura logica

Lo stack può essere rappresentato concettualmente nel seguente modo:

```text
                         ┌─────────────────────┐
                         │     OpenWebUI       │
                         │    AI User UI       │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │  Envoy AI Gateway   │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    Envoy Gateway    │
                         │     API Gateway     │
                         └──────────┬──────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
                    ▼               ▼               ▼
               AI Services      Applications    Other APIs


┌──────────────────────────────────────────────────────────────────┐
│                    Infrastructure Layer                          │
│                                                                  │
│  PostgreSQL Operator       Valkey       Redis                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘


┌──────────────────────────────────────────────────────────────────┐
│                    Observability Layer                           │
│                                                                  │
│  kube-prometheus-stack      OpenSearch      OpenSearch Dashboard │
│                                                                  │
│  Cluster Discovery Dashboard                                   │
└──────────────────────────────────────────────────────────────────┘


┌──────────────────────────────────────────────────────────────────┐
│                    GPU / Identity Layer                          │
│                                                                  │
│       NVIDIA Operator                  Keycloak                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

L'architettura è quindi composta da un **layer infrastrutturale**, un **layer di networking e AI Gateway**, un **layer di observability**, e infine dai componenti applicativi per **identity, GPU e accesso ai servizi AI**.
