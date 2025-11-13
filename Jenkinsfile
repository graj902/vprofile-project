pipeline {
    agent any
    tools {
        maven "Maven3.9"
        jdk "JDK17"
        // 1. ADD THE SONARSCANNER TOOL NAME
        // This MUST match the name in Manage Jenkins > Tools
        sonar 'Sonar-scanner' 
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
        NEXUS_LOGIN = 'nexuslogin'

        // --- SonarQube Variables ---
        // This MUST match the name in Manage Jenkins > System
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
                // We must run 'test' to generate the JaCoCo report for SonarQube
                sh 'mvn -s settings.xml test'
            }
        } 
        
        stage ('Checkstyle') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        // 2. ADD THE SONARQUBE STAGE (INSIDE 'stages')
        stage('SonarQube Analysis') {
            steps {
                // This wrapper gets the URL and Token from your 'Sonar-server' configuration
                withSonarQubeEnv(env.SONAR_SERVER) {
                    
                    // This is the correct command. It's a Maven project,
                    // so we use the Maven sonar plugin.
                    sh 'mvn -s settings.xml sonar:sonar'
                }
            }
        }
    } // <-- The 'stages' block ENDS HERE

    post {
        success {
            echo 'Build was successful. Archiving the .war file...'
            archiveArtifacts artifacts: 'target/vprofile-v2.war', fingerprint: true

            // 3. ADD THE QUALITY GATE CHECK
            // This pauses the pipeline and waits for the SonarQube webhook
            // to send back a "PASSED" or "FAILED" status.
            timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
        
        failure {
            echo 'Build failed!'
        }
    }
}