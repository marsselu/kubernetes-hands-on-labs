# Lab 21 - ArgoCD (GitOps)

## Objetivo

Validar o modelo GitOps utilizando ArgoCD, onde o estado do cluster Kubernetes é gerenciado automaticamente a partir de um repositório Git.

---

## Pré-requisitos

* Cluster Kubernetes (kind)
* ArgoCD instalado
* Repositório GitHub com manifests Kubernetes

Verifique:

```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
```

---

## Conceitos chave

* **GitOps** → Git como fonte de verdade
* **ArgoCD** → controlador que aplica estado desejado
* **Desired State** → estado definido no Git
* **Actual State** → estado atual no cluster
* **Sync** → processo de convergência entre os dois estados

---

## Arquitetura

```text
Git → ArgoCD → Kubernetes Cluster
```

---

## Etapa 1 — Criar manifests

Estrutura:

```text
manifests/nginx/
```

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-gitops
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-gitops
  template:
    metadata:
      labels:
        app: nginx-gitops
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-gitops
spec:
  selector:
    app: nginx-gitops
  ports:
  - port: 80
    targetPort: 80
```

---

## Etapa 2 — Commit e push

```bash
git add .
git commit -m "feat: add nginx gitops manifests"
git push
```

---

## Etapa 3 — Criar aplicação no ArgoCD

Na UI do ArgoCD:

* Application Name: `nginx-gitops`
* Project: `default`

### Source

* Repository URL: repositório GitHub
* Revision: `main`
* Path: `manifests/nginx`

### Destination

* Cluster: `https://kubernetes.default.svc`
* Namespace: `default`

### Sync Policy

* Auto Sync: habilitado

---

## Etapa 4 — Sincronização

O ArgoCD:

* lê o repositório
* aplica os manifests no cluster
* mantém o estado sincronizado

---

## Etapa 5 — Validar recursos

```bash
kubectl get pods
kubectl get svc
```

Resultado esperado:

* pods `nginx-gitops`
* service `nginx-gitops`

---

## Etapa 6 — Teste de GitOps

Alterar no repositório:

```yaml
replicas: 2 → replicas: 4
```

Commit e push:

```bash
git add .
git commit -m "feat: scale nginx via gitops"
git push
```

---

## Resultado esperado

Sem usar `kubectl apply`:

```bash
kubectl get pods
```

* ArgoCD detecta mudança
* aplica automaticamente
* cluster atualiza para 4 réplicas

---

## O que aconteceu

1. O estado desejado foi definido no Git
2. O ArgoCD monitorou o repositório
3. Detectou divergência entre Git e cluster
4. Aplicou automaticamente a mudança

---

## Troubleshooting

### Aplicação não sincroniza

```bash
kubectl logs -n argocd deploy/argocd-application-controller
```

---

### Status não fica Healthy

Verificar:

```bash
kubectl get pods
kubectl describe deployment nginx-gitops
```

---

### Mudança no Git não reflete

Verificar:

* Auto Sync habilitado
* branch correta (`main`)
* path correto

---

## Insights

* Git se torna a fonte única de verdade
* elimina necessidade de `kubectl apply`
* reduz erro humano
* permite rollback simples via Git
* padrão amplamente usado em produção

---

## Conclusão

O lab demonstrou a implementação de GitOps com ArgoCD, onde o cluster Kubernetes é automaticamente sincronizado com o estado definido no Git.

Esse modelo é fundamental em ambientes modernos, garantindo consistência, auditabilidade e automação na entrega de aplicações.

---
