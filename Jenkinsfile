pipeline {
    agent any


    environment {
        sonarAnalyzeResult = 'FAILURE'
		testRunResult = 'FAILURE'
    }

	stages {
	    stage ('Checkout scm') {
	        steps {
	            script {
	                checkout scm: [$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/mike-podolskiy90/java-junit-sample.git/']]]
	            }
	        }
	    }


		stage('SonarQube analysis') {
		    steps {
		        script {
			        sonarAnalyzeResult = build(job: "task7-sonar-analyze", propagate: false).result
			        echo "First job result: $sonarAnalyzeResult"

			        if(sonarAnalyzeResult == 'FAILURE') {
				        currentBuild.result = 'UNSTABLE'
			        }
		        }
		    }
		}

		stage('Tests run') {
		    steps {
		        script {
			        testRunResult = build(job: "task7-test-run", propagate: false).result
			        echo "Second job result: $testRunResult"

			        if(testRunResult == 'FAILURE') {
				        currentBuild.result = 'UNSTABLE'
			        }
		        }
		    }
		}

		stage('Check result') {
		    steps {
			    script {
				    if(sonarAnalyzeResult == 'UNSTABLE' && testRunResult == 'UNSTABLE') {
				    	currentBuild.result = 'FAILURE'
				    }


				    if (currentBuild.currentResult == 'FAILURE') { // Other values: SUCCESS, UNSTABLE

                        def checkLastCommit = bat (
                            script: "git show -s --format=short HEAD",
                            returnStdout: true
                        )

                        echo "$checkLastCommit"

                        def authorEmail = bat (
                            script: "git show -s --format='%%ae' HEAD",
                            returnStdout: true
                        ).split('\r\n')[2].trim()

                        echo "$authorEmail"

                        echo "The last commit was written by ${authorEmail}."

                        // Send an email only if the build status has changed from green/unstable to red
                        emailext subject: '$DEFAULT_SUBJECT',
                            body: '$DEFAULT_CONTENT',
                            recipientProviders: [
                                [$class: 'CulpritsRecipientProvider'],
                                [$class: 'DevelopersRecipientProvider'],
                                [$class: 'RequesterRecipientProvider']
                            ],
                            replyTo: '$DEFAULT_REPLYTO',
                            to: "${authorEmail}"
                    }

		    	}
		    }
		}
	}

}