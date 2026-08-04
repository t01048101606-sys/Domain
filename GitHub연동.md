# Git & GitHub 연동 가이드

까먹을 때마다 다시 보는 용도의 참고 문서입니다.

---

## 0. Git과 GitHub는 다른 것

- **Git**: 내 컴퓨터에서 파일 변경 이력을 기록하는 프로그램 (버전 관리 도구)
- **GitHub**: 그 Git 기록을 인터넷에 올려서 백업/공유하는 웹사이트

Git으로 로컬에서 이력을 관리하고, GitHub로 그걸 인터넷에 올리는 관계다.

---

## 1. 최초 1회 설정

컴퓨터 하나당 한 번만 하면 된다.

```bash
git config --global user.name "본인 이름"
git config --global user.email "본인 이메일"
```

병합 방식도 한 번 정해두면 편하다 (merge 방식 고정):

```bash
git config pull.rebase false
```

---

## 2. 기본 작업 흐름 (매번 반복하는 3줄)

코드를 수정한 뒤, 의미 있는 작업 단위가 끝날 때마다 실행한다.
**Ctrl+S로 저장하는 것과 GitHub 업데이트는 별개다 — 아래 3줄을 직접 실행해야 GitHub에 반영된다.**

```bash
git add -A
git commit -m "수정 내용을 설명하는 메시지"
git push
```

| 명령어 | 의미 |
|---|---|
| `git status` | 지금 뭐가 바뀌었는지 확인 |
| `git add -A` | 바뀐 파일을 전부 "커밋할 대상"으로 등록 |
| `git commit -m "메시지"` | 실제로 이력에 기록 (아직 GitHub엔 안 올라감) |
| `git push` | GitHub에 실제로 반영 |
| `git pull` | GitHub의 최신 내용을 내 컴퓨터로 받아오기 |
| `git log --oneline` | 커밋 이력 한 줄씩 보기 |

---

## 3. 로컬 프로젝트를 처음 GitHub와 연결하기

### 3-1. 로컬 저장소 만들기 (아직 안 했다면)

```bash
cd ~/work/프로젝트폴더
git init
```

### 3-2. `.gitignore` 먼저 만들기

DB 파일, 가상환경, 로그 파일이 실수로 올라가지 않도록 미리 걸러둔다.

```bash
cat > .gitignore << 'EOF'
mes/
venv/
.venv/
__pycache__/
*.pyc
sql/*.db
.streamlit/secrets.toml
*.log
EOF
```

### 3-3. 첫 커밋

```bash
git add -A
git status   # sql/*.db, *.log 같은 게 목록에 없는지 확인
git commit -m "초기 커밋"
```

### 3-4. GitHub에 이미 저장소가 있다면 연결

```bash
git remote add origin https://github.com/계정명/저장소이름.git
git remote -v   # 연결 확인
```

### 3-5. GitHub 쪽에 이미 파일이 있는 경우 (예: 웹으로 직접 올린 파일)

로컬과 GitHub의 커밋 역사가 서로 무관하므로, 옵션을 붙여서 pull한다.

```bash
git pull origin main --allow-unrelated-histories
```

**충돌(CONFLICT)이 뜨면**, 로컬 코드를 우선하고 싶을 때:

```bash
git checkout --ours .
git add -A
git commit -m "GitHub 기존 내용과 병합 (로컬 최신 코드 유지)"
```

### 3-6. Push

```bash
git branch -M main
git push -u origin main
```

`-u`는 처음 한 번만 필요하고, 이후로는 그냥 `git push`만 치면 된다.

---

## 4. 인증 — Personal Access Token (PAT)

GitHub는 push할 때 계정 비밀번호를 직접 쓰지 못하게 막아뒀다. **토큰**을 대신 써야 한다.

### 4-1. 토큰 만들기

1. GitHub 웹사이트 → 우측 상단 프로필 → **Settings**
2. 왼쪽 맨 아래 **Developer settings**
3. **Personal access tokens** → **Tokens (classic)**
4. **Generate new token** → **Generate new token (classic)**
5. Note(이름) 아무거나 입력, 만료기간 선택
6. 권한에서 **repo** 전체 체크
7. **Generate token** 클릭
8. 나오는 토큰 문자열을 **바로 복사** (이 화면 벗어나면 다시 안 보임)

### 4-2. 토큰 사용

`git push` 시 아래 프롬프트가 뜨면:

```
Username for 'https://github.com': 본인 계정명
Password for 'https://계정명@github.com':
```

- Username: GitHub 계정명
- Password 자리: **토큰을 붙여넣기** (계정 비밀번호 아님)

화면에 아무것도 안 보이고 커서도 안 움직이는 게 정상이다 (보안을 위해 입력값을 숨기는 것). 붙여넣고 Enter를 누르면 된다.

### 4-3. 토큰 저장해서 매번 안 치기

```bash
git config --global credential.helper store
```

한 번 입력하면 그 다음부터는 자동으로 기억한다.

---

## ⚠️ 절대 지켜야 할 것 — 토큰 보안

- **토큰은 터미널에만 입력한다.** 채팅창, 메모장, 코드 파일 등 어디에도 붙여넣지 않는다.
- 실수로 어딘가에 토큰을 노출했다면, **즉시 GitHub에서 그 토큰을 삭제(revoke)**하고 새로 발급받는다.
  - Settings → Developer settings → Personal access tokens → Tokens (classic) → 해당 토큰 → **Delete**
- 토큰은 사실상 비밀번호와 같다. 남에게 노출되면 내 GitHub 저장소에 접근할 수 있게 된다.

---

## 5. 이미 있는 GitHub 저장소를 새 컴퓨터로 받아오기 (clone)

```bash
git clone https://github.com/계정명/저장소이름.git
```

`git init` + `remote add` + `pull`을 한 번에 해주는 명령어다.

---

## 6. 자동화 (선택 사항)

매번 3줄 치는 게 익숙해지면, 아래 방법으로 더 편하게 만들 수 있다.

### 6-1. 단축 명령어 (alias)

```bash
git config --global alias.save '!f() { git add -A && git commit -m "$1" && git push; }; f'
```

이후로는:
```bash
git save "수정 내용"
```

### 6-2. 완전 자동 (파일 저장 감지 시 자동 push)

`watchdog` 라이브러리로 파일 변경을 감지해 자동 커밋+push 하는 스크립트도 있다 (별도 파일 `auto_git_sync.py` 참고). 다만 커밋 메시지가 의미 없어지고 미완성 코드도 올라갈 수 있어, 처음에는 수동 방식을 추천한다.

---

## 7. 자주 겪는 상황 정리

| 상황 | 해결 |
|---|---|
| `fatal: not a git repository` | 그 폴더에서 `git init` 안 한 상태. `git init`부터. |
| `divergent branches` 에러 | `git config pull.rebase false` 실행 후 다시 `git pull` |
| `CONFLICT (add/add)` | 로컬 버전 유지하려면 `git checkout --ours .` 후 add+commit |
| `! [rejected] ... (fetch first)` | GitHub에 로컬에 없는 커밋이 있음. `git pull` 먼저 하고 다시 `git push` |
| push 시 비밀번호 안 먹힘 | 계정 비밀번호가 아니라 Personal Access Token을 입력해야 함 |
| 매번 토큰 입력 귀찮음 | `git config --global credential.helper store` |
| `nano` 편집기가 갑자기 뜸 (병합 커밋 메시지 등) | 당황하지 말고 `Ctrl+O` → Enter(저장) → `Ctrl+X`(나가기) |
| `vim` 편집기가 갑자기 뜸 | `Esc` → `:wq` → Enter |
| 파일명을 대소문자 다르게 만들어버림 (예: `mes.md`) | `mv mes.md MES.md`로 이름 정정, 중복 생겼으면 `rm`으로 여분 삭제 |

---

## 8. 실전 예제 — 파일 하나 수정해서 GitHub에 반영하기

가장 자주 반복하게 될 흐름을 실제 예시로 정리한다. (`MES.md`에 한 줄 추가하는 경우)

### 8-1. 파일 열어서 수정

```bash
cd ~/work/mini_mes
nano MES.md
```

원하는 내용 추가 후 저장: `Ctrl+O` → Enter → `Ctrl+X`

### 8-2. 뭐가 바뀌었는지 확인

```bash
git status
```
```
Changes not staged for commit:
        modified:   MES.md
```

자세한 변경 내용까지 보고 싶으면:
```bash
git diff
```

### 8-3. Staging → Commit

```bash
git add MES.md
git commit -m "MES.md에 업데이트 로그 섹션 추가"
```

### 8-4. Push

```bash
git push
```

**여기서 거부(rejected)될 수 있다** — GitHub 쪽에 로컬에 없는 커밋이 있을 때 흔히 발생:
```
! [rejected]        main -> main (fetch first)
```

이 경우 당황하지 말고:

```bash
git pull
```

- 자동으로 조용히 합쳐지면 → 그대로 다음 단계로
- `nano`(또는 `vim`) 편집기가 병합 커밋 메시지 화면으로 뜨면 → 기본 메시지 그대로 두고 저장 후 나가기 (`Ctrl+O` → Enter → `Ctrl+X`)
- `CONFLICT` 뜨면 → 7번 표 참고

그 다음 다시:
```bash
git push
```

성공하면:
```
To https://github.com/계정명/저장소이름.git
   e255de3..494fa14  main -> main
```

### 8-5. 확인

GitHub 웹사이트에서 파일 열어서 내용 반영됐는지 확인. 저장소 커밋 목록에서 방금 쓴 커밋 메시지도 확인 가능.

> 여러 곳(다른 컴퓨터, GitHub 웹 등)에서 같은 저장소를 건드렸다면 8-4의 `rejected → pull → push` 패턴을 자주 겪게 된다. 당황하지 않고 pull 먼저 하면 대부분 해결된다.
| `divergent branches` 에러 | `git config pull.rebase false` 실행 후 다시 `git pull` |
| `CONFLICT (add/add)` | 로컬 버전 유지하려면 `git checkout --ours .` 후 add+commit |
| push 시 비밀번호 안 먹힘 | 계정 비밀번호가 아니라 Personal Access Token을 입력해야 함 |
| 매번 토큰 입력 귀찮음 | `git config --global credential.helper store` |
