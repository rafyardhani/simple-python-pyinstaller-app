pipeline {
    agent any
    environment {
        DOCKER_IMAGE_BUILD = 'python:2-alpine'
        DOCKER_IMAGE_TEST = 'qnib/pytest'
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Building source files...'
                script {
                    docker.image(env.DOCKER_IMAGE_BUILD).inside {
                        sh '''
                        set -e
                        python -m py_compile sources/add2vals.py sources/calc.py
                        '''
                    }
                }
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                script {
                    docker.image(env.DOCKER_IMAGE_TEST).inside {
                        sh '''
                        set -e
                        py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py
                        '''
                    }
                }
                echo 'Publishing test results...'
                junit 'test-reports/results.xml'
            }
        }
    }
    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check logs for more details.'
        }
    }
}
