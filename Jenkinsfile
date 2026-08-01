@Library('cloudbees-ci-engr-shared-libs') _ // Import shared library

pipeline {
  agent {
        label 'awsjenklinux'
    }

  options {
    timeout(time: 1, unit: 'HOURS') 
  }

parameters {
  string(name: 'ENV', defaultValue: 'Dev')
} 

  stages {
    stage('Agent') {
      agent {
        kubernetes(dynamicAgents(tools: ['dotnet:6.0','awscli:2']))
      }

      stages {
        stage('Git clone') {
          steps {
            script {
              if (params.ENV == 'Dev') {
             git branch: 'develop', credentialsId: 'msbuild', url: 'https://msstash.morningstar.com/scm/ret/min-backend.git'
			}
			if (params.ENV == 'Prod')
			{
			git branch: 'master', credentialsId: 'msbuild', url: 'https://msstash.morningstar.com/scm/ret/min-backend.git'
			}
         }
        }
		}
        stage('Build Application') {
          steps {
            container('dotnet') {
              sh '''
               	echo "Build now..."
                dotnet build Newsletters/ --no-incremental
                cd Newsletters/src/Newsletters/
                chmod 777 .
                apt-get install -y zip
                dotnet tool install -g Amazon.Lambda.Tools
                dotnet lambda package --output-package NewslettersAspNetCoreFunction-CodeUri-Or-ImageUri.zip  
                echo "Install Lambda Package"
              '''
            }
          }
        }
        stage('awscli') {
          steps{
    	container('awscli') {
        script{
		if (params.ENV == 'Dev') {
      	withAWS(roleAccount:'012775134384', role:'mstar-rpmn-backendapi-deployment', useNode:true) 
      	{    
          sh 'aws s3 cp Newsletters/src/Newsletters/NewslettersAspNetCoreFunction-CodeUri-Or-ImageUri.zip s3://newslettersbackendcode/'
          sh 'aws s3 cp --recursive integration-automation-test s3://newslettersbackendcode/integration-automation-test'          
          sh 'aws lambda update-function-code --function-name NewslettersNonVPC-Newsletters-7OSL0xcbxm5e --s3-bucket newslettersbackendcode --s3-key NewslettersAspNetCoreFunction-CodeUri-Or-ImageUri.zip  --region us-east-1' 
          
        }
        }
		if (params.ENV == 'Prod')
		{
		 withAWS(roleAccount:'835110494321', role:'mstar-rpmn-backendapi-deployment', useNode:true) 
      	{    
          sh 'aws s3 cp Newsletters/src/Newsletters/NewslettersAspNetCoreFunction-CodeUri-Or-ImageUri.zip s3://prodnewslettersbackendcode/'
          sh 'aws s3 cp Newsletters/src/Newsletters/NewslettersAspNetCoreFunction-CodeUri-Or-ImageUri.zip s3://prodnewslettersbackendcodedr/'          
          sh 'aws lambda update-function-code --function-name NewslettersMAAS-Newsletters-3QnbIOpcupQ3 --s3-bucket prodnewslettersbackendcode --s3-key NewslettersAspNetCoreFunction-CodeUri-Or-ImageUri.zip  --region us-east-1'
          sh 'aws lambda update-function-code --function-name NewslettersDR-Newsletters-tSEELfhQHdwa --s3-bucket prodnewslettersbackendcodedr --s3-key NewslettersAspNetCoreFunction-CodeUri-Or-ImageUri.zip  --region us-west-2'
        }
		}
          }
	   }
    }
  }
      }
        post {
             success {

				emailext body: 'Latest changes deployment successful on '+ params.ENV + ' server',

				subject: 'Deployment successful on '+ params.ENV + ' server',

				to: 'sandesh.patil1@morningstar.com,bharatkumar.dwivedi@morningstar.com,amey.shah@morningstar.com,gaurav.singh1@morningstar.com,pranitha.dharanikota@morningstar.com,sagar.rajput@morningstar.com,kirti.kundnani@morningstar.com,pranav.hanwante@morningstar.com,pranati.lokhande@morningstar.com,sangram.salunkhe@morningstar.com'

                }

                unsuccessful {

                   emailext body: 'Deployment unsuccessful on '+ params.ENV + ' server',

                    subject: 'Latest changes deployment unsuccessful on '+ params.ENV + ' server',

                    to: 'sandesh.patil1@morningstar.com,bharatkumar.dwivedi@morningstar.com,amey.shah@morningstar.com,gaurav.singh1@morningstar.com,pranitha.dharanikota@morningstar.com,sagar.rajput@morningstar.com,kirti.kundnani@morningstar.com,pranav.hanwante@morningstar.com,pranati.lokhande@morningstar.com,sangram.salunkhe@morningstar.com'

                         }
           }
    }
  }

}
