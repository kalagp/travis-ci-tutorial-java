pipeline {
    agent {
        node{
//          label 'maven-builder'
            label 'builder-06'
            customWorkspace "workspace/${env.JOB_NAME}"
            deleteDir(workspace/${env.JOB_NAME})
            }
    }
    environment {
        //GIT_CREDS = credentials('github-04')
        GITHUB_TOKEN = credentials('github-04')
    }
    tools {
        maven 'linux-maven-3.3.9'
        jdk 'linux-jdk1.8.0_102'
    }
    options { 
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timestamps()
//        properties([[$class: 'BuildBlockerProperty', blockLevel: <object of type hudson.plugins.buildblocker.BuildBlockerProperty.BlockLevel>, blockingJobs: 'simple-build-for-pipeline-plugin', scanQueueFor: <object of type hudson.plugins.buildblocker.BuildBlockerProperty.QueueScanScope>, useBuildBlocker: true], [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false], pipelineTriggers([])])
//        [$class: 'BuildBlockerProperty', blockLevel: <object of type hudson.plugins.buildblocker.BuildBlockerProperty.BlockLevel>, blockingJobs: 'root-parent', scanQueueFor: <object of type hudson.plugins.buildblocker.BuildBlockerProperty.QueueScanScope>, useBuildBlocker: true]
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
                        sh '''
                            mvn sonar:sonar \
                                -Dsonar.host.url=$SONAR_HOST_URL \
                                -Dsonar.login=$SONAR_AUTH_TOKEN \
                                -Dsonar.java.coveragePlugin=jacoco \
                                -Dsonar.junit.reportsPath=target/surefire-reports \
                                -Dsonar.jacoco.reportPath=target/coverage-reports/jacoco-ut.exec \
                                -Dsonar.jacoco.itReportPath=target/coverage-reports/jacoco-it.exec \
                                -Dsonar.dependencyCheck.reportPath=${WORKSPACE}/report/dependency-check-report.xml
                        '''
                }
            }
        }
        stage('Third Party Audit'){
            steps{
                  sh "echo skipping this"
//                    sh "mvn org.apache.maven.plugins:maven-dependency-plugin:2.10:analyze-report license:add-third-party org.apache.maven.plugins:maven-dependency-plugin:2.10:tree -DoutputType=dot -DoutputFile=${WORKSPACE}/report//dependency-tree.dot"
//                    archiveArtifacts 'target/generated-sources/license/*,report/*'
            }
        }
        stage('NexB Scan'){
            steps{
                dir('/opt') {
                    checkout([$class: 'GitSCM', 
                              branches: [[name: '*/master']], 
                              doGenerateSubmoduleConfigurations: false, 
                              extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'nexB-scancode']], 
                              submoduleCfg: [], 
                              userRemoteConfigs: [[url: 'https://github.com/nexB/scancode-toolkit.git']]])
                }
                dir('nexb-output'){
                    sh "sh /opt/nexB-scancode/scancode --help"
                    sh "sh /opt/nexB-scancode/scancode --format html-app ${WORKSPACE} scancode_result.html"
                    sh "sh /opt/nexB-scancode/scancode --format html ${WORKSPACE} minimal.html"
                }
                archiveArtifacts '**/nexb-output/**'
            }
        }
        stage('Copy Artifacts'){
            steps{
                step([$class: 'CopyArtifact', projectName: 'nexb-scan-test', filter: 'output/concerto/*'])
            }
        }
        stage('Github Release'){
            when{
                expression{
                    return env.BRANCH_NAME ==~ /master|sample_branch|release\/.*/
                }
            }
            steps{
                dir('/opt') {
                    sh "rm -f linux-amd64-github-release.tar.bz2"
                    sh "wget https://github.com/aktau/github-release/releases/download/v0.7.2/linux-amd64-github-release.tar.bz2"
                    sh "tar -xvjf linux-amd64-github-release.tar.bz2"
                    sh "yes | cp bin/linux/amd64/github-release /usr/bin/"
                    sh '''
                        github-release release \
                            --user chamap1 \
                            --repo travis-ci-tutorial-java \
                            --tag v0.0.1-${BRANCH_NAME}-${BUILD_ID} \
                            --name "travis-ci-tutorial-java" \
                            --description "travis-ci-tutorial-java release"
                        github-release upload \
                            --user chamap1 \
                            --repo travis-ci-tutorial-java \
                            --tag v0.0.1-${BRANCH_NAME}-${BUILD_ID} \
                            --name "travis-ci-tutorial-java" \
                            --file ${WORKSPACE}/target/travis-ci-tutorial.jar
                    '''
                }
            }
        }
    }
    post{
        always{
            emailext attachLog: true, body: 'This is a sample body', recipientProviders: [[$class: 'CulpritsRecipientProvider']], subject: 'Jenkins Job ${JOB_NAME}(${BUILD_NUMBER})', to: 'purna.chamala@vce.com'
        }
        success{
            //steps to be performed
            sh "echo Do Nothing"
        }
        failure{
            //steps to be performed
            sh "echo Do Nothing"
        }
        unstable{
            //steps to be performed
            sh "echo Do Nothing"
        }
    }
}
