pipeline {
    agent {
        node{
//          label 'maven-builder'
            label 'builder-06'
            customWorkspace "workspace/${env.JOB_NAME}"
            }
    }
    environment {
        GIT_CREDS = credentials('github-04')
        //GITHUB_TOKEN = credentials('github-04')
    }
    tools {
        maven 'linux-maven-3.3.9'
        jdk 'linux-jdk1.8.0_102'
    }
    stages {
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Deploy') {
            steps {
                sh "mvn clean install"
                sh "echo This is not deploying to any remote repository. Just for testing purpose."
            }
        }
        stage('Unit Tests') {
            steps {
                sh "mvn test"
            }
        }
        stage('Integration Tests') {
            steps {
                sh "echo No Integration tests defined for this repo!"
            }
        }
        stage('SonarQube Analysis') {
            steps{
                withSonarQubeEnv('SonarQube') { 
                    sh 'mvn sonar:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=f4c66c76562511e089a6fb02426c08e45e78e3d6 -Dsonar.java.coveragePlugin=jacoco -Dsonar.junit.reportsPath=target/surefire-reports -Dsonar.jacoco.reportPath=target/coverage-reports/jacoco-ut.exec -Dsonar.jacoco.itReportPath=target/coverage-reports/jacoco-it.exec -Dsonar.dependencyCheck.reportPath=${WORKSPACE}/report/dependency-check-report.xml'
 //                   sh 'mvn sonar:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=admin -Dsonar.password=Vcebrm01'
                }
            }
        }
        stage('Third Party Audit'){
            steps{
                sh "mvn org.apache.maven.plugins:maven-dependency-plugin:2.10:analyze-report license:add-third-party org.apache.maven.plugins:maven-dependency-plugin:2.10:tree -DoutputType=dot -DoutputFile=${WORKSPACE}/report//dependency-tree.dot"
                archiveArtifacts 'target/generated-sources/license/*,report/*'
            }
        }
        stage('NexB Scan'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'nexb-scancode']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/nexB/scancode-toolkit.git']]])
                sh "./nexb-scancode/scancode --help"
                sh "./nexb-scancode/scancode --format html-app ${WORKSPACE}/src/ scancode_result.html"
                sh "./nexb-scancode/scancode --format html ${WORKSPACE}/src/ minimal.html"
                archiveArtifacts 'nexb-scancode/scancode_result_files/,**/scancode_result.html,**/minimal.html'
            }
        }
        stage('Copy Artifacts'){
            steps{
                step([$class: 'CopyArtifact', projectName: 'nexb-scan-test', filter: 'output/concerto/*'])
            }
        }
        stage('Github Release'){
            steps{
                sh "rm -f linux-amd64-github-release.tar.bz2"
                sh "wget https://github.com/aktau/github-release/releases/download/v0.7.2/linux-amd64-github-release.tar.bz2"
                sh "tar -xvjf linux-amd64-github-release.tar.bz2"
                sh "yes | cp bin/linux/amd64/github-release /usr/bin/"
                sh '''
                    github-release release \
                        --user chamap1 \
                        --repo  travis-ci-tutorial-java \
                        --tag v0.0.1-${BUILD_ID} \
                        --name "travis-ci-tutorial-java release" \
                        --description "travis-ci-tutorial-java release"
                    github-release upload \
                        --user chamap1 \
                        --repo travis-ci-tutorial-java \
                        --tag v0.1.0-${BUILD_ID} \
                        --name "travis-ci-tutorial-java release" \
                        --file /opt/jenkins/workspace/gitorg-test-purna/travis-ci-tutorial-java/master/target/travis-ci-tutorial.jar"
            '''
            }
        }
    }
}
