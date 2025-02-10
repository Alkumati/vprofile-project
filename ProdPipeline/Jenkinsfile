    // Define color map for Slack notifications
def COLOR_MAP = [
        'SUCCESS': 'good', 
        'FAILURE': 'danger'
    ]
    
pipeline {
    agent any
    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.28.98'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
        SONAR_SCANNER_OPTS = "--add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED"
    }



    stages {
        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        // Quality Gate stage commented out
        /*
        stage("Quality Gate") {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        echo "Waiting for SonarQube Quality Gate..."
                        def qualityGate = waitForQualityGate abortPipeline: true
                        echo "Quality Gate Status: ${qualityGate.status}"
                    }
                }
            }
        }
        */

        // New stage to upload artifact to Nexus
        stage("UploadArtifact") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3', // Nexus version
                    protocol: 'http', // Protocol (http or https)
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}", // Nexus server URL
                    groupId: 'QA', // Group ID for the artifact
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}", // Version of the artifact
                    repository: "${RELEASE_REPO}", // Nexus repository to upload to
                    credentialsId: "${NEXUS_LOGIN}", // Jenkins credentials ID for Nexus
                    artifacts: [
                        [
                            artifactId: 'vproapp', // Artifact ID
                            classifier: '', // Classifier (optional)
                            file: 'target/vprofile-v2.war', // Path to the file to upload
                            type: 'war' // Type of the artifact
                        ]
                    ]
                )
            }
        }
    }

    // Post-build actions
    post {
        always {
            echo 'Sending Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}