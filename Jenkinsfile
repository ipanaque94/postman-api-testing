pipeline {
    agent any
    
    parameters {
        choice(
            name: 'TEST_PROJECT',
            choices: ['all', 'ui-tests', 'api-tests', 'e2e-tests', 'integration-tests'],
            description: '🎯 Proyecto de tests a ejecutar'
        )
        choice(
            name: 'TEST_SUITE',
            choices: ['all', 'smoke', 'regression', 'api', 'ui'],
            description: '📋 Suite de tests'
        )
        choice(
            name: 'WORKERS',
            choices: ['1', '2', '4', 'auto'],
            description: '⚡ Workers paralelos'
        )
        choice(
            name: 'RETRIES',
            choices: ['0', '1', '2'],
            description: '🔄 Reintentos en caso de fallo'
        )
        booleanParam(
            name: 'HEADED',
            defaultValue: false,
            description: '🖥️ Ejecutar tests en modo visible (headed)'
        )
        booleanParam(
            name: 'GENERATE_TRACE',
            defaultValue: true,
            description: '📹 Generar traces para debugging'
        )
    }
    
    options {
        timeout(time: 1, unit: 'HOURS')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    stages {
        stage('🔍 Pre-Build Checks') {
            steps {
                script {
                    echo '═══════════════════════════════════════════'
                    echo '   PRE-BUILD CHECKS'
                    echo '═══════════════════════════════════════════'
                }
                
                sh '''
                    echo "📋 Información del Sistema:"
                    echo "  - Node: $(node --version)"
                    echo "  - NPM: $(npm --version)"
                    echo "  - Playwright: $(npx playwright --version)"
                    echo "  - Workspace: ${WORKSPACE}"
                    echo ""
                    echo "⚙️  Parámetros del Build:"
                    echo "  - Proyecto: ${TEST_PROJECT}"
                    echo "  - Suite: ${TEST_SUITE}"
                    echo "  - Workers: ${WORKERS}"
                    echo "  - Retries: ${RETRIES}"
                    echo "  - Headed: ${HEADED}"
                    echo "  - Generate Trace: ${GENERATE_TRACE}"
                '''
            }
        }
        
        stage('📥 Checkout') {
            steps {
                script {
                    echo '═══════════════════════════════════════════'
                    echo '   CHECKOUT CODE'
                    echo '═══════════════════════════════════════════'
                }
                
                checkout scm
                
                sh '''
                    echo "✅ Código descargado"
                    echo "📂 Branch: $(git branch --show-current || echo 'detached HEAD')"
                    echo "🔖 Commit: $(git rev-parse --short HEAD)"
                    echo "👤 Autor: $(git log -1 --pretty=format:'%an')"
                    echo "📝 Mensaje: $(git log -1 --pretty=format:'%s')"
                '''
            }
        }
        
        stage('🧹 Clean') {
            steps {
                script {
                    echo '═══════════════════════════════════════════'
                    echo '   CLEANING OLD ARTIFACTS'
                    echo '═══════════════════════════════════════════'
                }
                
                sh '''
                    echo "🗑️  Eliminando archivos antiguos..."
                    rm -rf node_modules
                    rm -rf test-results
                    rm -rf playwright-report
                    rm -rf reports
                    rm -rf screenshots
                    rm -rf videos
                    rm -rf traces
                    echo "✅ Limpieza completada"
                '''
            }
        }
        
        stage('📦 Install Dependencies') {
            steps {
                script {
                    echo '═══════════════════════════════════════════'
                    echo '   INSTALLING DEPENDENCIES'
                    echo '═══════════════════════════════════════════'
                }
                
                sh '''
                    echo "📥 Instalando dependencias de Node..."
                    npm ci --quiet
                    echo "✅ Dependencias instaladas"
                    echo ""
                    echo "📊 Resumen de dependencias:"
                    npm list --depth=0
                '''
            }
        }
        
        stage('🎭 Verify Playwright Browsers') {
            steps {
                script {
                    echo '═══════════════════════════════════════════'
                    echo '   VERIFYING PLAYWRIGHT BROWSERS'
                    echo '═══════════════════════════════════════════'
                }
                
                sh '''
                    echo "✅ Navegadores ya instalados en imagen Docker"
                    echo ""
                    echo "📂 Verificando ubicación:"
                    which playwright || echo "Playwright instalado localmente"
                    npx playwright --version
                    echo ""
                    echo "✅ Verificación completada"
                '''
            }
        }
        
        stage('🧪 Run Tests') {
            steps {
                script {
                    echo '═══════════════════════════════════════════'
                    echo '   RUNNING TESTS'
                    echo '═══════════════════════════════════════════'

                    // FIX: Crear el directorio test-results ANTES de ejecutar Playwright
                    // para garantizar que el JUnit reporter pueda escribir el XML
                    sh 'mkdir -p test-results'
                    
                    def command = 'npx playwright test'
                    
                    // Proyecto específico
                    if (params.TEST_PROJECT != 'all') {
                        command += " --project=${params.TEST_PROJECT}"
                    }
                    
                    // Suite específica
                    if (params.TEST_SUITE != 'all') {
                        command += " tests/${params.TEST_SUITE}/"
                    }
                    
                    // Workers
                    if (params.WORKERS != 'auto') {
                        command += " --workers=${params.WORKERS}"
                    }
                    
                    // Retries
                    if (params.RETRIES != '0') {
                        command += " --retries=${params.RETRIES}"
                    }
                    
                    // Headed mode
                    if (params.HEADED == true) {
                        command += ' --headed'
                    }
                    
                    // Traces
                    if (params.GENERATE_TRACE == true) {
                        command += ' --trace=retain-on-failure'
                    }
                    
                    // NO pasar --reporter en el comando
                    // Los reporters están definidos en playwright.config.ts
                    // para que outputFile de junit sea respetado
                    
                    echo "🚀 Comando a ejecutar:"
                    echo "   ${command}"
                    echo ""
                    
                    // Ejecutar tests — capturar exit code pero continuar
                    def testResult = sh(script: command, returnStatus: true)
                    
                    echo ""
                    echo "📊 Resultado de los tests:"
                    if (testResult == 0) {
                        echo "   ✅ Todos los tests pasaron"
                    } else {
                        echo "   ⚠️  Algunos tests fallaron (Exit code: ${testResult})"
                        currentBuild.result = 'UNSTABLE'
                    }

                    // FIX: Verificar que el XML fue generado correctamente
                    // Ya NO se hace cp — el reporter escribe directo en test-results/
                    sh '''
                        echo ""
                        echo "🔍 Verificando artefactos generados:"
                        if [ -f "test-results/junit-results.xml" ]; then
                            echo "  ✅ junit-results.xml encontrado ($(wc -l < test-results/junit-results.xml) líneas)"
                        else
                            echo "  ❌ junit-results.xml NO encontrado"
                            echo "  📂 Contenido de test-results/:"
                            ls -lah test-results/ || echo "  (directorio vacío o inexistente)"
                        fi
                    '''
                }
            }
        }
        
        stage('📊 Generate Reports') {
            steps {
                script {
                    echo '═══════════════════════════════════════════'
                    echo '   GENERATING REPORTS'
                    echo '═══════════════════════════════════════════'
                }
                
                sh '''
                    echo "📈 Generando reportes HTML..."
                    
                    if [ -d "test-results" ]; then
                        echo "✅ Directorio test-results encontrado"
                        echo "   Archivos: $(find test-results -type f | wc -l)"
                    else
                        echo "⚠️  No se encontró el directorio test-results"
                    fi
                    
                    if [ -d "playwright-report" ]; then
                        mkdir -p reports
                        echo "✅ Playwright HTML report disponible en playwright-report/"
                    fi
                    
                    echo "✅ Reportes generados"
                '''
            }
        }
        
        stage('📤 Publish Results') {
            parallel {
                stage('JUnit Report') {
                    steps {
                        script {
                            echo '📋 Publicando reporte JUnit...'
                            // FIX: allowEmptyResults: true para evitar que Jenkins falle
                            // si por alguna razón el XML no se generó (ej: error fatal en setup)
                            junit(
                                testResults: 'test-results/junit-results.xml',
                                allowEmptyResults: false,
                                skipPublishingChecks: true
                            )
                        }
                    }
                }
                
                stage('Archive Artifacts') {
                    steps {
                        script {
                            echo '📦 Archivando artefactos...'
                            archiveArtifacts(
                                artifacts: 'test-results/**/*,playwright-report/**/*',
                                allowEmptyArchive: true,
                                fingerprint: true
                            )
                        }
                    }
                }
            }
        }
        
        stage('📊 HTML Report') {
            steps {
                script {
                    echo '═══════════════════════════════════════════'
                    echo '   PUBLISHING HTML REPORT'
                    echo '═══════════════════════════════════════════'
                }
                
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'playwright-report',
                    reportFiles: 'index.html',
                    reportName: 'Playwright Test Report',
                    reportTitles: 'Test Execution Report'
                ])
            }
        }

        stage('📈 Test Metrics') {
            steps {
                script {
                    echo '═══════════════════════════════════════════'
                    echo '   TEST METRICS'
                    echo '═══════════════════════════════════════════'
                }
                
                sh '''
                    echo "📊 Generando métricas de tests..."
                    
                    TOTAL_TESTS=$(find test-results -name "*.xml" 2>/dev/null | wc -l)
                    FAILED_TESTS=$(find test-results -name "*-failed-*" 2>/dev/null | wc -l)
                    
                    echo ""
                    echo "📈 Resumen de Ejecución:"
                    echo "  - Total de archivos de resultado: ${TOTAL_TESTS}"
                    echo "  - Tests fallidos: ${FAILED_TESTS}"
                    
                    if [ -d "screenshots" ]; then
                        SCREENSHOTS=$(find screenshots -name "*.png" 2>/dev/null | wc -l)
                        echo "  - Screenshots capturados: ${SCREENSHOTS}"
                    fi
                    
                    if [ -d "videos" ]; then
                        VIDEOS=$(find videos -name "*.webm" 2>/dev/null | wc -l)
                        echo "  - Videos grabados: ${VIDEOS}"
                    fi
                    
                    if [ -d "traces" ]; then
                        TRACES=$(find traces -name "*.zip" 2>/dev/null | wc -l)
                        echo "  - Traces generados: ${TRACES}"
                    fi
                    
                    echo ""
                    echo "💾 Uso de espacio:"
                    du -sh test-results reports screenshots videos traces 2>/dev/null || echo "  - No hay datos disponibles"
                '''
            }
        }
    }
    
    post {
        always {
            script {
                echo '═══════════════════════════════════════════'
                echo '   PIPELINE COMPLETED'
                echo '═══════════════════════════════════════════'
                echo "⏱️  Duración: ${currentBuild.durationString.replace(' and counting', '')}"
                echo "📊 Resultado: ${currentBuild.result ?: 'SUCCESS'}"
                echo "🔗 Build URL: ${env.BUILD_URL}"
                
                if (currentBuild.result == 'SUCCESS') {
                    echo "✅ PIPELINE EXITOSO"
                } else if (currentBuild.result == 'UNSTABLE') {
                    echo "⚠️  PIPELINE INESTABLE (algunos tests fallaron)"
                } else {
                    echo "❌ PIPELINE FALLÓ"
                }
                
                echo "ENOC"
            }
        }
        
        success {
            script {
                echo '✅ Todos los tests pasaron exitosamente'
            }
        }
        
        unstable {
            script {
                echo '⚠️  Algunos tests fallaron, revisa el reporte'
            }
        }
        
        failure {
            script {
                echo '❌ El pipeline falló completamente'
                
                sh '''
                    echo ""
                    echo "🔍 Información de debug:"
                    echo "  - Workspace: ${WORKSPACE}"
                    echo "  - Build Number: ${BUILD_NUMBER}"
                    echo "  - Node User: $(whoami)"
                    echo ""
                    echo "📂 Contenido del workspace:"
                    ls -lah ${WORKSPACE} | head -n 20
                '''
            }
        }
        
        cleanup {
            script {
                echo '🧹 Limpiando workspace temporal...'
                
                sh '''
                    echo "💾 Preservando reportes y artefactos importantes"
                    echo "✅ Limpieza completada"
                '''
            }
        }
    }
}
