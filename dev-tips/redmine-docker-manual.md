# Redmine Docker 설치 메뉴얼 22

작성일: 2026-05-13
태그: #redmine #docker #setup

---

## 개요

개발자 로컬 환경에서 Docker를 이용하여 Redmine을 설치하고 관리하는 방법을 설명합니다.

---

## 사전 준비

- [ ] Docker Desktop 설치 (https://www.docker.com/products/docker-desktop)
- [ ] 설치 후 재부팅
- [ ] 작업표시줄에 Docker 고래 아이콘 확인

---

## STEP 1. 작업 폴더 생성

```bash
mkdir /c/redmine
cd /c/redmine
```

---

## STEP 2. docker-compose.yml 파일 생성

`C:\redmine\docker-compose.yml` 파일을 만들고 아래 내용 붙여넣기

```yaml
version: '3'
services:
  redmine:
    image: redmine:latest
    restart: always
    ports:
      - "3000:3000"
    environment:
      REDMINE_DB_MYSQL: db
      REDMINE_DB_USERNAME: redmine
      REDMINE_DB_PASSWORD: redmine123
    depends_on:
      - db
    volumes:
      - redmine_data:/usr/src/redmine/files

  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: redmine
      MYSQL_USER: redmine
      MYSQL_PASSWORD: redmine123
    volumes:
      - db_data:/var/lib/mysql

volumes:
  redmine_data:
  db_data:
```

---

## STEP 3. 실행

PowerShell 또는 Git Bash에서

```bash
cd /c/redmine
docker-compose up -d
```

> 이미지 다운로드 때문에 처음 한 번만 2~3분 소요

---

## STEP 4. 접속

브라우저에서 `http://localhost:3000` 접속

- 초기 ID: `admin`
- 초기 PW: `admin`
- **로그인 후 즉시 비밀번호 변경 권장**

---

## 자주 쓰는 명령어

```bash
# 중지
docker-compose down

# 시작
docker-compose up -d

# 로그 확인
docker-compose logs -f

# 컨테이너 상태 확인
docker ps
```

---

## 다른 노트북으로 이전

### 현재 노트북에서 (백업)

```bash
# 컨테이너 중지
docker-compose down

# DB 볼륨 백업
docker run --rm -v db_data:/data -v $(pwd):/backup alpine \
  tar czf /backup/db_backup.tar.gz /data

# Redmine 파일 볼륨 백업
docker run --rm -v redmine_data:/data -v $(pwd):/backup alpine \
  tar czf /backup/redmine_backup.tar.gz /data
```

현재 폴더에 `.tar.gz` 파일 2개 생성됨

### 새 노트북으로 옮기기

1. `docker-compose.yml` + `.tar.gz` 파일 2개 복사
2. Docker Desktop 설치
3. 볼륨 복원 명령어 실행

```bash
# 볼륨 복원
docker run --rm -v db_data:/data -v $(pwd):/backup alpine \
  tar xzf /backup/db_backup.tar.gz -C /

docker run --rm -v redmine_data:/data -v $(pwd):/backup alpine \
  tar xzf /backup/redmine_backup.tar.gz -C /

# 실행
docker-compose up -d
```

---

## 간단한 백업 방법 (DB만)

```bash
# 백업 (MySQL)
docker exec redmine_db mysqldump -u redmine -predmine123 redmine > redmine_dump.sql

# 복원
docker exec -i redmine_db mysql -u redmine -predmine123 redmine < redmine_dump.sql
```

> 첨부파일(이슈에 올린 파일 등)까지 이전하려면 볼륨 백업 방식 사용

---

## 백업 방법 비교

| 방법 | 장점 | 단점 |
|------|------|------|
| 볼륨 tar 백업 | 첨부파일까지 완벽 이전 | 명령어 김 |
| DB 덤프 | 간단 | 첨부파일 별도 복사 필요 |
