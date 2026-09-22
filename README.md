# Thales Gaia K8s

Questo repository contiene i manifest e le configurazioni Kubernetes utilizzate per l'ambiente **Thales Gaia**.

* **`envoy-gateway/`** — Envoy Gateway, utilizzato come **API Gateway controller** per gestire il traffico verso i servizi esposti tramite Gateway API.

* **`envoy-gateway-redis/`** — Redis utilizzato come componente di supporto per il **rate limiting**, permettendo di mantenere lo stato necessario per applicare i limiti alle richieste.

* **`envoy-ai-gateway-crd/`** — CRD necessarie per le funzionalità di **AI Gateway**, che permettono di configurare e gestire i workload AI tramite le risorse Kubernetes dedicate.

* **`envoy-ai-gateway/`** — Configurazione dell'**AI Gateway**, utilizzata per gestire il routing e l'accesso ai servizi/modelli AI attraverso Envoy Gateway.

* **`openwebui/`** — Open WebUI, interfaccia web per interagire con i **modelli di intelligenza artificiale** e i relativi backend.

In sintesi, lo stack combina **Envoy Gateway + AI Gateway + Redis** per la gestione e il controllo del traffico AI, con **Open WebUI** come interfaccia utente.