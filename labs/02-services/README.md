# Lab 02 - Services, DNS e Comunicação entre Pods

## Objetivo

Demonstrar como expor aplicações no Kubernetes e permitir comunicação interna entre Pods utilizando Service e DNS.

---

## Pré-requisitos

* Cluster Kubernetes rodando (kind)
* Deployment nginx já criado (Lab 01)

Verifique:

```bash
kubectl get pods -l app=nginx
```

---

## Conceitos chave

* **Service (ClusterIP)** → ponto fixo de acesso
* **DNS interno** → resolve nome do serviço
* **Service Discovery** → comunicação via nome
* **Load Balancing** → distribuição entre Pods

---

## Arquitetura

```
Pod cliente
   ↓
DNS (nginx)
   ↓
Service (ClusterIP)
   ↓
Pods nginx
```

---

## Etapa 1 — Criar Service

```bash
kubectl apply -f manifests/nginx-service.yaml
```

---

## Etapa 2 — Validar Service

```bash
kubectl get svc
```

Esperado:

```
nginx   ClusterIP   10.x.x.x   <none>   80/TCP
```

---

## Etapa 3 — Ver endpoints

```bash
kubectl get endpoints nginx
```

Esperado:

* IPs dos Pods nginx

---

## Etapa 4 — Testar comunicação interna

Criar Pod de teste:

```bash
kubectl run teste --image=busybox --rm -it -- sh
```

Dentro do Pod:

```sh
wget -qO- nginx
```

---

## Resultado esperado

* Retorno HTML do nginx
* Comunicação feita usando apenas o nome `nginx`

---

## O que aconteceu

```
nginx (nome)
↓
DNS resolve
↓
Service recebe
↓
Distribui entre Pods
```

---

## Teste de Load Balancing

Execute múltiplas vezes:

```sh
wget -qO- nginx
```

→ O tráfego é distribuído entre os Pods

---

## Troubleshooting

### Service sem endpoints

```bash
kubectl get endpoints nginx
```

Se vazio:

* Verifique labels dos Pods

---

### DNS não resolve

```bash
kubectl get pods -n kube-system
```

Verifique:

* coredns está rodando

---

### Sem resposta

* Verifique Pods:

```bash
kubectl get pods
```

* Verifique logs:

```bash
kubectl logs <pod>
```

---

## Insights

* Pods são efêmeros, Services são estáveis
* DNS interno elimina necessidade de IP fixo
* Service é base da comunicação entre aplicações

---

## Conclusão

Esse padrão é utilizado em produção para permitir comunicação entre microsserviços de forma resiliente e desacoplada.
