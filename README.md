# insta-prac

기존 insta 수업의 순서를 따라 직접 다시 구현하는 FastAPI 게시판 연습.

## 1. 프로젝트 준비부터 게시글 목록 조회까지 — 2026-11-11

### 진행 순서

1. uv로 Python 프로젝트 생성, FastAPI 설치, 홈 응답 작성.
2. `psql -d postgres`로 접속해 연습용 DB 생성.
3. SQLAlchemy·psycopg·Alembic 설치.
4. DB 연결 주소와 engine 작성, Session 객체 생성 실험.
5. DeclarativeBase를 상속한 Base와 Post 모델 작성.
6. GET /posts에서 게시글을 조회하고 list.html로 전달.
7. 테이블이 없다는 오류를 확인한 뒤 Alembic 초기화·설정.
8. 첫 마이그레이션 생성·적용 후 /posts 접속 성공 확인.

```sql
CREATE DATABASE insta_prac OWNER sd4;
```

```bash
uv add "fastapi[standard]"
uv add sqlalchemy "psycopg[binary]" alembic
uv run alembic init alembic
uv run alembic revision --autogenerate -m "create posts"
uv run alembic upgrade head
uv run fastapi dev main.py
```

초기화와 첫 revision 생성은 이미 완료했다. 기존 프로젝트 실행 시 반복하지 않는다.

### 배운 개념

- DB owner와 접속 사용자는 별개다. 이번 DB의 owner는 sd4다.
- FastAPI 개발 서버 포트는 8000, PostgreSQL 접속 포트는 5432다.
- create_engine은 DB 연결을 준비하고 Session은 엔진을 사용해 DB 작업을 한다. Session 객체 출력만으로 실제 접속 성공을 확인할 수는 없다.
- Base.metadata에는 모델의 테이블 구조 정보가 모인다.
- `content: Mapped[str]`는 기본 설정의 문자열 컬럼이다.
- `mapped_column(primary_key=True)`로 기본 키를 명시한다. 이름이 id라는 이유만으로 기본 키가 되지 않는다.
- `db.scalars(select(Post)).all()`로 Post 객체 목록을 얻는다.
- context의 posts를 템플릿에서 `{{ posts }}`로 표시했다. 현재는 목록 자체를 표시하는 단계다.
- revision은 변경 파일 생성, upgrade는 실제 DB 적용이다.

### 실제 오류와 해결

| 문제 | 원인과 해결 |
| --- | --- |
| /posts에서 Internal Server Error | Post 클래스만 만들었고 실제 posts 테이블은 없었다. 마이그레이션을 적용했다. |
| No 'script_location' key found in configuration | Alembic 초기화가 빠졌다. `alembic init alembic`으로 설정과 폴더를 생성했다. |
| env.py에서 DB_URL을 import보다 먼저 사용 | 위에서 아래로 실행되므로 이름을 가져온 뒤 사용해야 한다. import를 상단으로 옮겼다. |

env.py는 main에서 DB_URL과 Base를 가져온다. `config.set_main_option('sqlalchemy.url', DB_URL)`로 접속 대상을, `target_metadata = Base.metadata`로 비교할 모델 구조를 지정했다.

### 확인 결과와 남은 작업

- 사용자 실행 로그에서 `633783d6509e` revision 생성과 upgrade 실행을 확인했다.
- 사용자가 /posts 재접속 성공을 확인했다.
- 작성·상세·수정·삭제 등은 아직 완료하지 않았다. main.py의 pass 함수와 중복 경로는 작업 중인 초안이다.
- Git 저장소 초기화와 원격 연결은 아직 하지 않았다.
- 공개 저장소에 올리기 전에 main.py의 DB 접속 정보를 환경변수로 분리해야 한다. .gitignore는 코드 안에 직접 적은 비밀번호를 숨겨 주지 않는다.

### 참고 문서

- [SQLAlchemy DB 연결 주소](https://docs.sqlalchemy.org/en/20/core/engines.html#database-urls)
- [DeclarativeBase와 모델 선언](https://docs.sqlalchemy.org/en/20/orm/quickstart.html#declare-models)
