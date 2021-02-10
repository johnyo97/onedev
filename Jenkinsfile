///////////////////////////////////////////////////////////////////
//  Groovy Jenkins Script
//
//  Script to build jenkins job
//  Script uses sshagent plugin (see SSH Agent Plugin on Jenkins)
//  to connect to git repo using ssh (for now there doesn't seem to
//  be a way to use the integrated 'Git Publisher' plugin through a
//  Groovy script)
///////////////////////////////////////////////////////////////////

def CheckoutRepo(String gitUrl, String gitCredentials, String branch) {
    git(
        url: gitUrl,
        credentialsId: gitCredentials,
        branches: [[name: "*/${branch}"]]
    )
}

def PushToRepo(String credentials, String branch) {
    sshagent([credentials]) {
        sh "git push origin ${branch}"
    }
}

def PullRepo(String credentials, String branch) {
    sshagent([credentials]) {
        sh "git pull --ff-only origin ${branch}"
    }
}

def CommitToRepo(String message) {
    sh "git commit -am '${message}'"
}

String GetCurrentBuildVersion(String filePath) {
    String version = new File(filePath).text
    if(version == null) {
        throw new Exception("File containing the version was empty")
    }
    return version
}

String ChangeBuildVersion(String filePath, String currentVersion) {
    if(currentVersion.length() == 0) {
        throw new Exception("Could not change build version, currentVersion string was empty")
    }

    String[] numbers = currentVersion.split('\\.')
    String newVersion = ''
    numbers[3] = (numbers[3].toInteger() + 1).toString()

    newVersion = "${numbers[0]}.${numbers[1]}.${numbers[2]}.${numbers[3]}"
    File file = new File(filePath)
    // This overwrites the file containing the version
    file.write(newVersion)
    CommitToRepo("Automatic commiting of new build version ${newVersion}")

    return newVersion
}

String CreateTag(String credentials, String currentVersion) {
    if(currentVersion.length() == 0) {
        throw new Exception("Couldn't create a tag, currentVersion string missing")
    }
    String[] numbers = currentVersion.split('\\.')
    String tagName = "${numbers[0]}.${numbers[1]}/${currentVersion}"

    sshagent([credentials]) {
        String gitTags = sh(returnStdout: true, script: "git tag -l").trim()
        try {
            if(!gitTags.contains(tagName)) {
                sh "git tag -a ${tagName} -m 'Automatic creation of tag ${tagName} for version ${currentVersion}'"
                sh "git push origin ${tagName}"
            }
        }
        catch(Exception ex) {
            String error = ex.toString()
            throw new Exception("Couldn't create tag name ${tagName}, ${error}")
        }
    }
    return tagName
}

def ResetToTag(String tag) {
    sh "git reset --hard ${tag}"
}

node {
    // List of variables needed to build
    // Update these as needed prior to building
    def GIT_CREDENTIALS = 'f3ed11b9-1c3d-43aa-a15b-bbf64eda6712'
    def GIT_URL = 'git@github.com:johnyo97/jenkins-test.git'
    def FILE_PATH = pwd() + "/version.txt"
    def BRANCH_NAME = 'master'

    // List of variables that will be set during the build
    // Do not need to be updated prior to build
    def CURRENT_VERSION
    def NEW_VERSION
    def TAG_NAME

    // List collecting errors, exceptions thrown
    def errorMessages = []

    stage("Clean local directory") {
        deleteDir()
    }
    stage("Clone repo") {
        CheckoutRepo(GIT_URL, GIT_CREDENTIALS, BRANCH_NAME)
        try {
            CURRENT_VERSION = GetCurrentBuildVersion(FILE_PATH)
        }
        catch(Exception ex) {
            errorMessages.add(ex.toString())
        }
    }
    stage("Create a tag") {
        try {
            TAG_NAME = CreateTag(GIT_CREDENTIALS, CURRENT_VERSION)
        }
        catch(Exception ex) {
            errorMessages.add(ex.toString())
        }
    }
    stage("Change Build Version") {
        try {
            NEW_VERSION = ChangeBuildVersion(FILE_PATH, CURRENT_VERSION)
            PushToRepo(GIT_CREDENTIALS, BRANCH_NAME)
            ResetToTag(TAG_NAME)
        }
        catch(Exception ex) {
            errorMessages.add(ex.toString())
        }
    }
    stage("Build") {
        if(errorMessages.size() != 0) {
            String errors = errorMessages.join(",")
            throw new Exception(errors)
        }
        sh 'xcodebuild -sdk iphonesimulator12.2 -destination "platform=iOS Simulator,OS=12.2,name=iPhone X" -configuration Release build'
    }
    stage("Check Build Folder") {
        def BUILD_FOLDER_PATH = '/Users/Shared/Jenkins/Home/workspace/jenkins-test-a-pipeline/build'
        def RELEASE_FOLDER_PATH = BUILD_FOLDER_PATH + '/Release-iphonesimulator'
        def EXECUTABLE_PATH = RELEASE_FOLDER_PATH + '/jenkins-test.app'

        File buildFolder = new File(BUILD_FOLDER_PATH)

        if(!buildFolder.exists()) {
            currentBuild.result = 'FAILURE'
            println "PROJECT WAS NOT SUCCESSFULLY BUILT"
        }
        else {
            sh "cd ${BUILD_FOLDER_PATH}"
            File filePackage = new File(RELEASE_FOLDER_PATH)
            if(!filePackage.exists()) {
                currentBuild.result = 'FAILURE'
                println "PROJECT WAS BUILT SUCCESSFULLY BUT PACKAGE WAS NOT GENERATED"
            }
            else {
                sh "cd ${RELEASE_FOLDER_PATH}"
                File executable = new File(EXECUTABLE_PATH)
                if(!executable.exists()) {
                    currentBuild.result = 'FAILURE'
                    println "PROJECT WAS BUILT SUCCESSFULLY BUT EXECUTABLE WAS NOT GENERATED"
                }
                else {
                    currentBuild.result = 'SUCCESS'
                    println "PROJECT WAS SUCCESSFULLY BUILT"
                }
            }
        }
    }
    stage("Reset git head") {
        PullRepo(GIT_CREDENTIALS, BRANCH_NAME)
    }
}