# 🖼️ Image Resizer - Versión de pruebas (Sergio)

## Placeholders a reemplazar

Antes de usar, busca y reemplaza estos valores en TODO el proyecto:

| Placeholder | Reemplazar con | Archivos afectados |
|-------------|---------------|-------------------|
| `__GITHUB_USER__` | Tu usuario de GitHub (minúsculas) | `k8s/*.yaml`, `helm/*/values.yaml`, `.github/workflows/ci.yml` |
| `__OPENSHIFT_NAMESPACE__` | Tu namespace de OpenShift Sandbox | `k8s/configmap.yaml`, `helm/*/values.yaml` |
| `__OPENSHIFT_DOMAIN__` | Dominio de tu Sandbox (ej: `apps.sandbox-m2.ll9k.p1.openshiftapps.com`) | `k8s/configmap.yaml`, `helm/*/values.yaml` |

### Reemplazo rápido (desde la raíz del proyecto):

```bash
# Linux/Mac
find . -type f \( -name "*.yaml" -o -name "*.yml" \) -exec sed -i '' \
  -e 's/__GITHUB_USER__/tu-usuario/g' \
  -e 's/__OPENSHIFT_NAMESPACE__/tu-namespace/g' \
  -e 's/__OPENSHIFT_DOMAIN__/apps.sandbox-m2.ll9k.p1.openshiftapps.com/g' {} +

# Git Bash en Windows
find . -type f \( -name "*.yaml" -o -name "*.yml" \) -exec sed -i \
  -e 's/__GITHUB_USER__/tu-usuario/g' \
  -e 's/__OPENSHIFT_NAMESPACE__/tu-namespace/g' \
  -e 's/__OPENSHIFT_DOMAIN__/apps.sandbox-m2.ll9k.p1.openshiftapps.com/g' {} +
```

### Secrets en GitHub

En tu repo → Settings → Secrets → Actions:

| Secret | Valor |
|--------|-------|
| `OPENSHIFT_SERVER` | URL de tu servidor OpenShift |
| `OPENSHIFT_TOKEN` | Tu token de login |

## Ejecución local

```bash
# Instalar dependencias
pip install -r requirements.txt

# Tests (no necesita Redis ni nada externo)
pytest tests/ -v

# Servidor local (solo API síncrona no funciona, necesita Celery)
# Para test local completo necesitas Redis:
#   docker run -d -p 6379:6379 --name redis redis:7-alpine
# Terminal 1: python -m app.app
# Terminal 2: celery -A app.celery_app:celery_app worker --loglevel=info --pool=solo
# Terminal 3: curl -X POST http://localhost:5000/resize -F "image=@test.jpg" -F "width=200" -F "height=200"
```

## Deploy a OpenShift

```bash
# Opción 1: Manifiestos directos
oc apply -f k8s/

# Opción 2: Helm
helm upgrade --install image-resizer helm/image-resizer/ \
  -f helm/image-resizer/values-dev.yaml \
  --wait --timeout 300s

# Verificar
oc get pods
oc get routes
```
