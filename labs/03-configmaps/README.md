# Lab 03 - ConfigMaps (env e volume)

## Objetivo

Demonstrar como externalizar configurações no Kubernetes usando ConfigMaps, tanto via variável de ambiente quanto via volume com atualização dinâmica.

---

## Pré-requisitos

* Cluster Kubernetes rodando (kind)
* Deployment nginx já existente

Verifique:

```bash
kubectl get pods -l app=nginx
```

---

## Conceitos chave

* ConfigMap → configuração externa ao container
* Env → configuração estática (não atualiza)
* Volume → configuração dinâmica (atualiza sem restart)
* Separação entre aplicação e configuração

---

## Etapa 1 — Criar ConfigMap

```bash
kubectl create configmap nginx-config --from-literal=mensagem="Hello Kubernetes"
```

---

## Etapa 2 — Validar ConfigMap

```bash
kubectl describe configmap nginx-config
```

---

## Etapa 3 — Usar como variável de ambiente

Editar Deployment:

```bash
kubectl edit deployment nginx
```

Adicionar no container:

```yaml
env:
  - name: MENSAGEM
    valueFrom:
      configMapKeyRef:
        name: nginx-config
        key: mensagem
```

---

## Etapa 4 — Validar no Pod

```bash
kubectl exec -it <pod> -- printenv | grep MENSAGEM
```

Esperado:

```text
MENSAGEM=Hello Kubernetes
```

---

## Observação importante

Alterar o ConfigMap NÃO atualiza automaticamente o valor em env.

---

## Etapa 5 — Usar como volume (config dinâmica)

Editar Deployment novamente:

### Dentro do container:

```yaml
volumeMounts:
  - name: config-volume
    mountPath: /etc/config
```

### Fora do container:

```yaml
volumes:
  - name: config-volume
    configMap:
      name: nginx-config
```

---

## Etapa 6 — Validar arquivo

```bash
kubectl exec -it <pod> -- cat /etc/config/mensagem
```

Esperado:

```text
Hello Kubernetes
```

---

## Etapa 7 — Testar atualização dinâmica

Editar ConfigMap:

```bash
kubectl edit configmap nginx-config
```

Alterar para:

```text
Hello DevOps
```

---

## Validar novamente (sem restart)

```bash
kubectl exec -it <pod> -- cat /etc/config/mensagem
```

Esperado:

```text
Hello DevOps
```

---

## O que aconteceu

```text
ConfigMap atualizado
↓
kubelet sincroniza
↓
arquivo atualizado no Pod
```

---

## Diferença crítica

| Método | Atualiza automaticamente |
| ------ | ------------------------ |
| Env    | ❌ Não                    |
| Volume | ✅ Sim                    |

---

## Troubleshooting

### Arquivo não aparece

* Verifique volumeMounts
* Verifique volumes
* Verifique nome do ConfigMap

---

### Valor não atualiza

* Aguarde alguns segundos (sync do kubelet)
* Confirme alteração no ConfigMap

---

## Insights

* ConfigMap desacopla config da imagem
* Volume permite atualização dinâmica
* Padrão usado em produção para apps configuráveis

---

## Conclusão

ConfigMaps são essenciais para manter aplicações flexíveis e adaptáveis sem necessidade de rebuild ou redeploy.
