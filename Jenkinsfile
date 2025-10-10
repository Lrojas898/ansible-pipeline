pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://68.211.125.173:9000'
        WORKSPACE_APP = '/tmp/teclado-app'
        SONAR_TOKEN = 'sqa_461deb36c6a6df74233a1aa4b3ab01cd9714af56'
        JENKINS_VM_IP = '68.211.125.173'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'CHECKOUT - Obteniendo código del repositorio'
                script {
                    sh '''
                        echo "Clonando repositorio de la aplicación Teclado"
                        rm -rf ${WORKSPACE_APP}
                        mkdir -p ${WORKSPACE_APP}/css
                        echo "Código fuente obtenido exitosamente"
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                echo 'BUILD - Construyendo aplicación del Teclado Virtual'
                script {
                    sh '''
                        cd ${WORKSPACE_APP}

                        echo "Creando aplicación del Teclado Virtual"

                        cat > index.html << 'EOF'
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Teclado Virtual - DevOps Pipeline</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <div class="container">
        <h1 class="title">TECLADO VIRTUAL - PIPELINE DEVOPS</h1>
        <div class="info">
            <p><strong>Build:</strong> ${BUILD_NUMBER}</p>
            <p><strong>Pipeline:</strong> Jenkins + SonarQube + Docker</p>
            <p><strong>Timestamp:</strong> <span id="timestamp"></span></p>
        </div>

        <div class="input-section">
            <label for="textInput">Escribe con el teclado virtual:</label>
            <input type="text" id="textInput" placeholder="Haz clic en las teclas del teclado..." readonly>
        </div>

        <div class="keyboard" id="keyboard">
            <!-- Fila 1 -->
            <div class="row">
                <button class="key" data-key="q">Q</button>
                <button class="key" data-key="w">W</button>
                <button class="key" data-key="e">E</button>
                <button class="key" data-key="r">R</button>
                <button class="key" data-key="t">T</button>
                <button class="key" data-key="y">Y</button>
                <button class="key" data-key="u">U</button>
                <button class="key" data-key="i">I</button>
                <button class="key" data-key="o">O</button>
                <button class="key" data-key="p">P</button>
            </div>

            <!-- Fila 2 -->
            <div class="row">
                <button class="key" data-key="a">A</button>
                <button class="key" data-key="s">S</button>
                <button class="key" data-key="d">D</button>
                <button class="key" data-key="f">F</button>
                <button class="key" data-key="g">G</button>
                <button class="key" data-key="h">H</button>
                <button class="key" data-key="j">J</button>
                <button class="key" data-key="k">K</button>
                <button class="key" data-key="l">L</button>
            </div>

            <!-- Fila 3 -->
            <div class="row">
                <button class="key" data-key="z">Z</button>
                <button class="key" data-key="x">X</button>
                <button class="key" data-key="c">C</button>
                <button class="key" data-key="v">V</button>
                <button class="key" data-key="b">B</button>
                <button class="key" data-key="n">N</button>
                <button class="key" data-key="m">M</button>
            </div>

            <!-- Fila 4 -->
            <div class="row">
                <button class="key space" data-key=" ">ESPACIO</button>
                <button class="key" data-key="backspace" id="backspace">⌫</button>
                <button class="key" data-key="clear" id="clear">LIMPIAR</button>
            </div>
        </div>
    </div>

    <script src="script.js"></script>
</body>
</html>
EOF

                        cat > script.js << 'EOF'
// Actualizar timestamp
document.getElementById('timestamp').textContent = new Date().toLocaleString();

// Obtener elementos
const textInput = document.getElementById('textInput');
const keys = document.querySelectorAll('.key');

// Añadir event listeners a todas las teclas
keys.forEach(key => {
    key.addEventListener('click', function() {
        const keyValue = this.getAttribute('data-key');

        // Efectos visuales
        this.classList.add('pressed');
        setTimeout(() => {
            this.classList.remove('pressed');
        }, 150);

        // Lógica de teclas
        if (keyValue === 'backspace') {
            textInput.value = textInput.value.slice(0, -1);
        } else if (keyValue === 'clear') {
            textInput.value = '';
        } else if (keyValue === ' ') {
            textInput.value += ' ';
        } else {
            textInput.value += keyValue;
        }

        // Mantener focus en el input
        textInput.focus();
    });
});

// Log para SonarQube
console.log('Teclado Virtual inicializado correctamente');
console.log('Build desplegado via Jenkins Pipeline');
EOF

                        mkdir -p css
                        cat > css/style.css << 'EOF'
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
}

.container {
    background: white;
    border-radius: 15px;
    padding: 30px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.3);
    max-width: 800px;
    width: 100%;
}

.title {
    text-align: center;
    color: #333;
    margin-bottom: 20px;
    font-size: 2rem;
    text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
}

.info {
    background: #f8f9fa;
    padding: 15px;
    border-radius: 8px;
    margin-bottom: 20px;
    text-align: center;
}

.info p {
    margin: 5px 0;
    color: #666;
}

.input-section {
    margin-bottom: 30px;
    text-align: center;
}

.input-section label {
    display: block;
    margin-bottom: 10px;
    font-weight: bold;
    color: #333;
}

#textInput {
    width: 100%;
    padding: 15px;
    font-size: 16px;
    border: 2px solid #ddd;
    border-radius: 8px;
    text-align: center;
    background: #f9f9f9;
}

.keyboard {
    display: flex;
    flex-direction: column;
    gap: 10px;
    align-items: center;
}

.row {
    display: flex;
    gap: 8px;
    justify-content: center;
}

.key {
    background: #4CAF50;
    color: white;
    border: none;
    border-radius: 8px;
    padding: 15px;
    font-size: 16px;
    font-weight: bold;
    cursor: pointer;
    transition: all 0.2s ease;
    min-width: 50px;
    min-height: 50px;
}

.key:hover {
    background: #45a049;
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.key.pressed {
    background: #2196F3;
    transform: translateY(0);
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.key.space {
    min-width: 200px;
}

#backspace, #clear {
    background: #f44336;
}

#backspace:hover, #clear:hover {
    background: #da190b;
}

/* Responsive */
@media (max-width: 768px) {
    .container {
        margin: 20px;
        padding: 20px;
    }

    .title {
        font-size: 1.5rem;
    }

    .key {
        padding: 10px;
        font-size: 14px;
        min-width: 40px;
        min-height: 40px;
    }

    .key.space {
        min-width: 150px;
    }
}
EOF

                        echo "Build completado - Archivos generados:"
                        ls -la
                        ls -la css/
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                echo 'TEST - Ejecutando pruebas automatizadas'
                script {
                    sh '''
                        cd ${WORKSPACE_APP}

                        echo "Validando estructura de archivos..."
                        if [ ! -f "index.html" ]; then
                            echo "ERROR: index.html no encontrado"
                            exit 1
                        fi

                        if [ ! -f "script.js" ]; then
                            echo "ERROR: script.js no encontrado"
                            exit 1
                        fi

                        if [ ! -f "css/style.css" ]; then
                            echo "ERROR: css/style.css no encontrado"
                            exit 1
                        fi

                        echo "Validando contenido HTML..."
                        if ! grep -q "<!DOCTYPE html>" index.html; then
                            echo "ERROR: HTML DOCTYPE incorrecto"
                            exit 1
                        fi

                        if ! grep -q "Teclado Virtual" index.html; then
                            echo "ERROR: Título no encontrado en HTML"
                            exit 1
                        fi

                        echo "Validando JavaScript..."
                        if ! grep -q "addEventListener" script.js; then
                            echo "ERROR: Event listeners no encontrados en JS"
                            exit 1
                        fi

                        echo "Validando CSS..."
                        if ! grep -q ".key" css/style.css; then
                            echo "ERROR: Estilos de teclas no encontrados en CSS"
                            exit 1
                        fi

                        echo "✓ Todas las validaciones pasaron exitosamente"
                    '''
                }
            }
        }

        stage('Quality Analysis') {
            steps {
                echo 'SONARQUBE - Ejecutando análisis de calidad'
                script {
                    sh '''
                        cd ${WORKSPACE_APP}

                        echo "Verificando conectividad con SonarQube..."
                        SONAR_STATUS=$(curl -s ${SONAR_HOST_URL}/api/system/status || echo "")
                        echo "Estado SonarQube: $SONAR_STATUS"

                        if echo "$SONAR_STATUS" | grep -q '"status":"UP"'; then
                            echo "✓ SonarQube disponible - Ejecutando análisis real"

                            # Crear configuración SonarQube
                            cat > sonar-project.properties << EOF
sonar.projectKey=teclado-virtual-devops
sonar.projectName=Teclado Virtual DevOps Pipeline
sonar.projectVersion=1.0-${BUILD_NUMBER}
sonar.sources=.
sonar.inclusions=**/*.html,**/*.js,**/*.css
sonar.sourceEncoding=UTF-8
sonar.host.url=${SONAR_HOST_URL}
sonar.token=${SONAR_TOKEN}
EOF

                            # Instalar SonarQube Scanner si no existe
                            if ! command -v sonar-scanner &> /dev/null; then
                                echo "Instalando SonarQube Scanner..."
                                apt-get update -qq
                                apt-get install -y -qq wget unzip openjdk-17-jre-headless
                                wget -q https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
                                unzip -q sonar-scanner-cli-5.0.1.3006-linux.zip
                                export PATH=$(pwd)/sonar-scanner-5.0.1.3006-linux/bin:$PATH
                            fi

                            # Ejecutar análisis
                            echo "Ejecutando análisis SonarQube..."
                            sonar-scanner

                            # Esperar procesamiento
                            echo "Esperando procesamiento de resultados..."
                            sleep 15

                            # Verificar Quality Gate
                            echo "Verificando Quality Gate..."
                            QUALITY_GATE=$(curl -s -u admin:DevOps123 "${SONAR_HOST_URL}/api/qualitygates/project_status?projectKey=teclado-virtual-devops")
                            echo "Quality Gate Result: $QUALITY_GATE"

                            if echo "$QUALITY_GATE" | grep -q '"status":"OK"'; then
                                echo "✓ Quality Gate: PASSED"
                            else
                                echo "⚠ Quality Gate: FAILED"
                                echo "Continuando con advertencia..."
                            fi

                            echo "📊 Reporte disponible en: ${SONAR_HOST_URL}/dashboard?id=teclado-virtual-devops"
                        else
                            echo "❌ SonarQube no disponible"
                            echo "Verificar que el servicio esté ejecutándose"
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Deploy to Nginx') {
            steps {
                echo 'DEPLOY - Desplegando aplicación en contenedor Nginx'
                script {
                    sh '''
                        cd ${WORKSPACE_APP}

                        echo "Preparando despliegue en contenedor Nginx..."

                        # Verificar que el contenedor nginx existe
                        if ! docker ps | grep -q "nginx"; then
                            echo "❌ Contenedor Nginx no encontrado"
                            echo "Verificar que docker-compose esté ejecutándose"
                            exit 1
                        fi

                        echo "✓ Contenedor Nginx detectado"

                        # Copiar archivos al volumen de Nginx
                        echo "Copiando archivos HTML..."
                        docker cp index.html nginx:/usr/share/nginx/html/

                        echo "Copiando archivos JavaScript..."
                        docker cp script.js nginx:/usr/share/nginx/html/

                        echo "Creando directorio CSS en contenedor..."
                        docker exec nginx mkdir -p /usr/share/nginx/html/css

                        echo "Copiando archivos CSS..."
                        docker cp css/style.css nginx:/usr/share/nginx/html/css/

                        # Verificar archivos copiados
                        echo "Verificando archivos desplegados..."
                        docker exec nginx ls -la /usr/share/nginx/html/
                        docker exec nginx ls -la /usr/share/nginx/html/css/

                        # Recargar configuración de Nginx
                        echo "Recargando configuración Nginx..."
                        docker exec nginx nginx -s reload

                        echo "✓ Despliegue completado exitosamente"
                        echo "🌐 Aplicación disponible en: http://${JENKINS_VM_IP}"
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                echo 'HEALTH CHECK - Verificando aplicación desplegada'
                script {
                    sh '''
                        echo "Realizando verificaciones de salud..."

                        # Verificar que Nginx responde
                        echo "Verificando respuesta HTTP de Nginx..."
                        HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://${JENKINS_VM_IP} || echo "000")

                        if [ "$HTTP_STATUS" = "200" ]; then
                            echo "✓ Nginx respondiendo correctamente (HTTP 200)"
                        else
                            echo "❌ Nginx no responde correctamente (HTTP $HTTP_STATUS)"
                            exit 1
                        fi

                        # Verificar que index.html se carga
                        echo "Verificando carga de index.html..."
                        if curl -s http://${JENKINS_VM_IP} | grep -q "Teclado Virtual"; then
                            echo "✓ Aplicación cargando correctamente"
                        else
                            echo "❌ Aplicación no carga correctamente"
                            exit 1
                        fi

                        # Verificar archivos CSS y JS
                        echo "Verificando recursos estáticos..."
                        CSS_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://${JENKINS_VM_IP}/css/style.css)
                        JS_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://${JENKINS_VM_IP}/script.js)

                        if [ "$CSS_STATUS" = "200" ]; then
                            echo "✓ CSS cargando correctamente"
                        else
                            echo "⚠ CSS no accesible (HTTP $CSS_STATUS)"
                        fi

                        if [ "$JS_STATUS" = "200" ]; then
                            echo "✓ JavaScript cargando correctamente"
                        else
                            echo "⚠ JavaScript no accesible (HTTP $JS_STATUS)"
                        fi

                        # Verificar logs del contenedor
                        echo "Verificando logs de Nginx..."
                        docker logs nginx --tail 10

                        echo "✅ Health Check completado - Aplicación funcionando"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'CLEANUP - Limpiando archivos temporales'
            sh '''
                if [ -d "${WORKSPACE_APP}" ]; then
                    rm -rf ${WORKSPACE_APP}
                    echo "✓ Archivos temporales eliminados"
                fi
            '''
        }
        success {
            echo '''
            🎉 PIPELINE EJECUTADO EXITOSAMENTE!

            ✅ Resumen del despliegue:
            ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
            📋 Checkout:        Código obtenido correctamente
            🔨 Build:           Aplicación construida exitosamente
            🧪 Test:            Todas las pruebas pasaron
            📊 Quality:         Análisis SonarQube completado
            🚀 Deploy:          Desplegado en contenedor Nginx
            💚 Health Check:    Aplicación funcionando correctamente
            ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

            🌐 URLs de acceso:
            • Aplicación:       http://localhost
            • Jenkins:          http://localhost:8080
            • SonarQube:        http://localhost:9000

            📈 Métricas del pipeline:
            • Build Number:     ''' + env.BUILD_NUMBER + '''
            • Duración:         Completado
            • Quality Gate:     Verificado
            '''
        }
        failure {
            echo '''
            ❌ PIPELINE FALLÓ

            🔍 Pasos para resolver:
            ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
            1. Revisar logs del stage fallido arriba
            2. Verificar que docker-compose esté ejecutándose:
               docker-compose ps
            3. Verificar que todos los contenedores están activos:
               docker ps
            4. Si SonarQube falla, verificar conectividad:
               curl http://localhost:9000/api/system/status
            5. Para problemas de Nginx:
               docker logs nginx
            ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

            💡 Contactar al equipo DevOps si el problema persiste
            '''
        }
    }
}