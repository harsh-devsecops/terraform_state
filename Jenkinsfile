pipeline {
    agent any
    options {
  ansiColor('css')
 }
    environment {
       // Define environment variables for Azure credentials
        ARM_SUBSCRIPTION_ID = credentials('SUBSCRIPTION_ID')
        ARM_TENANT_ID = credentials('TENANT_ID')
        ARM_CLIENT_ID = credentials('CLIENT_ID')
        ARM_CLIENT_SECRET = credentials('CLIENT_SECRET')
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from Git'
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'git_credentials', url: 'https://github.com/git01h/terraform_state.git']])
               
            }
        }
        stage('Terraform init') {
            steps {
                script {
                    sh 'terraform init'
                }
            }
        }
        stage('terraform validate') {
            steps {
                script {
                    sh 'terraform validate'
                }
            }
        } 
        stage('Terraform Plan') {
	when {
		
			expression{choice =='Plan'||'Apply'||'Destroy' &&  currentBuild.resultIsBetterOrEqualTo('SUCCESS')}
	}
            steps {
                script {
                    sh 'terraform plan -out=plan.out'
                }
			}
        }
	
      stage(' Terraform Apply') {
	when{
		
			expression{choice == 'Apply' &&  currentBuild.resultIsBetterOrEqualTo('SUCCESS')}
		}
    steps{
        script {
		input "Please approve to proceed with Apply"
            // Run Terraform apply using the saved plan file
            sh 'terraform apply "plan.out"'
        }
    }
      }
        stage('terraform destroy') {
		when{
		
			expression{choice =='Destroy' &&  currentBuild.resultIsBetterOrEqualTo('SUCCESS')}
		} 
            steps {
                script {
			input "Please approve to proceed with Destroy"
                    sh 'terraform destroy --auto-approve'
                }
	    }
	}
		    stage('Terraform Import') {
		    when{
			expression{choice =='Import' && params.Arguments != ""}
		}
            steps {
                script {
                    sh "terraform import ${params.Arguments}"
                }
            }
        }
	      //if (choice == 'State'){
          //terraform_state_option =params.Arguments.split()[0]

 	stage ('Terraform state'){
 	when{
 		expression{choice == 'State' && params.Arguments != " "}
 	}
 	steps {
            script {
		    def terraformStateAction = params.Arguments.split(' ')[0]
                def terraformStateResource = params.Arguments.split(' ')[1]
               stage("Terraform state ${terraformStateAction}")

    if (terraformStateAction == 'list') {
        sh 'terraform state list'
    } else if (terraformStateAction == 'show') {
        sh "terraform state show '${terraformStateResource}'"
    } else if (terraformStateAction == 'rm') {
        sh 'terraform state list'
        input 'Please confirm to proceed with Terraform State Remove'
        sh "terraform state rm '${terraformStateResource}'"
    } else {
        error "Invalid terraformStateAction: ${terraformStateAction}. Supported actions are 'list', 'show', and 'rm'."
    }
 }

	 

             }
         }
    
 }
}

   


    
