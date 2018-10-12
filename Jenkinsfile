#!/usr/bin/groovy
@Library('TAP-PIPELINE-LIBRARY')

def dummy

import com.tod.CustomPipelineUtil

nodeJsNode {
    ws {
        env.setProperty('FABRIC8_DOCKER_REGISTRY_SERVICE_HOST', registryHost)
        env.setProperty('FABRIC8_DOCKER_REGISTRY_SERVICE_PORT', registryPort)
        checkout scm

        def customConfigStr = readFile "${env.JOB_NAME}Config.json" // 读取配置文件
        def customConfig = CustomPipelineUtil.getJsonPipelineConfig(customConfigStr)
        def docker_image = ''
        def proj_version = "${customConfig.version}.${env.BUILD_NUMBER}"

        container(name: 'builder') {
            stage('build with config') {
                nodeJsBuild {
                    custom = customConfig
                }
            }

            stage('docker build and push image') {
                docker_image = customOtherRelease {
                    version = proj_version
                    dockerfilePath = customConfig.build.dockerfile
                }
                echo "Docker image : ${docker_image}"
            }

           stage('update docker image') {
               def res = applyDockerImage {
                   bind = customConfig.bind
                   image = docker_image
               }
           }
        }
    }
}
