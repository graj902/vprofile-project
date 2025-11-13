pipeline {
    agent any
    tools {
        maven "Maven3.9"
        jdk "JDK17"
    }
    
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'Maafa143@#'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vprofile-maven-central'
        NEXUSIP = '172.31.42.72'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vprofile--maven-group'
        NEXUS_LOGIN = 'nexuslogin'
    }

    stages {
        stage('Build'){
            steps {
                // Use 'package' to create the .war, skip tests
                sh 'mvn -s settings.xml -DskipTests package'
            }
        } 
        
        stage('Test'){
            steps {
                // Run tests to generate code coverage data
                sh 'mvn -s settings.xml test'
            }
        } 
        
        stage ('Checkstyle') {
            steps {
                // Run static code analysis
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
    } // <-- The 'stages' block ENDS HERE

    //
    // This is the correct 'post' block that runs AFTER all stages
    //
    post {
        success {
            steps {
                echo 'Build was successful. Archiving the .war file...'
                // This command finds the .war file and saves it
                // as a build artifact, just like in the video.
                archiveArtifacts artifacts: 'target/vprofile-v2.war', fingerprint: true
            }
        }
        
        failure {
            steps {
                echo 'Build failed!'
                // This is where you would add a Slack notification
            }
        }
    }
}