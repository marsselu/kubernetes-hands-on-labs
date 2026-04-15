# Lab 19 - StatefulSet (identidade e persistência por instância)

## Objetivo

Validar o comportamento de um StatefulSet no Kubernetes, comprovando que cada Pod possui identidade estável, nome previsível e volume persistente próprio.

---

## Pré-requisitos

* Cluster Kubernetes (kind)
* StorageClass funcional
* Provisionamento dinâmico disponível

Verifique:

```bash
kubectl get storageclass
```

---

## Conceitos chave

* **StatefulSet** → controlador para aplicações stateful
* **Identidade estável** → cada Pod mantém nome fixo
* **Headless Service** → permite descoberta individual dos Pods
* **volumeClaimTemplates** → cria um PVC por réplica
* **Persistência por instância** → cada Pod mantém seu próprio volume

---

## Arquitetura

```text
StatefulSet
   ├── nginx-stateful-0 → PVC próprio → volume próprio
   └── nginx-stateful-1 → PVC próprio → volume próprio
```

---

## Etapa 1 — Criar Service headless

Arquivo `nginx-headless.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx-stateful
  ports:
  - port: 80
    targetPort: 80
```

Aplicar:

```bash
kubectl apply -f nginx-headless.yaml
```

---

## Etapa 2 — Criar StatefulSet

Arquivo `nginx-stateful.yaml`:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-stateful
spec:
  serviceName: nginx-headless
  replicas: 2
  selector:
    matchLabels:
      app: nginx-stateful
  template:
    metadata:
      labels:
        app: nginx-stateful
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: storage
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: storage
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
```

Aplicar:

```bash
kubectl apply -f nginx-stateful.yaml
kubectl get pods
```

Resultado esperado:

* `nginx-stateful-0`
* `nginx-stateful-1`

---

## Etapa 3 — Validar identidade dos Pods

```bash
kubectl get pods -l app=nginx-stateful -o wide
kubectl get pvc
```

Resultado esperado:

* nomes fixos e previsíveis
* PVCs criados automaticamente para cada réplica

---

## Etapa 4 — Criar conteúdo diferente em cada Pod

No Pod 0:

```bash
kubectl exec -it nginx-stateful-0 -- sh
```

Dentro:

```sh
echo "POD 0" > /usr/share/nginx/html/index.html
exit
```

No Pod 1:

```bash
kubectl exec -it nginx-stateful-1 -- sh
```

Dentro:

```sh
echo "POD 1" > /usr/share/nginx/html/index.html
exit
```

---

## Etapa 5 — Validar cada Pod separadamente

Pod 0:

```bash
kubectl port-forward pod/nginx-stateful-0 8080:80
```

Acessar:

```text
http://localhost:8080
```

Resultado esperado:

```text
POD 0
```

Pod 1:

```bash
kubectl port-forward pod/nginx-stateful-1 8081:80
```

Acessar:

```text
http://localhost:8081
```

Resultado esperado:

```text
POD 1
```

---

## Etapa 6 — Simular falha

Deletar o Pod 0:

```bash
kubectl delete pod nginx-stateful-0
kubectl get pods
```

Resultado esperado:

* o Pod volta com o mesmo nome: `nginx-stateful-0`

---

## Etapa 7 — Validar persistência após recriação

Fazer novamente o port-forward:

```bash
kubectl port-forward pod/nginx-stateful-0 8080:80
```

Acessar:

```text
http://localhost:8080
```

Resultado esperado:

```text
POD 0
```

---

## O que aconteceu

1. O StatefulSet criou Pods com identidade estável
2. Cada réplica recebeu seu próprio PVC automaticamente
3. Cada Pod gravou dados no seu volume exclusivo
4. Após deletar `nginx-stateful-0`, ele foi recriado com o mesmo nome
5. O conteúdo foi preservado porque o volume permaneceu associado àquela instância

---

## Diferença entre Deployment e StatefulSet

| Recurso            | Deployment     | StatefulSet                        |
| ------------------ | -------------- | ---------------------------------- |
| Identidade do Pod  | efêmera        | estável                            |
| Nome do Pod        | aleatório      | previsível                         |
| Volume por réplica | não nativo     | nativo via `volumeClaimTemplates`  |
| Uso comum          | apps stateless | bancos, filas, aplicações stateful |

---

## Troubleshooting

### Pods não sobem

```bash
kubectl describe pod nginx-stateful-0
kubectl describe pod nginx-stateful-1
```

---

### PVCs não são criados ou não ficam `Bound`

```bash
kubectl get pvc
kubectl describe pvc
kubectl get storageclass
```

---

### Conteúdo não persiste

Verificar se cada Pod está usando seu PVC correto:

```bash
kubectl get pod nginx-stateful-0 -o yaml
kubectl get pod nginx-stateful-1 -o yaml
```

---

## Insights

* StatefulSet é a escolha correta para aplicações stateful
* Cada réplica possui identidade própria e persistente
* O headless service é parte importante da descoberta e identidade dos Pods
* Esse padrão é muito usado em bancos de dados e sistemas distribuídos

---

## Conclusão

O lab demonstrou que StatefulSets fornecem identidade estável e persistência por instância, dois requisitos fundamentais para aplicações stateful em Kubernetes.

Esse comportamento é essencial em ambientes produtivos que utilizam bancos de dados, filas e serviços que dependem de nome fixo e armazenamento durável.

---
