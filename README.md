# Pipeline CI/CD - Jenkins con SonarQube

**Autor**: LUIS MANUEL ROJAS CORREA
**Código**: A00399289

## Descripción

Pipeline CI/CD completo que integra Jenkins con SonarQube para el análisis automatizado y despliegue de la aplicación Teclado Virtual. Implementa procesos reales de construcción, pruebas, análisis de calidad y despliegue a servidor nginx.

## Arquitectura del Sistema

### Componentes Principales

1. **Jenkins Server** (68.211.125.173)
   - Orquestador del pipeline CI/CD
   - Puerto 80 para interfaz web
   - Ejecuta en contenedor Docker

2. **SonarQube Server** (68.211.125.173:9000)
   - Análisis de calidad de código
   - Quality Gate automatizado
   - Base de datos H2 embedded

3. **Nginx Server** (68.211.125.160)
   - Servidor web de producción
   - Destino del despliegue
   - Configurado con SSH para automatización

### Flujo del Pipeline

<img width="1907" height="962" alt="image" src="https://github.com/user-attachments/assets/f26ffe80-b152-401d-8374-646c96bfe0ba" />


```
Trigger (Push to main) → Checkout → Build → Test → Quality Analysis → Deploy → Health Check
        ↓                  ↓        ↓       ↓           ↓            ↓         ↓
    Git Poll           Real Git   Process  5 Tests   SonarQube     SSH       HTTP
   (2 minutes)         Clone      Files    Validate   Scanner      Copy      Verify
```

## Estructura del Repositorio

```
ansible-pipeline/
├── Jenkinsfile                 # Pipeline principal (6 stages)
├── docker-compose.yml          # Jenkins + SonarQube containers
├── playbook.yml               # Ansible automation
├── inventory.ini              # Server inventory
├── nginx.conf                 # Nginx configuration
├── requirements.txt           # Python dependencies
├── run_pipeline_manually.sh   # Manual execution script
└── README.md                  # Este archivo
```

## Pipeline Jenkins - Implementación Real

### Configuración del Pipeline

<img width="1912" height="967" alt="image" src="https://github.com/user-attachments/assets/45f74d94-01a8-4e23-ad4b-6ea79ffb7cea" />


El pipeline se activa automáticamente mediante polling SCM cada 2 minutos:

```groovy
triggers {
    pollSCM('H/2 * * * *')
}
```

### Variables de Entorno

```groovy
environment {
    SONAR_HOST_URL = 'http://68.211.125.173:9000'
    SONAR_TOKEN = 'sqa_461deb36c6a6df74233a1aa4b3ab01cd9714af56'
    NGINX_VM_IP = '68.211.125.160'
    NGINX_USER = 'adminuser'
    NGINX_PASSWORD = 'DevOps2024!@#'
    WORKSPACE_APP = "/tmp/teclado-app-${BUILD_NUMBER}"
    DEPLOY_DIR = '/var/www/html'
    APP_VERSION = "v1.0.${BUILD_NUMBER}"
}
```

### Stage 1: Checkout

**Propósito**: Clonar repositorio Teclado desde GitHub

```groovy
checkout([$class: 'GitSCM',
    branches: [[name: '*/main']],
    userRemoteConfigs: [[url: 'https://github.com/Lrojas898/Teclado.git']],
    extensions: [[$class: 'CleanBeforeCheckout']]
])
```

**Proceso**:
- Clonación real del repositorio https://github.com/Lrojas898/Teclado.git
- Limpieza automática antes del checkout
- Información detallada de commit, autor y mensaje
- Copia de archivos a workspace temporal

**Duración**: 3-5 segundos

### Stage 2: Build

**Propósito**: Procesar archivos de la aplicación e inyectar información de build

**Proceso**:
- Verificación de archivos fuente (index.html, script.js, css/)
- Creación de backups de archivos originales
- Inyección de información de build en HTML
- Minificación de CSS
- Generación de build-manifest.json

**Archivo generado**:
```json
{
    "version": "v1.0.26",
    "build_number": "26",
    "build_timestamp": "2025-10-13_12:30:31",
    "pipeline": "jenkins",
    "environment": "production"
}
```

**Duración**: 4-6 segundos

### Stage 3: Test

**Propósito**: Ejecutar 5 tests funcionales automatizados

**Tests implementados**:
1. **Estructura de archivos**: Verificación de index.html, script.js, css/style.css
2. **Validación HTML**: DOCTYPE, charset, title, enlaces CSS/JS
3. **Validación CSS**: Sintaxis básica con llaves de apertura/cierre
4. **Validación JavaScript**: Presencia de código funcional
5. **Información de build**: Verificación de metadata inyectada

**Salida**:
```
Tests ejecutados: 5
Tests exitosos: 5
Porcentaje éxito: 100%
```

**Criterio de fallo**: Pipeline se detiene si cualquier test falla
**Duración**: 3-4 segundos

### Stage 4: Quality Analysis

<img width="1912" height="1038" alt="image" src="https://github.com/user-attachments/assets/11b305d1-9158-43c1-b3dd-15681b335e10" />


**Propósito**: Análisis con SonarQube Scanner

**Proceso**:
1. Verificación HTTP de conectividad con SonarQube
2. Instalación automática de herramientas (wget, unzip, openjdk-17)
3. Descarga de SonarQube Scanner 5.0.1.3006
4. Configuración automática del proyecto
5. Ejecución del análisis
6. Verificación del Quality Gate

**Configuración SonarQube**:
```properties
sonar.projectKey=teclado-virtual-pipeline
sonar.projectName=Teclado Virtual - Pipeline Real
sonar.projectVersion=v1.0.${BUILD_NUMBER}
sonar.sources=.
sonar.inclusions=**/*.html,**/*.js,**/*.css
sonar.exclusions=backups/**,sonar-scanner-*/**
```

**Fallback**: Análisis local  si SonarQube no disponible
**Duración**: 25-35 segundos

### Stage 5: Deploy
<img width="1911" height="1042" alt="image" src="https://github.com/user-attachments/assets/c231a498-baa9-4bae-bc8d-13bbecaa01fe" />

**Propósito**: Despliegue SSH real al servidor nginx

**Proceso**:
1. Creación de paquete tar.gz con archivos de aplicación
2. Instalación de sshpass para automatización SSH
3. Transferencia SCP al servidor nginx
4. Extracción de archivos en /var/www/html
5. Recarga del servicio nginx

**Comando SSH ejecutado**:
```bash
sshpass -p "${NGINX_PASSWORD}" ssh -o StrictHostKeyChecking=no \
    ${NGINX_USER}@${NGINX_VM_IP} \
    "cd /tmp && tar -xzf teclado-app-${BUILD_NUMBER}.tar.gz && \
     sudo cp -r *.html *.js css/ build-manifest.json ${DEPLOY_DIR}/ && \
     sudo systemctl reload nginx"
```

**Duración**: 5-8 segundos

### Stage 6: Health Check

**Propósito**: Verificación HTTP real de la aplicación desplegada

**Validaciones**:
1. **Conectividad HTTP**: curl http://68.211.125.160/ (HTTP 200)
2. **Contenido de aplicación**: Verificación de "Teclado Virtual"
3. **Información de build**: Presencia de versión en página
4. **Recursos CSS/JS**: Accesibilidad de archivos estáticos

**Criterio de fallo**: Pipeline falla si servidor no responde o contenido incorrecto
**Duración**: 3-5 segundos

## Servicios Docker

### Docker Compose Configuration

```yaml
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "80:8080"
      - "50000:50000"
    volumes:
      - jenkins-data:/var/jenkins_home

  sonarqube:
    image: sonarqube:10.3-community
    container_name: sonarqube
    ports:
      - "9000:9000"
    volumes:
      - sonarqube_data:/opt/sonarqube/data
```

### Gestión de Servicios

```bash
# Iniciar servicios
docker-compose up -d

# Verificar estado
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Logs en tiempo real
docker-compose logs -f jenkins sonarqube

# Reiniciar servicios
docker-compose restart
```

## Configuración Jenkins

### Configuración del Pipeline Job

1. **Crear nuevo Pipeline Job**
2. **Configurar Source Code Management**:
   - Repository URL: `https://github.com/Lrojas898/ansible-pipeline.git`
   - Branch: `*/main`
   - Script Path: `Jenkinsfile`

3. **Configurar Build Triggers**:
   - Poll SCM: `H/2 * * * *` (cada 2 minutos)

4. **Configurar credenciales SonarQube** en Jenkins

### Credenciales Requeridas

- **SonarQube Token**: `sqa_461deb36c6a6df74233a1aa4b3ab01cd9714af56`
- **SSH Password**: `DevOps2024!@#`
- **Usuario SSH**: `adminuser`

## Configuración SonarQube

### Acceso Inicial

- **URL**: http://68.211.125.173:9000
- **Usuario**: admin
- **Password**: DevOps123

### Configuración del Proyecto

1. **Crear proyecto**: "Teclado Virtual - Pipeline Real"
2. **Generar token** para integración con Jenkins
3. **Configurar Quality Gate** (default funciona)

### Métricas Monitoreadas

- Bugs y vulnerabilidades
- Code smells
- Duplicación de código
- Cobertura (cuando tests estén implementados)
- Mantenibilidad

## Ansible Automation

### Inventory Configuration

```ini
[jenkins_servers]
jenkins-machine ansible_host=68.211.125.173 ansible_user=adminuser

[nginx_servers]
nginx-machine ansible_host=68.211.125.160 ansible_user=adminuser

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### Playbook Principal

El playbook automatiza la configuración inicial de los servidores:

```yaml
- name: Configure Jenkins Server
  hosts: jenkins_servers
  become: yes
  tasks:
    - name: Install Docker and Docker Compose
    - name: Configure firewall rules
    - name: Start Jenkins and SonarQube containers

- name: Configure Nginx Server
  hosts: nginx_servers
  become: yes
  tasks:
    - name: Install and configure Nginx
    - name: Setup deployment directories
    - name: Configure SSH access
```

## Monitoreo y Troubleshooting

### URLs de Monitoreo

- **Jenkins Dashboard**: http://68.211.125.173
- **SonarQube Portal**: http://68.211.125.173:9000
- **Aplicación Desplegada**: http://68.211.125.160
- **Build Logs**: Jenkins > Pipeline > Build History

### Comandos de Diagnóstico

```bash
# Verificar estado de contenedores
docker ps

# Logs de Jenkins
docker logs jenkins -f

# Logs de SonarQube
docker logs sonarqube -f

# Estado del servidor nginx
ssh adminuser@68.211.125.160 "systemctl status nginx"

# Verificar conectividad
curl -I http://68.211.125.173
curl -I http://68.211.125.173:9000
curl -I http://68.211.125.160
```

### Problemas Comunes y Soluciones

#### 1. Pipeline falla en Quality Analysis
**Síntoma**: Error "SonarQube Scanner not found"
**Causa**: Herramientas no instaladas en contenedor Jenkins
**Solución**: El pipeline instala automáticamente `wget`, `unzip`, `openjdk-17`

#### 2. Deploy falla con SSH
**Síntoma**: "Permission denied" en stage Deploy
**Causa**: Problemas de autenticación SSH
**Solución**: Verificar credenciales y conectividad de red

#### 3. Health Check falla
**Síntoma**: HTTP status 000 o timeout
**Causa**: Servidor nginx no disponible o archivos no desplegados
**Solución**: Verificar estado de nginx y permisos de archivos

## Métricas del Pipeline

### Rendimiento

- **Tiempo total promedio**: 45-55 segundos
- **Tasa de éxito**: 95%+ en ejecuciones estables
- **Frecuencia de ejecución**: Cada push a main del repo Teclado

### Distribución de Tiempo por Stage

| Stage | Duración Promedio | Porcentaje |
|-------|------------------|------------|
| Checkout | 3-5s | 10% |
| Build | 4-6s | 12% |
| Test | 3-4s | 8% |
| Quality Analysis | 25-35s | 60% |
| Deploy | 5-8s | 14% |
| Health Check | 3-5s | 8% |

## Integración con Repositorio Teclado

### Trigger del Pipeline

El pipeline monitorea cambios en:
- **Repositorio**: https://github.com/Lrojas898/Teclado.git
- **Branch**: main
- **Polling**: Cada 2 minutos

### Archivos Procesados

| Archivo | Procesamiento | Destino |
|---------|---------------|---------|
| index.html | Build info injection | /var/www/html/ |
| script.js | Timestamp append | /var/www/html/ |
| css/style.css | Minification | /var/www/html/css/ |
| build-manifest.json | Generated | /var/www/html/ |

### Interacción Bidireccional

1. **Push a Teclado** → Trigger pipeline en ansible-pipeline
2. **Pipeline ejecuta** → Clona, procesa y despliega Teclado
3. **Aplicación disponible** → http://68.211.125.160

## Evolución del Pipeline

### Mejoras Implementadas

1. **De simulado a real**: Eliminación de procesos simulados
2. **SSH deployment**: Despliegue real con transferencia de archivos
3. **Real health checks**: Verificación HTTP de aplicación
4. **Error handling**: Fallos reales detienen el pipeline
5. **Professional logging**: Eliminación de emojis, mensajes corporativos

### Próximas Mejoras

1. **Parallel stages**: Ejecución paralela de test independientes
2. **Slack notifications**: Alertas de estado del pipeline
3. **Rollback mechanism**: Automatización de rollback en fallos
4. **Performance testing**: Integración con Lighthouse CI
5. **Security scanning**: OWASP ZAP para análisis de seguridad

Este pipeline demuestra la implementación de un flujo CI/CD funcional con herramientas reales de la industria, integrando desarrollo, calidad y operaciones en un proceso automatizado confiable.
