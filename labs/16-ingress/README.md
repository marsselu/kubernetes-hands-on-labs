# Lab 16 - Ingress com Traefik

## Objetivo

Validar o roteamento de tráfego HTTP de entrada no Kubernetes utilizando Ingress com Traefik como Ingress Controller.

---

## Pré-requisitos

* Cluster Kubernetes (kind)
* Traefik instalado como Ingress Controller
* Deployment nginx ativo
* Service nginx funcionando

Verifique:

```bash
kubectl get pods | grep traefik
kubectl get svc traefik
kubectl get ingressclass
kubectl get svc nginx
kubectl get pods -l app=nginx
```

---

## Conceitos chave

* **Ingress** → define regras de entrada HTTP/HTTPS para serviços no cluster
* **Ingress Controller** → componente que implementa essas regras
* **Traefik** → Ingress Controller utilizado neste lab
* **Host-based routing** → roteamento baseado em domínio/host

---

## Arquitetura

```text
Cliente → Traefik (Ingress Controller) → Service nginx → Pods nginx
```

---

## Etapa 1 — Instalar o Traefik

Adicionar repositório Helm:

```bash
helm repo add traefik https://traefik.github.io/charts
helm repo update
```

Instalar:

```bash
helm install traefik traefik/traefik
```

Validar:

```bash
kubectl get pods | grep traefik
kubectl get svc | grep traefik
kubectl get ingressclass
```

Resultado esperado:

* pod do Traefik em `Running`
* service `traefik` criado
* ingress class `traefik` disponível

---

## Etapa 2 — Criar o recurso Ingress

Arquivo `ingress-nginx.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  ingressClassName: traefik
  rules:
  - host: nginx.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

Aplicar:

```bash
kubectl apply -f ingress-nginx.yaml
kubectl get ingress
kubectl describe ingress nginx-ingress
```

---

## Etapa 3 — Validar o backend

Verificar service e endpoints do nginx:

```bash
kubectl get svc nginx
kubectl get endpoints nginx
```

Resultado esperado:

* service `nginx` na porta 80
* endpoints apontando para os Pods nginx

---

## Etapa 4 — Testar roteamento interno

Criar um pod temporário com curl:

```bash
kubectl run teste --image=curlimages/curl --rm -it -- sh
```

Dentro do pod:

```sh
curl -H "Host: nginx.local" http://traefik.default.svc.cluster.local:80/
```

Resultado esperado:

* HTML do nginx retornado com sucesso

---

## O que esse teste faz

O comando abaixo envia uma requisição para o Traefik com o header HTTP `Host` definido manualmente:

```bash
curl -H "Host: nginx.local" http://traefik.default.svc.cluster.local:80/
```

Isso simula o comportamento de um cliente acessando o domínio `nginx.local`, permitindo que o Traefik encontre a regra correta do Ingress e encaminhe o tráfego para o service `nginx`.

---

## Troubleshooting encontrado neste lab

Durante a validação, o roteamento não funcionou inicialmente.

### Causa

As NetworkPolicies aplicadas no Lab 14 estavam bloqueando o acesso ao Traefik.

### Diagnóstico

Mesmo com:

* Ingress correto
* backend correto
* ingress class correta

o acesso ao pod/service do Traefik falhava por timeout.

### Solução

Remover temporariamente as policies:

```bash
kubectl delete networkpolicy --all
```

Após isso, o roteamento passou a funcionar normalmente.

---

## Troubleshooting adicional

### Ingress não responde

Verificar:

```bash
kubectl describe ingress nginx-ingress
kubectl logs deploy/traefik
```

---

### Backend sem resposta

Verificar:

```bash
kubectl get svc nginx
kubectl get endpoints nginx
```

---

### Service do Traefik existe, mas requisição falha

Verificar se há NetworkPolicies ativas:

```bash
kubectl get networkpolicy
```

---

## Insights

* Ingress depende do controller para funcionar
* Traefik implementa as regras definidas no recurso Ingress
* O header `Host` é fundamental para roteamento baseado em domínio
* Policies de rede podem bloquear o Ingress Controller e quebrar o fluxo externo
* O troubleshooting entre camadas é parte essencial de ambientes reais

---

## Conclusão

O lab demonstrou o funcionamento do Ingress com Traefik em Kubernetes, incluindo um cenário real de troubleshooting envolvendo NetworkPolicy.

Esse padrão é amplamente utilizado em produção para expor aplicações HTTP/HTTPS com roteamento centralizado e flexível.

---
