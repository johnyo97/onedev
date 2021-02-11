///////////////////////////////////////////////////////////////////
//  Groovy Jenkins Script
//
//  Script to build jenkins job
//  Script uses sshagent plugin (see SSH Agent Plugin on Jenkins)
//  to connect to git repo using ssh (for now there doesn't seem to
//  be a way to use the integrated 'Git Publisher' plugin through a
//  Groovy script)
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
	def WORKSPACE = pwd()

    stage("Clone Repo") {
        try {
			if (!new File(WORKSPACE).listFiles().length == 0) {
				CheckoutRepo(GIT_URL, BRANCH_NAME)
			}
        }
        catch(Exception ex) {
            errorMessages.add(ex.toString())
        }
    }
    stage("Build - 'mvn install'") {
        if(errorMessages.size() != 0) {
            String errors = errorMessages.join(",")
            throw new Exception(errors)
        }
		// Switch and build using Maven on CLI
        bat 'Z:/apache-maven-3.6.3/bin/mvn install'
    }
	stage("Deployment") {
		dir {
			bat 'Z:/apache-maven-3.6.3/bin/mvn exec:java -Dexec.mainClass="io.onedev.commons.launcher.bootstrap.Bootstrap"'
		}
	}
}