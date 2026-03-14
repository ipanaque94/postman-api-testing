pipeline {
    agent any

    options {
        timeout(time: 1, unit: 'HOURS')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Setup') {
            steps {
                echo '📦 Instalando Newman y reporters...'
                sh '''
                    export npm_config_prefix=/var/jenkins_home/.npm-global
                    export PATH=/var/jenkins_home/.npm-global/bin:$PATH
                    npm install -g newman
                    npm install -g newman-reporter-htmlextra
                    echo "✅ Newman version: $(newman --version)"
                '''
            }
        }

        stage('Run Dog API Tests') {
            steps {
                echo '🐶 Ejecutando tests de The Dog API...'
                sh '''
                    export PATH=/var/jenkins_home/.npm-global/bin:$PATH
                    mkdir -p reports
                    newman run dog-api/TheDogAPI.json \
                        --environment dog-api/ApiDogEnvironments.json \
                        --iteration-data dog-api/dog_breeds.csv \
                        -r cli,htmlextra,junit \
                        --reporter-htmlextra-export reports/reporte-dog.html \
                        --reporter-htmlextra-title "The Dog API - Test Report" \
                        --reporter-htmlextra-browserTitle "Dog API Tests" \
                        --reporter-junit-export reports/junit-dog.xml \
                        || true
                '''
            }
        }

        stage('Run Cat API Tests') {
            steps {
                echo '🐱 Ejecutando tests de The Cat API...'
                sh '''
                    export PATH=/var/jenkins_home/.npm-global/bin:$PATH
                    newman run cat-api/TheCatAPI.json \
                        --environment cat-api/ApiCatEnvironments.json \
                        --iteration-data cat-api/cat_breeds.csv \
                        -r cli,htmlextra,junit \
                        --reporter-htmlextra-export reports/reporte-cat.html \
                        --reporter-htmlextra-title "The Cat API - Test Report" \
                        --reporter-htmlextra-browserTitle "Cat API Tests" \
                        --reporter-junit-export reports/junit-cat.xml \
                        || true
                '''
            }
        }

        stage('Publish Reports') {
            steps {
                echo '📊 Publicando reportes HTML...'
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'reporte-dog.html',
                    reportName: 'Dog API Test Report'
                ])
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'reporte-cat.html',
                    reportName: 'Cat API Test Report'
                ])

                echo '📋 Publicando resultados JUnit...'
                junit(
                    testResults: 'reports/junit-dog.xml,reports/junit-cat.xml',
                    allowEmptyResults: true,
                    skipPublishingChecks: true
                )
            }
        }
    }

    post {
        always {
            echo '🏁 Pipeline finalizado'
            echo "⏱️  Duración: ${currentBuild.durationString}"
            echo "📊 Resultado: ${currentBuild.result ?: 'SUCCESS'}"
        }
        success {
            echo '✅ Todos los tests pasaron correctamente'
        }
        failure {
            echo '❌ Algunos tests fallaron - revisar reportes'
        }
    }
}
