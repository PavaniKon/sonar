pipeline {
    agent any

    environment {
        function_name = 'functionforjava'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Build'
                sh 'mvn clean package'
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
                script {
                    withSonarQubeEnv('Sonar') {
                        sh 'mvn sonar:sonar'
                    }
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
                    } catch (Exception ex) {
                        // Handle exception if needed
                    }
                }
            }
        }

        stage('Push') {
            steps {
                echo 'Push'
                sh "aws s3 cp target/sample-1.0.3.jar s3://bucket001306"
            }
        }

        stage('Deployments') {
            parallel {
                stage('Deploy to Dev') {
                    steps {
                        echo 'Deploy to Dev'
                        sh "aws lambda update-function-code --
