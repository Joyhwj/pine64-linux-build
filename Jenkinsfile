/**
properties([
  parameters([
    string(defaultValue: '1.0', description: 'Current version number', name: 'VERSION'),
    text(defaultValue: '', description: 'A list of changes', name: 'CHANGES'),
    choice(choices: 'all\nkernel-tarball\nlinux-package\nxenial-minimal-pinebook\nxenial-mate-pinebook\nstretch-i3-pinebook\nxenial-pinebook\nlinux-pinebook\nxenial-minimal-pine64\nlinux-pine64\nxenial-minimal-sopine\nlinux-sopine', description: 'What makefile build type to target', name: 'MAKE_TARGET')
    booleanParam(defaultValue: true, description: 'Whether to upload to Github for release or not', name: 'GITHUB_UPLOAD'),
    booleanParam(defaultValue: false, description: 'If build should be marked as pre-release', name: 'GITHUB_PRERELEASE'),
    string(defaultValue: 'joyhwj', description: 'GitHub username or organization', name: 'GITHUB_USER'),
    string(defaultValue: 'build-pine64-image', description: 'GitHub repository', name: 'GITHUB_REPO'),
  ])
])
*/

node('docker_linux-build') {
  timestamps {
    wrap([$class: 'AnsiColorBuildWrapper', colorMapName: 'xterm']) {
      stage('Environment') {
        checkout scm

        def environment = docker.build('build-environment:build-pine64-image', 'build-environment')
        environment.inside("--privileged -u 0:0") {
          withEnv([
            "USE_CCACHE=true",
            "RELEASE_NAME=$env.VERSION",
            "RELEASE=$BUILD_NUMBER"
          ]) {
              stage('Prepare') {
                sh '''#!/bin/bash
                  id
                  set +xe
                  export CCACHE_DIR=$WORKSPACE/ccache
                  ccache -M 0 -F 0
                  git config --global --add safe.directory '*'
                  git clean -ffdx -e ccache
                '''
              }

              stage('Build') {
                sh '''#!/bin/bash
                  set +xe
                  export CCACHE_DIR=$WORKSPACE/ccache
                  make -j4 $MAKE_TARGET
                '''
              }
          }
    
          withEnv([
            "VERSION=$env.VERSION",
            "CHANGES=$env.CHANGES",
            "GITHUB_PRERELEASE=$env.GITHUB_PRERELEASE",
            "GITHUB_USER=$env.GITHUB_USER",
            "GITHUB_REPO=$env.GITHUB_REPO"
          ]) {
            stage('Release') {
              if (params.GITHUB_UPLOAD) { 
                sh '''#!/bin/bash
                  set -xe
                  shopt -s nullglob
                  github-release release \
                      --tag "${VERSION}" \
                      --name "$VERSION: $BUILD_TAG" \
                      --description "${CHANGES}\n\n${BUILD_URL}" \
                      --draft
                  for file in *.xz *.deb; do
                    github-release upload \
                        --tag "${VERSION}" \
                        --name "$(basename "$file")" \
                        --file "$file" &
                  done
                  wait
                  if [[ "$GITHUB_PRERELEASE" == "true" ]]; then
                    github-release edit \
                      --tag "${VERSION}" \
                      --name "$VERSION: $BUILD_TAG" \
                      --description "${CHANGES}\n\n${BUILD_URL}" \
                      --pre-release
                  else
                    github-release edit \
                      --tag "${VERSION}" \
                      --name "$VERSION: $BUILD_TAG" \
                      --description "${CHANGES}\n\n${BUILD_URL}"
                  fi
                '''
              } else {
                 echo 'Flagged as an no upload release job'
              }
            }
          }
        }
      }
    }
  }
}
