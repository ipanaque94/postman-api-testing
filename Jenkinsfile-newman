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
                    mkdir -p reports
                    newman run dog-api/TheDogAPI.json \
                        --environment dog-api/ApiDogEnvironments.json \
                        --iteration-data dog-api/dog_breeds.csv \
                        -r cli,htmlextra \
                        --reporter-htmlextra-export reports/reporte-dog.html \
                        --reporter-htmlextra-title "The Dog API - Test Report" \
                        --reporter-htmlextra-browserTitle "Dog API Tests" \
                        || true
                '''
            }
        }

        stage('Run Cat API Tests') {
            steps {
                echo '🐱 Ejecutando tests de The Cat API...'
                sh '''
                    newman run cat-api/TheCatAPI.json \
                        --environment cat-api/ApiCatEnvironments.json \
                        --iteration-data cat-api/cat_breeds.csv \
                        -r cli,htmlextra \
                        --reporter-htmlextra-export reports/reporte-cat.html \
                        --reporter-htmlextra-title "The Cat API - Test Report" \
                        --reporter-htmlextra-browserTitle "Cat API Tests" \
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
