---
title: TypeORM 기존 운영 데이터를 유지하며 Prisma로 마이그레이션 하기
subTitle: Prisma를 서비스 도중에 적용하는 방법
tech: Prisma
category: 마이그레이션 가이드
tags:
	- NestJS
	- TypeORM
	- Prisma
	- Migration
date: 2022-04-05
---

TILog 서비스에 기능을 추가하니 기존 데이터를 유지하면서
테이블 및 컬럼을 추가, 수정, 삭제 해야 하는 요구 사항들이 많이 발생했습니다

하지만 기존 TypeORM에서 지원되는 마이그레이션은
변경, 롤백에 해당하는 로우 쿼리를 직접 작성하는 방식이기에

테이블 하나를 수정하는 작업이 너무 많은 시간을 소모하게 했습니다

이런 상황에서 TypeORM 대신 왜 Prisma를 선택 했는지
TILog 처럼 프로덕션에 유지 관리 해야 하는 데이터가 포함되어 있어 데이터베이스를 재설정 할 수 없는 경우
기존 데이터를 유지하면서 Prisma로 어떻게 마이그레이션 했는지에 대해 알아보겠습니다!

# Prisma를 선택한 이유

---

TypeORM대신 Prisma를 선택한 이유는 크게 두 가지 인데요

- prisma.schema 파일 변경으로 빠른 마이그레이션 지원
- TypeORM보다 뛰어난 타입 안정성

그 외에도 [Prisma 공식 문서](https://www.prisma.io/docs/concepts/more/comparisons/prisma-and-typeorm)에서 TypeORM과 Prisma의 차이점에 관해서 자세히 설명하고 있습니다

## 쉽고 빠른 마이그레이션

TypeORM은 작업, 작업을 되돌리는 로우 쿼리를 하나의 파일에 기록하는 방식으로 마이그레이션을 지원합니다
프로젝트 초기 단계에 이런 변경 사항을 일일이 작성하는 것이 번거롭고
데이터베이스를 변경하는 작업은 하나의 장벽처럼 느껴졌었는데요

→ Prisma는 schema.prisma 파일을 변경하는 것 만으로 자동으로 마이그레이션 파일을 생성하고 특정 지점으로 Rollback할 수 있습니다

## Type-safe

TypeORM은 컬럼을 Select하거나 Querybuilder를 사용할 때 컬럼명을 텍스트 배열로 전달하기 때문에
Typescript 컴파일러에서 리턴타입을 추정할 수 없어 해당 테이블의 Entity 유형만을 반환하는 문제가 있습니다
그로인해 존재하지 않는 프로퍼티를 참조하려고 시도하거나, 오타가 나는경우 런타임에러가 빈번해서
TypeORM을 사용할 때마다 디버깅해야 실제 데이터가 어떻게 나오는지 알 수 있고 결과에 따른 Interface도 따로 정의해야 했습니다

→ Prisma는 generate 기능을 통해 TypeScript 컴파일러와 데이터베이스 스키마 간에 정확한 리턴 Type을 작성해주고

이에 따라 이에 따라 디버깅하지 않고도 에디터에서 자동완성, 참조가 가능해 사이드 이펙트를 에디터 레벨에서 확인할 수 있습니다

## Error가 보인다!

TypeORM은 SELECT 대상 컬럼, 테이블을 텍스트로 전달하기 때문에
스키마에 변경사항이 발생하더라도 컴파일러에서 에러를 볼 수 없었습니다

→ Prisma는 정확한 Type을 정의하기때문에 일부 테이블의 스키마가 변경되면 에디터에서 오류를 확인할 수 있습니다

# Prisma 기존 프로젝트에 적용하기

---

## 개발, 프로덕션 데이터베이스 스키마 동기화 하기

TypeORM에서 데이터베이스 스키마를 Entity형태로 구성하는 것과 마찬가지로
Prisma를 사용하기 위해서는 기존 데이터베이스 스키마를 Prisma에서 지원하는 schema.prisma로 구성하는 것이 필요합니다

```shell
// prisma db pull
$ dotenv -e environments/.prod.env prisma db pull
```

dotenv-cli를 통해서 프로덕션 서버 정보를 저장하고 있는 .env 파일을 로드해서
`prisma db pull` 명령어를 실행했습니다

```json
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model category {
  id                         Int                          @id @default(autoincrement()) @db.UnsignedInt
  categoryName               String                       @db.VarChar(30)
  iconURL                    String?                      @db.VarChar(300)
  pinnedRepositoryCategories pinnedRepositoryCategories[]
  posts                      posts[]
}

model comments {
  id          BigInt    @id @default(autoincrement())
  usersID     Int       @db.UnsignedInt
  postsID     BigInt
  htmlContent String    @db.VarChar(300)
  replyTo     BigInt?
  replyLevel  Int       @default(0) @db.TinyInt
  createdAt   DateTime  @db.DateTime(0)
  updatedAt   DateTime? @db.DateTime(0)
  deletedAt   DateTime? @db.DateTime(0)
  posts       posts     @relation(fields: [postsID], references: [id], onDelete: Cascade, map: "FK_comments_postsID_posts_id")
  users       users     @relation(fields: [usersID], references: [id], onDelete: Cascade, map: "FK_comments_usersID_users_id")

  @@index([postsID], map: "FK_comments_postsID_posts_id")
  @@index([usersID], map: "FK_comments_usersID_users_id")
}
...
```

명령어를 통해 생성된 schema.prisma 파일의 일부입니다
프로덕션 서버의 스키마 정보를 담고 있습니다

```bash
// prisma db push
$ dotenv -e environments/.dev.env prisma db push
```

생성된 schema.prisma를 로컬 개발 서버 데이터베이스에 push해서 반영했습니다
위 과정을 통해서 로컬 개발서버, 프로덕션 서버의 데이터베이스 스키마를 정확하게 Sync 할 수 있습니다

## 프로덕션 데이터베이스에 마이그레이션 기존(Baselining) 설정하기

---

### 마이그레이션 기준 (Baselining) 설정 이란?

기준 설정은 TILog처럼 이미 Prisma 마이그레이션을 사용하지 않고 개발하며 유지 관리 중인 데이터가 있을 때 유용합니다
![image](https://user-images.githubusercontent.com/55491354/207647194-263c65d0-8789-43f9-817b-100a9ead8e91.png)
schema.prisma파일만 존재하는 상태로 마이그레이션 기록을 생성하게되면 위 이미지처럼
현재 데이터베이스 스키마를 온전히 재생성하는데 필요한 쿼리가 생성됩니다 (테이블 전체 스키마 백업)
결과적으로 이 마이그레이션 기록은 프로덕션 데이터베이스에서는 이미 생성된 기록임으로 반영할 필요가 없기에
기준 설정(Baselining)을 통해서 Prisma에게 이미 이 마이그레이션 기록이 적용된 상태라고 가정하도록 명령하는 것입니다.

### 첫 마이그레이션 기록 생성하기

```bash
// prisma migrate dev --name initial-migration --create-only
$ dotenv -e environments/.dev.env prisma migrate dev --name initial-migration --create-only
```

프로덕션 서버와 같은 스키마를 가지고 있는 로컬 개발 서버를 이용해서
첫 마이그레이션 기록을 생성합니다

### 마이그레이션 기준(Baselining) 설정

```sql
npx prisma migrate resolve --applied {마이그레이션 파일명}
```

resolve 명령어로 이 마이그레이션은 이미 적용된 상태라고 프로덕션 데이터베이스에 기록합니다
이후 프로덕션 데이터베이스 서버에 Prisma 전용 테이블이 생성되고 마이그레이션 히스토리가 저장됩니다

마이그레이션 기준 설정이 모두 끝났습니다. 앞으로 작성되는 새로운 마이그레이션 파일만 프로덕션 데이터베이스에 반영됩니다
하지만 테이블의 구조를 크게 변경하거나 특정 컬럼의 이름을 변경하는 경우 데이터가 초기화될 수 있습니다
이럴 때 확장 축소 패턴을 사용해서 스키마 작업을 해야 하는데요 관련 내용은 [Prisma 공식문서](https://www.prisma.io/docs/guides/database/developing-with-prisma-migrate/customizing-migrations)에 자세히 작성되어 있습니다

# Reference

---

[https://www.prisma.io/docs/guides/database/developing-with-prisma-migrate/baselining](https://www.prisma.io/docs/guides/database/developing-with-prisma-migrate/baselining)

[https://www.prisma.io/docs/reference/api-reference/command-reference#prisma-migrate](https://www.prisma.io/docs/reference/api-reference/command-reference#prisma-migrate)

[https://www.prisma.io/docs/guides/database/developing-with-prisma-migrate/add-prisma-migrate-to-a-project#update-the-development-environment](https://www.prisma.io/docs/guides/database/developing-with-prisma-migrate/add-prisma-migrate-to-a-project#update-the-development-environment)
