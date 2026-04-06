# Lab 06 - Resiliência e Alta Disponibilidade

## Objetivo

Validar como o Kubernetes mantém o estado desejado da aplicação, garantindo auto-recuperação (auto-healing) e alta disponibilidade.

---

## Pré-requisitos

* Deployment nginx com 3 réplicas
* Service nginx criado

Verifique:

```bash id="1x0q2a"
kubectl get pods -l app=nginx
```

---

## Conceitos chave

* Desired State → estado desejado definido no Deployment
* Auto-healing → Kubernetes recria Pods automaticamente
* Alta disponibilidade → múltiplas réplicas garantem continuidade

---

## Arquitetura

```id="3s1y8v"
Cliente
  ↓
Service (ClusterIP)
  ↓
Pods (3 replicas)
```

---

## Etapa 1 — Validar estado atual

```bash id="9o3p7k"
kubectl get pods -l app=nginx
```

Esperado:

* 3 Pods em estado Running

---

## Etapa 2 — Testar acesso

```bash id="n6b2fd"
kubectl run teste --image=busybox --rm -it -- sh
```

Dentro:

```sh id="x2k9lm"
wget -qO- nginx
```

---

## Etapa 3 — Simular falha (delete de Pod)

```bash id="q5z7wt"
kubectl delete pod -l app=nginx
```

---

## Etapa 4 — Observar comportamento

```bash id="u1v8cr"
kubectl get pods -w
```

---

## Resultado esperado

* Pods são terminados
* Novos Pods são criados automaticamente
* Número de réplicas volta para 3

---

## Etapa 5 — Validar continuidade

Enquanto os Pods são recriados:

```sh id="w7r4nj"
wget -qO- nginx
```

Esperado:

* Serviço continua respondendo

---

## Etapa 6 — Escalar aplicação

```bash id="z4m1xp"
kubectl scale deployment nginx --replicas=5
```

---

## Validar

```bash id="c6k8yt"
kubectl get pods -l app=nginx
```

---

## Resultado esperado

* 5 Pods em execução

---

## Etapa 7 — Reduzir escala

```bash id="p8n3js"
kubectl scale deployment nginx --replicas=2
```

---

## O que aconteceu

```text id="t4d9vc"
Deployment define estado desejado
↓
Kubernetes compara estado atual
↓
Ajusta automaticamente
```

---

## Teste avançado (opcional)

Forçar falha contínua:

```bash id="r2k5hb"
kubectl delete pod -l app=nginx --force --grace-period=0
```

---

## Troubleshooting

### Pods não recriam

* Verificar Deployment:

```bash id="e9x2kp"
kubectl get deployment
```

---

### Service não responde

* Verificar endpoints:

```bash id="h3f7ds"
kubectl get endpoints nginx
```

---

### Pods não ficam Running

* Ver logs:

```bash id="j6m1qa"
kubectl logs <pod>
```

---

## Insights

* Kubernetes trabalha com estado desejado
* Falhas são tratadas automaticamente
* Alta disponibilidade depende de múltiplas réplicas

---

## Conclusão

Deployments garantem resiliência e continuidade do serviço, sendo fundamentais para aplicações em produção.
