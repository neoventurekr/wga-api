<!-- TOC -->

* [WGA API](#wga-api)
* [개발 서버 정보](#개발-서버-정보)
    * [URL](#url)
    * [API Key](#api-key)
* [API 목록](#api-목록)
    * [Register Verification Event API](#register-verification-event-api)
        * [Authentication](#authentication)
        * [Request Body](#request-body)
        * [Fields](#fields)
        * [property Object](#property-object)
            * [mission](#mission)
            * [social](#social)
            * [game](#game)
            * [asset](#asset)
            * [device](#device)
        * [Responses](#responses)

<!-- TOC -->

# WGA API

# 개발 서버 정보

## URL

https://dev-api.wga.xyz/

## API Key

요청 시 X-API-KEY 헤더에 API Key 값을 전달해주시면 됩니다.

# API 목록

## Register Verification Event API

`POST /api/verification-events`

클라이언트(파트너 서비스)가 특정 유저 행동(소셜/게임/미션/자산 등)을 검증 이벤트로
등록하는 API입니다. WGA 서버는 이벤트를 저장하고 후속 검증/배치/온체인 앵커링 파이프라인에서 사용합니다.

### Authentication

이 API 는 WgaProjectContext.projectId()를 사용하므로, 요청은 프로젝트 인증(예: API Key 또는 JWT) 이 반드시 필요합니다.
프로젝트 인증 정보에서 projectId 가 추출되어 서버 내부에 주입됩니다.

### Request Body

```json
{
  "verificationEventType": "SOCIAL",
  "verificationPlatformType": "MOBILE",
  "verificationActionType": "POST_CREATE",
  "clientTimestamp": "2025-12-09T04:12:34Z",
  "clientUserId": "user_12345",
  "ip": "203.0.113.10",
  "userAgent": "Mozilla/5.0 ...",
  "country": "KR",
  "property": {
    "social": {
      "contentId": "post_abc",
      "contentType": "TEXT",
      "parentContentId": null,
      "textLength": 120,
      "mediaCount": 1
    },
    "device": {
      "os": "iOS",
      "osVersion": "17.1",
      "deviceModel": "iPhone15,3",
      "networkType": "WIFI"
    }
  }
}
```

### Fields

| Field                    | Type            | Required | Description                               |
|--------------------------|-----------------|----------|-------------------------------------------|
| verificationEventType    | enum            | ✅        | 이벤트 도메인 유형                                |
| verificationPlatformType | enum            | ✅        | 발생 플랫폼                                    |
| verificationActionType   | enum            | ✅        | 유저 행동 타입                                  |
| clientTimestamp          | ISO-8601 string | ✅        | 클라이언트 기준 발생 시각 (Instant)                  |
| clientUserId             | string          | ✅        | 파트너 서비스의 유저 식별자                           |
| ip                       | string          | ❌        | 클라이언트 IP                                  |
| userAgent                | string          | ❌        | User-Agent                                |
| country                  | string          | ❌        | 국가 코드 (ISO 3166-1 alpha-2) <br/>예: KR, US |
| property                 | object          | ❌        | 행동별 상세 메타데이터                              |

| Enum                  | Value                |
|-----------------------|----------------------|
| VerificationEventType | SOCIAL: 소셜, GAME: 게임 |

| Enum                     | Value                            |
|--------------------------|----------------------------------|
| VerificationPlatformType | MOBILE: 모바일, PC: PC, CONSOLE: 콘솔 |

| Enum                   | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| VerificationActionType | SIGN_UP: 회원가입, WITHDRAW: 회원탈퇴, LOG_IN: 로그인, LOG_OUT: 로그아웃,<br/>MISSION_ACCEPT: 미션 수락, MISSION_CLEAR: 미션 완료, MISSION_FAIL: 미션 실패,<br/>POST_CREATE: 게시글 작성, POST_UPDATE: 게시글 수정, POST_DELETE: 게시글 삭제,<br/>COMMENT_CREATE: 댓글 작성, COMMENT_UPDATE: 댓글 수정, COMMENT_DELETE: 댓글 삭제,<br/>LIKE_CREATE: 좋아요 생성, LIKE_DELETE: 좋아요 삭제,<br/>ASSET_INFLOW: 자산 입금, ASSET_OUTFLOW: 자산 출금,<br/>SESSION_START: 게임 세션 시작, SESSION_END: 게임 세션 종료, STAGE_CLEAR: 스테이지 클리어, STAGE_FAIL: 스테이지 실패 |

### property Object

property 는 선택이며, 행동 타입에 맞는 상세 정보를 담습니다. 구조는 아래처럼 필요한 섹션만 보내는 방식입니다.

```json
{
  "property": {
    "mission": {
      ...
    },
    "social": {
      ...
    },
    "game": {
      ...
    },
    "asset": {
      ...
    },
    "device": {
      ...
    }
  }
}
```      

#### mission

| Field            | Type            | Description    |
|------------------|-----------------|----------------|
| missionId        | string          | 미션 ID          |
| rewards          | string          | 보상 정보 (문자열 형태) |
| expiredTimestamp | ISO-8601 string | 만료 시각          |
| detail           | string          | 상세 정보          |
| failedReasons    | string          | 실패 사유          |

```json
{
  "mission": {
    "missionId": "m_1001",
    "rewards": "point:100",
    "expiredTimestamp": "2025-12-31T23:59:59Z",
    "detail": "Invite 3 friends",
    "failedReasons": null
  }
}
```

#### social

| Field           | Type   | Description          |
|-----------------|--------|----------------------|
| contentId       | string | 컨텐츠 ID               |
| contentType     | enum   | 컨텐츠 타입               |
| parentContentId | string | 부모 컨텐츠 ID (답글/대댓글 등) |
| textLength      | number | 텍스트 길이               |
| mediaCount      | number | 미디어 첨부 수             |      

| Enum        | Value                             |
|-------------|-----------------------------------|
| contentType | POST: 게시글, COMMENT: 댓글, LIKE: 좋아요 |

```json
{
  "social": {
    "contentId": "post_abc",
    "contentType": "TEXT",
    "parentContentId": null,
    "textLength": 120,
    "mediaCount": 1
  }
}
```

#### game

| Field           | Type    | Description |
|-----------------|---------|-------------|
| playDurationSec | number  | 플레이 타임 (초)  |
| success         | boolean | 성공 여부       |
| score           | number  | 점수          |

```json
{
  "game": {
    "playDurationSec": 340,
    "success": true,
    "score": 9850
  }
}
```

#### asset

| Field      | Type          | Description    |
|------------|---------------|----------------|
| type       | enum          | 자산 타입          |
| name       | string        | 자산 이름          |
| amount     | string/number | 수량 (정밀도 고려 권장) |
| actionType | enum          | 변동 유형          |
| isCash     | boolean       | 유료 재화 여부       |

| Enum       | Value                                                                 |
|------------|-----------------------------------------------------------------------|
| actionType | SPEND: 소비, EARN: 획득, TRANSFER: 이동, GRANT: 지급(시스템을 통한 획득), DISCARD: 폐기 |

```json
{
  "asset": {
    "type": "COIN",
    "name": "GOLD",
    "amount": "10.5",
    "actionType": "EARN",
    "isCash": false
  }
}
```

#### device

| Field       | Type   | Description |
|-------------|--------|-------------|
| os          | string | 운영체제        |
| osVersion   | string | OS 버전       |
| deviceModel | string | 디바이스 모델     |
| networkType | enum   | 네트워크 타입     |

| Enum        | Value                                                                    |
|-------------|--------------------------------------------------------------------------|
| networkType | WIFI: WIFI, CELL_3G: 모바일 3G, CELL_4G: 모바일 4G, CELL_5G: 모바일 5G, OTHER: 기타 |

```json
{
  "device": {
    "os": "Android",
    "osVersion": "14",
    "deviceModel": "SM-S918N",
    "networkType": "WIFI"
  }
}
```

### Responses

현재 컨트롤러는 별도 응답 바디가 없습니다.

- 200 OK
- 인증 실패 시: 403
