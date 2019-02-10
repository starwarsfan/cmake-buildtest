#!groovy

pipeline {
    agent {
        label "docker"
    }
    options {
        timestamps()
        timeout(time: 3, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '5'))
        disableConcurrentBuilds()
    }
    environment {
        // In case another branch beside master or develop should be deployed, enter it here
        BRANCH_TO_DEPLOY = 'xyz'
        GITHUB_TOKEN = credentials('GitHub-Token')
        DEVELOP_TAG = "Build${BUILD_NUMBER}"
        RELEASE_TAG = '1.0.1'
        GIT_TAG_TO_USE = "${DEVELOP_TAG}"
        GIT_COMMIT_SHORT = sh(
                script: "printf \$(git rev-parse --short ${GIT_COMMIT})",
                returnStdout: true
        )
        RELEASE_NAME = "Continuous build #${BUILD_NUMBER}"
        RELEASE_DESCRIPTION = "Build ${BUILD_NUMBER}"
        PRERELEASE = "true"
    }
    stages {
        stage('Feature branch') {
            // Only build the application, no upload a/o deployment to anywhere
            when {
                not {
                    anyOf { branch 'develop'; branch 'master'; branch "${BRANCH_TO_DEPLOY}" }
                }
            }
            //noinspection GroovyAssignabilityCheck
            parallel {
                stage('Build Debian binaries') {
                    steps {
                        script {
                            buildFeatureBranch('Docker/Debian/Dockerfile_noUpload', "starwarsfan/cmake-buildtest-debian:${GIT_TAG_TO_USE}")
                        }
                    }
                }
                stage('Build Fedora binaries') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            buildFeatureBranch('Docker/Fedora/Dockerfile_noUpload', "starwarsfan/cmake-buildtest-fedora:${GIT_TAG_TO_USE}")
                        }
                    }
                }
            }
        }
        stage('Prepare master branch build') {
            when {
                branch 'master'
            }
            stages {
                stage('Setup env vars to use') {
                    steps {
                        script {
                            GIT_TAG_TO_USE = "${RELEASE_TAG}"
                            RELEASE_NAME = "Release ${GIT_TAG_TO_USE}"
                            RELEASE_DESCRIPTION = "${WORKSPACE}/Releasenotes.md"
                            PRERELEASE = "false"
                        }
                    }
                }
            }
        }
        stage('Git tag handling') {
            when {
                anyOf { branch 'master'; branch 'develop'; branch "${BRANCH_TO_DEPLOY}" }
            }
            // We are on a branch from which the results should be uploaded to GitHub
            // So at first prepare the release tag on GitHub
            stages {
                stage('Create Git tag') {
                    steps {
                        sshagent(credentials: ['GitHub-SSH']) {
                            createTag(
                                    tag: "${GIT_TAG_TO_USE}",
                                    commit: "${GIT_COMMIT_SHORT}",
                                    comment: "Created tag ${GIT_TAG_TO_USE}"
                            )
                        }
                    }
                }
                stage('Remove Github release if already existing') {
                    when {
                        expression {
                            return isReleaseExisting(
                                    user: 'starwarsfan',
                                    repository: 'cmake-buildtest',
                                    tag: "${GIT_TAG_TO_USE}"
                            ) ==~ true
                        }
                    }
                    steps {
                        script {
                            removeRelease(
                                    user: 'starwarsfan',
                                    repository: 'cmake-buildtest',
                                    tag: "${GIT_TAG_TO_USE}"
                            )
                        }
                    }
                }
                stage('Create Github release') {
                    when {
                        expression {
                            return isReleaseExisting(
                                    user: 'starwarsfan',
                                    repository: 'cmake-buildtest',
                                    tag: "${GIT_TAG_TO_USE}"
                            ) ==~ false
                        }
                    }
                    steps {
                        script {
                            createRelease(
                                    user: 'starwarsfan',
                                    repository: 'cmake-buildtest',
                                    tag: "${GIT_TAG_TO_USE}",
                                    name: "${RELEASE_NAME}",
                                    description: "${RELEASE_DESCRIPTION}",
                                    preRelease: "${PRERELEASE}"
                            )
                        }
                    }
                }
            }
        }
        stage('Build steps') {
            when {
                anyOf { branch 'master'; branch 'develop'; branch "${BRANCH_TO_DEPLOY}" }
            }
            // We are on a branch from which the results should be uploaded to GitHub
            // Now build (and upload) the different binaries
            //noinspection GroovyAssignabilityCheck
            parallel {
                stage('Build Debian binaries') {
                    stages {
                        stage('Build Debian binaries') {
                            steps {
                                script {
                                    buildBranch('Docker/Debian/Dockerfile', "starwarsfan/cmake-buildtest-debian:${GIT_TAG_TO_USE}", "${GIT_TAG_TO_USE}", "${GIT_COMMIT_SHORT}")
                                }
                            }
                        }
                    }
                }
                stage('Build Fedora binaries') {
                    agent {
                        label "docker"
                    }
                    steps {
                        script {
                            buildBranch('Docker/Fedora/Dockerfile', "starwarsfan/cmake-buildtest-fedora:${GIT_TAG_TO_USE}", "${GIT_TAG_TO_USE}", "${GIT_COMMIT_SHORT}")
                        }
                    }
                }
                stage('Mac') {
                    agent any
                    steps {
                        script {
                            sh "echo 'Not yet implemented...'"
                        }
                    }
                }
                stage('Windows') {
                    agent any
                    steps {
                        script {
                            sh "echo 'Not yet implemented...'"
                        }
                    }
                }
            }
        }
    }
}

def buildFeatureBranch(String dockerfile, String tag) {
    withDockerRegistry(credentialsId: 'Docker-Registry') {
        sh(
                script: """
                    docker build \\
                        -f ${dockerfile} \\
                        --rm \\
                        -t ${tag} \\
                        .
                """
        )
    }
}

def buildBranch(String dockerfile, String dockerTag, String gitTag, String gitCommit) {
    withDockerRegistry(credentialsId: 'Docker-Registry') {
        sh(
                script: """
                    docker build \\
                        -f ${dockerfile} \\
                        --rm \\
                        --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} \\
                        --build-arg GIT_COMMIT=${gitCommit} \\
                        --build-arg RELEASE=${gitTag} \\
                        --build-arg REPLACE_EXISTING_ARCHIVE=--replace \\
                        -t ${dockerTag} \\
                        .
                """
        )
    }
}
