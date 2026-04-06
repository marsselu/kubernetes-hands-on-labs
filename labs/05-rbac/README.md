# Lab 05 - RBAC (controle de acesso)

## Objetivo

Demonstrar como controlar permissões no Kubernetes utilizando RBAC (Role-Based Access Control), aplicando o princípio do menor privilégio.

---

## Pré-requisitos

* Cluster Kubernetes rodando
* kubectl configurado

---

## Conceitos chave

* ServiceAccount → identidade dentro do cluster
* Role → permissões em nível de namespace
* RoleBinding → associação entre identidade e permissões
* Least Privilege → acesso mínimo necessário

---

## Etapa 1 — Criar ServiceAccount

```bash id="j9m4qs"
kubectl create serviceaccount dev-user
```

---

## Etapa 2 — Validar

```bash id="z8v1dx"
kubectl get sa
```

---

## Etapa 3 — Criar Role

Arquivo `manifests/rbac-role.yaml`:

```yaml id="z0k8rt"
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

Aplicar:

```bash id="s4p6yt"
kubectl apply -f manifests/rbac-role.yaml
```

---

## Etapa 4 — Criar RoleBinding

```bash id="t5n2bx"
kubectl create rolebinding dev-binding \
  --role=pod-reader \
  --serviceaccount=default:dev-user
```

---

## Etapa 5 — Validar permissões

```bash id="y1u8pq"
kubectl auth can-i list pods --as=system:serviceaccount:default:dev-user
```

Esperado:

```text id="v7k3dn"
yes
```

---

## Testar ação proibida

```bash id="r6c2jf"
kubectl auth can-i delete pods --as=system:serviceaccount:default:dev-user
```

Esperado:

```text id="m9b4xz"
no
```

---

## O que aconteceu

```text id="k2n7wr"
ServiceAccount → identidade
Role → define permissões
RoleBinding → conecta os dois
```

---

## Diferença importante

| Tipo        | Escopo          |
| ----------- | --------------- |
| Role        | Namespace       |
| ClusterRole | Cluster inteiro |

---

## Troubleshooting

### Permissão não aplicada

* Verificar namespace
* Verificar nome da ServiceAccount
* Verificar RoleBinding

---

### Teste não funciona

```bash id="q3h8tz"
kubectl get rolebinding
```

---

## Segurança (produção)

* Nunca usar permissões amplas desnecessárias
* Evitar `*` em verbs/resources
* Separar acessos por times

---

## Insights

* RBAC é essencial para segurança do cluster
* Permissões devem ser sempre explícitas
* ServiceAccounts são usadas por aplicações

---

## Conclusão

RBAC permite controlar quem pode fazer o quê dentro do cluster, sendo fundamental para ambientes seguros e multi-tenant.
