pipeline {
    
    agent {
        label 'test'
    }
   
    tools {
           label 'maven'
           label 'java'
    }
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10')
        disableConcurrentBuilds()
        timestamps()
    }
    
    parameters {
        string defaultValue: '', description: 'Version of the java application', name: 'app_version', trim: false
        choice choices: ['DEV', 'QA', 'INT', 'PRE_PROD'], description: 'Environment name for the code deployment', name: 'APP_ENV'
    }
    
    stages {
        stage('Code Checkout'){
            steps {
                echo "code checkout"
                git credentialsId: 'github-creds', url: 'https://github.com/drakeade/PetClinic.git'
            }
        }
        
        stage('Code Build'){
            steps {
                sh "mvn test-compile"
            }
        }
        
        stage('Code Analysis & Unit Tests'){
            failFast true
            parallel {
                stage('Unit test') {
                    steps {
                        sh "mvn test"
                    }
                }
                stage('SonarQube Scan'){
                    environment {
                        SCANNER_HOME = tool 'sonarscanner'
                    }
                    steps {
                        withSonarQubeEnv (installationName: 'sq1') {
                            sh "${SCANNER_HOME}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
                        }
                    }
                }
            }
        }
        stage('Code Package'){
            steps {
                sh "mvn package"
            }
        }
        stage('Nexus Upload'){
            steps {
                sh '''
                POM_VERSION=`grep "<version>" pom.xml | head -1 | awk -F "-" '{print $1}' | tail -c 6`
                curl -u admin:admin POST "http://ec2-18-207-239-251.compute-1.amazonaws.com:8081/service/rest/v1/components?repository=PetClinic" -H "accept: application/json" -H "Content-Type: multipart/form-data" -F "maven2.groupId=org.SampleOrg" -F "maven2.artifactId=petclinic" -F "maven2.version=${POM_VERSION}" -F "maven2.asset1=@${WORKSPACE}/target/petclinic.war" -F "maven2.asset1.extension=war"
                '''
            }
        }
        stage('Code Deployment'){
            steps {
                ansiblePlaybook disableHostKeyChecking: true, installation: 'ANSIBLE29', inventory: '/opt/inventory.txt', playbook: '/home/ubuntu/playbooks/deploy.yaml'
            }
        }
    }
}
