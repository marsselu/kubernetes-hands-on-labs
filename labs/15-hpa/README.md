# Lab 15 - HPA (Horizontal Pod Autoscaler)

## Objetivo

Validar o escalonamento automático de Pods no Kubernetes com base no uso de CPU, utilizando o Horizontal Pod Autoscaler (HPA).

---

## Pré-requisitos

* Cluster Kubernetes (kind)
* Metrics Server instalado
* Deployment nginx ativo

Verifique:

```bash
kubectl get pods -n kube-system | grep metrics
kubectl top pods
```

---

## Conceitos chave

* **HPA (Horizontal Pod Autoscaler)** → ajusta o número de Pods automaticamente
* **CPU utilization** → base para decisão de scaling
* **resources.requests** → referência para cálculo de uso
* **Auto scaling** → adaptação dinâmica à carga

---

## Arquitetura

```text
Cliente → nginx → HPA → escala Pods automaticamente
```

---

## Etapa 1 — Verificar deployment

```bash
kubectl get deployment nginx
```

---

## Etapa 2 — Garantir resources definidos

O HPA depende de `requests.cpu`.

Verifique:

```bash
kubectl get deployment nginx -o yaml | grep -A10 resources
```

Caso não exista, edite:

```bash
kubectl edit deployment nginx
```

Adicionar no container:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "64Mi"
  limits:
    cpu: "200m"
    memory: "128Mi"
```

---

## Etapa 3 — Criar HPA

```bash
kubectl autoscale deployment nginx --cpu-percent=50 --min=2 --max=10
```

Validar:

```bash
kubectl get hpa
```

---

## Etapa 4 — Validar métricas

```bash
kubectl top pods
```

Resultado esperado:

* valores de CPU visíveis

---

## Etapa 5 — Gerar carga

Criar pod gerador:

```bash
kubectl run load-generator --image=busybox --rm -it -- sh
```

Dentro do pod:

```sh
while true; do wget -q -O- http://nginx > /dev/null; done
```

---

## Etapa 6 — Observar scaling

Em outro terminal:

```bash
kubectl get hpa -w
```

e:

```bash
kubectl get pods -l app=nginx -w
```

---

## Resultado esperado

* uso de CPU aumenta
* HPA detecta aumento
* número de réplicas cresce automaticamente

Exemplo:

```text
44% / 50%
```

---

## O que aconteceu

1. HPA monitora uso de CPU dos Pods
2. Com base no `requests.cpu`, calcula percentual de uso
3. Se o uso se aproxima do target (50%), o número de réplicas aumenta

---

## Troubleshooting

### `<unknown>` no TARGET

Causa:

* ausência de `resources.requests.cpu`

Solução:

```yaml
resources:
  requests:
    cpu: "100m"
```

---

### HPA não escala

Verificar:

```bash
kubectl describe hpa nginx
```

---

### Métricas não aparecem

Verificar Metrics Server:

```bash
kubectl get pods -n kube-system | grep metrics
kubectl top pods
```

---

## Insights

* HPA utiliza **requests**, não limits
* Sem Metrics Server, HPA não funciona
* Escalonamento é baseado em média de uso dos Pods
* Esse modelo é amplamente usado em produção

---

## Conclusão

O HPA permite que aplicações em Kubernetes se adaptem automaticamente à carga, aumentando ou reduzindo o número de Pods conforme necessário.

Essa funcionalidade é essencial para garantir performance e eficiência de recursos em ambientes produtivos.

---
