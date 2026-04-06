# Lab 07 - Troubleshooting (cenário real de falha)

## Objetivo

Diagnosticar e corrigir um problema comum em Kubernetes: Service não consegue se comunicar com os Pods.

---

## Cenário

A aplicação está rodando, mas:

* O Service não responde
* A comunicação falha

---

## Conceitos chave

* Labels
* Selectors
* Endpoints
* Debug com kubectl

---

## Etapa 1 — Criar cenário quebrado

Criar um Service com selector errado:

Arquivo: `manifests/nginx-broken-service.yaml`

```yaml id="r4y8zp"
apiVersion: v1
kind: Service
metadata:
  name: nginx-broken
spec:
  selector:
    app: nginx-errado
  ports:
    - port: 80
      targetPort: 80
```

Aplicar:

```bash id="k2d9ls"
kubectl apply -f manifests/nginx-broken-service.yaml
```

---

## Etapa 2 — Validar Service

```bash id="f8n3qk"
kubectl get svc
```

---

## Etapa 3 — Ver endpoints

```bash id="m6z1wc"
kubectl get endpoints nginx-broken
```

---

## Resultado esperado

```text id="d7q2bx"
<none>
```

---

## 🔥 Diagnóstico

Service existe, mas:

* Não encontra Pods
* Não há endpoints

---

## Etapa 4 — Testar falha

```bash id="v9c4yr"
kubectl run teste --image=busybox --rm -it -- sh
```

Dentro:

```sh id="b3w8fz"
wget -qO- nginx-broken
```

---

## Resultado esperado

* Falha de conexão

---

## Etapa 5 — Investigar Pods

```bash id="g1t5xk"
kubectl get pods --show-labels
```

Exemplo:

```text id="x8m2sd"
app=nginx
```

---

## 🔎 Problema identificado

```text id="y5r9hn"
Service selector = nginx-errado
Pods label = nginx
```

---

## Etapa 6 — Corrigir Service

Editar:

```bash id="c4k8pz"
kubectl edit svc nginx-broken
```

Corrigir:

```yaml id="n7f2qe"
selector:
  app: nginx
```

---

## Etapa 7 — Validar correção

```bash id="z2m6yv"
kubectl get endpoints nginx-broken
```

Esperado:

* IPs dos Pods

---

## Testar novamente

```sh id="t9w3aj"
wget -qO- nginx-broken
```

---

## Resultado esperado

* Retorno HTML do nginx

---

## O que aconteceu

```text id="k8n4pt"
Selector errado
↓
Sem endpoints
↓
Service não funciona
```

---

## Fluxo correto

```text id="v2r7dx"
Service selector
↓
Match com labels
↓
Endpoints criados
↓
Tráfego funcionando
```

---

## Troubleshooting mental (guarde isso)

Quando Service não responde:

1. Verificar Pods:

```bash
kubectl get pods
```

2. Verificar Service:

```bash
kubectl get svc
```

3. Verificar endpoints:

```bash
kubectl get endpoints
```

4. Ver labels:

```bash
kubectl get pods --show-labels
```

---

## Insights

* 90% dos problemas são label/selector
* Endpoints são a chave do diagnóstico
* kubectl é sua principal ferramenta

---

## Conclusão

Saber diagnosticar problemas é o principal diferencial entre níveis em Kubernetes.

Este cenário simula um erro comum em produção e demonstra a abordagem correta para resolução.
