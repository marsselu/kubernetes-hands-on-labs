# Lab 04 - Secrets (dados sensíveis)

## Objetivo

Demonstrar como armazenar e consumir dados sensíveis no Kubernetes utilizando Secrets, além de entender suas limitações de segurança.

---

## Pré-requisitos

* Cluster Kubernetes rodando (kind)
* Deployment nginx existente

Verifique:

```bash id="4y6l2k"
kubectl get pods -l app=nginx
```

---

## Conceitos chave

* Secret → armazenamento de dados sensíveis
* Base64 → encoding, não criptografia
* Env vs Volume → formas de consumo
* Segurança depende de RBAC e configuração do cluster

---

## Etapa 1 — Criar Secret

```bash id="hxyg4g"
kubectl create secret generic app-secret --from-literal=senha=123456
```

---

## Etapa 2 — Validar Secret

```bash id="6r91gr"
kubectl describe secret app-secret
```

Observação:

* O valor NÃO é exibido diretamente

---

## Etapa 3 — Ver valor real (base64)

```bash id="mw3g6y"
kubectl get secret app-secret -o yaml
```

Exemplo:

```yaml id="p1x0xw"
data:
  senha: MTIzNDU2
```

---

## Etapa 4 — Decodificar

```bash id="n7q6f2"
echo MTIzNDU2 | base64 -d
```

Resultado:

```text id="8a2x8k"
123456
```

---

## Etapa 5 — Usar Secret como variável de ambiente

Editar Deployment:

```bash id="p8t2rj"
kubectl edit deployment nginx
```

Adicionar no container:

```yaml id="zq0m1a"
env:
  - name: SENHA
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: senha
```

---

## Etapa 6 — Validar no Pod

```bash id="3c9n5s"
kubectl exec -it <pod> -- printenv | grep SENHA
```

Esperado:

```text id="2r4h8y"
SENHA=123456
```

---

## ⚠️ Limitação importante

Dentro do container:

* O valor fica em texto plano
* Pode ser acessado via `printenv`

---

## Etapa 7 — Usar Secret como volume (opcional)

Adicionar no Deployment:

### Dentro do container:

```yaml id="k9t7e1"
volumeMounts:
  - name: secret-volume
    mountPath: /etc/secret
```

### Fora do container:

```yaml id="c2m8p0"
volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```

---

## Validar

```bash id="z7w4y3"
kubectl exec -it <pod> -- cat /etc/secret/senha
```

---

## O que aconteceu

```text id="m1k3v9"
Secret → injetado → Pod → env ou arquivo
```

---

## Segurança (ponto crítico)

Secrets NÃO são seguros por padrão:

* Apenas base64
* Qualquer acesso ao cluster pode ler

---

## Boas práticas (produção)

* Usar RBAC (controle de acesso)
* Ativar encryption at rest (etcd)
* Usar ferramentas externas:

  * HashiCorp Vault
  * AWS Secrets Manager
  * External Secrets Operator

---

## Troubleshooting

### Secret não encontrado

```bash id="8u2d7s"
kubectl get secrets
```

---

### Variável não aparece

* Verificar nome da key
* Verificar restart do Pod

---

## Insights

* Secrets devem ser protegidos com camadas adicionais
* Evitar expor secrets via env em aplicações críticas
* Volume pode ser mais seguro dependendo do cenário

---

## Conclusão

Secrets são fundamentais para gerenciamento de dados sensíveis, mas precisam ser combinados com boas práticas de segurança para uso em produção.
