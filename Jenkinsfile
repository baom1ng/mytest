pipeline{
    //制定构建的节点
    agent any
    //声明全局变量
    environment {
		harborRepo = 'repo'
        harborUser = 'admin'
		harborPasswd = 'qwer1.2.3.'
		harborAddress = '192.168.10.31:80'
    }
    stages {
        stage('获取Git仓库代码') {
            steps {
				checkout([$class: 'GitSCM', branches: [[name: '${tag}']], extensions: [], userRemoteConfigs: [[url: 'http://192.168.10.30:3000/root/mytest.git']]])
                }
            }    
            stage('通过maven构建项目') {
            steps {
				sh '/var/jenkins_home/maven/bin/mvn clean package -DskipTests'
                }
            }
            stage('代码质量检测') {
            steps {
				sh '/var/jenkins_home/sonar-scanner/bin/sonar-scanner -Dsonar.source=./ -Dsonar.projectname=${JOB_NAME} -Dsonar.projectKey=${JOB_NAME} -Dsonar.java.binaries=./target/ -Dsonar.login=sqa_45bbb9646f3999d6bae391aa619a0abaeee36d10 '
                }
            }    
            stage('构建Docker镜像') {
            steps {
				sh '''mv target/*.jar docker/
docker build -t ${JOB_NAME}:${tag} ./docker/ '''
                }
            }
            stage('推送Docker镜像到harbor') {
            steps {
                sh '''docker login -u ${harborUser} -p ${harborPasswd} ${harborAddress}
docker tag ${JOB_NAME}:${tag} ${harborAddress}/${harborRepo}/${JOB_NAME}:${tag}
docker push ${harborAddress}/${harborRepo}/${JOB_NAME}:${tag}'''
                }
            }    
            stage('通知目标服务器') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: '10.30', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: "deploy.sh $harborAddress $harborRepo $JOB_NAME $tag $port", execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                }
            }    
        }
}    
