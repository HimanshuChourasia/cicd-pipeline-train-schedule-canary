pipeline {
 environment {
    dockerhub_repo = 'himanshuchourasia/train-schedule'
    dockerhub_credential = 'dockerhub_credentials' 
 }

    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build and push  docker image') {
           when {
               branch 'master'
               
           }
           steps{
         	script {
    			def myImage = docker.build("${dockerhub_repo}")
    			docker.withRegistry('https://registry.hub.docker.com', dockerhub_credential) {
    			myImage.push("${env.BUILD_ID}")
    			myImage.push('latest')
    		
     }
    			
}     
               
           }
       }    
           stage('Canary deployment') {
              when {
                 branch 'master'
                 
             }
             environment {
                 CANARY_REPLICAS = 1
             }
             steps {
                kubernetesDeploy( 
				configs: 'train-schedule-kube-canary.yaml', 
				kubeconfigId: 'kube_config',
				enableConfigSubstitution: true
				)     			           
                 
                 
             }

           }
           
           stage('deploy container to production'){
             when {
                 branch 'master'
                 
             }
             environment {
                 CANARY_REPLICAS = 0
             }

             steps{
                
 				input 'Deploy container to production ?'
                milestone label:'container ready for production', ordinal:1
				kubernetesDeploy( 
				configs: 'train-schedule-kube-canary.yaml', 
				kubeconfigId: 'kube_config',
				enableConfigSubstitution: true
				)
				kubernetesDeploy( 
				configs: 'train-schedule-kube.yaml', 
				kubeconfigId: 'kube_config',
				enableConfigSubstitution: true
				)     			           


 				  

		
 			          
            }
         }                        
       }
     }
   