pipeline {
    agent {
      label 'AGENT-1'
    }
    options {
        timeout(time: 300, unit: 'SECONDS')
        disableConcurrentBuilds()
    }

    environment {
        DEBUG = 'true'
        appVersion = ''
        account_id = '596059882666'
        region = 'us-east-1'
        project = 'expense'
        environment = 'development'
        component = 'frontend'
    }
    stages {
        stage('Read Version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "App Version: ${appVersion}"
                }
            }
        }
        stage('Install Dependencies') {
          steps {
              sh "npm install"
          }
        }
        stage('Building image') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.us-east-1.amazonaws.com

                        docker build -t ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion} .

                        docker images

                        docker push ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion}
                    """
                }
            }
        }
        stage('Deploy') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name ${project}-${environment}
                        cd helm
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values-dev.yaml
                        helm upgrade --install ${component} -n ${project} -f values-dev.yaml .
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
        success {
            echo "Build Successful"
        }
        failure {
            echo "Build Failed"
        }
        unstable {
            echo "Build Unstable"
        }
        changed {
            echo "Status changed from previous build"
        }
    }
}
