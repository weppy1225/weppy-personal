# Obsidian + Git 연동 메뉴얼

작성일: 2026-05-13
태그: #obsidian #git #github #setup

---

## 개요

Obsidian 보관함(Vault)을 GitHub 저장소와 연동하여 Claude Web 대화 내용을 md로 정리하고 자동 백업하는 방법을 설명합니다.

**역할 분담**

| 도구 | 역할 |
|------|------|
| Claude Web | AI와 대화하며 작업 |
| Obsidian | 대화 내용 정리해서 md로 보관 |
| GitHub | Obsidian 노트 백업 및 동기화 |
| 각 프로젝트 Git | 소스코드 관리 (별도) |

---

## 저장소 구조

```
weppy-company/  ← 회사 관련 전부 (Private)
  project/
    ai-framework/      # AI프레임워크 구축 사업
    unified-fw/        # 통합 프레임워크 채택 및 고도화
    rag-system/        # 사내 RAG 시스템 구축
  work/
    draft-ai-fw/       # 기안서 - AI프레임워크
    draft-unified-fw/  # 기안서 - 통합프레임워크
    ai-agent/          # AI 에이전트 만들기

weppy-personal/ ← 개인 전부 (Public)
  project/
    class-checkin/     # 학원 출결 관리 메모
    study-note/        # 오답노트
  blog/
    etf-invest/        # ETF 투자 블로그
    wms-content/       # WMS 컨텐츠 제작 블로그
  cafe/
    claude-edu/        # 클로드 교육 카페
  life/
    future-plan/       # 미래 설계
    health/            # 건강
    travel/            # 여행
  dev-tips/
    obsidian-git/      # Obsidian + Git 연동 메뉴얼
    redmine-docker/    # Redmine Docker 설치 메뉴얼
```

---

## STEP 1. GitHub 저장소 2개 생성

1. https://github.com 접속
2. 우측 상단 `+` → `New repository`

### weppy-company
- Repository name: `weppy-company`
- **Private** 선택
- README 체크 해제
- `Create repository` 클릭

### weppy-personal
- Repository name: `weppy-personal`
- **Public** 선택
- README 체크 해제
- `Create repository` 클릭

---

## STEP 2. 로컬 폴더 생성 및 Git 연동

Git Bash 실행 후 순서대로 실행

### weppy-company
```bash
mkdir /c/zinide/weppy-company
cd /c/zinide/weppy-company
git init
git remote add origin https://github.com/weppy1225/weppy-company.git
```

### weppy-personal
```bash
mkdir /c/zinide/weppy-personal
cd /c/zinide/weppy-personal
git init
git remote add origin https://github.com/weppy1225/weppy-personal.git
```

---

## STEP 3. Claude Code로 폴더 구조 생성

### Claude Code 설치 (없다면)
```bash
npm install -g @anthropic-ai/claude-code
```

### weppy-company 폴더 구조 생성
```bash
cd /c/zinide/weppy-company
claude
```

Claude Code CLI에 아래 내용 붙여넣기:
```
현재 경로에 아래 폴더 구조를 생성해줘.
각 폴더 안에 README.md 파일도 만들어줘.
README.md 내용은 해당 폴더의 용도를 한국어로 간단히 설명해줘.

project/
  ai-framework/
  unified-fw/
  rag-system/
work/
  draft-ai-fw/
  draft-unified-fw/
  ai-agent/
```

### weppy-personal 폴더 구조 생성
```bash
cd /c/zinide/weppy-personal
claude
```

Claude Code CLI에 아래 내용 붙여넣기:
```
현재 경로에 아래 폴더 구조를 생성해줘.
각 폴더 안에 README.md 파일도 만들어줘.
README.md 내용은 해당 폴더의 용도를 한국어로 간단히 설명해줘.

project/
  class-checkin/
  study-note/
blog/
  etf-invest/
  wms-content/
cafe/
  claude-edu/
life/
  future-plan/
  health/
  travel/
dev-tips/
  obsidian-git/
  redmine-docker/
```

---

## STEP 4. GitHub 첫 Push

### weppy-company
```bash
cd /c/zinide/weppy-company
git add .
git commit -m "init: weppy-company 폴더 구조 생성"
git branch -M main
git push -u origin main
```

### weppy-personal
```bash
cd /c/zinide/weppy-personal
git add .
git commit -m "init: weppy-personal 폴더 구조 생성"
git branch -M main
git push -u origin main
```

> ⚠️ push 시 rejected 오류 발생하면
> ```bash
> git pull origin main --allow-unrelated-histories
> git add .
> git commit -m "merge"
> git push -u origin main
> ```

> ⚠️ vim 에디터 열리면 `:wq` 입력 후 엔터

---

## STEP 5. Obsidian 보관함 2개 열기

> ⚠️ **중요**: 이미 폴더가 존재하므로 **"보관함 폴더 열기"** 를 사용해야 합니다.
> "새 보관함 생성" 을 누르면 폴더 안에 중복 폴더가 생성되니 주의!

### 보관함 등록 방법

1. Obsidian 실행
2. 좌측 하단 보관함 이름 클릭 (없으면 자동으로 시작 화면 나옴)
3. **`보관함 폴더 열기`** → **`열기`** 클릭
4. 폴더 선택

### weppy-company 보관함 열기
- 폴더 선택: `C:\zinide\weppy-company`
- `폴더 선택` 클릭

### weppy-personal 보관함 열기
- 다시 보관함 목록 클릭 → `보관함 폴더 열기` → `열기`
- 폴더 선택: `C:\zinide\weppy-personal`
- `폴더 선택` 클릭

---

### 참고: "새 보관함 생성" vs "보관함 폴더 열기"

| 옵션 | 사용 시점 |
|------|----------|
| 새 보관함 생성 | Obsidian이 새 폴더를 만들어 줄 때 (빈 상태에서 시작) |
| **보관함 폴더 열기** | **이미 만들어진 폴더를 보관함으로 인식시킬 때 (오늘 케이스)** |

---

## STEP 6. Obsidian Git 플러그인 설치

각 보관함마다 동일하게 설정

1. `Ctrl + ,` → `커뮤니티 플러그인` → `커뮤니티 플러그인 사용` 클릭
2. `탐색` 클릭
3. `Git` 검색 (개발자: vinzent03)
4. `설치` → `활성화` 클릭
5. `옵션` 클릭 후 아래 설정

| 설정 항목 | 값 |
|----------|-----|
| Auto commit interval | 0 |
| Auto push interval | 10 |
| Auto pull interval | 10 |
| Pull on startup | ON |

---

## STEP 7. 동작 확인

1. Obsidian에서 테스트 노트 작성 (`Ctrl + N`)
2. `Ctrl + P` → `Git: Commit-and-sync` 실행
3. GitHub 저장소 접속하여 파일 업로드 확인

---

## 자주 쓰는 명령어

| 작업 | 단축키 |
|------|--------|
| 수동 커밋 + Push | `Ctrl + P` → `Git: Commit-and-sync` |
| 수동 Pull | `Ctrl + P` → `Git: Pull` |
| 변경사항 확인 | `Ctrl + P` → `Git: Open source control view` |

---

## 다른 노트북으로 이전 시

```bash
git clone https://github.com/weppy1225/weppy-company.git /c/zinide/weppy-company
git clone https://github.com/weppy1225/weppy-personal.git /c/zinide/weppy-personal
```

Obsidian에서 **`보관함 폴더 열기`** 로 각 폴더 열고 Git 플러그인 설치하면 완료!

---

## 문제 해결

### "Vault already exists" 오류
- 원인: 해당 위치에 이미 보관함이 존재
- 해결: **"새 보관함 생성"** 대신 **"보관함 폴더 열기"** 사용

### 폴더가 안 보임 (중복 폴더 발생)
- 원인: "새 보관함 생성"으로 만들었을 때 폴더명이 중복으로 생성됨
- 해결:
  ```bash
  cd /c/zinide/weppy-company
  rm -rf weppy-company    # 중복 폴더 삭제
  ```
  그 다음 Obsidian에서 **"보관함 폴더 열기"** 로 다시 진행

### git push 시 rejected
- 원인: 원격 저장소에 이미 파일 존재
- 해결:
  ```bash
  git pull origin main --allow-unrelated-histories
  git add .
  git commit -m "merge"
  git push -u origin main
  ```

### Master/Main 브랜치 불일치
- 원인: 로컬은 `master`, GitHub는 `main`
- 해결:
  ```bash
  git branch -M main
  git push -u origin main
  ```

---

## Claude 프로젝트 → Obsidian 폴더 매핑

| Claude 프로젝트 | Obsidian 저장소 | 폴더 |
|----------------|----------------|------|
| [회사-프로젝트] AI프레임워크 구축 사업 | weppy-company | project/ai-framework |
| [회사-프로젝트] 통합 프레임워크 채택 및 고도화 | weppy-company | project/unified-fw |
| [회사-프로젝트] 사내RAG시스템구축 | weppy-company | project/rag-system |
| [회사-업무] 기안서작성-AI프레임워크구축사업 | weppy-company | work/draft-ai-fw |
| [회사-업무] 기안서작성-통합프레임워크채택고도화 | weppy-company | work/draft-unified-fw |
| [회사-업무] AI에이전트만들기 | weppy-company | work/ai-agent |
| [개인-프로젝트] 클래스체크인 | weppy-personal | project/class-checkin |
| [개인-프로젝트] 오답노트 | weppy-personal | project/study-note |
| [개인-블로그] ETF | weppy-personal | blog/etf-invest |
| [개인-블로그] WMS컨텐츠제작 | weppy-personal | blog/wms-content |
| [개인-카페] 클로드교육 | weppy-personal | cafe/claude-edu |
| [개인-생활] 미래설계 | weppy-personal | life/future-plan |
| [개인-생활] 건강 | weppy-personal | life/health |
| [개인-생활] 여행 | weppy-personal | life/travel |
| 개발 메뉴얼 모음 | weppy-personal | dev-tips |
