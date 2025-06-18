## Node.js에서 DB bigint 타입이 문자열로 들어오는 이유와 해결법

Node.js로 프로젝트를 개발하면서 데이터베이스(PostgreSQL)의 bigint 타입 컬럼을 사용할 때, 가끔 이상한 일이 생김.

바로 DB에서 숫자를 가져왔는데, JavaScript에서는 숫자가 아니라 문자열로 나타나는 현상임!

### 왜 이런 일이 생길까?

* PostgreSQL의 bigint 타입은 매우 큰 정수(64비트)를 저장할 수 있음.
* 근데 JavaScript에서는 정수를 안전하게 다룰 수 있는 한계(`Number.MAX_SAFE_INTEGER ≈ 9천조`)가 있음.
* 이 한계를 넘는 정수를 안전하게 다루기 위해, 대부분의 PostgreSQL 드라이버(node-postgres 등)가 bigint 값을 **숫자가 아닌 문자열로 반환함**.

예시로 보면:

* DB에 `id = 1234567890123456789`라고 저장됨.
* 근데 Node.js에서 가져올 때는 문자열 `'1234567890123456789'`로 나타남!

### 이러면 뭐가 문제일까?

* 코드에서 숫자끼리의 비교, 연산을 할 때 문자열을 매번 숫자로 변환해야 함.
* 불편하고, 개발자가 깜빡하면 버그 발생 가능성이 높아짐.

### 해결 방법: Transformer 사용하기!

이럴 때, TypeORM의 `ValueTransformer`를 사용하면 편리하게 해결 가능함.

바로 이 코드임:

```typescript
import { ValueTransformer } from 'typeorm';

export class BigIntTransformer implements ValueTransformer {
  to(value: number | null): number | null {
    return value; // DB에 저장할 땐 숫자 그대로 저장
  }

  from(value: string | null): number | null {
    return value !== null ? parseInt(value, 10) : null; // DB에서 가져올 땐 문자열을 숫자로 변환
  }
}
```

이걸 어떻게 쓰냐면:

```typescript
import { Generated, PrimaryColumn } from 'typeorm';
import { BigIntTransformer } from './transformers/big-int.transformer';

export class BaseIdEntity {
  @Generated('increment')
  @PrimaryColumn({
    type: 'bigint',
    transformer: new BigIntTransformer(),
  })
  id: number;
}
```

이렇게 하면:

* DB → Node.js: bigint 값을 숫자로 편하게 다룰 수 있음!
* Node.js → DB: 숫자를 그대로 bigint로 저장함.

### 결론

* PostgreSQL의 bigint는 Node.js에서 문자열로 받아와서 불편함.
* Transformer를 사용하면 자동으로 변환해서 매우 편리해짐!

이제 bigint를 마음 편히 써보자!

그럼 즐코딩 😊!
