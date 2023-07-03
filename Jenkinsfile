pipeline {
    agent { label 'master'}

    environment {
        function_name = 'java-sample'

    }

    parameters {
        string(name: 'RollbackVersion', description: 'Please enter rollback version')
        choice(
            choices: ['Dev', 'Test', 'Prod'],
            name: 'Environment',
            description : 'Please select environment'
        )
    }

    stages {

        // CI Start
        stage('Build') {
            steps {
                echo 'Build'
                sh 'mvn package'
            }
        }


        stage("SonarQube analysis") {
            agent any

            when {
                anyOf {
                    branch 'feature/*'
                    branch 'main'
                }
            }
            steps {
                withSonarQubeEnv('Sonar') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    try {
                        timeout(time: 10, unit: 'MINUTES') {
                            waitForQualityGate abortPipeline: true
                        }
                    }
                    catch (Exception ex) {

                    }
                }
            }
        }

        stage('Push') {
            steps {
                echo 'Push'

                sh "aws s3 cp target/sample-1.0.3.jar s3://bermtecbatch31"
            }
        }

        // Ci Ended

        // CD Started

        stage('Deployments') {
            parallel {

                stage('Deploy to Dev') {
                    steps {
                        echo 'Build'

                        sh "aws lambda update-function-code --function-name $function_name --region us-east-1 --s3-bucket bermtecbatch31 --s3-key sample-1.0.3.jar"
                    }
                }

                stage('Deploy to test ') {
                    steps {
                        echo 'Build'

                        function_name = 'java-test'

                        sh "aws lambda update-function-code --function-name $function_name --region us-east-1 --s3-bucket bermtecbatch31 --s3-key sample-1.0.3.jar"
                    }
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                expression { return params.Environment == 'Prod'}
            }
            steps {
               input (
                    message: 'Are we good for Prod Deployment ?'
               )
            }
        }

        stage('Release to Prod') {
            when {
                branch 'main'
            }
            steps {
                function_name = 'java-prod'
                sh "aws lambda update-function-code --function-name $function_name --region us-east-1 --s3-bucket bermtecbatch31 --s3-key sample-1.0.3.jar"
            }
        }


        

        // CD Ended
    }

    post {
        always {
            echo "${env.BUILD_ID}"
            echo "${BRANCH_NAME}"
            echo "${JENKINS_URL}"

        }

        failure {
            echo 'failed'
            mail ()
        }
        aborted {
            echo 'aborted'
        }
    }
}
