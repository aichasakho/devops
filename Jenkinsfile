pipeline {
    agent any

    environment {
        mvn = tool 'Maven'
    }

    tools {
        jdk 'jdk11'
        maven 'Maven'
    }

    stages {
        stage('Git Check out') {
            steps {
                checkout scm
            }
        }

        stage('Maven build') {
            steps {
                sh "mvn clean package"
            }
        }
    stage('SonarQube Analysis') {
      steps{
        withSonarQubeEnv('sonar-server') {
        sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=devops-aicha -Dsonar.projectName=devops-aicha -Dsonar.login=sqp_778012cc8939d601645358638e0f7f9534ca1cd5"
      }
    }

  }

        stage('Publish to Nexus') {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    def artifactPath = filesByGlob[0].path
                    def artifactExists = fileExists artifactPath

                    if (artifactExists) {
                        echo "* File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTIFACT_VERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging]
                            ]
                        )
                    } else {
                        error "* File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
    }
}
