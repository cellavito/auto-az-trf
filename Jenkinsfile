pipeline {
    agent any

    parameters {
        choice(name: 'TF_ACTION', choices: ['plan', 'apply', 'destroy'], description: 'Select the Terraform action to perform.')
    }

    environment {
        // Binds Jenkins Credentials securely to standard Azure Terraform environment variables
            ARM_CLIENT_ID       = credentials('AZURE_CLIENT_ID')
            ARM_CLIENT_SECRET   = credentials('AZURE_CLIENT_SECRET')
            ARM_SUBSCRIPTION_ID = credentials('AZURE_SUBSCRIPTION_ID')
            ARM_TENANT_ID       = credentials('AZURE_TENANT_ID')
        }
   
    stages {
        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Format & Validate') {
            steps {
                sh 'terraform fmt -check'
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }

        stage('Terraform Apply / Destroy') {
            when {
                expression { params.TF_ACTION == 'apply' || params.TF_ACTION == 'destroy' }
            }
            steps {
                script {
                    if (params.TF_ACTION == 'apply') {
                        echo "Applying infrastructure updates..."
                        sh 'terraform apply -input=false tfplan'
                    } else if (params.TF_ACTION == 'destroy') {
                        echo "Destroying provisioned infrastructure..."
                        sh 'terraform destroy -auto-approve'
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace workspace files like the sensitive plan binary
            cleanWs()
        }
    }
}

