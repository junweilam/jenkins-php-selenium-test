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
							sh "chmod +x ${scriptPath}"
							sh "ls -l ${scriptPath}"
							sh ".${scriptPath}"
							input message: 'Finished using the web site? (Click "Proceed" to continue)'
							sh './jenkins/scripts/kill.sh'
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