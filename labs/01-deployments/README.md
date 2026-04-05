# Lab 01 - Kubernetes Deployments

## Objetivo

Demonstrar a criação e gerenciamento de aplicações utilizando Deployments no Kubernetes, incluindo escalabilidade e auto-recuperação.

---

## Conceitos abordados

* Deployment
* ReplicaSet
* Pods
* Auto-healing
* Escalabilidade

---

## Arquitetura

Deployment gerencia um ReplicaSet que, por sua vez, mantém o número desejado de Pods em execução.

```
Deployment
   ↓
ReplicaSet
   ↓
Pods (3 replicas)
```

---

## Execução

### Criar o Deployment

```bash
kubectl apply -f manifests/nginx-deployment.yaml
```

---

### Verificar Deployment

```bash
kubectl get deployments
```

---

### Verificar Pods

```bash
kubectl get pods -o wide
```

---

## Teste de resiliência

Delete um Pod manualmente:

```bash
kubectl delete pod -l app=nginx
```

---

## Resultado esperado

* O Pod será recriado automaticamente
* O número de réplicas será mantido em 3

---

## Insights

* O Kubernetes garante o estado desejado
* Deployments fornecem auto-recuperação
* ReplicaSets garantem a quantidade de Pods

---

## Nível de aplicação

Esse padrão é utilizado em ambientes de produção para garantir alta disponibilidade e resiliência de aplicações.
