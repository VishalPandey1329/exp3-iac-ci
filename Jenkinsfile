
pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
    }

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
        TF_IN_AUTOMATION = 'true'
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                bat 'terraform fmt -check -recursive -diff'
                bat 'terraform init -input=false'
                bat 'terraform validate'
            }
        }

        stage('Security Scan') {
            steps {
                bat 'tflint --init'
                bat 'tflint --format compact'
                bat 'tfsec . --minimum-severity HIGH'
            }
        }

        stage('Plan') {
            steps {
                bat 'terraform plan -input=false -out=tfplan'
                bat 'terraform show -no-color tfplan > tfplan.txt'
                archiveArtifacts artifacts: 'tfplan, tfplan.txt', fingerprint: true
            }
        }

        stage('Approval') {
            when {
                branch 'main'
            }
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    input message: 'Review the Terraform plan. Approve applying it to AWS?', ok: 'Apply'
                }
            }
        }

        stage('Apply') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Final confirmation: allow Terraform to apply changes to AWS?', ok: 'Proceed'
                bat 'terraform apply -input=false tfplan'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Inspect the stage that failed.'
        }
        always {
            echo 'Review the build log and clean up temporary files when safe.'
        }
    }
}
