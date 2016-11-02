def formElementPathVersion
node('hi-speed') {
    stage('plugin') {
        dir('plugin') {
            checkout scm
            timeout(30) {
                withMaven(jdk: 'Oracle JDK 1.8 (latest)',
                        maven: 'Maven 3.2.1') {
                    sh 'mvn clean install -B'
                    formElementPathVersion = sh script: 'cat target/form-element-path/META-INF/MANIFEST.MF | grep Implementation-Version | cut -f 2 -d ":" | xargs', returnStdout: true
                }
            }
        }
    }
    dir('ath') {
        stage('Acceptance Tests') {
            git branch: 'JENKINS-38928', changelog: false, poll: false, url: 'git@github.com:Vlatombe/acceptance-test-harness.git'
            step([$class: 'CopyArtifact',
                  filter: 'war/target/jenkins.war',
                  fingerprintArtifacts: true,
                  flatten: true,
                  projectName: 'jenkins_lts_branch',
                  selector: [
                          $class: 'StatusBuildSelector',
                          stable: false
                  ]
            ])
            wrap([$class: 'Xvnc', takeScreenshot: false, useXauthority: true]) {
                withMaven(jdk: 'Oracle JDK 8 (latest)',
                        maven: 'Maven 3.2.1',
                        mavenSettingsConfig: 'org.jenkinsci.plugins.configfiles.maven.MavenSettingsConfig1382087369423',
                        mavenSettingsFilePath: 'settings.xml') {
                    withEnv(['BROWSER=firefox',
                             'JENKINS_WAR=jenkins.war',
                             'JENKINS_JAVA_HOME=/opt/jdk/jdk1.7.latest/',
                             'JAVA_HOME=/opt/jdk/openjdk8.latest/',
                             'EXERCISEDPLUGINREPORTER=textfile',
                             "FORM_ELEMENT_PATH_VERSION=$formElementPathVersion"]) {
                        sh '''
unzip -p $JENKINS_WAR META-INF/MANIFEST.MF | perl -p -0777 -e 's/\\r?\\n //g' | fgrep Jenkins-Version
cp $SAUCE_SECRET ~/.sauce-ondemand || true

firefox --version
mvn clean test -Dmaven.test.failure.ignore=true -DforkCount=1 -Dtest=FreestyleJobTest -B
'''
                    }
                }
            }
            archiveArtifacts artifacts: 'target/exercised-plugins.properties', excludes: null
        }
    }
}
