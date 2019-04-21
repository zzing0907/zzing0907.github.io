도커는 환경설정을 모두 구성해 놓고 재사용이 용이하게 도와줍니다. 도커는 아래와 같이 설치할 수 있습니다.

```
curl -fsSL https://get.docker.com/ | sudo sh  
```



도커의 설치가 완료되었다면 도커를 이용할 수 있습니다. 이때 도커를 이용하려면 기본적으로 sudo 권한이 필요한데 도커를 편하게 이용하고 싶다면 현재 사용자를 도커 그룹에 추가해주면 됩니다.  

```bash
sudo usermod -aG docker $USER  ## 현재 접속중인 사용자에게 권한주기  
sudo usermod -aG docker "user_name"  ## user_name 사용자에게 권한주기  
```

  

도커 컨테이너를 만드는 명령어: run

```bash
(sudo) docker run -dit --name 'name' ubunut:16.04
## ubuntu 16.04가 설치된 컨테이너가 실행
## 실행후 apt update && apt upgrade를 실행해야 함
```



실행중인 도커에 접속하는 명령어: attach  

```bash
(sudo) docker attach 'name'
## or
(sudo) docker exec -it [컨테이너 이름] bash
## 새로운 bash로 도커에 접속하는 방법
```

  

실행 중인 컨테이너 확인하는 방법 명령어: ps

```bash
(sudo) docker ps  ## 실행중인 컨테이너 검색  
(sudo) docker ps -a  ## 꺼진 컨테이너도 검색  
```

   

실행중인 컨테이너를 멈추거나 다시 실행하는 명령어: stop / start

```bash
(sudo) docker stop [컨테이너 id 또는 컨테이너 이름]
```

  

만들어진 컨테이너를 지우는 명령어: rm

```bash
(sudo) docker rm [컨테이너 id 또는 컨테이너 이름]  
```

​    

도커 컨테이너 안으로 파일을 복사하는 명령어: cp  

```bash
(sudo) docker cp [호스트 파일 경로] [컨테이너 이름]:[디렉토리 경로] 
## or
(sudo) docker cp [컨테이너 이름]:[파일 경로] [host 디렉토리 경로]

## example
(sudo) docker cp example.py test:/root/src
```

 

### Docker에서 GPU 이용하기  

[nvidia-docker](https://github.com/NVIDIA/nvidia-docker)를 이용하면 host의 graphic driver, cuda 세팅으로 docker를 만들어줍니다. 아래 방법으로 nvidia-docker는 설치 가능합니다.

```
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -  

## ubuntu 16.04 기준
curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu16.04/amd64/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list  

sudo apt-get update  

sudo apt-get install -y nvidia-docker2  
```



이제 아래 명령을 이용해 docker를 만들면 그래픽 드라이버와 쿠다가 설치된 docker가 만들어집니다. 나머지 이용방법은 docker와 같습니다.

```
(sudo) nvidia-docker run -dit --name cuda_test nvidia/cuda:9.0-devel
```



### Dockerfile 만들기

docker 이미지를 미리 만들 수 있습니다. 아래와 같은 Dockerfile을 빌드해서 docker 이미지를 만들어 놓으면 여러가지 환경이 설정된 docker 컨테이너를 바로바로 만들어 낼 수 있습니다. docker의 큰 장점중 하나가 바로 이런 환경 관리가 쉽다는 점입니다. 아래 dockerfile은 tensorflow를 설치하는 예입니다.

```dockerfile
FROM		nvidia/cuda  
RUN			apt-get update  
RUN			apt-get -y upgrade  

RUN			apt-get -y install git build-essential python3-dev python3-pip 

ENV			PATH $PATH:/usr/local/cuda/bin  
ENV			LD_LIBRARY_PATH $LD_LIBRARY_PATH:/usr/local/cuda/lib64  

RUN			pip3 install numpy tensorflow-gpu
```



Dockerfile을 만들었으면 이제 이 파일을 이용해 docker 이미지를 만들어야 합니다. 먼저 아래 명령어로 현재 존재하는 이미지들을 확인 할 수 있습니다.

```bash
(sudo) docker images
```



dockerfile로 이미지를 만들려면 dockerfile이 있는 디렉토리에서 다음 명령어를 실행하면 됩니다.

```bash
(sudo) docker build -t tensorflow . ## tensorflow는 이미지 이름
```



위 명령을 실행하면 Dockerfile내부의 명령어들이 실행됩니다. 모두 빌드가 되고 나서 docker images로 확인하면 tensorflow라는 이미지가 생성된 것이 볼 수 있습니다. 지우고 싶다면 아래 명령어로 지울 수 있습니다.

```bash
(sudo) docker rmi tensorflow(:latest) ## ()안은 태그 이름, default가 latest이다.
```



또 만들어진 이미지로 컨테이너를 실행시키고 싶다면 아래 명령과 같이 실행하면 됩니다.

```bash
(sudo) nvidia-docker -dit --name 'name' tensorflow  ## tensorflow는 도커 이미지
```

