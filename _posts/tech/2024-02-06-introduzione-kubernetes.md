---
title: Introduzione a Kubernetes
date: 2024-02-06 12:00:00 +0100
categories: [tech]
tags: [container, orchestrazione, cloud]
# Se vuoi mettere un'immagine grande in alto:
# image:
#   path: /assets/img/headers/k8s-banner.jpg
---

Benvenuti nel mio nuovo blog. Oggi parliamo di **Kubernetes**.

Kubernetes (spesso abbreviato in K8s) è un sistema open-source per l'automazione del deployment, dello scaling e della gestione di applicazioni containerizzate.

## Perché usare Kubernetes?

Se vieni dal mondo delle macchine virtuali o di Docker semplice, potresti chiederti perché aggiungere questa complessità. Ecco i motivi principali:

1.  **Self-healing**: Riavvia i container che falliscono.
2.  **Scaling automatico**: Aumenta o diminuisce le risorse in base al traffico.
3.  **Rollout e Rollback**: Gestisce gli aggiornamenti senza downtime.

## Esempio di un Pod

In Kubernetes, l'unità base non è il container, ma il **Pod**. Ecco come appare la definizione di un Pod semplice in YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
