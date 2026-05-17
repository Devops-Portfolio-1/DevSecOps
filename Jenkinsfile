pipeline {
    agent any
    tools {
        nodejs 'Node.js 22.13.0'
    }
    environment {
        MONGO_URI = "mongodb+srv://shalindraperera151_db_user:hIgNO8IUucjbj3MW@cluster0.0awxr8z.mongodb.net/?appName=Cluster0"
        MONGO_USERNAME = credentials('mongo-db-username')
        MONGO_PASSWORD = credentials('mongo-db-password')
        SONAR_SCANNER_HOME = tool 'sonarqube-scanner-801'
        ECR_REPO_URL = '450647752372.dkr.ecr.ap-southeast-1.amazonaws.com'
        IMAGE_REPO = "${ECR_REPO_URL}/nodejs-app"
    
    }
    stages {
    stage('Installing Dependencies') {
         options { 
                timestamps() 
            }
        steps {
            sh 'npm install --no-audit'
        }
    }
    
    stage('Dependency Scanning') {
        parallel {
            stage('NPM Dependency Audit') {
                steps {
                    sh '''
                        npm audit --audit-level=critical
                        echo $?
                    '''
                }
            }
            //disable the yarn audit as we are using npm for this project
            stage('OWASP Dependency Check') {
                steps {
                    dependencyCheck additionalArguments: '''
                        --scan './'
                        --out './'
                        --format 'ALL'
                        --disableYarnAudit \
                        --prettyPrint
                    ''', odcInstallation: 'OWASP-DepCheck-10'
                    // Publish the dependency check report and fail the build if critical vulnerabilities are found
                    dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true

                    

                }
            }
        }
    }

    stage('Unit Testing') {
        options { 
                retry(2) 
            }
        steps {
                // withCredentials([usernamePassword(credentialsId: 'mongodb-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                //     sh 'npm test'
                // }
                sh 'echo Colon-Separated - $MONGO_DB_Creds'
                sh 'echo Username - $MONGO_DB_Creds_USR'
                sh 'echo Password - $MONGO_DB_Creds_PSW'
                sh 'npm test'

                
            }
        }
   
   
    stage('Code Coverage') {
        steps {
            // withCredentials([usernamePassword(credentialsId: 'mongodb-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
            //     }
            // }
            // Run the code coverage command and catch any errors to prevent the build from failing
                catchError(buildResult: 'SUCCESS', message: 'Oops! it will be fixed in future releases', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                }
            
        }
}

      stage('SAST -SonarQube') {
        steps {
            // timeout the SonarQube analysis to prevent it from hanging indefinitely
            // timeout(time: 300, unit: 'SECONDS') {
            withSonarQubeEnv('SonarQubeServer') {
               
                        sh "echo SonarQube Scanner Home: ${SONAR_SCANNER_HOME}"
                        sh '''
                        
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=my-project \
                        -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info 

                        '''
                    
            
            }
            waitForQualityGate abortPipeline: true
        }
      }
      stage('Build Docker Image') {

        steps{
            sh "docker build -t ${IMAGE_REPO}:$BUILD_NUMBER ."
        }
      }

      stage('Trivy Scan'){
        steps{
         sh '''
            docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                -v /tmp/trivy-cache:/root/.cache/trivy \
                -v /tmp:/output \
                aquasec/trivy image \
                --severity HIGH,CRITICAL \
                --exit-code 0 \
                --quiet \
                --format json -o /output/trivy-results.json \
                ${IMAGE_REPO}:${BUILD_NUMBER}
        '''
      }
        post {
        always {
            sh '''
            docker run --rm \
                -v /tmp/trivy-cache:/root/.cache/trivy \
                -v /tmp:/output \
                aquasec/trivy convert \
                --format template \
                --template "@/contrib/html.tpl" \
                --output /output/trivy-results.html \
                /output/trivy-results.json
            
            cp /tmp/trivy-results.html ${WORKSPACE}/trivy-results.html
        '''
                
    }
        }
    //   stage('ECR Push') {
    //         steps {
    //             script {
    //                 echo "building the docker image..."
    //                 withCredentials([usernamePassword(credentialsId: 'ecr-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
    //                     sh "echo $PASS | docker login -u $USER --password-stdin ${ECR_REPO_URL}"
    //                     sh "docker push ${IMAGE_REPO}:$BUILD_NUMBER"
    //                 }
    //             }
    //         }
    //     }

 
    
    


    post {
        always {
            
            //unit test results
            junit allowEmptyResults: true, keepProperties: true, testResults: 'test-results.xml'
            // Publish the code coverage report and fail the build if coverage is below 90%

            // Publish the code coverage report
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code Coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])


            // Publish the dependency check report and fail the build if critical vulnerabilities are found
            junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml'

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Dependency check HTML Report', reportTitles: '', useWrapperFileDirectly: true])

            //publish the trivy scan report
            publishHTML([
            allowMissing: true,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: '.',
            reportFiles: 'trivy-results.html',
            reportName: 'Trivy Security Report',
            reportTitles: '',
            useWrapperFileDirectly: true
        ])


            // echo 'Cleaning up workspace...'
            // cleanWs() 
        }
    }
}
    }



