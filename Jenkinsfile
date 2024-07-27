pipeline {
    agent any

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
                }
            }
        }*/

        //stage('Security SAST') {
         //   parallel {
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
                                //archiveArtifacts artifacts: "report_gitleaks.json"
                                stash includes: 'report_gitleaks.json', name: 'report_gitleaks.json'
                            }
                        }
                    }
                }
                stage('NPMAudit-Scan') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            args '-u root:root -v ${WORKSPACE}:/src'
                        }
                    }                    
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            script {
                                sh "npm audit --registry=https://registry.npmjs.org -audit-level=moderate --json > report_npmaudit.json"
                                //archiveArtifacts artifacts: "report_npmaudit.json"
                                stash includes: 'report_npmaudit.json', name: 'report_npmaudit.json'
                            }
                        }
                    }
                }
                stage('Semgrep-Scan') {
                    agent {
                        docker {
                            image 'returntocorp/semgrep'
                            args '-u root:root -v ${WORKSPACE}:/src'
                        }
                    }                     
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            script {
                                sh "semgrep ci --json --exclude=package-lock.json --output report_semgrep.json --config auto --config p/ci"
                                stash includes: 'report_semgrep.json', name: 'report_semgrep.json'
                                //archiveArtifacts artifacts: "/src/report_semgrep.json"
                            }
                        }
                    }
                }    
                /*
                stage('Snyk-Code-Scan'){
                    agent {
                        docker {
                            image 'snyk/snyk:node'
                            args '--entrypoint="" -e SNYK_TOKEN=$SNYK_CREDENTIALS -u root:root -v ${WORKSPACE}:/src'
                        }
                    }                     
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            script {
                                sh "snyk test --json --file=package.json --severity-threshold=high --print-deps --print-deps-uses --print-vulnerabilities --print-trace --print-all-environment --json-file-output=report_snyk.json"
                                stash includes: 'report_snyk.json', name: 'report_snyk.json'
                            }
                        }
                    }
                }*/
                stage('Horusec-Scan') {                   
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            script {
                                sh ''' 
                                    docker run --rm \
                                        -v /var/run/docker.sock:/var/run/docker.sock \
                                        -v $(pwd):/src \
                                        horuszup/horusec-cli:v2.9.0-beta.3 \
                                        horusec start \
                                        -p /src \
                                        -P "$(pwd)/src" \
                                        -e="true" \
                                        -o="json" \
                                        -O=report_horusec.json || true
                                '''
                                stash includes: 'report_horusec.json', name: 'report_horusec.json'
                                //archiveArtifacts artifacts: "/src/report_horusec.json"
                            }
                        }
                    }
                }                                           
            //}
        //}

        stage('Build') {
            steps {
                echo 'Compilando el código...'
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
            }
        }
    }
}