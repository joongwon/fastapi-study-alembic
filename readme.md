## 설치
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

## 설정
수정할 내용
```python
# alembic/env.py
from sql_app import models

target_metadata = models.Base.metadata
```
```ini
sqlalchemy.url = sqlite:///./sql_app.db
```

## 첫번째 마이그레이션 생성
일단 sql_app.db를 지우고 시작한다. 아무것도 없는 상태에서 데이터베이스 생성부터 하는 게 좋을 것 같다.
```shell
alembic revision --autogenerate -m "Add user and item table"
```

자동생성은 완벽하지 않기 때문에 직접 실행 내용을 확인하고 수정하여야 한다.
지금은 수정할 내용이 없기 때문에 패스.


## 데이터들을 만들어주자
...

## 두번째 마이그레이션