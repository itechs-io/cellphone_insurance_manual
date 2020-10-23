# 인슈어테크서비스 휴대전화보험 청약 API 개요
```diff
- API는 현재 디자인 단계입니다. 참고용으로만 활용해주세요.
```
이 API는 휴대전화보험 청약 연동을 쉽게 할 수 있도록 제공되는 서비스 입니다.

## 문의
홍찬의, 010-2484-6313, cuhong@itechs.io

## 문서이력
- 2020-10-22
  - 가입가능 디바이스 확인 API 추가
  - 이미지 업로드 응답 수정(이미지 `id` 추가)
  - 최초 가입정보 등록시 요청 필드 수정
- 2020-10-19
  - 최초작성

## API 기본정보
### 인코딩
`UTF-8`을 기본으로 사용합니다.

### 인증방식
요청 헤더에 `X-ITECHS-API-KEY` 항목으로 API Key를 포함해 요청합니다.
API Key는 별도로 전달됩니다.

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

*재작성중*
<!-- <img src="https://github.com/itechs-io/cellphone_insurance_manual/blob/main/process.png?raw=true" width="90%"></img> -->

## 청약 API
청약은 아래 단계로 진행됩니다. 각 단계에선 이전단계의 작업을 수행할 수 없습니다.
### 가입가능 기기 정보 조회
`GET: /cp-insurance/candidate/`
가입가능한 단말기의 정보를 반환합니다.

#### 응답
```json
{
  "code": 200,
  "data": {
    "device_list": [
      {"id": "16e98243-33bb-413b-ac2c-9d51db71151d", "name": "삼성 갤럭시 노트 10", "model_name": "SM-N971N"},
      {"id": "47d3c6cc-5d0d-44e5-9226-a5a85f775979", "name": "삼성 갤럭시 노트 10+", "model_name": "SM-N976N"}
    ]
  },
  "error": null
}
```

### 가입가능 기기 확인
`POST: /cp-insurance/candidate/`
요청한 기기 정보를 확인하고 가입 가능 여부를 응답합니다.

#### 요청
```json
# 아이폰 SE의 경우
{
  "os": "ios",
  "device_name": "iPhone8,4"
}

# 삼성 갤럭시 노트 20의 경우
{
  "os": "aos",
  "device_name": "SM-N981N"
}
```

#### 응답
`available`(가입가능여부) 항목을 true/false 로 반환한다.
```json
{
  "code": 200,
  "data": {
    "available": true
  },
  "error": null
}
```


### 최초 가입정보 등록
청약절차 흐름상 *1-rq. 가입정보 송신*, *1-rs. 계약 고유키 응답* 부분에 해당하는 API 입니다.
이 단계가 완료되면 `contract`의 `status`는 `0`(계약 요청)으로 설정됩니다.

#### 요청
`POST: /cp-insurance/contract/create/`
```json
{
  "customer": {
    "name": "홍찬의",
    "ssn": "8609061000000",
    "cellphone": "01024846313",
    "email": "cuhong@itechs.io",
    "extra_1": "",
    "extra_2": "",
    "extra_3": null
  },
  "device": {
    "os": "ios",
    "name": "iPhone8,4"
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
이 단계가 완료되면 `contract`의 `status`는 `1`(기기정보 등록)으로 설정됩니다.

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
이 단계가 완료되면 `contract`의 `status`는 `2`(결제 대기)으로 설정됩니다.
#### 요청
`POST: /cp-insurance/contract/{contract_id}/step-2/`
```json
{
  "front_image": ["733d49b5-1414-426c-858e-0a52e199dc78", "541fc883-a29a-43ec-97ef-da429c81d10a", "7dac2d4e-792e-4f62-8bb9-3098dfaa02a5"],
  "rear_image": ["2508b0a7-df51-48ab-8499-5363d67e269b", "f5c1b795-862e-4f8e-a98d-21277836d7c2", "3595d0a9-6c3b-4d6a-8ab5-d5f72d6afa79"],,
  "location": {
    "lat": "33.45048921971038",
    "lng": "126.56945764814947"
  },
  "checkout_url": "https://..."
}
```

#### 응답
```json
{
  "code": 200,
  "data": {
    "checkout": {
      "due_datetime": "2020-12-12 13:22:20"
    }
  },
  "error": null
}
```

### 결제정보 등록
청약절차 흐름상 *4-rq. 결제 타입(일시납/월납), 결제 성공일시, 결제금액 전달* 부분에 해당하는 API 입니다.
이 단계가 완료되면 `contract`의 `status`는 `3`(검수요청)으로 설정됩니다.
[후면 촬영 이미지 및 결제 딥링크 등록] 에서 응답받은 `due_datetime`이 경과했을 경우 `contract_id`를 파기하시기 바랍니다.
#### 요청
`POST: /cp-insurance/contract/{contract_id}/checkout/`
```json
{
  "type": "0",
  "amount": 10000,
  "datetime": "2020-12-13 09:22:22"
}
```

#### 응답
```json
{
  "code": 200,
  "data": {
    "result": true
  },
  "error": null
}
```

### 결제취소
이 단계는 `contract`의 `status`가 `3`, `4`, `5`일 경우만 가능합니다.
[후면 촬영 이미지 및 결제 딥링크 등록] 에서 응답받은 `due_datetime`이 경과했을 경우 `contract_id`를 파기하시기 바랍니다.
#### 요청
`POST: /cp-insurance/contract/{contract_id}/checkout/cancel/`
```json
{
  "datetime": "2020-12-13 09:22:22"
}
```

#### 응답
```json
{
  "code": 200,
  "data": {
    "result": true
  },
  "error": null
}
```

### 검수결과 webhook
***이 API는 고객사가 개발하셔야 합니다.***
#### 검수결과
*5-rq. 검수실패 사유 및 검수자료 갱신 deadline 전달*, *5-rs. 재검수 딥링크 응답*, *6-rq. 검수완료일자, 보험 게시일자, 만료일자, 다음 보험료 납입기한(월납) 전달*, *6-rs. 가입정보 조회 딥링크 응답.* 단계에 해당하는 부분입니다.
이 단계가 완료되면 `contract`의 `status`는 `4`(재검수요청), `5`(검수완료), `6`(인수거절), `7`(계약완료)로 설정됩니다.
만약 이 정보를 `POST`한 후 응답이 정상적으로 수신되지 않았을 경우 일정 시간을 두고 재전송 할 수 있습니다.(재전송 인터벌은 고객사 협의)

##### 검수완료(일시납)
검수가 완료된 건은 검수 완료 시점에서 `status`가 5로 변경되며, webhook으로 전송하고 정상 응답을 수신할 경우 `7`(계약완료)로 설정됩니다.(고객에게 알림톡이 발송됩니다.)
###### 요청
```json
{
  "result": 0,
  "inspection_complete": "2020-12-14 13:22:22",
  "insurance": {
    "insurance_start": "2020-12-15",
    "insurance_end": "2021-12-14"
  }
}
```

###### 응답
수신이 성공했을 경우 고객의 계약정보를 조회할 수 있는 링크를 생성하여 응답해주시면 됩니다.
```json
{
  "result": true,
  "contract_url": "https://..."  
}
```

수신은 성공했으나 처리중 문제가 발생하였을 경우 아래와 같이 간단한 에러 메시지(식별이 용이하도록 고객사에서 관리하는 sequence를 보내주시면 더 좋습니다)와 함께 응답처리 해주시면 별도로 대응하겠습니다.
```json
{
  "result": false,
  "error": "tell me something about your gloomy exception..."
}
```

###### 검수완료(월납)
검수가 완료된 건은 검수 완료 시점에서 `status`가 5로 변경되며, webhook으로 전송하고 정상 응답을 수신할 경우 `contract`의 `7`(계약완료)로 설정됩니다.(고객에게 알림톡이 발송됩니다.)
```json
{
  "result": 0,
  "inspection_complete": "2020-12-14 13:22:22",
  "insurance": {
    "insurance_start": "2020-12-15",
    "insurance_end": "2021-12-14"
  },
  "checkout": {
    "due_date": "2021-1-31",
    "amount": 30000
  }
}
```

###### 응답
수신이 성공했을 경우 고객의 계약정보를 조회할 수 있는 링크를 생성하여 응답해주시면 됩니다.
```json
{
  "result": true,
  "contract_url": "https://..."  
}
```

수신은 성공했으나 처리중 문제가 발생하였을 경우 아래와 같이 간단한 에러 메시지와 함께 응답처리 해주시면 별도로 대응하겠습니다.
```json
{
  "result": false,
  "error": "tell me something about your gloomy exception..."
}
```

###### 재검수요청
당사에서 이 내용을 webhook으로 전송하고 정상 응답을 수신할 경우 `contract`의 `status`가 `4`(재검수요청)로 설정됩니다.(고객에게 알림톡이 발송됩니다.)
```json
{
  "result": 1,
  "due_datetime": "2020-12-15 13:12:22",
  "detail": {
    "front": {
      "result": false,
      "msg": "전면 액정이 손가락에 가려져 있습니다"
    },
    "rear": {
      "result": true,
      "msg": null
    }
  }
  "msg": "재촬영이 필요합니다."
}
```

###### 응답
수신이 성공했을 경우 고객이 검수 데이터를 다시 등록할 수 있는 `url`을 생성하여 보내주십쇼.
```json
{
  "result": true,
  "reinspection_url": "https://..."  
}
```

수신은 성공했으나 처리중 문제가 발생하였을 경우 아래와 같이 간단한 에러 메시지와 함께 응답처리 해주시면 별도로 대응하겠습니다.
```json
{
  "result": false,
  "error": "tell me something about your gloomy exception..."
}
```

###### 인수거절
당사에서 이 내용을 webhook으로 전송하고 정상 응답을 수신할 경우 `contract` `status`가 `6`(인수거절)로 설정됩니다.(고객에게 알림톡이 발송됩니다.)
```json
{
  "result": 2,
  "reject_type": 0,
  "msg": "전면액정 파손으로 가입이 거절되었습니다."
}
```

###### 응답
수신이 성공했을 경우 아래와 같이 응답 부탁드립니다.
```json
{
  "result": true,
}
```

수신은 성공했으나 처리중 문제가 발생하였을 경우 아래와 같이 간단한 에러 메시지와 함께 응답처리 해주시면 별도로 대응하겠습니다.
```json
{
  "result": false,
  "error": "tell me something about your gloomy exception..."
}
```

### 재검수 이미지 등록
이 단계가 완료되면 `contract`의 `status`는 `3`(검수요청)으로 설정됩니다.
#### 요청
`POST: /cp-insurance/contract/{contract_id}/step-2/update/`
```json
{
  "front_image": ["733d49b5-1414-426c-858e-0a52e199dc78", "541fc883-a29a-43ec-97ef-da429c81d10a", "7dac2d4e-792e-4f62-8bb9-3098dfaa02a5"],
  "rear_image": ["2508b0a7-df51-48ab-8499-5363d67e269b", "f5c1b795-862e-4f8e-a98d-21277836d7c2", "3595d0a9-6c3b-4d6a-8ab5-d5f72d6afa79"],,
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
#### 응답
이미지 url이 반환됩니다. 이 url은 15초간만 유효합니다.
```json
{
  "code": 201,
  "data": {
    "id": "61a5861a-4f03-4213-80b2-df6546e10201",
    "url": "https://..."
  },
  "error": null
}
```

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
