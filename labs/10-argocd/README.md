# Lab 10 - GitOps com ArgoCD

## Objetivo

Demonstrar deploy automatizado utilizando GitOps com ArgoCD.

---

## Conceitos

* GitOps → Git como fonte da verdade
* ArgoCD → controlador de deploy
* Sincronização automática

---

## Fluxo

```text
Git → ArgoCD → Kubernetes
```

---

## Execução

### Instalar ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

### Acessar

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

---

### Login

* user: admin
* password via secret

---

### Criar aplicação

* Repo: repositório Git
* Path: chart Helm
* Sync: automático

---

## Teste

Alterar `values.yaml` e fazer commit.

---

## Resultado esperado

* ArgoCD detecta mudanças
* Atualiza cluster automaticamente

---

## Insights

* Remove deploy manual
* Garante consistência
* Facilita auditoria

---

## Conclusão

GitOps é padrão moderno de deploy em Kubernetes, trazendo automação, rastreabilidade e segurança.
