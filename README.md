# worker-template

Template repository para **workers/consumers** sem entrada HTTP — apps que processam
filas (kafka/sqs/rabbitmq), eventos ou tarefas em background. Deploy via
[`platform-k8s-worker-chart`](https://github.com/pagamericantech/platform-charts/tree/main/platform-k8s-worker-chart).

Para outros tipos de workload, use o template correspondente:

- [`web-template`](https://github.com/pagamericantech/web-template) — apps HTTP (Deployment + HTTPRoute + Argo Rollouts canary).
- **`worker-template`** (este) — consumers/workers sem entrada HTTP.
- [`cronjob-template`](https://github.com/pagamericantech/cronjob-template) — jobs agendados.

## Estrutura

```
.
├── .github/workflows/build_and_deploy.yaml   # chama _reusable-docker-build-push.yml
├── chart/
│   └── values-k8s-shared-services.yaml       # override do platform-k8s-worker-chart
├── Dockerfile                                # build do app (placeholder)
├── .dockerignore
└── .gitignore
```

O fluxo é idêntico ao `web-template`: push em `main` → build/push pra ECR → bump
da `image.tag` no values → reconcile do ArgoCD → deploy via chart compartilhado.

## Como criar um novo worker a partir desse template

```bash
gh repo create pagamericantech/<worker-name> \
  --template pagamericantech/worker-template \
  --private \
  --clone

cd <worker-name>

APP=meu-worker
IMAGE=pga-meu-worker
OWNER=meu-time
LANG=go

find . -type f \( -name '*.yaml' -o -name '*.yml' \) -not -path './.git/*' -print0 | \
  xargs -0 sed -i '' \
    -e "s/__APP_NAME__/${APP}/g" \
    -e "s/__IMAGE_NAME__/${IMAGE}/g" \
    -e "s/__OWNER__/${OWNER}/g" \
    -e "s/__LANGUAGE__/${LANG}/g"
```

## Configuração específica de worker

Após substituir os placeholders, ajuste `chart/values-k8s-shared-services.yaml`:

- **Trigger de autoscaling** — KEDA já vem habilitado com cpu/memory. Workers
  geralmente escalam por **profundidade de fila**. O bloco `autoscaling.sqs` está
  comentado como exemplo; outros scalers (`kafka`, `rabbitmq`, `datadog`) estão
  documentados em
  [`platform-k8s-worker-chart/values.yaml`](https://github.com/pagamericantech/platform-charts/blob/main/platform-k8s-worker-chart/values.yaml).
- **`replicaCount`** — define o `minReplicaCount` do `ScaledObject`. Para workers
  caros, manter `1`; para workers com pico de carga súbita, considerar `2+` pra
  reduzir cold-start.

## Marcar como template no GitHub

```bash
gh repo edit pagamericantech/worker-template --template
```
