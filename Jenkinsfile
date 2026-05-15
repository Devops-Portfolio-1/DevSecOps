pipeline {
    agent any
    tools {
        nodejs 'Node.js 26.1.0'
    }
    environment {
        MONGO_URI = "mongodb+srv://shalindraperera151_db_user:hIgNO8IUucjbj3MW@cluster0.0awxr8z.mongodb.net/?appName=Cluster0"
    }
    stages {
    stage('Installing Dependencies') {
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
            
            stage('OWASP Dependency Check') {
                steps {
                    dependencyCheck additionalArguments: '''
                        --scan './'
                        --out './'
                        --format 'ALL'
                        --prettyPrint
                    ''', odcInstallation: 'OWASP-DepCheck-10'
                    // Publish the dependency check report and fail the build if critical vulnerabilities are found
                    dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true

                    junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml'

                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Dependency check HTML Report', reportTitles: '', useWrapperFileDirectly: true])


                }
            }
        }
    }

    stage('Unit Testing') {
        steps {
                withCredentials([usernamePassword(credentialsId: 'mongodb-cred', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                    sh 'npm test'
                }
                junit allowEmptyResults: true, keepProperties: true, testResults: 'test-results.xml'
                // Publish the code coverage report and fail the build if coverage is below 90%
            }
        }
    }
}