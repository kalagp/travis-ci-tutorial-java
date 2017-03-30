pipeline {
    agent {
        label 'builder-06'
        //        label 'maven-builder'
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
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        stage('SonarQube Analysis') {
            steps{
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
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
//                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'nexb-scancode']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/nexB/scancode-toolkit.git']]])
//                sh "mkdir nexb-scancode"
                dir('sample'){
                    dir('nexb-scancode'){
                        git "https://github.com/nexB/scancode-toolkit.git" 
                    }
                }
//                sh "cd ${WORKSPACE}/nexb-scancode"  
                sh "./nexb-scancode/scancode --help"
                sh "./nexb-scancode/scancode --format html-app ${WORKSPACE}/ scancode_result.html"
                sh "./nexb-scancode/scancode --format html ${WORKSPACE}/ minimal.html"
                archiveArtifacts 'nexb-scancode/scancode_result_files/,**/scancode_result.html,**/minimal.html'
            }
        }
    }
}
