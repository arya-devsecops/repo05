parameters {
    choice choices: ['Plan', 'Apply', 'Destroy','State' , 'Import'], description: 'pickup any one', name: 'Action'
    string description: 'Terraform Arguments', name: 'ARGUMENTS'
}

pipeline {
    agent any
options {
  ansiColor('xterm')
    }
    environment {
        ARM_CLIENT_ID         = credentials('CLIENT_ID')
        ARM_CLIENT_SECRET     = credentials('CLIENT_SECRET')
        ARM_SUBSCRIPTION_ID   = credentials('SUBSCRIPTION_ID')
        ARM_TENANT_ID         = credentials('TENANT_ID')
        }
    stages {
        stage('Git Checkout') {
            steps {
               checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/arya-devsecops/repo05.git']])
                
            }
        }
          stage('Terraform Init') {
            steps {
                script {
                    sh 'terraform init'
                }
            }
        }
        stage('Terraform code format check') {
            steps{
                script {
                    sh 'terraform fmt'
                }
            }
        } 
        stage('Terraform validate') {
            steps {
                script {
                    sh 'terraform validate'
                }
            }
        }
         stage('Terraform Plan') {
             when {
                expression { params.Action in ['Plan' , 'Apply' , 'Destroy' , 'Import' , 'State'] && currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                script {
                    echo 'Executing terraform plan...'
                    sh 'terraform plan -out=plan.out'
                }
            }
        }

        stage('Terraform Apply') {
            when {
                expression { params.Action =='Apply' && currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                script {
                    input "Please approve to proceed Apply"
                    echo 'Executing terraform apply...'
                    sh 'terraform apply "plan.out"'
                }
            }
        }

        stage('Terraform Destroy') {
            when {
                expression { params.Action =='Destroy' && currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                script {
                    input "Please approve to proceed Destroy"
                    echo 'Executing terraform destroy...'
                    sh 'terraform destroy -auto-approve'
                }
            }
        }
        stage('Terraform Import') {
            when {
                expression { params.Action == 'Import' && params.ARGUMENTS != "" }
            }
            steps {
                script {
                    echo "Executing terraform import for azure account, Importing Terraform Statefile...."
                    sh 'terraform import ${params.ARGUMENTS}'
                }
            }
        }
         stage('Terraform State Show') {
            when {
                expression { params.Action == 'State' && currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                script {
                    // Replace 'azurerm_resource_group.example02' with the actual address of the resource you want to show
            def resourceAddress = 'azurerm_resource_group.example02'
            
            echo "Executing terraform state show for ${resourceAddress}..."
            sh "terraform state show ${resourceAddress}"
                }
            }
        }

        stage('Terraform State List') {
            when {
                expression { params.Action == 'State' && currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                script {
                    echo 'Executing terraform state list...'
                    sh 'terraform state list'
                }
            }
        }

       stage('Terraform State Remove') {
    when {
        expression { params.Action == 'State' && currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
    }
    steps {
        script {
            // Run terraform state list to identify the resource
            sh 'terraform state list'

            // Replace 'azurerm_resource_group.example02' with the actual address of the resource you want to remove
            def resourceAddress = 'azurerm_resource_group.example02'
            
            echo "Executing terraform state rm for ${resourceAddress}..."
            sh "terraform state rm ${resourceAddress}"
                }
            }
        }
    }
}
