pipeline {
    agent any

    environment {
        py = "python"

//        usr = "Yan"
//        crs_dir = "C:\\Users\\${usr}\\Desktop\\dev-ops-course"
        crs_dir = '%USERPROFILE%\\Desktop\\dev-ops-course'
        env_dir = "${crs_dir}\\env"
        py_dir = "${crs_dir}\\py"

        pkgs_dir = "${py_dir}\\venv\\Lib\\site-packages"
    }

    stages {
        stage("Stage-1: Handle Git") {
            steps {
                script {
                    properties([
                        buildDiscarder(logRotator(
                                        artifactDaysToKeepStr: '',
                                        artifactNumToKeepStr: '',
                                        daysToKeepStr: '5',
                                        numToKeepStr: '1')),
//                                        numToKeepStr: '20')),

                        pipelineTriggers([pollSCM('30 * * * *')])
                    ])
                }

                git 'https://github.com/yan-nagler-sw/proj-3.git'
            }
        }

        stage("Stage-2: Handle prerequisites") {
            steps {
                echo "Running Python script: backend_testing_db.py..."
                bat '''
                    set PYTHONPATH=%PYTHONPATH%;${pkgs_dir}
                    ${py} backend_testing_db.py
                '''

                echo "Copying Selenium WebDriver - Chrome..."
                sh '''
                    cp ${env_dir}/chromedriver .
                '''
            }
        }

        stage("Stage-3: Launch REST server") {
            steps {
                echo "Running Python script: rest_app.py..."
                sh '''
                    set PYTHONPATH=%PYTHONPATH%;${pkgs_dir}
                    start /min ${py} rest_app.py
                '''
            }
        }

        stage("Stage-4: Launch BE test") {
            steps {
                echo "Running Python script: backend_testing.py..."
                sh '''
                    set PYTHONPATH=%PYTHONPATH%;${pkgs_dir}
                    ${py} backend_testing.py
                '''
            }
        }

        stage("Stage-5: Clean environment") {
            steps {
                echo "Running Python script: clean_environment.py..."
                sh '''
                    set PYTHONPATH=%PYTHONPATH%;${pkgs_dir}
                    ${py} clean_environment.py
                '''
            }
        }
    }

    post {
        always {
            echo "post - always"
        }
        success {
            echo "post - success"
/*
            mail to: 'yan.nagler@gmail.com',
                 subject: "post - success: ${currentBuild.fullDisplayName}",
                 body: "post - success: ${env.BUILD_URL}"
*/
        }
        failure {
            echo "post - failure"
/*
            mail to: 'yan.nagler@gmail.com',
                 subject: "post - failure: ${currentBuild.fullDisplayName}",
                 body: "post - failure: ${env.BUILD_URL}"
*/
        }
        unstable {
            echo "post - unstable"
        }
        changed {
            echo "post - changed"
        }
    }
}
