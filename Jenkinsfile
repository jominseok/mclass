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
                // 리눅스 쉘 명령어(sh)로 복구
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Prepare Jar') {
            steps {
                // 리눅스 파일 복사 명령어(cp) 및 경로 슬래시(/)로 복구
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        }
        
        stage('Local Docker Build & Deploy') {
            steps {
                // Jenkins 서버 내부에서 도커 빌드 및 실행 (리눅스 명령어 sh 사용)
                
                // 1. 기존 컨테이너가 있으면 삭제 (없어도 에러 없이 넘어가도록 || true 처리)
                sh "docker rm -f ${CONTAINER_NAME} || true"
                
                // 2. 도커 이미지 빌드
                sh "docker build -t ${DOCKER_IMAGE} ."
                
                // 3. 도커 컨테이너 실행
                sh "docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE}"
            }
        }
    }
}