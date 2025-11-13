pipeline {
    agent any
    tools {
        maven "Maven3.9"
        jdk "JDK17"
        // This is the line I removed:
        // sonar 'Sonar-scanner'  <-- THIS IS NOT VALID SYNTAX HERE
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
        // This name MUST match what you configured in Manage Jenkins > System
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
                // THIS IS THE CORRECT WAY to load the SonarQube Scanner tool.
                // This 'tool' step finds the scanner you named 'Sonar-scanner'
                // in Manage Jenkins > Tools.
                scannerHome = tool 'Sonar-scanner'
            }
            steps {
                // This wrapper gets the URL and Token from your 'Sonar-server' configuration
                withSonarQubeEnv(env.SONAR_SERVER) { 
                    
                    // This command uses the 'scannerHome' variable from the environment block above
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
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    } // <-- The 'stages' block ENDS HERE

    post {
        success {
            echo 'Build was successful. Archiving the .war file...'
            archiveArtifacts artifacts: 'target/vprofile-v2.war', fingerprint: true

            // This is the Quality Gate, it runs after all stages succeed
            timeout(time: 10, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
        
        failure {
            echo 'Build failed!'
        }
    }
}