# Lab 11 - Observabilidade com Prometheus

## Objetivo

Monitorar aplicações e o cluster Kubernetes utilizando Prometheus.

---

## Conceitos

* Métricas
* Monitoramento
* Saúde da aplicação
* Observabilidade

---

## Arquitetura

```text
Aplicação → Prometheus → Visualização
```

---

## Instalação

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-prometheus prometheus-community/kube-prometheus-stack
```

---

## Acesso

```bash
kubectl port-forward svc/kube-prometheus-kube-prome-prometheus 9090
```

---

## Testes

Query:

```text
up
```

---

## Métricas úteis

* kube_pod_status_phase
* container_cpu_usage_seconds_total
* container_memory_usage_bytes

---

## Teste de falha

```bash
kubectl delete pod -l app=nginx
```

---

## Resultado esperado

* Métricas refletem mudanças
* Novo pod aparece automaticamente

---

## Insights

* Observabilidade é essencial para produção
* Permite detectar falhas rapidamente
* Base para alertas e automação

---

## Conclusão

Prometheus é uma das principais ferramentas de monitoramento em Kubernetes, sendo amplamente utilizada em ambientes produtivos.
