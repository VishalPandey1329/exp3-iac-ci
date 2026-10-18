pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
    }

    parameters {
        booleanParam(
            name: 'APPLY_INFRASTRUCTURE',
            defaultValue: false,
            description: 'Use AWS credentials for a refreshed plan and enable the separately approved apply stages.'
        )
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
                script {
                    if (params.APPLY_INFRASTRUCTURE) {
                        bat 'terraform plan -input=false -var=ci_plan_mode=false -out=tfplan'
                    } else {
                        withEnv(['AWS_ACCESS_KEY_ID=ci-plan-placeholder', 'AWS_SECRET_ACCESS_KEY=ci-plan-placeholder', 'AWS_EC2_METADATA_DISABLED=true']) {
                            bat 'terraform plan -input=false -refresh=false -var=ci_plan_mode=true -out=tfplan'
                        }
                    }
                }
                bat 'terraform show -no-color tfplan > tfplan.txt'
                archiveArtifacts artifacts: 'tfplan, tfplan.txt', fingerprint: true
            }
        }

        stage('Approval') {
            when {
                allOf {
                    branch 'main'
                    expression { params.APPLY_INFRASTRUCTURE }
                }
            }
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    input message: 'Review the refreshed Terraform plan. Approve applying it to AWS?', ok: 'Apply'
                }
            }
        }

        stage('Apply') {
            when {
                allOf {
                    branch 'main'
                    expression { params.APPLY_INFRASTRUCTURE }
                }
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
