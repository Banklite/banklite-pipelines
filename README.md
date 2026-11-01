# banklite-pipelines

Workflows reutilizables de GitHub Actions para todos los repositorios de BankLite (organizacion **Banklite**).
Un repositorio por microservicio, un pipeline por repositorio, un solo lugar donde esta definido el proceso.

| Workflow | Lo usan | Pasos |
|---|---|---|
| `service-pipeline.yml` | los 8 microservicios y `banklite-api-gateway` (Python) | `test` -> `docker` -> `deploy` |
| `panel-pipeline.yml` | `banklite-web-panel` (Next.js) | `test` -> `docker` -> `deploy` |

## Como funciona

1. **test**: instala dependencias, compila, ejecuta `pytest` y comprueba que la aplicacion importa. Si falla, los jobs siguientes no corren (`needs`).
2. **docker**: construye la imagen. Solo en `push` a `main` la publica en GitHub Container Registry (`ghcr.io/banklite/<servicio>`), con las etiquetas `latest` y el SHA del commit.
3. **deploy**: solo en `push` a `main`, llama al **Deploy Hook** de Render del servicio. Si el secret no existe, avisa y no despliega.

En pull requests solo corren `test` y la construccion de la imagen (sin publicarla ni desplegar).

## Como lo invoca cada repositorio

`.github/workflows/deploy.yml` del microservicio:

```yaml
name: CI-CD identity-service
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }
  workflow_dispatch:
permissions:
  contents: read
  packages: write
jobs:
  pipeline:
    uses: Banklite/banklite-pipelines/.github/workflows/service-pipeline.yml@main
    with:
      service-name: banklite-identity-service
    secrets: inherit
```

## Secrets (por repositorio: Settings > Secrets and variables > Actions)

| Secret | Valor | Donde se obtiene |
|---|---|---|
| `RENDER_DEPLOY_HOOK_URL` | URL secreta del servicio | Render > servicio > Settings > Deploy Hook |

La URL del Deploy Hook es una credencial: nunca va en el codigo ni en el README. El push de imagenes usa el `GITHUB_TOKEN` que GitHub crea en cada ejecucion; no hace falta guardar ninguna clave del registro.
