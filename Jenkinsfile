pipeline {
	agent none
	stages {
		stage('Integration UI Test') {
			parallel {
				stage('Check User') {
					agent any
					steps {
						script {
							def current_user = sh(script: 'whoami', returnStdout: true).trim()
							echo "Current user: ${current_user}"

							// Print Docker information
                    		sh 'docker info'
                    		sh 'docker version'
                    		sh 'docker images'
						}
					}
            }
				stage('Deploy') {
					agent any
					steps {
						script{
							def scriptPath = "${WORKSPACE}/jenkins/scripts/deploy.sh"
							def phpPath = "${WORKSPACE}/src/test/java/mycompany/app"
							sh "chmod -R 755 ${phpPath}"
							sh "chmod +x ${scriptPath}"
							sh "ls -l ${scriptPath}"						
                    		sh "tr -d '\r' < ${scriptPath} > ${scriptPath}.tmp && mv ${scriptPath}.tmp ${scriptPath}"
							sh "chmod +x ${scriptPath}"
							sh "${scriptPath}"
							input message: 'Finished using the web site? (Click "Proceed" to continue)'
							def killPath = "${WORKSPACE}/jenkins/scripts/kill.sh"
							sh "chmod +x ${killPath}"
							sh "tr -d '\r' < ${killPath} > ${killPath}.tmp && mv ${killPath}.tmp ${killPath}"
							sh "chmod +x ${killPath}"
							// sh './jenkins/scripts/kill.sh'
							sh "${killPath}"
						}
						
					}
				}
				stage('Headless Browser Test') {
					agent {
						docker {
							image 'maven:3-alpine' 
							args '-v /root/.m2:/root/.m2' 
						}
					}
					steps {
						sh 'mvn -B -DskipTests clean package'
						sh 'mvn test'
					}
					post {
						always {
							junit 'target/surefire-reports/*.xml'
						}
					}
				}
			}
		}
	}
}