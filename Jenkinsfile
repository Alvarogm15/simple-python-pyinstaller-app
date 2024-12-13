pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:3.12.0-alpine3.18'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'  // Monta el socket Docker para usar Docker dentro de contenedor
                }
            }
            steps {
                script {
                    // Asegúrate de que las rutas son correctas y 'sources' existe en el repositorio
                    sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                    stash(name: 'compiled-results', includes: 'sources/*.py*')
                }
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'  // Monta el socket Docker también en este contenedor
                }
            }
            steps {
                script {
                    sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
                }
            }
            post {
                always {
                    junit 'test-reports/results.xml'  // Archiva los resultados de pruebas siempre
                }
            }
        }
        stage('Deliver') {
            agent any
            environment {
                VOLUME = '${WORKSPACE}/sources:/src'  // Utiliza WORKSPACE en lugar de $(pwd)
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: 'sources') {
                    unstash(name: 'compiled-results')  // Deshaz el 'stash' de los archivos compilados
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} pyinstaller -F add2vals.py"  // Usa la ruta correcta
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: "${env.WORKSPACE}/sources/dist/add2vals", allowEmptyArchive: true
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} rm -rf build dist"  // Limpia los directorios generados por PyInstaller
                }
            }
        }
    }
}
