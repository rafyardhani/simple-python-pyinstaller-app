pipeline {
    agent any
    environment {
        DOCKER_IMAGE_BUILD = 'python:2-alpine'
        DOCKER_IMAGE_TEST = 'qnib/pytest'
        DOCKER_IMAGE_DEPLOY = 'python:2-slim'
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
                        echo "Compiling Python source files..."
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
                        echo "Executing pytest..."
                        py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py
                        '''
                    }
                }
                echo 'Publishing test results...'
                junit 'test-reports/results.xml'
            }
        }
        stage('Manual Approval') {
            steps {
                input message: 'Do you want to proceed to Deploy stage?', ok: 'Proceed'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying artifact...'
                script {
                    docker.image(env.DOCKER_IMAGE_DEPLOY).inside('-u root') {
                        sh '''
                        set -e
                        echo "Installing dependencies and PyInstaller..."
                        apt-get update && apt-get install -y build-essential libffi-dev
                        pip install pyinstaller==3.6
                        echo "Creating executable..."
                        pyinstaller --onefile sources/add2vals.py
                        '''
                    }
                }
                echo 'Archiving built artifacts...'
                archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true
                echo 'Transferring artifact to remote server...'
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'deployment-server',
                        transfers: [
                            sshTransfer(
                                cleanRemote: false,
                                sourceFiles: 'dist/add2vals',
                                remoteDirectory: 'dicoding-ci-cd'
                            ),
                            sshTransfer(
                                cleanRemote: false,
                                execCommand: 'cd /home/ubuntu/dicoding-ci-cd/dist && chmod +x add2vals && ./add2vals 2 5'
                            )
                        ],
                        verbose: true
                    )
                ])
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
