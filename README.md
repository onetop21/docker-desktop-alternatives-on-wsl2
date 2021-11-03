# Docker Desktop Alternatives on WSL2
Docker Desktop 이 유료화 됨에 따라 윈도 환경에서 다른 대체 환경을 사용하는 방법 정리 <br>
<sub>개인적으로 Mac은 가지고 있지 않아 Docker Desktop for Windows만 정리</sub>

⚠ Docker Desktop for Windows를 대체를 목적으로 정리하였기 때문에, WSL2 환경(Hyper-V)이 갖춰져있다고 가정하였습니다.<br>

|대체 도구|Image Build|Container Run|Orchestartion|비고|
|---|:---:|:---:|:---:|---|
|Docker.io(Docker-CE)🔗|✔|✔|✔|데몬을 직접 실행하거나 서비스 등록 필요|
|⭐Podman🔗|✔|✔||쉬운 설치. Docker와 명령어 유사|
|Buildah|✔|||OCI 빌드 지원|
|⭐minikube🔗||✔|✔|Podman을 드라이버로 설정하여 Build및 WSL2사용 가능|
|microK8S||✔|✔|WSL2에서는 Snapd 설치 필요. Windows용은 별도 Hyper-V 인스턴스 생성 필요.|
|⭐Rancher Desktop(K3S)🔗|✔|✔|✔|Docker Desktop 과 가장 유사. CLI는 kubectl 사용해야 함.|

위 표에서와 같이 각 대체 도구별 지원 기능이 달라 조합하여 사용할 필요가 있습니다.<vr>
이에 따라 추천하는 조합은 아래와 같습니다.
* Docker가 편하다면
  * Docker.io(Docker-CE) 설치 + 서비스 등록
  * Podman 설치 + alias 설정
* Kubernetes 사용이 가능하다면
  * Rancher Desktop
* Docker와 Kubernetes 모두 사용해야 한다면
  * Podman + minikube(podman driver)

## Docker.io (Docker-CE)
1. docker engine 설치
  > docker.io로 설치 시
  ```bash
  sudo apt-get update
  sudo apt-get install docker.io
  ```
  > docker-ce(community edition)로 설치 시 #1
  ```bash
  sudo apt-get remove docker docker-engine docker.io containerd runc
  sudo apt-get update
  sudo apt-get install ca-certificates curl gnupg lsb-release
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt-get update
  sudo apt-get install docker-ce docker-ce-cli containerd.io
  ```
  > docker-ce(community edition)로 설치 시 #2
  ```bash
  curl -fsSL https://get.docker.com | sudo sh -
  ```
2. Docker 그룹 등록 및 확인
  ```bash
  sudo groupadd docker
  sudo usermod -aG docker $USER
  ```
3. Docker daemon 서비스 실행
  ```bash
  sudo service docker start
  ```
4. 자동 실행 추가 (Optional)
  > $HOME/.profile 파일 수정
  ```bash
  if service docker status 2>&1 | grep -q "is not running"; then
    wsl.exe -d "${WSL_DISTRO_NAME}" -u root -e /usr/sbin/service docker start >/dev/null 2>&1
  fi
  ```
### 출처 : https://dev.to/felipecrs/simply-run-docker-on-wsl2-3o8
## Podman (Recommended)
1. Podman 설치
  ```bash
  . /etc/os-release
  sudo sh -c "echo 'deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/x${NAME}_${VERSION_ID}/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list"
  wget -nv https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable/x${NAME}_${VERSION_ID}/Release.key -O Release.key
  sudo apt-key add - < Release.key
  sudo apt-get update -qq
  sudo apt-get -qq -y install podman
  sudo mkdir -p /etc/containers
  echo -e "[registries.search]\nregistries = ['docker.io', 'quay.io']" | sudo tee /etc/containers/registries.conf
  ```
2. 아래 파일을 복사 및 수정하여 권한 및 로그 이벤트 수정
  > 설정 파일 복사
  ```bash
  sudo cp /usr/share/containers/containers.conf /etc/containers/
  ```
  > /etc/containers/containers.conf 파일 수정
  ```toml
  [engine]
  #...
  cgroup_manager = "cgroupfs"
  #...
  events_logger = "file"
  ```
3. Podman을 Docker로 alias 설정 (Optional)
  ```bash
  alias docker=podman
  ```
                  
### 출처 : https://www.redhat.com/sysadmin/podman-windows-wsl2

## MiniKube (Podman Driver)
  1. podman 설치
  2. minikube 설치
  ```bash
  curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
  sudo dpkg -i minikube_latest_amd64.deb
  ```
  3. minikube 시작
  ```bash
  minikube start --driver=podman
  ```
  4. alias 등록
  ```bash
  alias kubectl="minikube kubectl --"
### 출처 : https://gist.github.com/fardjad/6c95cda623d061bb830538c6c631d2e6

### Rancher Desktop (Recommended)
  1. 최신 바이너리 다운로드 및 실행
  https://github.com/rancher-sandbox/rancher-desktop/releases
  
  2. WSL Integration > [] {Distribute Name} 체크
  
  ❗ Build는 kim(kubernetes image manager)를 이용한다.
  ```bash
  kim build
  kim pull ubuntu:latest
  ...
  ```
