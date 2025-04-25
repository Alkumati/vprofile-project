def buildNumber = Jenkins.instance.getItem('cicd-jenkins-bean-stage').lastSuccessfulBuild.number

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
        NEXUSPASS = credentials("nexuspass")
        ARTIFACT_NAME = "vprofile-v${BUILD_ID}.war"
        AWS_S3_BUCKET = 'vprocicdbean2025'
        AWS_EB_APP_NAME = 'vproapp'
        AWS_EB_ENVIRONMENT = 'Vproapp-env'
        AWS_EB_APP_VERSION = "${buildNumber}"
        BUILD_TIMESTAMP = ""
    }

    stages {
        stage('Initialize Timestamp') {
            steps {
                script {
                    env.BUILD_TIMESTAMP = new Date().format('yyyy-MM-dd-HHmm', TimeZone.getTimeZone('UTC'))
                }
            }
        }

        // stage('Build') {
        //     steps {
        //         sh 'mvn -s settings.xml -DskipTests install'
        //     }
        //     post {
        //         success {
        //             echo "Now Archiving."
        //             archiveArtifacts artifacts: '**/*.war'
        //         }
        //     }
        // }

        // stage('Test') {
        //     steps {
        //         sh 'mvn -s settings.xml test'
        //     }
        // }

        // stage('Checkstyle Analysis') {
        //     steps {
        //         sh 'mvn -s settings.xml checkstyle:checkstyle'
        //     }
        // }

        // stage('Sonar Analysis') {
        //     environment {
        //         scannerHome = tool "${SONARSCANNER}"
        //     }
        //     steps {
        //         withSonarQubeEnv("${SONARSERVER}") {
        //             sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
        //             -Dsonar.projectName=vprofile \
        //             -Dsonar.projectVersion=1.0 \
        //             -Dsonar.sources=src/ \
        //             -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
        //             -Dsonar.junit.reportsPath=target/surefire-reports/ \
        //             -Dsonar.jacoco.reportsPath=target/jacoco.exec \
        //             -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
        //         }
        //     }
        // }

        // stage("UploadArtifact") {
        //     steps {
        //         nexusArtifactUploader(
        //             nexusVersion: 'nexus3',
        //             protocol: 'http',
        //             nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
        //             groupId: 'QA',
        //             version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        //             repository: "${RELEASE_REPO}",
        //             credentialsId: "${NEXUS_LOGIN}",
        //             artifacts: [
        //                 [
        //                     artifactId: 'vproapp',
        //                     classifier: '',
        //                     file: 'target/vprofile-v2.war',
        //                     type: 'war'
        //                 ]
        //             ]
        //         )
        //     }
        // }

        stage('Deploy to Prod Bean') {
            steps {
                withAWS(credentials: 'awsbeancreds', region: 'us-east-1') {

                    sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
                }
            }
        }

        // Uncomment this block if needed
        // stage('Ansible Deploy to staging') {
        //     steps {
        //         ansiblePlaybook([
        //             inventory: 'ansible/stage.inventory',
        //             playbook: 'ansible/site.yml',
        //             installation: 'ansible',
        //             colorized: true,
        //             credentialsId: 'applogin',
        //             disableHostKeyChecking: true,
        //             extraVars: [
        //                 USER: "admin",
        //                 PASS: "${NEXUSPASS}",
        //                 nexusip: "${NEXUSIP}",
        //                 reponame: "${RELEASE_REPO}",
        //                 groupid: "QA",
        //                 time: "${env.BUILD_TIMESTAMP}",
        //                 build: "${env.BUILD_ID}",
        //                 artifactid: "vproapp",
        //                 vprofile_version: "vproapp-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war"
        //             ]
        //         ])
        //     }
        // }
    }

    post {
        always {
            script {
                def COLOR_MAP = [
                    'SUCCESS': 'good',
                    'FAILURE': 'danger'
                ]
                echo 'Sending Slack Notifications.'
                slackSend channel: '#jenkinscicd',
                    color: COLOR_MAP[currentBuild.currentResult],
                    message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
            }
        }
    }
}
