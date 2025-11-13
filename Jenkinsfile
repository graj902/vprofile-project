pipeline {
    agent any
    tools {
        maven "Maven3.9"
        jdk "JDK17"
        // This name MUST match what you configured in Manage Jenkins > Tools
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

        //
        // THIS IS YOUR STAGE, NOW IN THE CORRECT LOCATION
        //
        stage('SonarQube Analysis') {
            environment {
                // This finds the tool named 'Sonar-scanner' (from the 'tools' block) 
                // and puts its location into a new variable called 'scannerHome'
                scannerHome = tool 'Sonar-scanner'
            }
            steps {
                // This wrapper gets the URL and Token from your 'Sonar-server' configuration
                withSonarQubeEnv(env.SONAR_SERVER) { 
                    
                    // This is Imran's 'sonar-scanner' command with the -Dsonar.java.binaries path FIXED
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