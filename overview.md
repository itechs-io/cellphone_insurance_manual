# 인슈어테크서비스 휴대전화보험 청약 API 개요
이 API는 휴대전화보험 청약 연동을 쉽게 할 수 있도록 제공되는 서비스 입니다.

## API 기본정보
### 인코딩
`UTF-8`을 기본으로 사용합니다.

### 인증방식
요청 헤더에 `X-ITECHS-API-KEY` 항목으로 API Key를 포함해 요청합니다.

### SSL
모든 요청/응답은 `HTTPS` 로만 이루어집니다. `HTTP` 요청은 모두 `HTTPS`로 redirect 됩니다.

### endpoint URL
- 개발 : 미정
- 운영 : 미정

### 응답 format
당사 제공 API는 모두 아래와 같은 형태의 json 형태로 응답합니다.
```json
{
  "code": 200,
  "data": {
    "key_1": "value_1",
    "key_2": "value_2",
    "key_3": []
  },
  "error": null
}
```

## 청약절차 흐름
모든 청약 절차는 [1-rs. 계약 고유키 응답] 단계에서 응답받은 계약 고유키(`contract_id`)를 기준으로 처리됩니다.
<img src="https://github.com/itechs-io/cellphone_insurance_manual/blob/main/%EB%A7%88%EC%9D%B4%EB%B1%85%ED%81%AC%20%EC%B2%AD%EC%95%BD%20%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4_20201019.png?raw=true" width="90%"></img>

## 청약 API
### 최초 가입정보 등록
청약절차 흐름상 *1-rq. 가입정보 송신*, *1-rs. 계약 고유키 응답* 부분에 해당하는 API 입니다.

#### 요청
`POST: /cp-insurance/contract/create/`
```json
{
  "customer": {
    "name": "홍찬의",
    "birthdate": "1986-09-06",
    "cellphone": "01024846313",
    "email": "cuhong@itechs.io",
    "extra_1": "",
    "extra_2": "",
    "extra_3": null
  },
  "affiliate_uid": "63b2ca8a-259e-49c8-84f9-7d95d64459ef",
  "affiliate_cid": "9d36d220-2d92-4800-82ba-880abadbdc49"
}
```

#### 응답
```json
{
  "code": 201,
  "data": {
    "contract_id": "44608d7d-fe3f-40f4-8b3c-dad6897a2212",
    "affiliate_uid": "63b2ca8a-259e-49c8-84f9-7d95d64459ef",
    "affiliate_cid": "9d36d220-2d92-4800-82ba-880abadbdc49"
  },
  "error": null
}
```

### 캡쳐 이미지 및 부가정보 등록
청약절차 흐름상 *2-rq. 캡쳐 이미지 및 부가정보 송신*, *2-rs. OCR 성공 여부, 부가정보 일치여부, QR생성키 응답* 부분에 해당하는 API 입니다.
이 단계를 진행하지 않고 이탈된 경우 [최초 가입정보 등록] 단계에서 발급 받은 `contract_id`는 반드시 파기하시기 바랍니다.
#### 요청
`POST: /cp-insurance/contract/{contract_id}/step-1/`
```json
{
  "device_info": {
    "key": "value"
  },
  "device_info_image": "18c5afd5-055e-46f5-a3d9-f4dafe1acd83",
  "location": {
    "lat": "33.45048921971038",
    "lng": "126.56945764814947"
  }
}
```

#### 응답
```json
{
  "code": 200,
  "data": {
    "ocr": {
      "result": true,
      "imei": "123456789012345",
      "sn": "abcdefg1234567890"
    },
    "insepection": {
      "result": true,
      "msg": "성공"
    }
  },
  "error": null
}
```

### 전후면 촬영 이미지 및 결제 딥링크 등록
청약절차 흐름상 *3-rq. 전후면 촬영 이미지, 결제 딥링크 송신*, *3-rs. 결제 deadline timestamp 응답* 부분에 해당하는 API 입니다.
이 단계를 진행하지 않고 이탈된 경우 [최초 가입정보 등록] 단계에서 발급 받은 `contract_id`는 반드시 파기하시기 바랍니다.
#### 요청
`POST: /cp-insurance/contract/{contract_id}/step-2/`
```json
{
  "front_image": ["733d49b5-1414-426c-858e-0a52e199dc78", "541fc883-a29a-43ec-97ef-da429c81d10a", "7dac2d4e-792e-4f62-8bb9-3098dfaa02a5"],
  "back_image": ["2508b0a7-df51-48ab-8499-5363d67e269b", "f5c1b795-862e-4f8e-a98d-21277836d7c2", "3595d0a9-6c3b-4d6a-8ab5-d5f72d6afa79"],
  "checkout_url": "https://..."
}
```

#### 응답
```json
{
  "code": 200,
  "data": {
    "checkout": {
      "due_datetime": ""2020-12-12 13:22:20
    }
  },
  "error": null
}
```
## 기타 API
청역 진행 및 계약 조회를 위한 API는 아래와 같습니다.

### 계약정보 조회 
청약 진행 상태와 무관하게 계약 현황을 조회합니다.

#### 요청
`GET: /cp-insurance/contract/{contract_id}/`

#### 응답
```json
{
  "code": 200,
  "data": {
    "key_1": "value_1",
    "key_2": "value_2",
    "key_3": []
  },
  "error": null
}
```

### 이미지 업로드
청약 과정에서 당사로 전달하는 이미지는 모두 이 API로 업로드한 후 결과로 받은 `image_id`로 지정한다.
#### 요청
`POST: /cp-insurance/image/`

### 이미지 조회
회원사가 업로드한 이미지만 조회 가능합니다.
#### 요청
`GET: /cp-insurance/image/{image_id}/`
#### 응답
이미지 url이 반환됩니다. 이 url은 15초간만 유효합니다.
```json
{
  "code": 200,
  "data": {
    "url": "https://..."
  },
  "error": null
}
```
