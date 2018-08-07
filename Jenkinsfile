#!/usr/bin/groovy
@Library('TAP-PIPELINE-LIBRARY')

def dummy

import com.tod.CustomPipelineUtil

mavenNode {
    ws {
        env.setProperty('FABRIC8_DOCKER_REGISTRY_SERVICE_HOST', registryHost)
        env.setProperty('FABRIC8_DOCKER_REGISTRY_SERVICE_PORT', registryPort)
        checkout scm

        def customConfigStr = readFile "${env.JOB_NAME}Config.json" // 读取配置文件
        def customConfig = CustomPipelineUtil.getJsonPipelineConfig(customConfigStr)
        def docker_image = ''
        def proj_version = "${customConfig.version}.${env.BUILD_NUMBER}"

        container(name: 'maven') {
            stage('build with config') {
                buildWithConfig {
                    custom = customConfig
                }
            }

            stage('docker build and push image') {
                docker_image = customMavenRelease {
                    version = proj_version
                    pom = customConfig.build.pomfile
                    dockerfilePath = customConfig.build.dockerfile
                }
                echo "Docker image : ${docker_image}"
            }

            stage('deploy application') {
                def yaml = getCustomK8SYaml{
                    deployment = customConfig.deployment
                    version = proj_version
                    image = docker_image
                }
                kubernetesApply(file: yaml, environment: DEPLOY_NAMESPACE)
            }
        }
    }
}
