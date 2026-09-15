node('assignment-4-agent') {

    properties([
        parameters([
            booleanParam(
                name: 'SKIP_STABILITY',
                defaultValue: false,
                description: 'Skip Code Stability Scan (Checkstyle)'
            ),
            booleanParam(
                name: 'SKIP_QUALITY',
                defaultValue: false,
                description: 'Skip Code Quality Analysis (SonarQube)'
            ),
            booleanParam(
                name: 'SKIP_COVERAGE',
                defaultValue: false,
                description: 'Skip Code Coverage Analysis (JaCoCo)'
            )
        ])
    ])

    try {

        stage('Code Checkout') {

            deleteDir()

            echo 'Checking out source code...'

            checkout scm

            echo 'Source code checkout completed.'
        }


        stage('Parallel Code Scans') {

            def parallelStages = [:]


            if (!params.SKIP_STABILITY) {

                parallelStages['Code Stability'] = {

                    stage('Code Stability') {

                        echo 'Running Code Stability Scan using Checkstyle...'

                        withEnv([
                            'JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64',
                            "PATH=/usr/lib/jvm/java-11-openjdk-amd64/bin:${env.PATH}"
                        ]) {

                            sh '''
                                echo "Java version:"
                                java -version

                                echo "Maven version:"
                                mvn -version

                                mvn checkstyle:checkstyle
                            '''
                        }

                        echo 'Code Stability Scan completed.'
                    }
                }
            }


            if (!params.SKIP_QUALITY) {

                parallelStages['Code Quality'] = {

                    stage('Code Quality') {

                        echo 'Running Code Quality Analysis using SonarQube...'

                        withEnv([
                            'JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64',
                            "PATH=/usr/lib/jvm/java-21-openjdk-amd64/bin:${env.PATH}"
                        ]) {

                            withSonarQubeEnv('SonarQube') {

                                sh '''
                                    echo "Java version:"
                                    java -version

                                    echo "Maven version:"
                                    mvn -version

                                    mvn -DskipTests \
                                    -Dfindbugs.skip=true \
                                    compile \
                                    org.sonarsource.scanner.maven:sonar-maven-plugin:3.10.0.2594:sonar \
                                    -Dsonar.projectKey=spring3hibernate \
                                    -Dsonar.projectName=spring3hibernate
                                '''
                            }
                        }

                        echo 'Code Quality Analysis completed.'
                    }
                }
            }


            if (!params.SKIP_COVERAGE) {

                parallelStages['Code Coverage'] = {

                    stage('Code Coverage') {

                        echo 'Running Code Coverage Analysis using JaCoCo...'

                        withEnv([
                            'JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64',
                            "PATH=/usr/lib/jvm/java-11-openjdk-amd64/bin:${env.PATH}"
                        ]) {

                            sh '''
                                echo "Java version:"
                                java -version

                                echo "Maven version:"
                                mvn -version

                                mvn clean \
                                org.jacoco:jacoco-maven-plugin:0.8.13:prepare-agent \
                                test \
                                org.jacoco:jacoco-maven-plugin:0.8.13:report
                            '''
                        }

                        echo 'Code Coverage Analysis completed.'
                    }
                }
            }


            if (parallelStages.isEmpty()) {
                error('At least one code scan must be enabled.')
            }

            parallel parallelStages
        }


        stage('Generate Reports') {

            echo 'Generating reports...'

            junit(
                testResults: 'target/surefire-reports/*.xml',
                allowEmptyResults: true
            )

            publishHTML(
                target: [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'target/site',
                    reportFiles: 'checkstyle.html',
                    reportName: 'Checkstyle Report'
                ]
            )

            publishHTML(
                target: [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'target/site/jacoco',
                    reportFiles: 'index.html',
                    reportName: 'JaCoCo Coverage Report'
                ]
            )

            echo 'Reports generated successfully.'
        }


        stage('Approval for Publication') {

            echo 'Waiting for approval before publishing artifact...'

            input(
                message: 'Approve artifact publication?',
                ok: 'Approve'
            )

            echo 'Artifact publication approved.'
        }


        stage('Publish Artifacts') {

            echo 'Building WAR artifact...'

            withEnv([
                'JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64',
                "PATH=/usr/lib/jvm/java-11-openjdk-amd64/bin:${env.PATH}"
            ]) {

                sh '''
                    mvn package -DskipTests
                '''
            }

            echo 'Publishing WAR artifact...'

            archiveArtifacts(
                artifacts: 'target/*.war',
                fingerprint: true
            )

            echo 'Artifact published successfully.'
        }


        echo '========================================'
        echo 'PIPELINE SUCCESS'
        echo 'Artifact published successfully.'
        echo '========================================'


        emailext(
            to: 'prajwaldekate6@gmail.com',
            subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """Build successful.

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Artifact: WAR published successfully.
Build URL: ${env.BUILD_URL}
"""
        )


    } catch (err) {

        echo '========================================'
        echo 'PIPELINE FAILED OR ABORTED'
        echo '========================================'


        emailext(
            to: 'prajwaldekate6@gmail.com',
            subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """Build failed or was aborted.

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Please check Jenkins console output.
Build URL: ${env.BUILD_URL}
"""
        )


        throw err
    }
}
