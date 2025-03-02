pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1' 
    }
    stages {
        stage('Set AWS Credentials') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS_SECRET_ACCESS_KEY' 
                ]]) {
                    sh '''
                    echo "AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID"
                    aws sts get-caller-identity
                    '''
                }
            }
        }
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/derrickSh43/autoScale' 
            }
        }
        stage('Initialize Terraform') {
            steps {
                sh '''
                terraform init
                '''
            }
        }
        stage('Plan Terraform') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    terraform plan -out=tfplan
                    '''
                }
            }
        }
        stage('Apply Terraform') {
            steps {
                input message: "Approve Terraform Apply?", ok: "Deploy"
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    terraform apply -auto-approve tfplan
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Terraform deployment completed successfully!'
        }
        failure {
            echo 'Terraform deployment failed!'
        }
    }
}
pipeline {
    agent any

    tools {
        // This assumes that you've configured a tool installation for JFrog CLI named 'jfrog-cli-01'
        jfrog 'jfrog-cli'
    }

    stages {
        stage('Testing') {
            steps {
                // Check JFrog CLI version
                jf '-v'
                
                // Show configured JFrog CLI settings
                jf 'c show'
                
                // Ping Artifactory via JFrog CLI
                jf 'rt ping'
                
                // Create a test file
                sh 'touch test-file'
                
                // Upload the test file to the repository or path defined by your tool config
                jf 'rt u test-file jfrog-cli/'
                
                // Build promotion (if applicable) via JFrog CLI
                jf 'rt bp'
                
                // Download the test file from Artifactory
                jf 'rt dl jfrog-cli/test-file'
            }
        }
    }
}

