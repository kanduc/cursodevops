pipeline {
    agent any

    environment {
        //DOCKER_HUB_LOGIN = credentials('docker-hub')
        VERSION = "3.0.0"
        REPO = "getting-started"
        REGISTRY = credentials('registry-hub')
    }

    stages {

        /*stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root:root'
                }
            }
            steps {
                script {
                    sh 'npm install'
                    //sh 'npm audit'
                }
            }
        }*/

        stage('Gitleaks-Scan') {
            agent {
                docker {
                    image 'zricethezav/gitleaks'
                    args '--entrypoint="" -u root -v ${WORKSPACE}:/src'
                }
            }                    
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script {
                        sh "gitleaks detect --verbose --source . -f json -r report_gitleaks.json"
                        sh "ls -la"
                        archiveArtifacts artifacts: "report_gitleaks.json"
                        //stash includes: 'report_gitleaks.json', name: 'report_gitleaks.json'
                    }
                }
            }
        }
        /*
        stage('NPMAudit-Scan') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root:root'
                }
            }                    
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script {
                        sh "npm audit -audit-level=moderate --json > /src/report_npmaudit.json"
                        sh "ls -la"
                        archiveArtifacts artifacts: "report_npmaudit.json"
                        //stash includes: 'report_npmaudit.json', name: 'report_npmaudit.json'
                    }
                }
            }
        }*/

        /*
        stage('SonarQube'){
            environment {
                scannerHome = tool 'sonar-scanner'
            }
            steps {
                withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonarqube'){
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }

        }*/

        stage('Build') {
            steps {
                echo 'Compilando el código...'
                sh "docker build -t $REGISTRY/$REPO:$VERSION ."
            }
        }

        stage('Trivy-Scan') {
            agent {
                docker {
                    image 'aquasec/trivy:0.48.1'
                    args '--entrypoint="" -u root -v /var/run/docker.sock:/var/run/docker.sock -v ${WORKSPACE}:/src'
                }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script {
                        sh "trivy image --format json --output report_trivy.json $REGISTRY/$REPO:$VERSION"
                        archiveArtifacts artifacts: "report_trivy.json"
                    }
                }
            }
        }

        stage('Pruebas') {
            steps {
                echo 'Ejecutando pruebas...'
            }
        }

        stage('Despliegue') {
            steps {
                echo 'Desplegando la aplicación...'
                echo 'Docker push'
 
                withCredentials([usernamePassword(
                    credentialsId:"docker-hub-user", 
                    usernameVariable: "DOCKER_USERNAME", 
                    passwordVariable: "DOCKER_PASSWORD")]){
 
                    sh '''
                        docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                        docker push ${DOCKER_HUB_REGISTRY}/${IMAGE_NAME}:${VERSION_TAG}
                    '''
                }
            }
        }

        /*
        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t $REGISTRY/$REPO:$VERSION ."
                }
            }
        }*/

        /*
        stage('Trivy-Scan') {
            agent {
                docker {
                    image 'aquasec/trivy:0.48.1'
                    args '--entrypoint="" -u root -v /var/run/docker.sock:/var/run/docker.sock -v ${WORKSPACE}:/src'
                }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script {
                        sh "trivy image --format json --output /src/report_trivy.json $REGISTRY/$REPO:$VERSION"
                        stash includes: 'report_trivy.json', name: 'report_trivy.json'
                    }
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    echo "hola"
                    //sh '''
                    //    docker login -u $DOCKER_HUB_LOGIN_USR -p $DOCKER_HUB_LOGIN_PSW
                    //    docker push $REGISTRY/$REPO:$VERSION
                    //'''
                }
            }
        }   
        */               
    }
}