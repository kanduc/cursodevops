pipeline {
    agent any

    environment {
        //DOCKER_HUB_LOGIN = credentials('docker-hub')
        VERSION = "3.0.0"
        REPO = "getting-started"
        REGISTRY = credentials('registry-hub')
    }

    stages {

        stage('Scaneo Datos Sensibles') {
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
                    }
                }
            }
        }

        stage('SonarQube'){
            environment {
                scannerHome = tool 'sonar-scanner'
            }
            steps {
                withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonarqube'){
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Compilando el código...'
                //sh "npm install"
            }
        }

        stage('Pruebas') {
            steps {
                echo 'Ejecutando pruebas...'
            }
        }

        stage('Despliegue') {
            agent {
                docker {
                    image 'jenkins-ansible:1.0.0'
                    args '--entrypoint="" -u root'
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

                        withCredentials([usernamePassword(
                            credentialsId: "username-webapp-node-tec",
                            usernameVariable: "username_webapp",
                            passwordVariable: "password_webapp")]){

                            sh "git init"
                            sh "git config --local user.email \"myapp@gmail.com\""
                            sh "git config --local user.name \"myapp\""
                            sh "git add *"
                            sh "git commit -m \"Initial commit\""
                            sh "git checkout -b master"
                            sh "git remote add azure https://\\${username_webapp}:${password_webapp}@myfirstwebappnodetec.scm.azurewebsites.net:443/myfirstwebappnodetec.git"
                            sh "git push -u azure master -f"
                        }
                    }
                }
            }
        }
             
    }
}