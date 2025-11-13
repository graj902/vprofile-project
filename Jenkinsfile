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
        
        // 1. FIXED: Correct variable name and credential ID (with dash)
        NEXUS_LOGIN_ID = 'nexus-login' 

        // --- SonarQube Variables ---
        SONAR_SERVER = 'Sonar-server'
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests package'
            }
        } 
        
        stage('Test'){
            steps {
                sh 'mvn -s settings.xml test'
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
        
        // 2. FIXED: The 'UploadArtifact' stage now uses the correct variable
        stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${env.NEXUSIP}:${env.NEXUSPORT}",
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: "${env.RELEASE_REPO}",
                  credentialsId: env.NEXUS_LOGIN_ID, // <-- THIS IS THE FIX
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
        success {
            echo 'Build was successful. Archiving the .war file...'
            // We use a wildcard (*) to archive the timestamped .war file
            archiveArtifacts artifacts: 'target/vprofile-v2*.war', fingerprint: true

            // 3. This is the correct and only Quality Gate check
            timeout(time: 10, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        } 
        
        failure {
            echo 'Build failed!'
        }
    }
}