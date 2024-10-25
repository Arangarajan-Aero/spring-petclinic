pipeline {
    agent { label 'UAT' }
    
    tools{
        // jdk 'jdk17'
        maven 'Maven3'
    }
        environment {
        SCANNER_HOME=tool 'sonarscanner'
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "3.91.233.169:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "p-app"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "Nexus"
        ARTIFACT_VERSION = "${BUILD_NUMBER}"
    }
    
     stages{
            
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
        stage("Quality Gate") {
             steps {
                script {
                    timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                    //We have added timeout so if pipeline will not complete in 1 hour then it will fail . 
                    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv , check quality gate status .
                    if (qg.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
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
            }               

          }
        stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;

                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTIFACT_VERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging]
                            ]
                        );

                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        }

}
