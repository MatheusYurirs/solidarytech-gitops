# Checklist — repo `solidarytech-gitops` (só manifestos)

Este repo contém **apenas** YAML e Helm values. O ArgoCD observa este repo e sincroniza o cluster.
O código, o Terraform e os pipelines estão no repo `solidarytech`.

Estrutura:
```
apps/            manifestos de cada serviço (donation, ngo, volunteer)
argocd/          Applications do ArgoCD
platform/
  observability/ Prometheus, Loki, Grafana (Helm values e dashboards)
  otel-collector/ OTel Collector
  alerting/      regras de alerta
  velero/        backup (opcional, DR opção A)
```

## ✅ Feito
- [x] Esqueleto de pastas criado
- [ ] (nada mais ainda: tudo abaixo está pendente)

## 🔲 Falta

**Etapa 3 — Deploy via GitOps (PDF p.3)**
- [ ] `apps/<serviço>/`: Deployment, Service, ConfigMap e Secret (via script, sem senha no Git)
  - **Sempre com `requests` e `limits`**
  - Probes em `/health`, porta 8081 (ngo), 8082 (donation), 8083 (volunteer)
- [ ] Ingress com as rotas dos 3 serviços (Nginx Ingress)
- [ ] HPA ao menos para o `donation-service` (hot path)
- [ ] `argocd/`: Application (ou app-of-apps) com sync automático para os 3 serviços
- [ ] Instalar o ArgoCD no cluster e ver os 3 serviços "Synced / Healthy"
- [ ] Validar que o CI do repo `solidarytech` consegue atualizar a tag da imagem nos Deployments (formato fácil de alterar via `sed`/`yq`)

**Etapa 4 — Observabilidade (PDF p.3–4)**
- [ ] `platform/observability/`: Helm values de Prometheus, Loki e Grafana (base: `entregues-fase-4/togglemaster-gitops/observability`)
- [ ] `platform/otel-collector/`: Collector recebendo OTLP e roteando métricas para o Prometheus, logs para o Loki e traces para o New Relic
- [ ] Applications do ArgoCD para cada componente acima
- [ ] Dashboard geral no Grafana (recursos do cluster, requisições, logs)

**Etapa 5 — SRE (PDF p.4)**
- [ ] Dashboard Grafana só de SLO e Error Budget do `donation-service` (como código, em JSON/ConfigMap)
- [ ] `platform/alerting/`: alertas de taxa de erro e latência ligados aos SLOs

**Etapa 6 — FinOps (PDF p.4)**
- [ ] Rightsizing: olhar o consumo real de CPU e memória no Prometheus e ajustar `requests` e `limits` de cada serviço
- [ ] Registrar o antes e depois (print) para o relatório

**Etapa 8 — DR (PDF p.5), opcional**
- [ ] `platform/velero/`: Velero com backup agendado para bucket S3 (a opção B, standby no Terraform, já cobre o requisito)
- [ ] O standby deve usar este mesmo repo: apontar o ArgoCD do cluster de us-west-2 para ele

## Regras
1. Nada de `kubectl apply` manual: só mudanças via commit neste repo.
2. Nunca commitar secrets, senhas ou tokens.
3. Todo pod precisa de `requests` e `limits`.
4. Imagem sempre com tag de commit (nunca `latest`).
