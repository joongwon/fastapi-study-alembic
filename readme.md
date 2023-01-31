# Alembic으로 DB 마이그레이션하기

---

우리가 만든 FastAPI 앱은 완벽하다.
우리가 원하는 기능을 모두 구현했고, 버그도 없다.
데이터베이스에도 문제가 없다.
그렇게 믿었다.
그런데 아뿔싸!
물건에 가격을 안 넣었잖아!
물건에 가격이 없다는 것은 있을 수 없는 일이다.
완벽하게 만들기 위해서는 데이터베이스를 수정해야 한다.
하지만, 데이터베이스를 수정하면 기존 데이터가 사라진다.
그래서 데이터베이스를 수정하기 전에 기존 데이터를 백업해야 한다.
그리고 데이터베이스를 수정한 후에 기존 데이터를 복원해야 한다.
이런 작업을 하기 위해서는 데이터베이스를 수정하는 코드를 직접 작성해야 한다.
이런 작업은 귀찮고, 실수하기 쉽다.
그래서 데이터베이스를 수정하는 코드를 자동으로 생성해주는 도구가 필요하다.
이런 도구가 Alembic이다.

---

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

---

아무거나 실행해볼까?
```http request
GET /users/?skip=0&limit=100 HTTP/1.1
```

---

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

---

## 설치
```shell
pip install alembic
alembic init alembic # 마지막 인자는 폴더 이름
```

---

생성된 파일들
* alembic.ini: alembic 설정
* alembic/env.py: alembic 관련 환경변수
* alembic/README: 여기에 데이터베이스 설정하는 법을 쓰면 된다
* script.py.mako: 마이그레이션 스크립트 템플릿
* alembic/versions/: 여기에 마이그레이션이 생성된다

---

## 설정
수정할 내용
```python
# alembic/env.py
from sql_app import models

target_metadata = models.Base.metadata
```
```ini
# alembic.ini
sqlalchemy.url = sqlite:///./sql_app.db
```

---

## 첫번째 마이그레이션 생성

깔끔하게 하려면 비어있는 데이터베이스에서 시작하는 게 좋겠지만, 아무래도 기존 데이터도 유지하는 것이 좋겠다.
비어있는 더미 데이터베이스를 만들고, 그 DB를 바탕으로 마이그레이션을 생성한다.
수정했던 모델 파일을 다시 원래대로 돌려놓고 마이그레이션을 생성한다. (git에 잘 올려놨다면 checkout으로 되돌릴 수 있다!)
```ini
# alembic.ini
# sqlalchemy.url = sqlite:///./sql_app.db
sqlalchemy.url = sqlite:///./sql_app_dummy.db
```

```shell
alembic revision --autogenerate -m "Add user and item table"
```

강좌에서도 거듭 강조하는데, 자동 생성된 마이그레이션은 완벽하지 않다.
따라서 직접 실행 내용을 확인하고 수정하여야 한다.
지금은 수정할 내용이 없기 때문에 패스.

---

## 첫번째 마이그레이션 실행
더미 데이터베이스에 마이그레이션을 실행해보면 잘 되는 것을 확인할 수 있다.
```shell
alembic upgrade head # 마지막 인자는 revision 이름
```

---

## 다시 기존 데이터베이스로 연결해주고...
모델에도 처음에 추가하고자 했던 price column을 다시 만들어준다. (역시 git checkout)

---

## 두번째 마이그레이션
```shell
alembic revision --autogenerate -m "Add items.price column"
```
새로운 마이그레이션 스크립트 확인해보고... 괜찮은 것 같다.
```shell
alembic upgrade head
```

---

## 다시 실행
다시 아까 하려고 했던 거 실행해보면 잘 된다. 생성도 잘 됨.

---

## 보너스
### 컬럼에 nullable=False 추가해보기

---

```python
class Item(BaseModel):
    title = Column(String, index=True, nullable=False)
```
```shell
alembic revision --autogenerate -m "change items.title to non-nullable"
```
```shell
alembic upgrade head
```

---

안타깝게도 sqlite에는 기존 테이블의 컬럼을 **수정**하는 명령어 자체가 없다.

---
postgresql는 컬럼 수정하는 기능이 있으므로, postgresql로 다시 해보자.
database.py랑 alembic.ini를 수정하고 마이그레이션을 실행한다.
```
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 44a7dab1d587, Add user and item table
INFO  [alembic.runtime.migration] Running upgrade 44a7dab1d587 -> eda70f402b62, Add items.price column
INFO  [alembic.runtime.migration] Running upgrade eda70f402b62 -> 29495339f15c, change items.title to non-nullable
```

---

한편 이미 null 항목이 있는 컬럼에 `nullable=False`를 설정한다면??

---

```
sqlalchemy.exc.IntegrityError: (psycopg2.errors.NotNullViolation) column "description" of relation "items" contains null values
```
실패. null이 있는 행을 어떻게 처리할지 alembic은 모른다. 이때는 디비를 직접 수정해서 적절한 값을 설정한 뒤에 실행해야 한다.
자동생성해준다고 만능이 아니라는 것을 알 수 있다.

---

끝이에요