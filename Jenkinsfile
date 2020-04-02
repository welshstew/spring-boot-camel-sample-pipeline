openshift.withCluster() {
  env.APP_NAME = "swinches-boot-camel"
  env.PIPELINES_NAMESPACE = "cicd"
  env.BUILD_NAMESPACE = "dev"
  env.DEV_NAMESPACE = "dev"
  env.RELEASE_NAMESPACE = "release"
  env.GIT_URL = "https://github.com/welshstew/spring-boot-camel-sample-pipeline.git"
  echo "Starting Pipeline for ${APP_NAME}..."
}

def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
//   slackSend (color: colorCode, message: summary)
}



pipeline {
    agent {
        label "master"
    }

    environment {
        GIT_SSL_NO_VERIFY = true
        GIT_CREDENTIALS = credentials('cicd-github-secret')

        NEXUS_CREDENTIALS = credentials('nexus-credentials')

        MAVEN_MIRROR_URL = "http://nas25c3f7.lan:10081/repository/maven-public/"

        MAVEN_SERVER_USERNAME = "${NEXUS_CREDENTIALS_USR}"
        MAVEN_SERVER_PASSWORD = "${NEXUS_CREDENTIALS_PSW}"

        // PRE_TAG = "${JOB_NAME}.${BUILD_NUMBER}"
        // JENKINS_TAG = "${PRE_TAG}".substring("${PRE_TAG}".lastIndexOf("/") + 1)
        // RELEASE_TAG = "release"

        // NEXUS_VERSION = "nexus3"
        // // This can be http or https
        // NEXUS_PROTOCOL = "http"
        // // Where your Nexus is running
        // NEXUS_URL = "172.17.0.3:8081"
        // // Repository where we will upload the artifact
        // NEXUS_REPOSITORY = "repository-example"
        // // Jenkins credential id to authenticate to Nexus OSS
        // NEXUS_CREDENTIAL_ID = "nexus-credentials"
    }


    stages {

        stage("Pipeline Start") {
            steps {
                notifyBuild('STARTED')
            }
            post {
                failure {
                    notifyBuild('FAIL')
                }
            }
        }

        stage("Checkout the code") {
            steps {
                script {
                    // Let's clone the source
                    git 'https://github.com/welshstew/spring-boot-camel-sample-pipeline.git';
                }
            }
        }

        stage("Build Java App") {
            agent {
                node { 
                    label "maven"
                }
            }
            steps {
                echo 'Build the java app'

                sh  '''
                printenv
                mvn -s /home/jenkins/nexus/settings.xml versions:set -DnewVersion=1.0.${BUILD_ID}
                mvn -s /home/jenkins/nexus/settings.xml clean install deploy
                '''
            }
            post {
                failure {
                    notifyBuild('FAIL')
                }
            }
        }

        stage("Create Build objects in Openshift") {
            agent {
                node { 
                    label "jenkins-slave-helm"
                }
            }
            steps {
                echo 'deploy the helm chart to create the build objects for this application'

                sh  '''
                printenv
                helm install simple ./charts/swinches-spring-boot-build
                '''
            }
            post {
                failure {
                    notifyBuild('FAIL')
                }
            }
        }




        // stage("mvn build") {
        //     steps {
        //         script {
        //             sh "mvn package -DskipTests=true"
        //         }
        //     }
        // }
        // stage("publish to nexus") {
        //     steps {
        //         script {
        //             // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
        //             pom = readMavenPom file: "pom.xml";
        //             // Find built artifact under target folder
        //             filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
        //             // Print some info from the artifact found
        //             echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
        //             // Extract the path from the File found
        //             artifactPath = filesByGlob[0].path;
        //             // Assign to a boolean response verifying If the artifact name exists
        //             artifactExists = fileExists artifactPath;
        //             if(artifactExists) {
        //                 echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
        //                 nexusArtifactUploader(
        //                     nexusVersion: NEXUS_VERSION,
        //                     protocol: NEXUS_PROTOCOL,
        //                     nexusUrl: NEXUS_URL,
        //                     groupId: pom.groupId,
        //                     version: pom.version,
        //                     repository: NEXUS_REPOSITORY,
        //                     credentialsId: NEXUS_CREDENTIAL_ID,
        //                     artifacts: [
        //                         // Artifact generated such as .jar, .ear and .war files.
        //                         [artifactId: pom.artifactId,
        //                         classifier: '',
        //                         file: artifactPath,
        //                         type: pom.packaging],
        //                         // Lets upload the pom.xml file for additional information for Transitive dependencies
        //                         [artifactId: pom.artifactId,
        //                         classifier: '',
        //                         file: "pom.xml",
        //                         type: "pom"]
        //                     ]
        //                 );
        //             } else {
        //                 error "*** File: ${artifactPath}, could not be found";
        //             }
        //         }
        //     }
        // }
    }
}