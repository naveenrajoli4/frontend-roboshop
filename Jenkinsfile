pipeline {
    agent {
        label 'agent-2-label'
    }
    options{
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        //retry(1)
    }
    
    environment {
            DEBUG = 'true'
            appversion = ''
            region = 'us-east-1'
            acc_ID = '135808959960'
            project = 'roboshop'
            environment = 'prod'
            component = 'frontend'
        }

    stages {
        stage('Read the version') {
            steps {
                script{
                    def packageJson = readJSON file: 'package.json'
                    appversion = packageJson.version
                    echo "App version: ${appversion}"
                }
            }
        }
        stage('Docker build') {
            
            steps {
                withAWS(region: 'us-east-1', credentials: "aws-creds-${environment}") {
                    sh """                   
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${acc_ID}.dkr.ecr.${region}.amazonaws.com

                        docker build -t ${acc_ID}.dkr.ecr.${region}.amazonaws.com/kdp-${project}-${environment}/${component}:${appversion} .

                        docker images

                        docker push ${acc_ID}.dkr.ecr.${region}.amazonaws.com/kdp-${project}-${environment}/${component}:${appversion}                 

                    """
                }
            }
        }
        stage('Deploy'){
            steps{
                withAWS(region: 'us-east-1', credentials: "aws-creds-${environment}") {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name kdp-${project}-${environment}-eks
                        cd helm
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml
                        helm upgrade --install ${component} -n rnk-${project} -f values-${environment}.yaml .
                    """
                }
            }
        }
    }

    post {
        always{
            echo "This sections runs always"
            deleteDir()
        }
        success{
            echo "This section run when pipeline success"
        }
        failure{
            echo "This section run when pipeline failure"
        }
    }
}