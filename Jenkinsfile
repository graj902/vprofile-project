// This color map MUST be outside the pipeline block
def colorMap = [
    'SUCCESS': 'good',
    'UNSTABLE': 'warning',
    'FAILURE': 'danger'
]

pipeline {
    agent any
    tools {
        maven "Maven3.9"
        jdk "JDK17"
    }
    
    environment {
        // --- Nexus Variables ---
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'Maafa143@#'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vprofile-maven-central'
        NEXUSIP = '172.31.42.72'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vprofile--maven-group'
        NEXUS_LOGIN_ID = 'nexus-login' 

        // --- SonarQube Variables ---
        SONAR_SERVER = 'Sonar-server'

        // --- Slack Variables ---
        SLACK_CHANNEL = 'jenkins-cicd'
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests package'
            }
        } 
        
        stage('Test'){
            steps {
                sh 'mvn -s settings.xml test-5ftests'
            }
        } 
        
        stage ('Checkstyle') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'Sonar-scanner'
            }
            steps {
                withSonarQubeEnv(env.SONAR_SERVER) { 
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                           -Dsonar.projectName=vprofile-repo \
                           -Dsonar.projectVersion=1.0 \
                           -Dsonar.sources=src/ \
                           -Dsonar.java.binaries=target/classes \
                           -Dsonar.junit.reportsPath=target/surefire-reports/ \
                           -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                           -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        } 
        
        stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${env.NEXUSIP}:${env.NEXUSPORT}",
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: "${env.RELEASE_REPO}",
                  credentialsId: env.NEXUS_LOGIN_ID,
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }
    } // <-- The 'stages' block ENDS HERE

    post {
        always {
            archiveArtifacts artifacts: 'target/vprofile-v2*.war', fingerprint: true
            
            timeout(time: 10, unit: 'MINUTES') {
                // This is the corrected Quality Gate syntax
                waitForQualityGate abortPipeline: true
            }

            echo 'Sending Slack notification...'
            slackSend (
                color: colorMap.get(currentBuild.currentResult, 'warning'), 
                channel: env.SLACK_CHANNEL,
                message: "${currentBuild.currentResult}: Job '${env.JOB_NAME}' build #${env.BUILD_NUMBER} \nMore info at: ${env.BUILD_URL}"
            )
        }
    }
} // <-- This was the missing '}'