# Lab 14 - NetworkPolicy (Segurança de rede)

## Objetivo

Controlar e validar a comunicação entre Pods no Kubernetes utilizando NetworkPolicies, aplicando o conceito de **Zero Trust** (negação por padrão e liberação explícita).

---

## Pré-requisitos

* Cluster Kubernetes (kind)
* CNI com suporte a NetworkPolicy
* Calico instalado
* Deployment nginx ativo
* Service nginx funcionando

Verifique:

```bash
kubectl get pods -n kube-system | grep calico
kubectl get pods -l app=nginx
kubectl get svc nginx
```

---

## Conceitos chave

* **NetworkPolicy** → controla tráfego entre Pods
* **Ingress** → tráfego de entrada
* **Egress** → tráfego de saída
* **Default deny** → negar tudo por padrão
* **Allow list** → liberar explicitamente
* **Zero Trust** → nenhum acesso é confiável por padrão

---

## Arquitetura

```text
Pod cliente → Service nginx → Pods nginx
          (controlado por NetworkPolicy)
```

---

## Etapa 1 — Garantir suporte a NetworkPolicy

No kind, é necessário instalar um CNI compatível (Calico):

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml
```

Validar:

```bash
kubectl get pods -n kube-system | grep calico
```

Resultado esperado:

* `calico-node` → Running
* `calico-kube-controllers` → Running

---

## Etapa 2 — Validar comunicação sem policy

Criar Pod de teste:

```bash
kubectl run teste --image=busybox --rm -it -- sh
```

Dentro do Pod:

```sh
wget -qO- nginx
```

Resultado esperado:

✔ HTML do nginx retornado

---

## Etapa 3 — Aplicar bloqueio total (default deny)

Arquivo `deny-all.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

Aplicar:

```bash
kubectl apply -f deny-all.yaml
kubectl get networkpolicy
```

---

## Etapa 4 — Validar bloqueio

```bash
kubectl run teste --image=busybox --rm -it -- sh
```

Dentro:

```sh
wget -qO- nginx
```

Resultado esperado:

❌ Falha de comunicação

---

## Etapa 5 — Liberar acesso ao nginx

Arquivo `allow-nginx.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector: {}
  policyTypes:
  - Ingress
```

Aplicar:

```bash
kubectl apply -f allow-nginx.yaml
```

---

## Etapa 6 — Validar liberação

```bash
kubectl run teste --image=busybox --rm -it -- sh
```

Dentro:

```sh
wget -qO- nginx
```

Resultado esperado:

✔ HTML do nginx volta a ser exibido

---

## O que aconteceu

1. Sem NetworkPolicy → comunicação totalmente aberta
2. Com `deny-all` → todo tráfego de entrada foi bloqueado
3. Com `allow-nginx` → acesso aos Pods nginx foi liberado explicitamente

---

## Troubleshooting

### Policy não bloqueia nada

Verifique se há CNI compatível:

```bash
kubectl get pods -n kube-system
```

Sem Calico, Cilium ou similar, NetworkPolicy não funciona.

---

### Policy aplicada no namespace errado

```bash
kubectl get networkpolicy -A
```

---

### Nada comunica

```bash
kubectl delete networkpolicy --all
```

---

### Labels não batem

```bash
kubectl get pods --show-labels
```

---

## Insights

* Kind não suporta NetworkPolicy nativamente sem CNI
* Calico é necessário para ativar controle de rede
* NetworkPolicy implementa modelo **Zero Trust**
* Segurança de rede depende do plugin de rede (CNI)
* Loki/Prometheus não substituem controle de tráfego (camadas diferentes)

---

## Conclusão

A validação prática demonstrou como NetworkPolicies permitem controlar explicitamente o tráfego dentro do cluster Kubernetes.

Esse mecanismo é essencial para segurança em ambientes produtivos, permitindo isolamento de serviços e redução de superfície de ataque.

---

