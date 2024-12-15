node {
    try {
        // Environment variables
        def DOCKER_IMAGE_BUILD = 'python:2-alpine'
        def DOCKER_IMAGE_TEST = 'qnib/pytest'

        stage('Checkout') {
            echo 'Checking out source code...'
            checkout scm
        }

        stage('Build') {
            echo 'Building source files...'
            docker.image(DOCKER_IMAGE_BUILD).inside {
                sh '''
                set -e
                python -m py_compile sources/add2vals.py sources/calc.py
                '''
            }
        }

        stage('Test') {
            echo 'Running tests...'
            docker.image(DOCKER_IMAGE_TEST).inside {
                sh '''
                set -e
                py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py
                '''
            }
            echo 'Publishing test results...'
            junit 'test-reports/results.xml'
        }
        
        stage('Manual Approval') {
            input "Lanjutkan ke tahap Deploy?"
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        echo "Pipeline failed: ${e.getMessage()}"
    } finally {
        stage('Cleanup') {
            echo 'Cleaning up workspace...'
            cleanWs()
        }

        if (currentBuild.result == 'SUCCESS') {
            echo 'Pipeline completed successfully!'
        } else {
            echo 'Pipeline failed! Check logs for more details.'
        }
    }
}
