# Lab 08 - Helm

## Objetivo

Demonstrar o uso do Helm para empacotar e gerenciar aplicações no Kubernetes.

---

## Conceitos chave

* Helm → gerenciador de pacotes
* Chart → pacote de aplicação
* values.yaml → configuração
* Templates → manifests dinâmicos

---

## Etapas

### Criar chart

```bash
helm create nginx-chart
```

---

### Instalar

```bash
helm install nginx-release ./nginx-chart
```

---

### Validar

```bash
kubectl get pods
```

---

### Alterar configuração

Editar `values.yaml`:

```yaml
replicaCount: 5
```

---

### Aplicar upgrade

```bash
helm upgrade nginx-release ./nginx-chart
```

---

## Resultado esperado

* Deploy dinâmico
* Configuração reutilizável
* Facilidade de upgrade

---

## Insights

* Helm padroniza deploy
* Evita YAML duplicado
* Facilita gestão em escala

---

## Conclusão

Helm é amplamente utilizado em ambientes produtivos para gerenciamento e padronização de aplicações Kubernetes.


## EXTRA

# Helm vs kubectl apply

## Diferença real

### kubectl apply

Aplica arquivos YAML diretamente no cluster.

* Simples e direto
* Usa manifests estáticos
* Sem parametrização nativa
* Difícil de reutilizar em larga escala

---

### Helm

Gerencia aplicações como pacotes (charts).

* Usa templates dinâmicos
* Permite reutilização
* Possui versionamento
* Facilita upgrades e rollbacks
* Padroniza deploys

---

## Explicação objetiva

O `kubectl apply` aplica manifests estáticos diretamente no cluster.

O Helm adiciona uma camada de abstração, permitindo parametrização, reutilização e versionamento dos manifests, facilitando o gerenciamento de aplicações em ambientes complexos e produtivos.

---

## Insight importante

Helm não substitui o kubectl.

Ele utiliza o Kubernetes por baixo (API), funcionando como uma camada superior de gerenciamento.

---

## Quando usar cada um

| Cenário                   | Ferramenta |
| ------------------------- | ---------- |
| Teste rápido              | kubectl    |
| Deploy simples            | kubectl    |
| Produção escalável        | Helm       |
| Padronização de ambientes | Helm       |

---

## Conclusão

O kubectl é essencial para operações diretas e simples.

O Helm é fundamental para ambientes produtivos onde há necessidade de organização, padronização e escalabilidade.
