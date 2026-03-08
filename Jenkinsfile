pipeline {
    agent any 

    tools {
        maven 'maven 3.8.7' 
    }

    environment {
        DOCKER_IMAGE = "demo-app" 
        CONTAINER_NAME = "springboot-container" 
        JAR_FILE_NAME = "app.jar" 
        PORT = "8081" 
    }

    stages {
        stage('Git checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Prepare Jar') {
            steps {
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        }
        
        stage('Local Docker Build & Deploy') {
            steps {
                // 1. [추가된 핵심 로직] 8081 포트를 점유 중인 프로세스(이전 컨테이너나 수동 실행된 앱)를 찾아 강제 종료합니다.
                // || true 를 붙여서 포트를 쓰고 있는 프로세스가 없더라도 파이프라인이 실패하지 않고 자연스럽게 넘어가게 합니다.
                sh "lsof -t -i:${PORT} | xargs -r kill -9 || true"
                
                // 2. 기존에 떠 있던 동일한 이름의 도커 컨테이너 삭제
                sh "docker rm -f ${CONTAINER_NAME} || true"
                
                // 3. 최신 코드로 도커 이미지 빌드
                sh "docker build -t ${DOCKER_IMAGE} ."
                
                // 4. 8081 포트로 새로운 도커 컨테이너 실행
                sh "docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE}"
            }
        }
    }
}