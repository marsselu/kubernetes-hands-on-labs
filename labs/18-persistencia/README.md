# Lab 18 - Persistência com PVC e StorageClass

## Objetivo

Validar persistência de dados no Kubernetes utilizando PersistentVolumeClaim (PVC), comprovando que o armazenamento sobrevive à recriação do Pod.

---

## Pré-requisitos

* Cluster Kubernetes (kind)
* StorageClass disponível no cluster

Verifique:

```bash
kubectl get storageclass
```

---

## Conceitos chave

* **PersistentVolume (PV)** → volume persistente no cluster
* **PersistentVolumeClaim (PVC)** → solicitação de armazenamento feita pelo usuário
* **StorageClass** → define como o volume será provisionado
* **Provisionamento dinâmico** → criação automática de PV a partir de um PVC
* **Persistência** → os dados sobrevivem à recriação do Pod

---

## Arquitetura

```text
Pod → PVC → StorageClass → PV (dinâmico)
```

---

## Etapa 1 — Criar o PVC

Arquivo `pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Aplicar:

```bash
kubectl apply -f pvc.yaml
kubectl get pvc
```

### Resultado esperado

* Em alguns ambientes, o PVC pode inicialmente ficar em `Pending`
* Após a criação do Pod consumidor, o PVC deve mudar para `Bound`

---

## Etapa 2 — Criar Pod usando o PVC

Arquivo `nginx-pvc.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pvc
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: nginx-pvc
```

Aplicar:

```bash
kubectl apply -f nginx-pvc.yaml
kubectl get pods
kubectl get pvc
```

Resultado esperado:

* Pod `nginx-pvc` em `Running`
* PVC em `Bound`

---

## Etapa 3 — Criar conteúdo no volume

Entrar no Pod:

```bash
kubectl exec -it nginx-pvc -- sh
```

Dentro do container:

```sh
echo "FUNCIONOU PVC" > /usr/share/nginx/html/index.html
exit
```

---

## Etapa 4 — Validar conteúdo

Fazer port-forward:

```bash
kubectl port-forward pod/nginx-pvc 8080:80
```

Acessar:

```text
http://localhost:8080
```

Resultado esperado:

```text
FUNCIONOU PVC
```

---

## Etapa 5 — Simular falha / recriação do Pod

Deletar o Pod:

```bash
kubectl delete pod nginx-pvc
```

Recriar:

```bash
kubectl apply -f nginx-pvc.yaml
kubectl get pods
```

---

## Etapa 6 — Validar persistência

```bash
kubectl port-forward pod/nginx-pvc 8080:80
```

Acessar:

```text
http://localhost:8080
```

### Resultado esperado

* O conteúdo `FUNCIONOU PVC` continua disponível

---

## O que aconteceu

1. O PVC solicitou armazenamento ao cluster
2. O StorageClass provisionou um PV automaticamente (provisionamento dinâmico)
3. O Pod utilizou o volume via PVC
4. Mesmo após deletar o Pod, o volume persistiu
5. Um novo Pod reutilizou o mesmo volume com os dados intactos

---

## Troubleshooting

### PVC permanece em `Pending`

Verificar StorageClass:

```bash
kubectl get storageclass
```

Verificar modo de binding:

```bash
kubectl describe storageclass
```

Observação:

* Em StorageClasses com `WaitForFirstConsumer`, o PVC só será associado após a criação de um Pod

---

### Pod não sobe

```bash
kubectl describe pod nginx-pvc
kubectl describe pvc nginx-pvc
```

---

### Conteúdo não persiste

Verificar se o Pod está usando o PVC correto:

```bash
kubectl get pod nginx-pvc -o yaml
```

---

## Insights

* PVC desacopla armazenamento da vida do Pod
* PV pode ser criado automaticamente (provisionamento dinâmico)
* Containers não devem armazenar dados críticos localmente
* Persistência é essencial para aplicações stateful (bancos, uploads, etc.)
* `WaitForFirstConsumer` é comum em ambientes modernos e impacta o estado inicial do PVC

---

## Conclusão

O lab demonstrou como Kubernetes garante persistência de dados utilizando PVC e StorageClass, mesmo após a recriação de Pods.

Esse padrão é essencial em ambientes produtivos, permitindo resiliência e integridade de dados em aplicações distribuídas.

---
