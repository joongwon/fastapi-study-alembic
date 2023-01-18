```shell
pip install alembic
alembic init alembic # 마지막 인자는 폴더 이름
```

생성된 파일들
* alembic.ini: alembic 설정
* alembic/env.py: alembic 관련 환경변수
* alembic/README: 여기에 데이터베이스 설정하는 법을 쓰면 된다
* script.py.mako: 마이그레이션 스크립트 템플릿
* alembic/versions/: 여기에 마이그레이션이 생성된다