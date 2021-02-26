pipeline {
    agent any

    parameters {
        choice(
            name: "test_type",
            choices: ['3', '1', '2'],
            description: ''
        )
    }

    environment {
        usr = "yannagler"
        crs_dir = "/Users/${usr}/Desktop/dev-ops-course"
        env_dir = "${crs_dir}/env"
        py_dir = "${crs_dir}/py"

        py = "python3.9"
        pkgs_dir = "${py_dir}/venv/lib/${py}/site-packages"
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
                                        numToKeepStr: '20')),
                                        
                        pipelineTriggers([pollSCM('30 * * * *')])
                    ])
                }

                git 'https://github.com/yan-nagler-sw/proj-2.git'
            }
        }

        stage("Stage-2: Handle prerequisites") {
            steps {
                echo "test_type: ${params.test_type}"

                echo "Running Python script: backend_testing_db.py..."
                sh '''
                    export PYTHONPATH="${pkgs_dir}:\\$PYTHONPATH"
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
                    export PYTHONPATH="${pkgs_dir}:\\$PYTHONPATH"
                    nohup $py rest_app.py &
                '''
            }
        }

        stage("Stage-4: Launch Web server") {
            steps {
                echo "Running Python script: web_app.py..."
                sh '''
                    export PYTHONPATH="${pkgs_dir}:\\$PYTHONPATH"
                    nohup ${py} web_app.py &
                '''
            }
        }

        stage("Stage-5: Launch BE test") {
            when {
                expression {
                    params.test_type == '2'
                }
            }

            steps {
                echo "test_type: ${params.test_type}"

                echo "Running Python script: backend_testing.py..."
                sh '''
                    export PYTHONPATH="${pkgs_dir}:\\$PYTHONPATH"
                    ${py} backend_testing.py
                '''
            }
        }

        stage("Stage-6: Launch FE test") {
            when {
                expression {
                    params.test_type == '1'
                }
            }

            steps {
                echo "test_type: ${params.test_type}"

                echo "Running Python script: frontend_testing.py..."
                sh '''
                    export PYTHONPATH="${pkgs_dir}:\\$PYTHONPATH"
                    ${py} frontend_testing.py
                '''
            }
        }

        stage("Stage-7: Launch combined test") {
            when {
                expression {
                    params.test_type == '3'
                }
            }

            steps {
                echo "test_type: ${params.test_type}"

                echo "Running Python script: combined_testing.py..."
                sh '''
                    export PYTHONPATH="${pkgs_dir}:\\$PYTHONPATH"
                    ${py} combined_testing.py
                '''
            }
        }

        stage("Stage-8: Clean environment") {
            steps {
                echo "Running Python script: clean_environment.py..."
                sh '''
                    export PYTHONPATH="${pkgs_dir}:\\$PYTHONPATH"
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

            mail to: 'yan.nagler@gmail.com',
                 subject: "post - success: ${currentBuild.fullDisplayName}",
                 body: "post - success: ${env.BUILD_URL}"
        }
        failure {
            echo "post - failure"

            mail to: 'yan.nagler@gmail.com',
                 subject: "post - failure: ${currentBuild.fullDisplayName}",
                 body: "post - failure: ${env.BUILD_URL}"
        }
        unstable {
            echo "post - unstable"
        }
        changed {
            echo "post - changed"
        }
    }
}
