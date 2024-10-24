pipeline {
    agent { label 'UAT' }
    
    tools{
        // jdk 'jdk17'
        maven 'Maven3'
    }
        environment {
        SCANNER_HOME=tool 'sonarscanner'
    }
    
     stages{
                
        //  stage("Test Cases"){
        //     steps{
        //         sh "./mvnw test -Dcheckstyle.skip"
        //     }
        // }

        stage("compile"){
            steps{
                sh " mvn compile"
            }
        }
        
        stage('test'){
            steps{
                sh " mvn test"
            }
         }

         stage('install'){
         
            steps{
                sh " mvn clean install"
            }

         }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
    
                }
            }
        }
         stage('Owasp dependency check'){
                steps {
             catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE')
              {
              timeout(time: 60, unit: 'MINUTES') {
                     dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dependency-check'
                     dependencyCheckPublisher pattern: 'dependency-check-report.xml'}
            }
            }//steps
               

          }
        // stage("OWASP Dependency Check"){
        //     steps{
        //         dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'dependency-check'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        
         

        // stage("Compile"){
        //     steps{
        //         sh "./mvnw package"
        //     }
        // }
 


        
     }
}
