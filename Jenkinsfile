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
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root:root -v ${WORKSPACE}:/src'
                }
            }
            steps {
                echo 'Compilando el código...'
                //sh "docker build -t $REGISTRY/$REPO:$VERSION ."
                sh "npm install "
                //sh "node build"
                //sh "node src/index.js"
                sh "ls -la"

            }
        }

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
                        sh "trivy image --format json --output report_trivy.json $REGISTRY/$REPO:$VERSION"
                        archiveArtifacts artifacts: "report_trivy.json"
                    }
                }
            }
        }*/

        stage('Pruebas') {
            steps {
                echo 'Ejecutando pruebas...'
            }
        }

        stage('Despliegue') {
            agent {
                docker {
                    image 'jenkins-ansible:1.0.0'
                    args '--entrypoint="" -u root -v ${WORKSPACE}:/src'
                }
            } 
            steps {
                echo 'Desplegando la aplicación...'
                script {
                    withCredentials([azureServicePrincipal('sp-iac-azure')]){
                        echo "Iniciando sesion Azure"

                        sh "az account clear"
                        sh "az login --service-principal --username ${AZURE_CLIENT_ID} --password ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}"
                        sh "az account set --subscription ${AZURE_SUBSCRIPTION_ID}"

                        //sh "az webapp update --resource-group myResourceGroupAppNode --name myfirstWebAppNode"
                        withCredentials([usernamePassword(
                            credentialsId:"user-deploy-webapp", 
                            usernameVariable: "WEBAPP_USERNAME", 
                            passwordVariable: "WEBAPP_PASSWORD")]){
                            sh "git init"
                            sh "git remote add azure https://\\${WEBAPP_USERNAME}:${WEBAPP_PASSWORD}@myfirstwebappnode.scm.azurewebsites.net:443"
                            sh "git config --local user.email \"myapp@example.com\""
                            sh "git config --local user.name \"myapp\""
                            sh "git add *"
                            sh "git commit -m \"Initial commit\""
                            sh "git push azure master -f"

                        }
                    }
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