# Pipeline CI/CD con Jenkins y SonarQube

**Autor**: LUIS MANUEL ROJAS CORREA
**Código**: A00399289
**Proyecto**: DevOps Multi-Repositorio

## Descripción

Este repositorio contiene la configuración completa del pipeline CI/CD que se ejecuta automáticamente cuando se realizan cambios en el repositorio [Teclado](https://github.com/Lrojas898/Teclado). El pipeline incluye análisis de calidad con SonarQube, testing automatizado y simulación de despliegue.

## Arquitectura Multi-Repositorio

### Repositorios del Proyecto
1. **[Teclado](https://github.com/Lrojas898/Teclado)**: Código fuente de la aplicación web
2. **[ansible-pipeline](https://github.com/Lrojas898/ansible-pipeline)** (este repo): Configuración del pipeline CI/CD
3. **[terraform_for_each_vm](https://github.com/Lrojas898/terraform_for_each_vm)**: Infraestructura como código

### Flujo de Integración
```
┌─────────────────┐    webhook    ┌─────────────────┐
│  Push to Main   │ ──────────── ▶│  Jenkins Server │
│  (Teclado repo) │               │                 │
└─────────────────┘               └─────────────────┘
                                           │
                                           ▼
                                  ┌─────────────────┐
                                  │ Read Jenkinsfile│
                                  │ (ansible-pipeline)│
                                  └─────────────────┘
                                           │
                                           ▼
                                  ┌─────────────────┐
                                  │ Clone Source    │
                                  │ (Teclado repo)  │
                                  └─────────────────┘
                                           │
                                           ▼
                                  ┌─────────────────┐
                                  │ Execute Pipeline│
                                  │ 6 Stages        │
                                  └─────────────────┘
```

## Configuración del Pipeline

### Webhook Configuration
- **Trigger Repository**: https://github.com/Lrojas898/Teclado
- **Jenkins URL**: http://68.211.125.173
- **Webhook Endpoint**: `/generic-webhook-trigger/invoke?token=teclado-webhook-token`
- **Event**: Push to main branch
- **Status**: ✅ Configurado y funcionando

### GenericTrigger Setup
```groovy
triggers {
    GenericTrigger(
        genericVariables: [
            [key: 'ref', value: '$.ref'],
            [key: 'repository_url', value: '$.repository.html_url']
        ],
        causeString: 'Triggered by push to Teclado repository',
        token: 'teclado-webhook-token',
        regexpFilterText: '$ref,$repository_url',
        regexpFilterExpression: 'refs/heads/main,https://github.com/Lrojas898/Teclado'
    )
}
```

## Pipeline Stages - Implementación Completa

### 📥 Stage 1: Checkout
**Propósito**: Clonar código fuente desde el repositorio Teclado

**Funcionalidad**:
- Checkout automático desde `https://github.com/Lrojas898/Teclado.git`
- Copia archivos específicos: `*.html`, `*.js`, `*.md`, `css/`
- Validación de estructura de archivos

**Status**: ✅ Funcionando

### 🔨 Stage 2: Build
**Propósito**: Procesar archivos y agregar metadata del build

**Funcionalidad**:
- Procesa archivos HTML, CSS y JavaScript del repositorio Teclado
- Agrega información dinámica del build:
  - Número de build
  - Commit hash
  - Branch
  - Timestamp
- Manejo de fallback si faltan archivos

**Cambios Implementados**:
- ✅ Corrección de script de copia de archivos
- ✅ Eliminación de heredoc problemático
- ✅ Implementación con archivos temporales para metadata

**Status**: ✅ Funcionando

### 🧪 Stage 3: Test
**Propósito**: Validar estructura y contenido de la aplicación

**Validaciones Automáticas**:
- Verificación de archivos requeridos (`index.html`, `script.js`, `css/style.css`)
- Validación de DOCTYPE HTML correcto
- Verificación de título "Teclado Virtual"
- Validación de archivos CSS y JavaScript no vacíos

**Status**: ✅ Funcionando

### 📊 Stage 4: Quality Analysis (SonarQube)
**Propósito**: Análisis de calidad de código con SonarQube

**Configuración**:
- **SonarQube Server**: http://68.211.125.173:9000
- **Project Key**: teclado-virtual
- **Analysis Scope**: `**/*.html`, `**/*.js`, `**/*.css`

**Funcionalidad**:
- Detección automática de SonarQube disponible
- Instalación automática de herramientas (wget, unzip, openjdk-17, nodejs)
- Descarga y configuración de SonarQube Scanner
- Ejecución de análisis completo

**Cambios Implementados**:
- ✅ Eliminación de comandos `sudo` para compatibilidad con contenedores
- ✅ Configuración automática de proyecto SonarQube

**Status**: ✅ Funcionando

### 🚀 Stage 5: Deploy
**Propósito**: Simulación de despliegue a servidor Nginx

**Funcionalidad**:
- Preparación de archivos para despliegue
- Creación de tarball (`teclado-app.tar.gz`)
- Simulación de transferencia a servidor Nginx (68.211.125.160)
- Logging de información de despliegue

**Status**: ✅ Funcionando

### 🏥 Stage 6: Health Check
**Propósito**: Verificación post-despliegue

**Verificaciones**:
- Conectividad con servidor
- Respuesta HTTP
- Carga de recursos CSS y JS
- Funcionalidad del teclado virtual

**Status**: ✅ Funcionando

## Estructura del Proyecto

```
ansible-pipeline/
├── Jenkinsfile                 # Pipeline principal (6 stages implementadas)
├── docker-compose.yml          # Jenkins + SonarQube
├── run_pipeline_manually.sh    # Ejecución manual
├── playbook.yml               # Ansible playbook
├── inventory.ini              # Inventario de servidores
├── nginx.conf                 # Configuración Nginx
├── requirements.txt           # Dependencias Python
└── README.md                  # Esta documentación
```

## Infraestructura

### Servidores
- **Jenkins Server**: 68.211.125.173 (Puerto 80)
- **SonarQube Server**: 68.211.125.173:9000
- **Nginx Server**: 68.211.125.160

### Componentes Instalados
- Jenkins con plugins:
  - Generic Webhook Trigger Plugin
  - Pipeline Plugin
  - Git Plugin
- SonarQube Community Edition
- SonarQube Scanner CLI

## Troubleshooting y Cambios Realizados

### Problemas Resueltos

#### 1. ❌ → ✅ Error de Sintaxis en Heredoc
**Problema**: `Syntax error: redirection unexpected`
**Causa**: Heredoc `<< 'EOF'` no compatible con Jenkins pipeline
**Solución**: Reemplazado con múltiples comandos `echo` para fallback HTML

#### 2. ❌ → ✅ Error en Comando sed Multi-línea
**Problema**: Variables con espacios causaban errores de parsing
**Causa**: Expansión de `$(date)` en comando sed complejo
**Solución**: Implementación con archivo temporal y placeholders

#### 3. ❌ → ✅ Comandos sudo No Disponibles
**Problema**: `sudo: not found`
**Causa**: Jenkins corriendo en contenedor sin sudo
**Solución**: Eliminación de comandos sudo, ejecución directa

#### 4. ❌ → ✅ Stages Saltándose por Condiciones when
**Problema**: Todas las stages se saltaban
**Causa**: Condiciones `when` con validación de branch incorrecta
**Solución**: Eliminación de condiciones when restrictivas

#### 5. ❌ → ✅ Archivos No Encontrados en Checkout
**Problema**: Solo se copiaba README.md
**Causa**: Script de copia con pattern matching problemático
**Solución**: Simplificación con comandos `cp` directos

### Configuración Jenkins

#### Pipeline Configuration
- **Type**: Pipeline script from SCM
- **SCM**: Git
- **Repository URL**: `https://github.com/Lrojas898/ansible-pipeline.git`
- **Branch**: `*/main`
- **Script Path**: `Jenkinsfile`

#### Triggers
- **NO configurar triggers manuales en Jenkins UI**
- **Triggers definidos en Jenkinsfile**: GenericTrigger

## Ejecución Manual

### Ejecutar Pipeline Manualmente
1. Acceder a Jenkins: http://68.211.125.173
2. Seleccionar job "Teclado-Pipeline"
3. Hacer clic en "Build Now"

### Ejecutar con Script
```bash
chmod +x run_pipeline_manually.sh
./run_pipeline_manually.sh
```

## Monitoring y Logs

### URLs de Monitoreo
- **Jenkins Dashboard**: http://68.211.125.173
- **Pipeline Teclado**: http://68.211.125.173/job/Teclado-Pipeline/
- **SonarQube Dashboard**: http://68.211.125.173:9000
- **Proyecto en SonarQube**: http://68.211.125.173:9000/dashboard?id=teclado-virtual

### Verificar Webhook
- **GitHub → Teclado → Settings → Webhooks**
- **Recent Deliveries**: Verificar status 200 OK

## Estado Actual

✅ **Webhook**: Configurado y funcionando
✅ **Pipeline**: 6 stages implementadas y funcionando
✅ **SonarQube**: Integración completa
✅ **Testing**: Validaciones automáticas implementadas
✅ **Deploy**: Simulación funcionando
✅ **Health Check**: Verificaciones post-despliegue

### Última Actualización
- **Fecha**: Octubre 2025
- **Cambios**: Pipeline completamente funcional con todas las stages
- **Status**: ✅ PRODUCCIÓN - FUNCIONANDO