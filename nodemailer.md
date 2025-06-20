## NestJS에서 Nodemailer로 이메일 쉽게 보내기 📧

오늘은 NestJS로 이메일 전송 기능을 쉽게 구현하는 방법을 소개할 것임. 이메일 기능이 필요할 때 자주 쓰이는 **Nodemailer**를 중심으로 간단하게 설명함.

---

### 📌 Nodemailer란?

`Nodemailer`는 Node.js에서 이메일을 보내게 해주는 대표적인 라이브러리임.

쉽게 비유하면,

* **Nodemailer** = 이메일을 보내주는 **집배원**
* **SMTP** = 이메일을 실제로 발송하는 **우체국**

우리가 편지를 보낼 때, 직접 우체국에 가지 않고 집배원에게 맡기듯, 코드에서도 Nodemailer가 이메일을 대신 전송해주는 것임.

---

### 📌 필요한 개념 미리 알고 가기

이메일을 NestJS로 보낼 때, 알아두면 좋은 개념들을 간단히 정리함.

| 개념                   | 설명                        | 비유           |
| -------------------- | ------------------------- | ------------ |
| SMTP                 | 이메일 전송 프로토콜               | 우체국          |
| Nodemailer           | 이메일 전송을 도와주는 라이브러리        | 집배원          |
| ConfigService        | 환경 변수(비밀번호 등)를 관리하는 서비스   | 비밀 메모장       |
| 의존성 주입               | 필요한 도구를 클래스에 자동으로 넣어주는 방식 | 알아서 준비된 도구박스 |
| 비동기 처리 (async/await) | 시간이 걸리는 작업을 기다리는 방법       | 배달 완료 기다리기   |

---

### 📌 NestJS로 Nodemailer를 쓰는 방법

#### 1️⃣ Nodemailer 설치

```bash
yarn add nodemailer
```

#### 2️⃣ 이메일 유틸리티 클래스 만들기

아래 코드처럼 간단히 구성할 수 있음.

```typescript
import { Injectable, BadRequestException } from '@nestjs/common';
import * as nodemailer from 'nodemailer';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class EmailUtil {
  private transporter;

  constructor(private readonly configService: ConfigService) {
    this.transporter = nodemailer.createTransport({
      service: 'Gmail', // Gmail을 이용할 거임
      auth: {
        user: this.configService.get<string>('NODEMAILER_EMAIL'),
        pass: this.configService.get<string>('NODEMAILER_PASSWORD'),
      },
    });
  }

  async sendCodeEmail(to: string, subject: string, text: string): Promise<void> {
    try {
      await this.transporter.sendMail({
        from: '"GPASS" <your-email@gmail.com>', // 실제 Gmail 주소여야 함
        to,
        subject,
        text,
      });
    } catch (error) {
      throw new BadRequestException({ message: '이메일 전송 실패', errors: { message: error.message } });
    }
  }
}
```

* `ConfigService`를 써서 이메일, 비밀번호를 안전하게 관리함.
* 이메일 전송 실패 시에도 친절한 예외 처리로 문제를 쉽게 파악할 수 있음.

---

### 📌 환경변수(.env) 설정법

`.env` 파일에 이메일과 비밀번호를 설정하면 됨.

```bash
NODEMAILER_EMAIL=your-email@gmail.com
NODEMAILER_PASSWORD=your-app-password
```

* Gmail의 경우 **앱 비밀번호**를 발급받아 사용하는 게 더 안전하고 필수임.

---

### 📌 이메일 보내기 실제 활용 예시

서비스 내부에서 이메일 유틸리티를 호출하는 방법은 다음과 같음.

```typescript
// 예시 서비스 코드
import { Injectable } from '@nestjs/common';
import { EmailUtil } from './email.util';

@Injectable()
export class UserService {
  constructor(private readonly emailUtil: EmailUtil) {}

  async sendVerificationEmail(userEmail: string, code: string) {
    const subject = '회원가입 인증 코드';
    const text = `인증 코드는 ${code}입니다.`;

    await this.emailUtil.sendCodeEmail(userEmail, subject, text);
  }
}
```

실제로는 컨트롤러에서 서비스를 호출하는 방식으로 구성함.

---

### ✅ 마치며

NestJS 환경에서 Nodemailer를 쓰면 이메일 전송 기능을 간편하고 안전하게 구현할 수 있음. 환경 변수를 이용한 보안 관리와 예외 처리를 통해서 안정적인 이메일 발송을 유지할 수 있음!

지금 바로 활용해보길 추천함! ✨
