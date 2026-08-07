# PostgreSQL 시작하기 (Ubuntu 24.04 + WSL)

이 문서는 Ubuntu 24.04(WSL) 환경에서 PostgreSQL을 설치하고,
`psql` 콘솔 도구를 이용하여 간단한 CRUD(Create, Read, Update, Delete)를 수행하는 방법을 설명합니다.

---

# 1. PostgreSQL 설치

패키지 목록을 최신으로 갱신합니다.

```bash
sudo apt update
```

PostgreSQL을 설치합니다.

```bash
sudo apt install postgresql postgresql-contrib -y
```

설치가 완료되면 버전을 확인합니다.

```bash
psql --version
```

예시

```text
psql (PostgreSQL) 16.x
```

---

# 2. PostgreSQL 서비스 시작

WSL에서는 `service` 명령을 사용하는 것이 편리합니다.

```bash
sudo service postgresql start
```

서비스 상태 확인

```bash
sudo service postgresql status
```

또는

```bash
sudo systemctl status postgresql
```

---

# 3. PostgreSQL 접속

기본 관리자 계정(postgres)으로 접속합니다.

```bash
sudo -u postgres psql
```

성공하면 다음과 같은 프롬프트가 나타납니다.

```text
postgres=#
```

---

# 4. 데이터베이스 생성

현재 데이터베이스 목록 확인

```sql
\l
```

새 데이터베이스 생성

```sql
CREATE DATABASE school;
```

생성 확인

```sql
\l
```

---

# 5. 데이터베이스 선택

생성한 데이터베이스로 이동합니다.

```sql
\c school
```

출력 예시

```text
You are now connected to database "school".
```

---

# 6. 테이블 생성

현재 테이블 목록 확인

```sql
\dt
```

학생(Student) 테이블 생성

```sql
CREATE TABLE student
(
    id SERIAL PRIMARY KEY,
    name VARCHAR(30),
    age INT
);
```

테이블 생성 확인

```sql
\dt
```

---

# 7. INSERT

학생 1명 추가

```sql
INSERT INTO student(name, age)
VALUES ('홍길동', 20);
```

여러 명 추가

```sql
INSERT INTO student(name, age)
VALUES
('김철수',22),
('이영희',19),
('박민수',25);
```

---

# 8. SELECT

전체 조회

```sql
SELECT *
FROM student;
```

예시 결과

```text
 id |  name  | age
----+--------+-----
1   | 홍길동 | 20
2   | 김철수 | 22
3   | 이영희 | 19
4   | 박민수 | 25
```

---

# 9. WHERE

20세 이상 조회

```sql
SELECT *
FROM student
WHERE age >= 20;
```

---

# 10. UPDATE

나이 수정

```sql
UPDATE student
SET age = 30
WHERE id = 1;
```

확인

```sql
SELECT *
FROM student;
```

---

# 11. DELETE

학생 삭제

```sql
DELETE
FROM student
WHERE id = 2;
```

확인

```sql
SELECT *
FROM student;
```

---

# 12. ORDER BY

나이 오름차순

```sql
SELECT *
FROM student
ORDER BY age;
```

나이 내림차순

```sql
SELECT *
FROM student
ORDER BY age DESC;
```

---

# 13. LIKE 검색

이름이 '김'으로 시작하는 학생 조회

```sql
SELECT *
FROM student
WHERE name LIKE '김%';
```

---

# 14. COUNT

학생 수 조회

```sql
SELECT COUNT(*)
FROM student;
```

---

# 15. 테이블 구조 확인

```sql
\d student
```

---

# 16. 테이블 목록

```sql
\dt
```

---

# 17. 데이터베이스 목록

```sql
\l
```

---

# 18. 현재 접속 정보

```sql
\conninfo
```

---

# 19. PostgreSQL 종료

`psql` 종료

```sql
\q
```

서비스 종료

```bash
sudo service postgresql stop
```

---

# 자주 사용하는 psql 메타 명령어

| 명령어 | 설명 |
|---------|------|
| `\l` | 데이터베이스 목록 |
| `\c DB명` | 데이터베이스 변경 |
| `\dt` | 테이블 목록 |
| `\d 테이블명` | 테이블 구조 |
| `\conninfo` | 현재 접속 정보 |
| `\q` | psql 종료 |

---

# 전체 실습 흐름

1. PostgreSQL 설치
2. 서비스 시작
3. psql 접속
4. 데이터베이스 생성
5. 데이터베이스 선택
6. 테이블 생성
7. INSERT
8. SELECT
9. WHERE
10. UPDATE
11. DELETE
12. ORDER BY
13. LIKE
14. COUNT
15. 테이블 구조 확인
16. psql 종료

---

# 다음 단계

기본 CRUD를 익혔다면 다음 내용을 학습하는 것을 추천합니다.

- 데이터 타입
- PRIMARY KEY
- FOREIGN KEY
- UNIQUE
- CHECK
- DEFAULT
- INDEX
- VIEW
- JOIN
- GROUP BY
- Transaction
- PostgreSQL 사용자 생성 및 권한 관리
- Java(JDBC) 연동
- C++(libpqxx) 연동
- Python(psycopg) 연동
- Spring Boot + PostgreSQL

# PostgreSQL 학생 데이터 Streamlit 화면에 출력하기

이 문서는 PostgreSQL의 `student` 테이블 데이터를 Python으로 조회하고, Streamlit 화면에 출력하는 과정을 설명합니다.

전체 흐름은 다음과 같습니다.

```text
PostgreSQL
    ↓
psycopg
    ↓
Pandas DataFrame
    ↓
Streamlit
```

---

## 1. 프로젝트 폴더 생성

WSL Ubuntu 터미널에서 프로젝트 폴더를 생성합니다.

```bash
mkdir postgresql
cd postgresql
```

프로젝트 구조는 다음과 같습니다.

```text
postgresql/
├── app.py
├── .env
├── .gitignore
└── requirements.txt
```

---

## 2. Python 가상환경 생성

```bash
python3 -m venv pgsql
```

가상환경을 활성화합니다.

```bash
source pgsql/bin/activate
```

`venv` 모듈이 설치되어 있지 않다면 다음 명령으로 설치합니다.

```bash
sudo apt install python3-venv -y
```

---

## 3. 필요한 라이브러리 설치

```bash
pip install streamlit pandas "psycopg[binary]" python-dotenv
```

설치된 라이브러리를 `requirements.txt` 파일에 기록합니다.

```bash
pip freeze > requirements.txt
```

간단하게 직접 작성할 수도 있습니다.

```text
streamlit
pandas
psycopg[binary]
python-dotenv
```

---

## 4. PostgreSQL 애플리케이션 사용자 생성

PostgreSQL 관리자 계정으로 접속합니다.

```bash
sudo -u postgres psql
```

Streamlit 프로그램에서 사용할 PostgreSQL 사용자를 생성합니다.

```sql
CREATE USER student_app
WITH PASSWORD 'student1234';
```

`school` 데이터베이스 접속 권한을 부여합니다.

```sql
GRANT CONNECT ON DATABASE school TO student_app;
```

`school` 데이터베이스로 이동합니다.

```sql
\c school
```

스키마 사용 권한을 부여합니다.

```sql
GRANT USAGE ON SCHEMA public TO student_app;
```

`student` 테이블 조회 권한을 부여합니다.

```sql
GRANT SELECT ON TABLE student TO student_app;
```

> `GRANT` 명령은 PostgreSQL 관리자 계정 또는 해당 데이터베이스 객체의 소유자 계정으로 실행해야 합니다. 일반 사용자 계정은 자신에게 테이블 권한을 부여할 수 없습니다.

PostgreSQL을 종료합니다.

```sql
\q
```

---

## 5. PostgreSQL 접속 테스트

다음 명령으로 `student_app` 계정 접속을 확인합니다.

```bash
psql -h localhost -U student_app -d school
```

비밀번호를 입력합니다.

```text
student1234
```

학생 데이터를 조회합니다.

```sql
SELECT *
FROM student
ORDER BY id;
```

정상적으로 조회되면 PostgreSQL을 종료합니다.

```sql
\q
```

---

## 6. 환경변수 파일 작성

프로젝트 폴더에 `.env` 파일을 생성합니다.

```bash
nano .env
```

다음 내용을 작성합니다.

```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=school
DB_USER=student_app
DB_PASSWORD=student1234
```

`.env` 파일에는 데이터베이스 접속 정보가 들어 있으므로 GitHub에 올리지 않습니다.

---

## 7. `.gitignore` 작성

`.gitignore` 파일을 생성합니다.

```bash
nano .gitignore
```

다음 내용을 작성합니다.

```gitignore
# Python 가상환경
.venv/

# Python 캐시
__pycache__/
*.pyc
*.pyo

# 환경변수
.env

# Streamlit 설정
.streamlit/secrets.toml

# IDE
.vscode/
.idea/
```

---

## 8. Streamlit 코드 작성

`app.py` 파일을 생성합니다.

```bash
nano app.py
```

다음 코드를 작성합니다.

```python
import os

import pandas as pd
import psycopg
import streamlit as st
from dotenv import load_dotenv


# .env 파일의 환경변수를 읽어옵니다.
load_dotenv()


def get_connection():
    """PostgreSQL 연결 객체를 반환합니다."""
    return psycopg.connect(
        host=os.getenv("DB_HOST"),
        port=os.getenv("DB_PORT"),
        dbname=os.getenv("DB_NAME"),
        user=os.getenv("DB_USER"),
        password=os.getenv("DB_PASSWORD"),
    )


def fetch_students() -> pd.DataFrame:
    """student 테이블의 전체 데이터를 조회합니다."""
    sql = """
        SELECT
            id,
            name,
            age
        FROM student
        ORDER BY id
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql)

            rows = cursor.fetchall()
            columns = [column.name for column in cursor.description]

    return pd.DataFrame(rows, columns=columns)


st.set_page_config(
    page_title="학생 관리",
    layout="wide",
)

st.title("학생 목록")
st.write("PostgreSQL의 `student` 테이블 데이터를 조회합니다.")

try:
    students = fetch_students()

    if students.empty:
        st.info("등록된 학생이 없습니다.")
    else:
        st.metric("전체 학생 수", len(students))

        st.dataframe(
            students,
            width="stretch",
            hide_index=True,
        )

except psycopg.Error as error:
    st.error("PostgreSQL 데이터베이스 연결 또는 조회 중 오류가 발생했습니다.")
    st.code(str(error))
```

---

## 9. Streamlit 실행

PostgreSQL 서버를 시작합니다.

```bash
sudo service postgresql start
```

Streamlit 프로그램을 실행합니다.

```bash
streamlit run app.py
```

또는 다음 명령을 사용할 수 있습니다.

```bash
python -m streamlit run app.py
```

실행하면 터미널에 접속 주소가 표시됩니다.

```text
Local URL: http://localhost:8501
```

Windows 브라우저에서 다음 주소로 접속합니다.

```text
http://localhost:8501
```

---

## 10. 실행 결과

Streamlit 화면에는 PostgreSQL의 `student` 테이블 데이터가 표시됩니다.

```text
학생 목록

전체 학생 수: 3

┌────┬────────┬─────┐
│ id │ name   │ age │
├────┼────────┼─────┤
│ 1  │ 홍길동 │ 30  │
│ 2  │ 이영희 │ 19  │
│ 3  │ 박민수 │ 25  │
└────┴────────┴─────┘
```

---

## 11. 프로그램 데이터 흐름

```text
PostgreSQL student 테이블
        ↓
psycopg.connect()
        ↓
SELECT id, name, age FROM student
        ↓
cursor.fetchall()
        ↓
Pandas DataFrame
        ↓
st.dataframe()
        ↓
웹 브라우저
```

### PostgreSQL 연결

```python
connection = psycopg.connect(
    host="localhost",
    port="5432",
    dbname="school",
    user="student_app",
    password="student1234",
)
```

### 학생 데이터 조회

```python
cursor.execute(
    """
    SELECT id, name, age
    FROM student
    ORDER BY id
    """
)

rows = cursor.fetchall()
```

### Pandas DataFrame 생성

```python
students = pd.DataFrame(
    rows,
    columns=["id", "name", "age"],
)
```

### Streamlit 화면 출력

```python
st.dataframe(
    students,
    width="stretch",
    hide_index=True,
)
```

---

## 12. 컬럼명을 한글로 출력하기

SQL에서 별칭을 사용하면 Streamlit 화면의 컬럼명을 한글로 표시할 수 있습니다.

```python
sql = """
    SELECT
        id AS "학생번호",
        name AS "이름",
        age AS "나이"
    FROM student
    ORDER BY id
"""
```

화면에는 다음과 같이 표시됩니다.

```text
학생번호 | 이름 | 나이
```

---

## 13. 최소 나이 검색 기능 추가

`fetch_students()` 함수에 최소 나이 조건을 추가합니다.

```python
def fetch_students(minimum_age: int = 0) -> pd.DataFrame:
    sql = """
        SELECT
            id,
            name,
            age
        FROM student
        WHERE age >= %s
        ORDER BY id
    """

    with get_connection() as connection:
        with connection.cursor() as cursor:
            cursor.execute(sql, (minimum_age,))

            rows = cursor.fetchall()
            columns = [column.name for column in cursor.description]

    return pd.DataFrame(rows, columns=columns)
```

Streamlit 입력 위젯을 추가합니다.

```python
minimum_age = st.number_input(
    "최소 나이",
    min_value=0,
    max_value=100,
    value=0,
    step=1,
)

students = fetch_students(minimum_age)
```

Psycopg에서는 SQL 조건값을 직접 문자열로 조합하지 않고 `%s` 자리표시자와 매개변수를 사용합니다.

```python
cursor.execute(sql, (minimum_age,))
```

---

## 14. 최종 프로젝트 구조

```text
postgresql_streamlit/
├── .venv/
├── app.py
├── .env
├── .gitignore
└── requirements.txt
```

실행 순서는 다음과 같습니다.

```bash
cd postgresql_streamlit
source .venv/bin/activate
sudo service postgresql start
streamlit run app.py
```

이번 단계에서는 PostgreSQL의 학생 데이터를 조회하여 Streamlit 화면에 출력했습니다.

다음 단계에서는 학생 등록, 수정, 삭제 기능을 추가하여 CRUD 프로그램으로 확장할 수 있습니다.
