# Lab 12 - Alertas com Prometheus (Alertmanager)

## Objetivo

Criar alertas automáticos no Kubernetes com base em métricas coletadas pelo Prometheus.

---

## Conceitos

* Métricas → dados coletados
* Regras → condições
* Alertas → ações baseadas em condições

---

## Arquitetura

```text
Aplicação → Métricas → Prometheus → Regra → Alerta
```

---

## Etapa 1 — Criar regra

Arquivo: `alert-rule.yaml`

```bash
kubectl apply -f labs/12-alerts/alert-rule.yaml
```

---

## Etapa 2 — Validar

```bash
kubectl get prometheusrules
```

---

## Etapa 3 — Testar alerta

```bash
kubectl delete pod -l app=nginx
```

---

## Resultado esperado

* Alerta é disparado após alguns segundos
* Visível no Prometheus/Grafana

---

## Troubleshooting

### Regra não aparece

```bash
kubectl get prometheusrules
```

---

### Alerta não dispara

* Verificar query (expr)
* Verificar se métrica existe

---

## Insights

* Alertas permitem reação automática a falhas
* Base para SRE e operação
* Dependem de boas métricas

---

## Conclusão

Alertmanager permite detectar problemas automaticamente, reduzindo tempo de resposta a incidentes.
