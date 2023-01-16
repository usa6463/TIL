# Alembic

Python, SQLAlchemy 기반의 database migration tool

## 설치 및 환경 구성

- pip install alembic
- 이후 빈 폴더에에서 `alembic init .`
  - 이걸 migration environment를 만든다고 한다.

## directory structure

- alembic/
  - env.py
  - README
  - script.py.mako
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

