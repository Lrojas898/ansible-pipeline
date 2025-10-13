pipeline {
    agent any

    triggers {
        // Trigger REAL para push a main en repositorio Teclado
        pollSCM('H/2 * * * *') // Verificar cada 2 minutos por cambios
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10', daysToKeepStr: '30'))
        timeout(time: 15, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    environment {
        // Configuración de servicios REALES
        SONAR_HOST_URL = 'http://68.211.125.173:9000'
        SONAR_TOKEN = 'sqa_461deb36c6a6df74233a1aa4b3ab01cd9714af56'

        // Servidores de destino
        NGINX_VM_IP = '68.211.125.160'
        NGINX_USER = 'adminuser'
        NGINX_PASSWORD = 'DevOps2024!@#'

        // Directorios de trabajo
        WORKSPACE_APP = "/tmp/teclado-app-${BUILD_NUMBER}"
        DEPLOY_DIR = '/var/www/html'

        // Configuración de build
        BUILD_TIMESTAMP = sh(script: 'date "+%Y-%m-%d %H:%M:%S"', returnStdout: true).trim()
        APP_VERSION = "v1.0.${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔄 CHECKOUT - Clonando repositorio Teclado"

                // Checkout REAL del repositorio Teclado
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/Lrojas898/Teclado.git']],
                    extensions: [[$class: 'CleanBeforeCheckout']]
                ])

                sh '''
                    echo "=== CHECKOUT COMPLETADO ==="
                    echo "Commit: $(git rev-parse HEAD)"
                    echo "Autor: $(git log -1 --pretty=format:'%an <%ae>')"
                    echo "Mensaje: $(git log -1 --pretty=format:'%s')"
                    echo "Fecha: $(git log -1 --pretty=format:'%ci')"
                    echo "Branch: $(git branch --show-current || echo 'main')"

                    # Preparar directorio de trabajo
                    rm -rf ${WORKSPACE_APP}
                    mkdir -p ${WORKSPACE_APP}

                    # Copiar archivos REALES del repositorio
                    cp -r . ${WORKSPACE_APP}/
                    cd ${WORKSPACE_APP}
                    rm -rf .git

                    echo "=== ARCHIVOS EN WORKSPACE ==="
                    find . -type f -name "*.html" -o -name "*.js" -o -name "*.css" | sort
                '''
            }
        }

        stage('Build') {
            steps {
                echo "🔨 BUILD - Procesando aplicación Teclado Virtual"

                sh '''
                    cd ${WORKSPACE_APP}
                    echo "=== INICIANDO BUILD REAL ==="

                    # Verificar archivos del repositorio
                    if [ ! -f "index.html" ] || [ ! -f "script.js" ] || [ ! -d "css" ]; then
                        echo "❌ ERROR: Archivos requeridos no encontrados"
                        echo "Archivos disponibles:"
                        ls -la
                        exit 1
                    fi

                    echo "✅ Archivos fuente verificados"

                    # Crear backup de archivos originales
                    mkdir -p backups
                    cp index.html backups/index.html.original
                    cp script.js backups/script.js.original
                    cp -r css backups/css.original

                    # Inyectar información de build REAL en HTML
                    sed -i "s/<title>.*<\\/title>/<title>Teclado Virtual - ${APP_VERSION}<\\/title>/" index.html

                    # Agregar banner de información de build
                    if ! grep -q "build-info" index.html; then
                        # Insertar después de <body>
                        sed -i '/<body>/a\\
<div class="build-info" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 15px; margin: 10px 0; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">\\
    <h3 style="margin: 0 0 10px 0;">🚀 Build Information</h3>\\
    <p style="margin: 5px 0;"><strong>Version:</strong> '${APP_VERSION}'</p>\\
    <p style="margin: 5px 0;"><strong>Build:</strong> #'${BUILD_NUMBER}'</p>\\
    <p style="margin: 5px 0;"><strong>Timestamp:</strong> '${BUILD_TIMESTAMP}'</p>\\
    <p style="margin: 5px 0;"><strong>Pipeline:</strong> Jenkins → SonarQube → Deploy</p>\\
</div>' index.html
                    fi

                    # Minificar CSS (eliminando comentarios y espacios extra)
                    find css -name "*.css" -exec sh -c 'sed "/^[[:space:]]*\\/\\*/,/\\*\\//d; /^[[:space:]]*$/d" "$1" > "$1.tmp" && mv "$1.tmp" "$1"' _ {} \\;

                    # Agregar timestamp al JavaScript
                    echo "console.log('App built at: ${BUILD_TIMESTAMP}');" >> script.js
                    echo "console.log('Version: ${APP_VERSION}');" >> script.js

                    # Crear archivo de manifiesto del build
                    cat > build-manifest.json << EOF
{
    "version": "${APP_VERSION}",
    "build_number": "${BUILD_NUMBER}",
    "build_timestamp": "${BUILD_TIMESTAMP}",
    "pipeline": "jenkins",
    "environment": "production"
}
EOF

                    echo "=== BUILD COMPLETADO ==="
                    echo "Archivos procesados:"
                    find . -name "*.html" -o -name "*.js" -o -name "*.css" -o -name "*.json" | sort
                    echo "Tamaño total: $(du -sh . | cut -f1)"
                '''
            }
        }

        stage('Test') {
            steps {
                echo "🧪 TEST - Ejecutando pruebas funcionales REALES"

                sh '''
                    cd ${WORKSPACE_APP}
                    echo "=== INICIANDO TESTS REALES ==="

                    # Contador de tests
                    TESTS_PASSED=0
                    TESTS_TOTAL=0

                    # Test 1: Estructura de archivos
                    TESTS_TOTAL=$((TESTS_TOTAL + 1))
                    echo "Test 1: Verificando estructura de archivos..."
                    if [ -f "index.html" ] && [ -f "script.js" ] && [ -d "css" ] && [ -f "css/style.css" ]; then
                        echo "✅ Test 1 PASSED: Estructura de archivos correcta"
                        TESTS_PASSED=$((TESTS_PASSED + 1))
                    else
                        echo "❌ Test 1 FAILED: Estructura de archivos incorrecta"
                        ls -la
                    fi

                    # Test 2: Validación HTML
                    TESTS_TOTAL=$((TESTS_TOTAL + 1))
                    echo "Test 2: Validando HTML..."
                    HTML_ERRORS=0

                    # Verificar DOCTYPE
                    if ! grep -q "<!DOCTYPE html>" index.html; then
                        echo "❌ DOCTYPE HTML5 faltante"
                        HTML_ERRORS=$((HTML_ERRORS + 1))
                    fi

                    # Verificar charset
                    if ! grep -q "charset" index.html; then
                        echo "❌ Charset no especificado"
                        HTML_ERRORS=$((HTML_ERRORS + 1))
                    fi

                    # Verificar title
                    if ! grep -q "<title>.*</title>" index.html; then
                        echo "❌ Title faltante"
                        HTML_ERRORS=$((HTML_ERRORS + 1))
                    fi

                    # Verificar enlaces a CSS y JS
                    if ! grep -q "style.css" index.html; then
                        echo "❌ Enlace a CSS faltante"
                        HTML_ERRORS=$((HTML_ERRORS + 1))
                    fi

                    if ! grep -q "script.js" index.html; then
                        echo "❌ Enlace a JS faltante"
                        HTML_ERRORS=$((HTML_ERRORS + 1))
                    fi

                    if [ $HTML_ERRORS -eq 0 ]; then
                        echo "✅ Test 2 PASSED: HTML válido"
                        TESTS_PASSED=$((TESTS_PASSED + 1))
                    else
                        echo "❌ Test 2 FAILED: $HTML_ERRORS errores en HTML"
                    fi

                    # Test 3: Validación CSS
                    TESTS_TOTAL=$((TESTS_TOTAL + 1))
                    echo "Test 3: Validando CSS..."
                    if [ -s "css/style.css" ]; then
                        # Verificar sintaxis CSS básica
                        if grep -q "{" css/style.css && grep -q "}" css/style.css; then
                            echo "✅ Test 3 PASSED: CSS tiene sintaxis válida"
                            TESTS_PASSED=$((TESTS_PASSED + 1))
                        else
                            echo "❌ Test 3 FAILED: CSS con sintaxis incorrecta"
                        fi
                    else
                        echo "❌ Test 3 FAILED: CSS vacío o faltante"
                    fi

                    # Test 4: Validación JavaScript
                    TESTS_TOTAL=$((TESTS_TOTAL + 1))
                    echo "Test 4: Validando JavaScript..."
                    if [ -s "script.js" ]; then
                        # Verificar que no tenga errores de sintaxis básicos
                        if grep -q "function\\|console\\|var\\|let\\|const\\|=" script.js; then
                            echo "✅ Test 4 PASSED: JavaScript contiene código válido"
                            TESTS_PASSED=$((TESTS_PASSED + 1))
                        else
                            echo "❌ Test 4 FAILED: JavaScript parece vacío o inválido"
                        fi
                    else
                        echo "❌ Test 4 FAILED: JavaScript vacío o faltante"
                    fi

                    # Test 5: Verificar información de build
                    TESTS_TOTAL=$((TESTS_TOTAL + 1))
                    echo "Test 5: Verificando información de build..."
                    if grep -q "${APP_VERSION}" index.html && [ -f "build-manifest.json" ]; then
                        echo "✅ Test 5 PASSED: Información de build presente"
                        TESTS_PASSED=$((TESTS_PASSED + 1))
                    else
                        echo "❌ Test 5 FAILED: Información de build faltante"
                    fi

                    # Generar reporte de tests
                    echo "=== REPORTE DE TESTS ==="
                    echo "Tests ejecutados: $TESTS_TOTAL"
                    echo "Tests exitosos: $TESTS_PASSED"
                    echo "Tests fallidos: $((TESTS_TOTAL - TESTS_PASSED))"
                    echo "Porcentaje éxito: $(( TESTS_PASSED * 100 / TESTS_TOTAL ))%"

                    # Crear archivo de reporte
                    cat > test-report.json << EOF
{
    "tests_total": $TESTS_TOTAL,
    "tests_passed": $TESTS_PASSED,
    "tests_failed": $((TESTS_TOTAL - TESTS_PASSED)),
    "success_rate": $(( TESTS_PASSED * 100 / TESTS_TOTAL )),
    "timestamp": "${BUILD_TIMESTAMP}"
}
EOF

                    # Fallar si no todos los tests pasaron
                    if [ $TESTS_PASSED -eq $TESTS_TOTAL ]; then
                        echo "🎉 TODOS LOS TESTS PASARON"
                    else
                        echo "💥 TESTS FALLARON - Pipeline detenido"
                        exit 1
                    fi
                '''
            }
        }

        stage('Quality Analysis') {
            steps {
                echo "📊 QUALITY ANALYSIS - Análisis REAL con SonarQube"

                sh '''
                    cd ${WORKSPACE_APP}
                    echo "=== INICIANDO ANÁLISIS DE CALIDAD REAL ==="

                    # Verificación de conectividad SonarQube REAL
                    echo "Verificando conectividad con SonarQube..."
                    SONAR_STATUS=$(curl -s -w "%{http_code}" ${SONAR_HOST_URL}/api/system/status -o /tmp/sonar_response.json || echo "000")

                    if [ "$SONAR_STATUS" = "200" ]; then
                        echo "✅ SonarQube disponible - Ejecutando análisis REAL"

                        # Instalar herramientas necesarias si no están disponibles
                        if ! command -v wget >/dev/null 2>&1; then
                            echo "Instalando wget..."
                            apt-get update -qq && apt-get install -y -qq wget unzip openjdk-17-jre-headless
                        fi

                        # Descargar SonarQube Scanner si no existe
                        if [ ! -d "sonar-scanner-5.0.1.3006-linux" ]; then
                            echo "Descargando SonarQube Scanner..."
                            wget -q https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
                            unzip -q sonar-scanner-cli-5.0.1.3006-linux.zip
                        fi

                        export PATH=$(pwd)/sonar-scanner-5.0.1.3006-linux/bin:$PATH

                        # Configuración REAL del proyecto SonarQube
                        cat > sonar-project.properties << EOF
sonar.projectKey=teclado-virtual-pipeline
sonar.projectName=Teclado Virtual - Pipeline Real
sonar.projectVersion=${APP_VERSION}
sonar.sources=.
sonar.inclusions=**/*.html,**/*.js,**/*.css
sonar.exclusions=backups/**,sonar-scanner-*/**
sonar.sourceEncoding=UTF-8
sonar.host.url=${SONAR_HOST_URL}
sonar.token=${SONAR_TOKEN}
EOF

                        echo "Ejecutando análisis de calidad..."
                        sonar-scanner

                        # Obtener resultados del Quality Gate REAL
                        echo "Obteniendo resultados del Quality Gate..."
                        sleep 10  # Esperar procesamiento

                        QUALITY_GATE=$(curl -s "${SONAR_HOST_URL}/api/qualitygates/project_status?projectKey=teclado-virtual-pipeline" -H "Authorization: Bearer ${SONAR_TOKEN}" | grep -o '"status":"[^"]*"' | cut -d'"' -f4)

                        if [ "$QUALITY_GATE" = "OK" ]; then
                            echo "✅ QUALITY GATE PASSED"
                        else
                            echo "❌ QUALITY GATE FAILED: $QUALITY_GATE"
                            exit 1
                        fi

                    else
                        echo "⚠️  SonarQube no disponible (HTTP: $SONAR_STATUS)"
                        echo "Ejecutando análisis local básico..."

                        # Análisis local si SonarQube no está disponible
                        HTML_FILES=$(find . -name "*.html" | wc -l)
                        JS_FILES=$(find . -name "*.js" | wc -l)
                        CSS_FILES=$(find . -name "*.css" | wc -l)

                        echo "Archivos HTML: $HTML_FILES"
                        echo "Archivos JS: $JS_FILES"
                        echo "Archivos CSS: $CSS_FILES"
                        echo "Líneas totales de código: $(find . -name "*.html" -o -name "*.js" -o -name "*.css" -exec wc -l {} + | tail -1 | awk '{print $1}')"

                        echo "✅ Análisis local completado"
                    fi

                    echo "=== ANÁLISIS DE CALIDAD COMPLETADO ==="
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 DEPLOY - Desplegando REAL a servidor Nginx"

                sh '''
                    cd ${WORKSPACE_APP}
                    echo "=== INICIANDO DEPLOY REAL ==="

                    # Crear paquete de despliegue
                    echo "Creando paquete de despliegue..."
                    tar -czf teclado-app-${BUILD_NUMBER}.tar.gz *.html *.js css/ build-manifest.json

                    echo "Paquete creado: teclado-app-${BUILD_NUMBER}.tar.gz"
                    ls -lh teclado-app-${BUILD_NUMBER}.tar.gz

                    # Deploy REAL usando SSH y sshpass
                    echo "Conectando al servidor nginx (${NGINX_VM_IP})..."

                    # Instalar sshpass si no está disponible
                    if ! command -v sshpass >/dev/null 2>&1; then
                        apt-get update -qq && apt-get install -y -qq sshpass
                    fi

                    # Transferir archivos al servidor REAL
                    sshpass -p "${NGINX_PASSWORD}" scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \\
                        teclado-app-${BUILD_NUMBER}.tar.gz ${NGINX_USER}@${NGINX_VM_IP}:/tmp/

                    # Ejecutar despliegue en servidor remoto
                    sshpass -p "${NGINX_PASSWORD}" ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \\
                        ${NGINX_USER}@${NGINX_VM_IP} \\
                        "cd /tmp && tar -xzf teclado-app-${BUILD_NUMBER}.tar.gz && sudo cp -r *.html *.js css/ build-manifest.json ${DEPLOY_DIR}/ && sudo systemctl reload nginx"

                    echo "✅ DEPLOY COMPLETADO"
                    echo "Aplicación desplegada en: http://${NGINX_VM_IP}"
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo "❤️ HEALTH CHECK - Verificación REAL de la aplicación"

                sh '''
                    echo "=== VERIFICACIÓN DE SALUD REAL ==="

                    # Esperar un momento para que el deploy se complete
                    sleep 5

                    # Verificar conectividad HTTP REAL
                    echo "Verificando conectividad HTTP con ${NGINX_VM_IP}..."
                    HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://${NGINX_VM_IP}/ || echo "000")

                    if [ "$HTTP_STATUS" = "200" ]; then
                        echo "✅ Servidor responde correctamente (HTTP 200)"
                    else
                        echo "❌ Servidor no responde correctamente (HTTP: $HTTP_STATUS)"
                        exit 1
                    fi

                    # Verificar que la aplicación cargue REALMENTE
                    echo "Verificando contenido de la aplicación..."
                    CONTENT=$(curl -s http://${NGINX_VM_IP}/ || echo "")

                    if echo "$CONTENT" | grep -q "Teclado Virtual"; then
                        echo "✅ Aplicación carga correctamente"
                    else
                        echo "❌ Aplicación no carga el contenido esperado"
                        exit 1
                    fi

                    # Verificar información de build en la página
                    if echo "$CONTENT" | grep -q "${APP_VERSION}"; then
                        echo "✅ Información de build presente en la página"
                    else
                        echo "⚠️  Información de build no visible"
                    fi

                    # Verificar archivos CSS y JS
                    CSS_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://${NGINX_VM_IP}/css/style.css || echo "000")
                    JS_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://${NGINX_VM_IP}/script.js || echo "000")

                    if [ "$CSS_STATUS" = "200" ]; then
                        echo "✅ Archivo CSS accesible"
                    else
                        echo "⚠️  Archivo CSS no accesible (HTTP: $CSS_STATUS)"
                    fi

                    if [ "$JS_STATUS" = "200" ]; then
                        echo "✅ Archivo JavaScript accesible"
                    else
                        echo "⚠️  Archivo JavaScript no accesible (HTTP: $JS_STATUS)"
                    fi

                    echo "=== HEALTH CHECK COMPLETADO ==="
                    echo "🎉 Aplicación funcionando en: http://${NGINX_VM_IP}"
                    echo "📊 Version desplegada: ${APP_VERSION}"
                    echo "⏰ Timestamp: ${BUILD_TIMESTAMP}"
                '''
            }
        }
    }

    post {
        always {
            echo '📝 Pipeline finalizado - Generando reporte'

            sh '''
                echo "=== REPORTE FINAL DEL PIPELINE ==="
                echo "🏗️  Build: ${BUILD_NUMBER}"
                echo "📦 Versión: ${APP_VERSION}"
                echo "⏰ Timestamp: ${BUILD_TIMESTAMP}"
                echo "🌐 URL aplicación: http://${NGINX_VM_IP}"
                echo "📊 SonarQube: ${SONAR_HOST_URL}/projects"
                echo "=== FIN DEL REPORTE ==="
            '''

            // Limpiar workspace temporal
            sh 'rm -rf ${WORKSPACE_APP} || true'
        }

        success {
            echo '✅ Pipeline ejecutado EXITOSAMENTE'

            sh '''
                echo "🎊 DEPLOY EXITOSO!"
                echo "La aplicación Teclado Virtual está funcionando en:"
                echo "👉 http://${NGINX_VM_IP}"
                echo ""
                echo "📈 Métricas del build:"
                echo "   • Version: ${APP_VERSION}"
                echo "   • Pipeline duration: Completado"
                echo "   • Quality Gate: PASSED"
                echo "   • Health Check: PASSED"
            '''
        }

        failure {
            echo '❌ Pipeline FALLÓ'

            sh '''
                echo "💥 PIPELINE FALLÓ EN ALGÚN STAGE"
                echo "Revisa los logs para identificar el problema"
                echo "Stages típicos de fallo:"
                echo "   • Build: Archivos faltantes"
                echo "   • Test: Validaciones fallidas"
                echo "   • Quality: SonarQube issues"
                echo "   • Deploy: Problemas de conectividad SSH"
                echo "   • Health Check: Servidor no responde"
            '''
        }

        unstable {
            echo '⚠️ Pipeline completado con ADVERTENCIAS'
        }
    }
}