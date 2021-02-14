///////////////////////////////////////////////////////////////////
//  Groovy Jenkins Script
//
//  Script to build jenkins job
///////////////////////////////////////////////////////////////////

def CheckoutRepo(String gitUrl, String branchName) {
    git( [url: gitUrl, branch: branchName] )
}

node {
    // List of variables needed to build
    // Update these as needed prior to building
    def GIT_URL = 'https://github.com/johnyo97/onedev.git'
    def BRANCH_NAME = 'main'

    // List collecting errors, exceptions thrown
    def errorMessages = []

    stage("Clean local directory") {
        deleteDir()
    }
    stage("Clone repo") {
        try {
			CheckoutRepo(GIT_URL, BRANCH_NAME)
        }
        catch(Exception ex) {
            errorMessages.add(ex.toString())
        }
    }
    stage("Build - 'mvn install'") {
        if(errorMessages.size() != 0) {
			// We don't want to build if something wrong happened before this stage
            String errors = errorMessages.join(",")
            throw new Exception(errors)
        }
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
}