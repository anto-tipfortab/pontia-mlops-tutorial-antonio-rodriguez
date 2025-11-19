# Pipeline MLOps con GitHub Actions

Pipeline CI/CD completo para entrenar, validar y desplegar un modelo de Machine Learning que predice niveles de ingresos usando el dataset UCI Adult.

## Arquitectura

```
┌─────────────────┐      ┌──────────────────┐      ┌─────────────────┐
│  Integration    │      │   Build Model    │      │  Deploy Model   │
│  (Tests en PR)  │──>───│  (Entrenamiento) │──>───│  (Producción)   │
└─────────────────┘      └──────────────────┘      └─────────────────┘
```

---

## Workflows

### 1. 🔄 Integration (`Integration.yml`)

**Se activa en:** Pull Requests

**Qué hace:**
- Configura entorno Python 3.10
- Instala dependencias
- Ejecuta tests unitarios con pytest
- Genera reportes de cobertura
- Publica resultados automáticamente como comentario en el PR

**Conceptos DevOps:** Integración Continua (CI), testing automatizado, validación de código antes de merge.

---

### 2. 🏗️ Build Model (`Build Model.yml`)

**Se activa en:** Push a main, ejecución manual

**Qué hace:**
- Descarga dataset UCI Adult
- Entrena el modelo con `src/main.py`
- Registra experimentos en MLflow
- Ejecuta tests de validación del modelo
- Registra modelo en MLflow Model Registry

**Variables de entorno:**
- `AZURE_STORAGE_CONNECTION_STRING`: Conexión a Azure Blob Storage
- `EXPERIMENT_NAME`: Nombre del experimento en MLflow
- `MLFLOW_URL`: URL del servidor MLflow
- `MODEL_NAME`: Nombre para registro del modelo

**Conceptos DevOps:** Entrenamiento continuo, tracking de experimentos, versionado de modelos, almacenamiento de artefactos.

---

### 3. 🚀 Deploy Model (`Deploy Model.yml`)

**Se activa en:** Ejecución manual

**Qué hace:**
- Autenticación en Azure (ACR y ACI)
- Construye imagen Docker desde `./deployment`
- Sube imagen a Azure Container Registry
- Despliega container en Azure Container Instances
- Configura DNS público y expone API en puerto 8080
- Verifica deployment con health check

**Configuración:**
- Recursos: 0.5 CPU, 1GB RAM
- Región: East US
- Endpoint: `http://{image-name}-{run-id}.eastus.azurecontainer.io:8080`

**Conceptos DevOps:** Despliegue continuo (CD), containerización con Docker, Infrastructure as Code, monitoreo de servicios.

---

## Stack Tecnológico

- **CI/CD:** GitHub Actions
- **ML:** Python 3.10, scikit-learn, pandas
- **Tracking:** MLflow
- **Cloud:** Azure (ACR, ACI, Blob Storage)
- **Testing:** pytest con reportes de cobertura
- **Containers:** Docker

---

## Secrets Necesarios

Configurar en Settings > Secrets del repositorio:

| Secret | Descripción |
|--------|-------------|
| `AZURE_STORAGE_CONNECTION_STRING` | Conexión Azure Blob Storage |
| `AZURE_CREDENTIALS` | Credenciales service principal |
| `ACR_USERNAME` | Usuario Container Registry |
| `ACR_PASSWORD` | Password Container Registry |
| `ACR_NAME` | Nombre del registry |
| `AZURE_RESOURCE_GROUP` | Grupo de recursos Azure |

## Variables Necesarias

| Variable | Descripción |
|----------|-------------|
| `EXPERIMENT_NAME` | Nombre experimento MLflow |
| `MLFLOW_URL` | URL servidor MLflow |
| `MODEL_NAME` | Identificador del modelo |
| `MODEL_ALIAS` | Alias versión (ej: "champion") |
| `IMAGE_NAME` | Nombre imagen Docker |

---

## Estructura del Proyecto

```
.
├── .github/workflows/
│   ├── Integration.yml       # Tests en PRs
│   ├── Build Model.yml       # Entrenamiento
│   └── Deploy Model.yml      # Despliegue
├── src/main.py               # Script entrenamiento
├── model_tests/              # Tests validación modelo
├── unit_tests/               # Tests unitarios
├── scripts/                  # Scripts auxiliares
├── deployment/Dockerfile     # Definición container
└── requirements.txt          # Dependencias Python
```

---

## Flujo de Trabajo

1. Crear Pull Request → Tests automáticos
2. Merge a main → Entrenamiento del modelo
3. Trigger manual → Despliegue a producción

---

## Uso de la API

```bash
# Health check
curl http://{dns-name}.eastus.azurecontainer.io:8080/health

# Predicciones
curl -X POST http://{dns-name}.eastus.azurecontainer.io:8080/predict \
  -H "Content-Type: application/json" \
  -d '{"data": [[...]]}'
```

---
