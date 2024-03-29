# Alembic

Python, SQLAlchemy 기반의 database migration tool

## 참조
- https://alembic.sqlalchemy.org/en/latest/tutorial.html
- https://alembic.sqlalchemy.org/en/latest/autogenerate.html

## 설치 및 환경 구성

- pip install alembic
- 이후 빈 폴더에에서 `alembic init .`
  - 이걸 migration environment를 만든다고 한다.

## directory structure

- alembic/
  - env.py
  - README
  - script.py.mako
  - alembic.ini
  - versions/
    - 3512b954651e_add_account.py
    - 2b1ae634e5cd_add_order_id.py
    - 3adcc9a56557_rename_username_field.py

init 하고 나면 보통 위와 같은 구조를 가짐 

- alembic
  - migration env의 루트 디렉토리. 
  - 꼭 alembic 이라는 이름이 아니어도 된다.
  - 하나의 repo에서 여러 DB을 관리할 수도 있으므로 이러한 루트 디렉토리가 Repo에 여러개 있을 수 있다.
- env.py
  - migration tool 실행시 같이 실행되는 python 스크립트
  - SQLAlchemy engine을 설정하고, 생성한다
  - engine으로 부터 connection을 획득하고, 이를 사용해 migration 실행함
  - Migration 환경에 대한 것이나, 어떻게 connect 할지, engine은 몇개 띄울지 등 migration 실행에 대해 커스터마이징 가능
- script.py.mako
  - migration script를 생성하기 위한 mako 탬플릿 파일
  - versions/ 폴더에 새 파일을 생성하기 위해 사용됨
  - 이 파일을 수정해서 version/에 생기는 파일들을 컨트롤 가능 (upgrade, downgrade 함수 구조 변경, standard import 추가 등)
- versions/
  - 파일들은 GUID를 파일명으로 가짐.
  - 각 버전별로 스크립트 파일이 생긴다
  - 버전 스크립트 파일의 순서는 파일내 지시어에 상대적이다
  - 이론적으로 파일들간 연결이 가능하고, 병합되도록 다른 분기의 시퀀스들을 마이그레이션 하는 것을 허용한다.


## 템플릿

`alembic list_templates` 명령어로 사용할 수 있는 환경 템플릿 확인 가능

`alembic init --template generic .` 이런식으로 탬플릿 기반 생성 가능


## .ini 파일

- migration 실행할때 참조하는 설정 파일
- 다양한 설정이 있지만 당장 alembic을 시작해보고자 한다면 아래 설정만 있으면 된다
  - `sqlalchemy.url = postgresql://scott:tiger@localhost/test`

## Migration script 만들기

- `alembic revision -m "{script 이름}"`
- 위 명령어를 치면 스크립트 파일이 생긴다
- 스크립트엔 아래와 같은 항목이 있다
  - revision = '1975ea83b712'
    - 파일에 대한 식별자
  - down_revision = None
    - 첫번째 실행 스크립트는 이 값이 None이라고 함
    - 이 값으로 다른 스크립트의 revision을 쓰면 스크립트 실행 순서가 결정된다
  - branch_labels = None
  - def upgrade():
    - migration시 실행될 내용. 
    - alembic 지시어(op.create_table() 같은거)를 이용한 테이블 생성 등의 명령어가 보통 들어감
  - def downgrade():
    - migration을 되돌리때 사용?
    - 테이블 삭제 등이 보통 들어감
- alembic 지시어는 create_table 외에도 많으니 문서 참고

## migration 실행
- `alembic upgrade {revision}`
- revison에 들어갈 수 있는 값
  - head : 가장 최신 revision을 의미
  - revision 값: 해당 revision으로 업그레이드
- Alembic은 upgrade 수행시 연결된 DB에서 alembic_version 이라는 테이블을 참고한다(없으면 생성한다)
- 그리고 명시한 revision 까지 수행하기 위한 경로를 계산한다. 
- 계산된 경로에 있는 모든 Revision 스크립트에서 upgrade를 수행한다

## partial revision identifier
- 예를들어 스크립트들 중 1975ea83b712 이라는 revision을 가진 스크립트가 있을때 1975ea83b712의 일부분만 써도 이 revision을 식별할 수 있다면 그 일부분만 사용해도 된다
- `alembic upgrade 197` 이렇게만 써도 된다 (197로 시작하는 revision이 더 없다면.)

## Relative Migration Identifiers
- 상대적인값(+1, +2, -2 등)으로 얼마만큼 업그레이드하거나 다운그레이드할지 명시 가능
  - ex) `alembic upgrade +2`
- 특정 Revision 기준으로도 가능 
  - ex) `alembic upgrade ae10+2`

## 상태 확인
- `alembic current`
  - 현재 revision 등 정보 제공
- `alembic history --verbose`
  - 실행한 Revision history 제공
  - `-r` 옵션을 사용하면 범위도 지정 가능
    - ex) `alembic history -r-3:current`
    - 더 자세한 범위 규칙은 문서 참고

## downgrading
- `alembic downgrade base`
  - 진행한 Upgrade를 모두 초기화

## Auto Generating Migration
- alembic은 db의 status를 확인하고 table metadata 비교 가능함. 그리고 obvious migration을 이 비교 기반으로 생성한다
- `alembic revision` 명령어에서 `--autogenerate` 옵션을 사용하여 수행 가능
- candidate Migration이라 불리는 것이 새로운 Migration file에 추가된다.
- 이걸 하기 위해선 env.py 에서 table metadata 접근할 수 있게 해야 한다
  - target_metadata에 metadata 객체를 넣어주면 됨.
  - 리스트로 객체를 넘겨줘서 여러개를 지정할 수도 있다.
- auto-generate가 감지할 수 있는 것, option으로 감지하는 것, 감지하지 못하는 사항이 문서에 정리되어 있음. 문서 참고해서 주의하여 사용할 것
- auto-generate 쓰더라도 결국 revision 결과물을 직접 보는게 좋다. 이 기능은 완벽함 지는것을 의도하지 않았다고 한다
- DB state scan시 기본적으로 Default schema가 참조되는데, EnvironmentContext.configure.include_schemas가 true면 non default schema도 스캔한다
  - include_schemas 설정을 켰을 때 모든 스키마가 scan되는게 싫다면 EnvironmentContext.configure.include_name 설정을 통해 필터링걸 수 있다.
  - table도 include_name을 통해 필터링 가능
- include_object 라는 설정도 있는데, 이를 통해 Table이나 그 안의 객체(column 등)에 대해 scan할지 필터링 가능
- comparing and rendering
  - DB 스크립트 비교하고 렌더링하는데 챌린지가 있음. 스크립트에 매우 다양한 종류의 타입이 렌더링 되어야 하는것
  - 모듈 프리픽스 컨트롤
    - 커스텀 타입을 만들었는데, Module path 때문에 너무 길어질 떄가 있다. myapp.models.utils.types.MyCustomType()
    - myapp.migration_types 이라는 모듈을 만들고 이 안에서 import를 한다 `from myapp.models.utils.types import MyCustomType`
    - script.py.mako 파일에서 위 모듈을 Import 한다 `import myapp.migration_types`
    - env.py의 run_migrations_online에서 `user_module_prefix="myapp.migration_types.",` 와 같이 설정한다.
    - 이후 해당 타입을 사용할 때 `Column("my_column", myapp.migration_types.MyCustomType())` 이런식으로 사용 가능 (myapp.models.utils.types 가 아니라 myapp.migration_types 가 되었다.)

## 그 외 인지는 하고 있으면 좋을만한 키워드들. 
  - 자세한 내용은 필요해질때 찾아보자
  - 커스텀 타입을 위한 comparing 로직을 추가할 수 있는듯 
  - revision 명령어로 생성되는 revision 스크립트에 대한 Post processing 가능
    - 포맷팅을 고치던가, 코드 분석하던가, 다시 작성하는데 사용 가능
  - `alembic check` 명령어로 새로운 revision이 필요한 상황인지 체크 가능
