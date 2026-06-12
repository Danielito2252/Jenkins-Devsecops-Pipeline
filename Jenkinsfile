pipeline {
    agent any

    environment {
        // Usamos un sitio de prueba legal y seguro de OWASP para el escaneo dinámico
        SCAN_TARGET = "https://juice-shop.herokuapp.com" 
    }

    stages {
        // Etapa 1: Clonado automático del repositorio del pipeline [cite: 13, 27]
        stage('1. Checkout') {
            steps {
                echo 'Clonando el repositorio del proyecto...'
                checkout scm
            }
        }

        // Etapa 2: SCA (Software Composition Analysis) [cite: 51]
        stage('2. Security Scan - SCA') {
            steps {
                echo 'Ejecutando análisis de vulnerabilidades en dependencias (SCA)...'
                sh 'npm audit || true' 
            }
        }

        // Etapa 3: SAST (Static Application Security Testing) con Trivy [cite: 51]
        stage('3. Security Scan - SAST') {
            steps {
                echo 'Ejecutando análisis de vulnerabilidades del código con Trivy...'
                sh '''
                    docker run --rm \
                    -v ${WORKSPACE}:/apps \
                    aquasec/trivy:latest fs /apps \
                    --format json \
                    --output /apps/trivy-report.json || true
                '''
                echo 'Escaneo de Trivy finalizado. Reporte guardado en trivy-report.json.'
            }
        }

        // Etapa 4: Security Gate (El Guardián) [cite: 52]
        stage('4. Security Gate') {
            steps {
                echo 'Evaluando criterios de seguridad (Security Gate)...'
                script {
                    if (!fileExists('trivy-report.json')) {
                        error "Fallo del Security Gate: No se encontró el reporte de seguridad."
                    }
                    
                    def trivyReport = readJSON file: 'trivy-report.json'
                    def vulnerabilidadesCriticas = 0
                    def vulnerabilidadesAltas = 0
                    
                    if (trivyReport.Results) {
                        for (result in trivyReport.Results) {
                            if (result.Vulnerabilities) {
                                for (vuln in result.Vulnerabilities) {
                                    if (vuln.Severity == 'CRITICAL') {
                                        vulnerabilidadesCriticas++
                                    } else if (vuln.Severity == 'HIGH') {
                                        vulnerabilidadesAltas++
                                    }
                                }
                            }
                        }
                    }
                    
                    echo "--- RESUMEN DE SEGURIDAD ---"
                    echo "Vulnerabilidades Críticas encontradas: ${vulnerabilidadesCriticas}"
                    echo "Vulnerabilidades Altas encontradas: ${vulnerabilidadesAltas}"
                    echo "----------------------------"
                    
                    if (vulnerabilidadesCriticas > 0 || vulnerabilidadesAltas > 0) {
                        currentBuild.result = 'FAILURE'
                        error "El Security Gate ha FALLADO debido a riesgos de seguridad altos o críticos. Deteniendo despliegue."
                    } else {
                        echo "¡Security Gate APROBADO! No se encontraron riesgos críticos ni altos."
                    }
                }
            }
        }

        // CONEXIÓN ENTRE REPOS: Descargamos tu proyecto estrella aquí [cite: 27, 34, 35]
        stage('Descargar Cypress Suite') {
            steps {
                echo 'Descargando suite de pruebas desde el repositorio estrella de Cypress...'
                dir('cypress-tests') {
                    // RECUERDA: Cambia TU_USUARIO por tu nombre de usuario real de GitHub
                    git url: 'https://github.com/TU_USUARIO/cypress-e2e-suite.git', branch: 'main'
                }
            }
        }

        // Etapa 5: Security Scan - DAST con OWASP ZAP [cite: 51]
        stage('5. Security Scan - DAST') {
            steps {
                echo "Iniciando análisis dinámico de seguridad (DAST) contra: ${SCAN_TARGET}"
                sh '''
                    docker run --rm \
                    -v ${WORKSPACE}:/zap/wrk/:rw \
                    owasp/zap2docker-stable zap-baseline.py \
                    -t ${SCAN_TARGET} \
                    -r zap-report.html || true
                '''
                echo 'Escaneo DAST finalizado. El reporte se guardó como zap-report.html.'
            }
        }
    }

    post {
        always {
            echo 'Limpiando el espacio de trabajo...'
            cleanWs()
        }
        success {
            echo '¡Pipeline ejecutado con éxito! Estado: SEGURO.'
        }
        failure {
            echo 'El pipeline ha fallado. Revisar los reportes de seguridad inmediatamente.'
        }
    }
}