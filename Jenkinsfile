pipeline {
    agent any 

    tools {
        maven 'maven 3.8.7' 
    }

    environment {
        DOCKER_IMAGE = "demo-app" 
        CONTAINER_NAME = "springboot-container" 
        JAR_FILE_NAME = "app.jar" 
        PORT = "8081" // 말씀하신 대로 8081 포트 고정
    }

    stages {
        stage('Git checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                // 우분투 환경에 맞게 리눅스 쉘(sh) 사용
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Prepare Jar') {
            steps {
                // 우분투 파일 복사 명령어(cp) 사용
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        }
        
        stage('Local Docker Build & Deploy') {
            steps {
                // 1. [핵심] 우분투 환경에서 8081 포트를 사용 중인 프로세스가 있다면 강제 종료 (fuser 활용)
                sh "fuser -k ${PORT}/tcp || true"
                
                // 2. 기존 도커 컨테이너 삭제
                sh "docker rm -f ${CONTAINER_NAME} || true"
                
                // 3. 도커 빌드
                sh "docker build -t ${DOCKER_IMAGE} ."
                
                // 4. 도커 컨테이너 실행 (8081:8081)
                sh "docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE}"
            }
        }
    }
}