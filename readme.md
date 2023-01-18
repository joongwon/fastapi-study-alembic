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


## 첫번째 마이그레이션 실행
```shell
alembic upgrade head # 마지막 인자는 revision 이름
```


## 데이터들을 만들어주자
...

## 모델 수정
```python
# models.py
class Item(Base):
    price = Column(Integer)
```
```python
# schemas.py
class ItemCreate(ItemBase):
    price: int
class Item(BaseModel):
    price: Union[int, None] = None
```

아무거나 실행해볼까?
```http request
GET /users/?skip=0&limit=100 HTTP/1.1
```

500 에러가 난다!
```
sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) no such column: items.price
[SQL: SELECT items.id AS items_id, items.title AS items_title, items.description AS items_description, items.price AS items_price, items.owner_id AS items_owner_id 
FROM items 
WHERE ? = items.owner_id]
[parameters: (1,)]
```
item을 불러오려 했는데 우리가 원하는 items.price 컬럼이 존재하지 않는다!
처음에는 알아서 디비를 다 만들어줬지만 기존 디비를 수정하는 일은 하지 못하는 것 같다...
그렇다고 기존 데이터를 다 밀어버릴 수도 없는 노릇...

## 두번째 마이그레이션
```shell
alembic revision --autogenerate -mm "Add items.price column"
```
새로운 마이그레이션 스크립트 확인해보고... 괜찮은 것 같다.
```shell
alembic upgrade head
```

## 다시 실행
다시 아까 하려고 했던 거 실행해보면 잘 된다. 생성도 잘 됨.


## 보너스
컬럼에 nullable=False 추가해보기