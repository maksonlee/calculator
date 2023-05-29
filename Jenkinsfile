podTemplate(cloud: 'kubernetes', label: 'test', yaml: '''
apiVersion: v1
kind: Pod
metadata:
  name: test
  namespace: jenkins
spec:
  containers:
  - name: jdk17
    image: amazoncorretto:17
    command:
    - sleep
    args:
    - 99d
    volumeMounts:
      - name: jenkins-agent-gradle-config-volume
        mountPath: /root/.gradle/init.gradle
        subPath: init.gradle
  - name: docker-build
    image: moby/buildkit:master
    command:
    - sleep
    args:
    - 99d
    securityContext:
      privileged: true
    env:
    - name: DOCKER_CONFIG
      value: "/home/jenkins/.docker"
    volumeMounts:
      - name: jenkins-docker-config-volume
        mountPath: /home/jenkins/.docker
      - name: jenkins-agent-docker-buildkit-config-volume
        mountPath: /etc/buildkit
  - name: kubectl
    image: bitnami/kubectl:1.26
    command:
    - sleep
    args:
    - 99d
    securityContext:
      runAsUser: 1000
      runAsGroup: 1000
  volumes:
  - name: jenkins-docker-config-volume
    secret:
      defaultMode: 0600
      secretName: docker-regcred
      items:
      - key: .dockerconfigjson
        path: config.json
  - name: jenkins-agent-docker-buildkit-config-volume
    configMap:
      name: jenkins-agent-docker-buildkit-config
  - name: jenkins-agent-gradle-config-volume
    configMap:
      name: jenkins-agent-gradle-config
''') {
    node('test') {
        container('jdk17') {
            stage('Checkout') {
                git 'https://github.com/maksonlee/calculator.git'
            }
            stage('Compile') {
                sh "./gradlew compileJava"
            }
            stage("Unit test") {
                sh "./gradlew test"
            }
            stage('Package') {
                sh "./gradlew build"
            }
        }
        container('docker-build') {
            stage('Docker build and push') {
                sh "buildctl-daemonless.sh build --frontend dockerfile.v0 -local context=./ --local dockerfile=./ --output type=image,name=docker.arimacomm.com.tw/test/calculator,push=true"
            }
            
        }
        container('kubectl') {
            stage('Deploy to staging') {
                withKubeConfig([namespace: "jenkins"]) {
                    sh "kubectl apply -f app.yaml"
                }
            }
        }
        stage("Acceptance test") {
            sh "./acceptance_test.sh"
        }
    }
}
