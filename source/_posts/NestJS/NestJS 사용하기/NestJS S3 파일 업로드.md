---
title: NestJS S3 파일업로드
subTitle: AWS S3와 서비스 DB 사이 정합성에 대해
tech: NestJS
category: NestJS 사용하기
tags:
	- NestJS
	- AWS
	- AWS S3
date: 2021-12-02
---

# 요구조건

- AWS S3 버킷을 사용해 파일을 업로드합니다
- 업로드 기록을 데이터베이스에 저장해야합니다
- 데이터베이스에 저장된 기록은 S3에 대조 했을 때 항상 정합성을 유지해야합니다
- 탈퇴한 유저인 경우 스케쥴링을 통해 해당 유저가 소유한 파일을 삭제할 수 있어야합니다

---

# 정합성에 대해

`정합성`이 보장된다는것은 데이터들의 값이 서로 일치하다는 뜻입니다.

데이터베이스의 경우 FK, 트랜잭션을 통해 데이터의 일관성과 정합성을 보장해주고 있지만
AWS S3와 같이 우리 서비스와 동 떨어져 있는 상황에서 어떻게하면 위 요구조건을 만족할 수 있을까요?

---

# 설계

![image](https://user-images.githubusercontent.com/55491354/193412101-670d536d-12d8-4a18-b72b-65142f1c0b89.png)
트랜잭션을 통해서 업로드 정보를 기록하는 것 까지는 쉬워 보이지만
AWS S3가 포함될 경우 데이터 정합성을 일부 보장할 수 없다는 문제가 있습니다

S3 버킷에 파일을 업로드했더라도 데이터베이스에서 쓰기가 실패했으면 롤백을 진행해야 하지만
S3는 롤백의 개념이 없기 때문에 추가로 요청된 삭제요청이 실패할 수 있는 것이죠

그렇기에 S3 삭제 요청 실패로 삭제되지 못한 파일이 남아 있을 수 있는 가능성에 대해서는 인정하되
데이터베이스에 기록된 파일 리스트에 대한 데이터 정확성은 반드시 지켜져야 한다 판단하고 플로우를 작성했습니다

---

# ERD

```sql
-- imageUpload Table Create SQL
CREATE TABLE imageUpload
(
    `id`             BIGINT          NOT NULL        AUTO_INCREMENT COMMENT '이미지 업로드 아이디',
    `usersID`        INT UNSIGNED    NULL    COMMENT '유저 아이디',
    `pathUrl`        VARCHAR(300)    NOT NULL    COMMENT '이미지 URL 정보',
    `fileSizeBytes`  INT             NOT NULL    COMMENT '파일 사이즈 정보',
    `fileType`       VARCHAR(20)     NOT NULL    COMMENT '파일 타입 정보',
    `createdAt`      DATETIME        NOT NULL    COMMENT '파일 업로드일',
    CONSTRAINT PK_imageUpload PRIMARY KEY (id)
);

ALTER TABLE imageUpload COMMENT '이미지 업로드';

-- 외래키 제약 설정
ALTER TABLE imageUpload
    ADD CONSTRAINT FK_imageUpload_usersID_users_id FOREIGN KEY (usersID)
        REFERENCES users (id) ON DELETE SET NULL ON UPDATE CASCADE;
```

- 유저가 탈퇴(하드 딜리트)되었을 경우의 지표로 사용하기위해 외래키 제약을
- ON DELETE SET NULL로 설정했습니다

---

# ImageUpload Service

## constructor

```tsx
@Injectable()
export class FileUploadsService {
  private readonly S3_BUCKET_NAME: string;
  private readonly S3: AWS.S3;

  constructor(private readonly configService: ConfigService, private connection: Connection) {
    // S3 초기화
    this.S3_BUCKET_NAME = this.configService.get<string>("AWS_S3_BUCKET", "noName");
    this.S3 = new AWS.S3({
      accessKeyId: this.configService.get<string>("AWS_S3_ACCESS_KEY", "noKey"),
      secretAccessKey: this.configService.get<string>("AWS_S3_KEY_SECRET", "noKey"),
    });
  }
}
```

AWS S3버킷 이름과 S3 서비스 객체를 생성합니다

S3 서비스 객체를 생성할 때 사용된 IAM의 권한은 `AmazonS3FullAccess` 입니다.

## s3FileUpload

```tsx
/**
   * s3 파일 업로드를 요청합니다.
   * @author seongrokLee <argon1025@gmail.com>
   * @version 1.0.0
   * @throws {S3FileUploadFail}
   */
  private async s3FileUpload(requestData: S3FileUploadDto): Promise<S3FileUploadResponseDto> {
    try {
      // 파일 업로드를 요청합니다.
      const s3FileUploadRequestResult = await this.S3.upload({
        Bucket: this.S3_BUCKET_NAME,
        Key: requestData.fileName,
        Body: requestData.fileRaw,
        ACL: 'public-read',
        ContentType: requestData.mimeType,
        ContentDisposition: 'inline',
      }).promise();

      // DTO Mapping
      let response: S3FileUploadResponseDto = new S3FileUploadResponseDto();
      response.fileName = s3FileUploadRequestResult.Key;
      response.location = s3FileUploadRequestResult.Location;

      return response;
    } catch (error) {
      // 에러 반환
      throw new S3FileUploadFail(`fileUpload.service.s3FileUpload.${!!error.message ? error.message : 'Unknown_Error'}`);
    }
  }
```

파일 업로드를 요청하는 메서드 입니다

## s3FileDelete

```typescript
/**
   * s3 파일 삭제를 요청합니다.
   * @author seongrokLee <argon1025@gmail.com>
   * @version 1.0.0
   */
  private async s3FileDelete(requestData: S3FileDeleteDto): Promise<boolean> {
    try {
      /**
       * 파일 삭제를 요청합니다
       * @Returns void
       */
      await this.S3.deleteObject({ Bucket: this.S3_BUCKET_NAME, Key: requestData.key }).promise();
      return true;
    } catch (error) {
      return false;
    }
  }
```

파일 삭제를 요청하는 메서드 입니다

## imageFileUpload : public

```tsx
/**
   * 하나의 이미지를 업로드하고 URL을 반환합니다.
   * - 파일버퍼, 커스텀 파일이름, 유저 아이디
   * @author seongrokLee <argon1025@gmail.com>
   * @version 1.0.0
   */
  public async imageFileUpload(requestData: ImageFileUploadDto): Promise<ImageFileUploadResponseDto> {
    // 파일 타입
    let FILE_TYPE: string;
    // 파일 확장자
    let FILE_EXTENSION: string;

    // 쿼리러너 객체 생성
    const queryRunner = this.connection.createQueryRunner();

    // 데이터 베이스 연결
    await queryRunner.connect();

    // 트랜잭션 시작
    await queryRunner.startTransaction();

    try {
      // 파일의 타입, 확장자를 저장합니다 'image/png'
      const FILE_INFO = requestData.file.mimetype.split('/');
      FILE_TYPE = FILE_INFO[0];
      FILE_EXTENSION = FILE_INFO[1];

      // 파일 타입이 이미지가 아닐 경우 오류를 반환합니다
      if (FILE_TYPE != 'image') {
        throw new Error('THIS_IS_NOT_IMAGE');
      }

      // 파일 업로드 DTO를 작성합니다.
      let s3FileUploadDTO = new S3FileUploadDto();
      s3FileUploadDTO.fileName = `${requestData.fileName}.${FILE_EXTENSION}`;
      s3FileUploadDTO.fileRaw = requestData.file.buffer;
      s3FileUploadDTO.mimeType = requestData.file.mimetype;

      // 파일 업로드를 요청하고 결과를 받습니다.
      const s3FileUploadResult = await this.s3FileUpload(s3FileUploadDTO);

      // 데이터베이스에 파일 업로드정보를 기록합니다.
      const imageUploadQuery = queryRunner.manager
        .createQueryBuilder()
        .insert()
        .into(ImageUpload)
        .values([
          {
            usersId: requestData.usersId,
            pathUrl: s3FileUploadResult.location,
            fileSizeBytes: requestData.file.size,
            fileType: requestData.file.mimetype,
            createdAt: Time.nowDate(),
          },
        ])
        .updateEntity(false);

      /**
       *
       * @Returns InsertResult {
       *   identifiers: [],
       *   generatedMaps: [],
       *   raw: ResultSetHeader {
       *     fieldCount: 0,
       *     affectedRows: 1,
       *     insertId: 2,
       *     info: '',
       *     serverStatus: 3,
       *     warningStatus: 0
       *   }
       * }
       */
      const imageUploadQueryResult = await imageUploadQuery.execute();

      // 테이블 삽입이 반영되었는지 확인합니다.
      if (imageUploadQueryResult.raw.affectedRows === 0) {
        throw new Error('imageUploadQueryResult_AFFECTED_IS_0');
      }

      // response DTO Mapping
      const response = new ImageFileUploadResponseDto();
      response.pathUrl = s3FileUploadResult.location;

      // 트랜잭션 커밋
      await queryRunner.commitTransaction();

      return response;
    } catch (error) {
      // 트랜잭션 롤백
      await queryRunner.rollbackTransaction();

      // 데이터베이스 파일 업로드 정보 기록에 실패했을 경우 등록된 파일의 삭제를 요청합니다.
      if (error.message === 'imageUploadQueryResult_AFFECTED_IS_0') {
        this.s3FileDelete({ key: `${requestData.fileName}.${FILE_EXTENSION}` });
      }

      // 최종 에러 생성
      throw new ImageUploadFail(`service.file-uploads.imageFileUpload.${!!error.message ? error.message : 'Unknown_Error'}`);
    } finally {
      // 데이터베이스 커넥션 해제
      await queryRunner.release();
    }
  }
```

위 두개의 메서드를 통해 파일 업로드서비스 로직을 작성했습니다.

---

# 부가 목표

- 작업 큐를 통한 업로드, 삭제 처리
  - S3 성능 최적화 관련 이슈
- 스케쥴러를 통한 탈퇴회원 업로드 파일 관리 처리

---

# Reference

AWS S3 성능 최적화 모범사례 패턴

[https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/optimizing-performance.html](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/optimizing-performance.html)

Node에서 multipart/form-data 를 다루기 위한 모듈 Multer 공식 문서

[https://github.com/expressjs/multer/blob/master/doc/README-ko.md](https://github.com/expressjs/multer/blob/master/doc/README-ko.md)
