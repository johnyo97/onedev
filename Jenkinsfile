///////////////////////////////////////////////////////////////////
//  Groovy Jenkins Script
//
//  Script to build jenkins job
///////////////////////////////////////////////////////////////////

node {
    // List collecting errors, exceptions thrown
    def errorMessages = []

    stage("Build - 'mvn install'") {
		// Switch and build using Maven on CLI
		try {
			bat 'Z:/apache-maven-3.6.3/bin/mvn install'
		}
		catch (Exception ex) {
			// We still want to catch error that could occur during the build phase
			errorMessages.add(ex.toString())
		}
    }
	stage("Unit Test - 'mvn test'") {
		try {
			bat 'Z:/apache-maven-3.6.3/bin/mvn test'
		}
		catch (Exception ex) {
			// We want to catch errors at this stage as well
			errorMessages.add(ex.toString);
		}
	}
	stage("Regression Test") {
		try {
			// We do not implement this test
		}
		catch (Exception ex) {
			// We want to catch errors at this stage as well
			errorMessages.add(ex.toString);
		}
	}
	stage("Integration Test") {
		try {
			// We do not implement this test
		}
		catch (Exception ex) {
			// We want to catch errors at this stage as well
			errorMessages.add(ex.toString);
		}
	}
	stage("Status") {
		// We need to verify whether or not exceptions were thrown
		if (errorMessages.size() != 0) {
			// If exceptions were thrown, we'll set the build result to 'FAILURE'
			currentBuild.result = 'FAILURE'
		} else { 
			// Otherwise, we'll set the result to 'SUCCESS'
			currentBuild.result = 'SUCCESS'
		}
	}
}