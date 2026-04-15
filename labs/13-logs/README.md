# Lab 13 - Logs com Loki (Observabilidade)

## Objetivo

Demonstrar a centralização e consulta de logs no Kubernetes utilizando Loki, além de validar a integração com Grafana.

---

## Pré-requisitos

* Cluster Kubernetes (kind)
* Grafana instalado (via kube-prometheus-stack)
* Loki instalado no cluster

Verifique:

```bash
kubectl get pods | grep loki
kubectl get pods | grep grafana
```

---

## Conceitos chave

* **Logs** → registros detalhados de eventos
* **Loki** → armazenamento de logs
* **Promtail** → coleta de logs (opcional no lab local)
* **Grafana** → visualização e consulta

---

## Arquitetura

```
Pods → Logs → Loki → Grafana
```

---

## Etapa 1 — Validar Loki

```bash
kubectl get svc | grep loki
```

Esperado:

* Service `loki` ativo na porta 3100

Testar endpoint:

```bash
kubectl port-forward svc/loki 3100:3100
```

Em outro terminal:

```bash
curl http://localhost:3100/ready
```

Resultado esperado:

```
ready
```

---

## Etapa 2 — Validar comunicação interna

```bash
kubectl get pods | grep grafana
```

Testar acesso ao Loki de dentro do Grafana:

```bash
kubectl exec -it <grafana-pod> -- curl http://loki.default.svc.cluster.local:3100/ready
```

Resultado esperado:

```
ready
```

---

## Etapa 3 — Configurar datasource no Grafana

Acesse:

```
http://localhost:3000
```

Adicionar datasource:

* Tipo: **Loki**
* URL:

```
http://loki.default.svc.cluster.local:3100
```

Salvar e testar conexão.

---

## Etapa 4 — Gerar logs

Selecionar um pod nginx:

```bash
kubectl get pods -l app=nginx
```

Ver logs:

```bash
kubectl logs <pod-nginx>
```

---

## Etapa 5 — Consultar logs no Grafana

No Grafana:

* Vá em **Explore**
* Selecione o datasource **Loki**
* Execute:

```text
{app="nginx"}
```

---

## Resultado esperado

* Conexão Grafana ↔ Loki funcional
* Possibilidade de consulta de logs

---

## ⚠️ Observação importante (ambiente local)

Em ambientes locais (como kind), pode ocorrer falha na consulta de logs mesmo com a conexão válida.

Isso acontece porque:

* O **Promtail (coletor de logs)** pode não funcionar corretamente
* O Loki não recebe logs automaticamente

A conectividade foi validada manualmente com:

```bash
kubectl exec -it <grafana-pod> -- curl http://loki.default.svc.cluster.local:3100/ready
```

---

## Diferença importante

| Componente | Função         |
| ---------- | -------------- |
| Loki       | Armazena logs  |
| Promtail   | Coleta logs    |
| Grafana    | Visualiza logs |

---

## Troubleshooting

### Loki não responde

```bash
kubectl get pods | grep loki
kubectl logs <loki-pod>
```

---

### Grafana não conecta

Verificar:

* URL correta (`loki.default.svc.cluster.local`)
* Sem headers customizados
* Sem autenticação ativa

---

### Sem logs na query

Causa comum:

* Promtail não coletando logs

---

## Insights

* Logs dependem de coleta (Promtail)
* Loki sozinho não gera logs
* Ambientes locais têm limitações de observabilidade
* Grafana + Loki é padrão de mercado

---

## Conclusão

A arquitetura de observabilidade foi validada com sucesso.

Mesmo com limitações do ambiente local, a integração entre Grafana e Loki foi comprovada, refletindo um cenário real de produção com coleta de logs adequada.

---
