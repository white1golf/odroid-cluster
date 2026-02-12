# ODROID Cluster Setup with Ansible

## Phase 1: Base Setup

### 프로젝트 구조
```
odroid-cluster/
├── cluster-info.txt          # ODROID 정보 (IP, MAC, 비밀번호)
├── inventory.ini             # Ansible 인벤토리 (호스트 정보)
├── ansible.cfg               # Ansible 설정 파일
├── playbooks/
│   ├── 00-test-connectivity.yml      # 연결 테스트
│   ├── 01-base-setup.yml             # 기본 환경 설정
├── roles/                    # Ansible 역할 (향후 사용)
└── group_vars/               # 그룹 변수 (향후 사용)
```

### 전제 조건

**Windows에서 Ansible 실행:**
```powershell
# WSL 설치 (Windows Subsystem for Linux)
wsl --install

# WSL에서 Ansible 설치
wsl
sudo apt-get update
sudo apt-get install ansible sshpass

# 또는 Windows용 직접 설치
pip install ansible
pip install sshpass  # 비밀번호 인증용
```

### 실행 단계

#### Step 1: 연결 테스트
```bash
cd C:\Users\san\dev\integrit\odroid-cluster

# 모든 ODROID에 연결 테스트
ansible-playbook playbooks/00-test-connectivity.yml -k

# 개별 호스트 테스트
ansible odroid_cluster -m ping -i inventory.ini -k
```

**-k 옵션:** 비밀번호 프롬프트 표시 (초기 설정 시 필요)

#### Step 2: 기본 환경 설정 실행
```bash
# 첫 번째 실행 (패키지 업데이트만)
ansible-playbook playbooks/01-base-setup.yml -k

# 재부팅이 필요한 경우 (재부팅 포함)
ansible-playbook playbooks/01-base-setup.yml -k --extra-vars="reboot=true"
```

### 주요 설정 항목

**inventory.ini에서:**
- `ansible_host`: 각 ODROID의 IP 주소
- `ansible_user`: SSH 사용자명 (odroid)
- `ansible_password`: SSH 비밀번호 (odroid)
- `ansible_python_interpreter`: Python 경로

**01-base-setup.yml에서:**
1. 패키지 업데이트 (apt update & upgrade)
2. 필수 도구 설치:
   - curl, wget, git, vim, net-tools, htop
   - SSH, Python3, build-tools
   - Docker 설치 준비 (apt-transport-https, ca-certificates, gnupg)
3. 호스트명 설정 (odroid-1, odroid-2, odroid-3, odroid-4)
4. sudo 비밀번호 없이 실행 가능하도록 설정
5. SSH 서비스 활성화

### SSH 키 설정 (향후)

비밀번호 인증 대신 SSH 키를 사용하려면:

```bash
# 로컬에서 SSH 키 생성 (한 번만)
ssh-keygen -t rsa -b 4096 -f odroid-cluster-key

# 공개 키를 각 ODROID에 배포
ansible-playbook playbooks/02-setup-ssh-keys.yml -k
```

### 문제 해결

**연결 실패:**
```bash
# 상세 로그 보기
ansible-playbook playbooks/00-test-connectivity.yml -k -vvv

# 특정 호스트만 테스트
ansible odroid-1 -m ping -i inventory.ini -k -vvv
```

**비밀번호 인증 관련:**
- 초기에는 비밀번호로 연결하기 때문에 `-k` 옵션 필수
- SSH 키 설정 후 비밀번호 인증 비활성화 가능

### 다음 단계 (Phase 2)
- Docker 설치 및 설정
- 컨테이너 테스트

### 참고사항
- ODROID의 기본 사용자명/비밀번호: odroid / odroid
- 클러스터의 모든 노드는 192.168.1.x 네트워크에 연결
- 호스트명 설정 후 시스템 재부팅 권장
