pipeline {
  agent any
  tools {
    maven "Maven"
  }

  environment {
      def mvn = tool 'Maven'

      NEXUS_VERSION = "nexus3"
      NEXUS_PROTOCOL = "http"
      NEXUS_URL = "http://172.20.5.10:8081"
      NEXUS_REPOSITORY = "RepositoryJenkins"
      NEXUS_CREDENTIAL_ID = "nexusCredential"
      ARTIFACT_VERSION = "${BUILD_NUMBER}"
  }
  
  stages {
    stage('Git Check out') {
      steps{
        checkout scm
      } 
    }

    stage('Maven build') {
      steps {
        sh "${mvn}/bin/mvn clean package "
      }
    }

    stage('SonarQube Analysis') {
      steps{
        withSonarQubeEnv('sonar-server') {
        sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=bom-boum  -Dsonar.projectName=bom-boum"
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
