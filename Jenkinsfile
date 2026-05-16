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
                        sh 'echo $WORKSPACE'

                        sh '''
                        export PYTHONPATH=$WORKSPACE

                        pytest test/unit \
                        --junitxml=result-unit.xml
                        '''
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

                        sh '''
                        export PYTHONPATH=$WORKSPACE

                        echo "Starting Flask API..."

                        nohup python3 app/api.py > flask.log 2>&1 &

                        sleep 5

                        echo "Starting WireMock..."

                        cd wiremock

                        nohup java -jar wiremock.jar \
                        --port 9090 \
                        > wiremock.log 2>&1 &

                        cd ..

                        sleep 10

                        echo "Running REST tests..."

                        pytest test/rest \
                        --junitxml=result-rest.xml
                        '''
                    }

                    post {
                        always {
                            junit 'result-rest.xml'

                            sh '''
                            pkill -f "python3 app/api.py" || true
                            pkill -f "wiremock.jar" || true
                            '''
                        }
                    }
                }

                stage('Static') {
                    agent { label 'python-agent' }

                    steps {
                        sh 'whoami'
                        sh 'hostname'
                        sh 'echo $WORKSPACE'

                        sh '''
                        export PYTHONPATH=$WORKSPACE

                        flake8 app \
                        --exit-zero \
                        --format=pylint \
                        > flake8-report.txt
                        '''
                    }
                }

                stage('Security') {
                    agent { label 'python-agent' }

                    steps {
                        sh 'whoami'
                        sh 'hostname'
                        sh 'echo $WORKSPACE'

                        sh '''
                        export PYTHONPATH=$WORKSPACE

                        bandit -r app \
                        -f txt \
                        -o bandit-report.txt || true
                        '''
                    }
                }

                stage('Coverage') {
                    agent { label 'python-agent' }

                    steps {
                        sh 'whoami'
                        sh 'hostname'
                        sh 'echo $WORKSPACE'

                        sh '''
                        export PYTHONPATH=$WORKSPACE

                        coverage run --branch \
                        -m pytest test/unit

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

                        sh '''
                        jmeter -n \
                        -t performance/test.jmx \
                        -l result.jtl
                        '''
                    }

                    post {
                        always {
                            perfReport sourceDataFiles: 'result.jtl'
                        }
                    }
                }
            }
        }
    }
}
