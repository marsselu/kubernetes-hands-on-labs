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
