parameters {
    choice choices: ['Plan', 'Apply', 'Destroy'], description: 'pickup any one', name: 'Action'
    choice choices: ['Import'], description: 'to import state file', name: 'Terraform Import'
    activeChoice choiceType: 'PT_CHECKBOX', description: 'pickup one', filterLength: 1, filterable: false, name: 'Terraform State', randomName: 'choice-parameter-189674809471088', script: groovyScript(fallbackScript: [classpath: [], oldScript: '', sandbox: true, script: 'return[\'error\']'], script: [classpath: [], oldScript: '', sandbox: true, script: 'return[\'Show\' , \'List\' , \'Remove \']'])
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
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'repo', url: 'https://github.com/Arya5596/repo05']])
                
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
                expression { params.Action == 'Import'}
            }
            steps {
                script {
                    echo "Executing terraform import for azure account, Importing Terraform Statefile...."
                    sh 'terraform import azurerm_resource_group.example02 /subscriptions/44072bee-09d8-4fd1-9150-46a2662697d7/resourceGroups/example02'
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
