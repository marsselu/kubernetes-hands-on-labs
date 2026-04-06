# Lab 09 - Helm Avançado

## Objetivo

Demonstrar o uso avançado do Helm com ConfigMaps, Secrets e parametrização dinâmica.

---

## Conceitos abordados

* Templates Helm
* values.yaml
* ConfigMap dinâmico
* Secret dinâmico
* Deploy configurável

---

## Estrutura

* templates/configmap.yaml
* templates/secret.yaml
* templates/deployment.yaml
* values.yaml

---

## Execução

### Instalar

```bash
helm install nginx-advanced ./nginx-chart
```

---

### Validar

```bash
kubectl get pods
kubectl get configmap
kubectl get secret
```

---

### Testar variáveis

```bash
kubectl exec -it <pod> -- printenv
```

---

### Upgrade

```bash
helm upgrade nginx-advanced ./nginx-chart
```

---

## Insights

* Helm permite centralizar configuração
* Secrets e ConfigMaps podem ser parametrizados
* Deploy se torna reutilizável e escalável

---

## Conclusão

Helm é essencial para ambientes produtivos, permitindo padronização e gerenciamento eficiente de aplicações Kubernetes.
