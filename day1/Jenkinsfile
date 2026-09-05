pipeline {
    agent {
        label 'dev'
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['sit', 'ppte', 'prod'],
            description: 'Select the environment'
        )
    }

    stages {

        stage('Terraform Init') {
            agent {
                docker {
                    image 'hashicorp/terraform:latest'
                    args '--entrypoint="" --user 0'
                    reuseNode true
                }
            }

            steps {
                echo "Selected Environment: ${params.ENVIRONMENT}"

                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        terraform version
                        terraform init
                    '''
                }
            }
        }

        stage('Terraform Plan') {
            agent {
                docker {
                    image 'hashicorp/terraform:latest'
                    args '--entrypoint="" --user 0'
                    reuseNode true
                }
            }

            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh 'terraform plan'
                }
            }
        }

        stage('Approval') {
            steps {
                input(
                    message: "Do you want to APPLY Terraform changes for ${params.ENVIRONMENT}?",
                    ok: 'Proceed'
                )
            }
        }

        stage('Terraform Apply') {
            agent {
                docker {
                    image 'hashicorp/terraform:latest'
                    args '--entrypoint="" --user 0'
                    reuseNode true
                }
            }

            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Terraform Output') {
            agent {
                docker {
                    image 'hashicorp/terraform:latest'
                    args '--entrypoint="" --user 0'
                    reuseNode true
                }
            }

            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh 'terraform output'
                }
            }
        }
    }
}
