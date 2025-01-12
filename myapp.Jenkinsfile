pipeline {
   agent any
    environment {
        def scannerHome = tool 'sonarScanner5.0.1'
    }
    stages {
        stage("Build"){
            steps {
               script {
                sh "mvn install"
                sh "mv target/*.jar target/spring-boot-2-hello-world-1.0.2-SNAPSHOT-${BUILD_NUMBER}.jar"
               }
            }
        }

        stage("Unit-Test"){
            steps {
                script {
                    sh "mvn test"
                }
            }
        }

        stage("Code Analysis"){
            steps {
                withSonarQubeEnv('mysonarQube') {
                    sh """ ${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=spring-boot-hello-world \
                    -Dsonar.projectName=spring-boot-hello-world \
                    -Dsonar.sources=. \
                    -Dsonar.java.binaries=target/classes \
                    -Dsonar.sourceEncoding=UTF-8
                    """
                }
            }
        }

        stage("Quality Gate") {
            steps {
              timeout(time: 2, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage("Upload Artifacts"){
            steps{
                
                rtServer (
                        id: 'jfrog-server',
                        url: 'http://683b06656b2c.mylabserver.com/artifactory/',
                        // If you're using username and password:
                        username: 'admin',
                        password: 'Admin@123',
                        timeout: 300
                )
                rtUpload (
                    serverId: 'jfrog-server',
                    spec: '''{
                        "files": [
                            {
                            "pattern": "target/*.jar",
                            "target": "example-repo-local/spring-boot-hello-world/"
                            }
                        ]
                    }''',
                )    
            }
        } 

        // stage("Deploy - Dev"){
        //     steps {
        //         sshagent(['ssh-creds']) {
        //            sh """      
        //                scp   -o StrictHostKeyChecking=no  target/*.jar    cloud_user@683b06656b2c.mylabserver.com:/home/cloud_user
        //             """
        //         }
        //     }

        // }

        // stage("Deploy - UAT"){
        //     steps {

        //         sshagent(['ssh-creds']) {
        //         sh """      
        //             scp  -o StrictHostKeyChecking=no   target/*.jar    cloud_user@683b06656b3c.mylabserver.com:/home/cloud_user
        //         """
        //     }
        //     }

        // }

        // stage("Deploy - Prod"){
        //     input{
        //          message "Do you want to proceed for production deployment?"
        //     }

        //     steps {

        //         sshagent(['ssh-creds']) {
        //         sh """      
        //             scp    -o StrictHostKeyChecking=no  target/*.jar    cloud_user@683b06656b3c.mylabserver.com:/home/cloud_user
        //         """
        //     }
        //     }

        // }
    }
    post {
        always {
             cleanWs()
        }
    }
}