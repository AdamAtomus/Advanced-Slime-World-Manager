pipeline {
    agent any

    tools {
        maven '3.6.3'
        jdk '8u262'
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "https"
        NEXUS_URL = "repo.rapture.pw"
        NEXUS_RELEASE_REPOSITORY = "maven-releases"
	NEXUS_SNAPSHOT_REPOSITORY = "maven-snapshots"
        NEXUS_CREDENTIAL_ID = "rapture.pw-nexus"
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    wget https://cdn.getbukkit.org/spigot/spigot-1.16.1.jar && mvn install:install-file -Dfile=spigot-1.16.1.jar -DgroupId=org.spigotmc -DartifactId=spigot -Dversion=1.16.1-R0.1-SNAPSHOT -Dpackaging=jar
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
            }
        }

        stage ('Build') {
            steps {
                dir('.'){
                    sh 'mvn -Dmaven.test.failure.ignore=true install'
                }
            }
        }

        stage('Release') {
              steps {
                    dir('.'){
                        echo 'Creating artifacts...';
                        sh "mkdir -p output"
                        sh "mv slimeworldmanager-api/target/slimeworldmanager*.jar output/"
                        sh "mv slimeworldmanager-classmodifier/target/slimeworldmanager*.jar output/"
                        sh "mv slimeworldmanager-plugin/target/slimeworldmanager*.jar output/"
                        sh "mv slimeworldmanager-importer/target/slimeworldmanager*.jar output/"
                        archiveArtifacts artifacts: 'output/*'
                    }
              }
        }

        stage('Maven Publish') {
              when {
                anyOf {
                  branch 'master';
                  branch 'develop';
                }
              }
              steps {
                echo 'Publishing artifacts to Nexus...';
                sh 'mvn deploy';
              }
        }
    }

    post {
        always {
            cleanWs();
            withCredentials([string(credentialsId: 'cloudnet-discord-ci-webhook', variable: 'url')]) {
                    discordSend description: 'New build for Advanced Slime World manager!', footer: 'New build!', link: env.BUILD_URL, successful: currentBuild.resultIsBetterOrEqualTo('SUCCESS'), title: JOB_NAME, webhookURL: url
            }
        }
    }
}
