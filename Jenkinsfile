pipeline {
    agent any

    tools {
        nodejs 'NodeJS' // Make sure this matches the configured NodeJS tool name in Jenkins
        jdk 'JAVA'      // Make sure this matches the configured JDK tool name in Jenkins
    }

    stages {
        stage('Verify JDK Installation') {
            steps {
                // Verify if jar command is available
                bat 'jar --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
                bat 'npm audit fix || exit 0'
            }
        }

        stage('Build') {
            steps {
                bat 'npm run build -- --configuration production'
            }
        }

        stage('Verify Build Output') {
            steps {
                bat 'dir dist'
                bat 'dir dist\\country-view'
            }
        }

        stage('Package') {
            steps {
                script {
                    // Package the build output as a WAR file using PowerShell
                    powershell '''
                    $distPath = "dist\\country-view"
                    $warPath = "dist\\war"
                    
                    # Ensure the war directory exists
                    if (-Not (Test-Path -Path $warPath)) {
                        New-Item -ItemType Directory -Path $warPath
                    }
                    
                    # Copy files from dist\\country-view to dist\\war
                    Copy-Item -Path "$distPath\\*" -Destination $warPath -Recurse -Force
                    
                    # Change to the war directory
                    Set-Location -Path $warPath
                    
                    # Create the WAR file
                    & jar -cvf country-view.war *
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Define variables
                    def tomcatPath = "C:\\Program Files\\Apache Software Foundation\\Tomcat 9.0\\webapps"
                    def warFile = "dist\\war\\country-view.war"

                    // Check if Tomcat is already stopped
                    def tomcatStatus = bat(script: 'sc query Tomcat9 | findstr /C:"STOPPED"', returnStatus: true)
            
                    if (tomcatStatus == 0) {
                        echo "Tomcat is already stopped."
                    } else {
                        // Stop Tomcat
                        bat "net stop Tomcat9"
                    }

                    // Copy WAR file to Tomcat webapps directory
                    def copyCommand = "copy \"${warFile}\" \"${tomcatPath}\\country-view.war\""
                    echo "Executing: ${copyCommand}"
                    bat copyCommand

                    // Start Tomcat
                    bat "net start Tomcat9"
                }
            }
        }
    }
}
