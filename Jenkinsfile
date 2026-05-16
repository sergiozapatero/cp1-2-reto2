pipeline {
    agent none

    stages {

        stage('Get Code') {
            agent any

            steps {
                sh 'whoami'
                sh 'hostname'
                sh 'echo $WORKSPACE'

                git 'https://github.com/sergiozapatero/cp1-2-reto2.git'
                sh 'ls -l'
            }
        }

        stage('Parallel Tests') {

            parallel {

                stage('Unit') {
                    agent { label 'python-agent' }

                    steps {
                        sh 'whoami'
                        sh 'hostname'
			sh 'export PYTHONPATH=$WORKSPACE'
                        sh 'echo $WORKSPACE'
                        sh 'pytest test/unit --junitxml=result-unit.xml'
                    }

                    post {
                        always {
                            junit 'result-unit.xml'
                        }
                    }
                }

                stage('Rest') {
                    agent { label 'python-agent' }

                    steps {
                        sh 'whoami'
                        sh 'hostname'
                        sh 'echo $WORKSPACE'

                        sh 'pytest test/rest --junitxml=result-rest.xml'
                    }

                    post {
                        always {
                            junit 'result-rest.xml'
                        }
                    }
                }

                stage('Static') {
                    agent { label 'python-agent' }

                    steps {
                        sh 'whoami'
                        sh 'hostname'
                        sh 'echo $WORKSPACE'

                        sh 'flake8 app --exit-zero --format=pylint > flake8-report.txt'
                    }
                }

                stage('Security') {
                    agent { label 'python-agent' }

                    steps {
                        sh 'whoami'
                        sh 'hostname'
                        sh 'echo $WORKSPACE'

                        sh 'bandit -r app -f txt -o bandit-report.txt || true'
                    }
                }

                stage('Coverage') {
                    agent { label 'python-agent' }

                    steps {
                        sh 'whoami'
                        sh 'hostname'
                        sh 'echo $WORKSPACE'

                        sh '''
                        coverage run --branch -m pytest test/unit
                        coverage xml -o coverage.xml
                        coverage report
                        '''
                    }
                }

                stage('Performance') {
                    agent { label 'perf-agent' }

                    steps {
                        sh 'whoami'
                        sh 'hostname'
                        sh 'echo $WORKSPACE'

                        sh 'jmeter -n -t performance/test.jmx -l result.jtl'
                    }
                }
            }
        }
    }

post {
    always {
        any node {
            script {
                if (fileExists('result.jtl')) {
                    perfReport sourceDataFiles: 'result.jtl'
                } else {
                    echo "Skipping perfReport"
                }
            }
        }
    }
}
}
