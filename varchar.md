# 데이터베이스 비밀번호 컬럼 길이 설정 📌

비밀번호 컬럼의 길이를 어떻게 설정할지 고민하고 있나요? 간단하면서도 중요한 내용을 쉽게 정리해봤어요!

## 1. 왜 길이를 설정해야 할까? 🤔

비밀번호는 절대 평문으로 저장하면 안 됨. 대부분의 서비스는 **해시값**(암호화된 값)을 저장함. 이 해시값 길이는 사용하는 알고리즘에 따라 다르기 때문에, 적절한 길이를 설정하는 것이 중요.

## 2. 해시 알고리즘별 추천 길이 🔑

자주 사용되는 해시 알고리즘과 권장 길이:

| 알고리즘    | 해시 결과 길이              | 추천 컬럼 길이                         |
| ------- | --------------------- | -------------------------------- |
| bcrypt  | 60자 고정                | `CHAR(60)` 또는 `VARCHAR(60)`      |
| Argon2  | 약 95자 내외(파라미터에 따라 다름) | `VARCHAR(128)` 또는 `VARCHAR(255)` |
| SHA-256 | 64자 (헥사)              | `CHAR(64)` 또는 `VARCHAR(64)`      |
| SHA-512 | 128자 (헥사)             | `CHAR(128)` 또는 `VARCHAR(128)`    |

### ✅ 간단한 예시:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  password VARCHAR(255) NOT NULL
);
```

이처럼 보편적으로 사용하는 `VARCHAR(255)`를 선택하면, 나중에 알고리즘을 바꿔도 무리 없이 대응할 수 있음!

## 3. PostgreSQL과 TypeORM에서는 어떨까? 🐘

TypeORM에서 길이를 생략하고 그냥 `VARCHAR`를 쓰면 PostgreSQL은 무제한 길이 문자열로 인식함.

예시 코드:

```typescript
@Column("varchar")
password: string;
```

이는 PostgreSQL에서는 `TEXT` 타입과 동일하게 작동.

## 4. 다른 DB에서는 길이 설정이 필수? 📚

대부분의 DB는 길이 설정이 필수입니다!

| DBMS       | 길이 지정 필수 여부 | 안 하면 어떻게 될까? |
| ---------- | ----------- | ------------ |
| MySQL      | ✔️ 필수       | 오류 발생 🚫     |
| SQL Server | ✔️ 필수       | 테이블 컬럼 오류 🚫 |
| Oracle     | ✔️ 필수       | 오류 발생 🚫     |

## 5. 길이를 설정하지 않으면 어떤 문제가 생길까? ⚠️

* **성능 저하**: 예상보다 큰 문자열이 저장돼 시스템이 느려질 수 있어요.
* **인덱스 문제**: 인덱스를 걸기 어렵거나, 제대로 동작하지 않을 수 있어요.
* **관리 문제**: 나중에 길이를 바꾸려면 번거롭게 마이그레이션 작업을 해야 해요.
* **보안 문제**: 입력값 검증이 어려워져 보안 이슈가 생길 수 있어요.

## 🎯 결론 & 베스트 프랙티스

* 비밀번호는 꼭 암호화(해싱)된 값으로 저장하세요!
* 대부분 상황에서는 `VARCHAR(255)`가 가장 무난하고 좋습니다.
* 명확한 길이를 설정해서 성능과 보안을 동시에 챙기세요! 😊
