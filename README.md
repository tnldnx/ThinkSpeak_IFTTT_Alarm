# ThinkSpeak_IFTTT_Alarm

## 1.  목표

- ThingSpeak를 이용하여 가변저항값과 LED 상태 모니터링
- React를 이용하여 센서값의 임계치 초과 여부 확인
- ThingHTTP와 IFTTT Webhooks 연동
- 특정 조건 충족 시 스마트폰으로 푸시 알림 전송

## 2. 시스템 동작 흐름

```text
[ThingSpeak API 요청]
         |
         | field1=900, field2=1
         v
[ThingSpeak Channel]
         |
         | 가변저항값 저장
         v
[React]
         |
         | Field 1 > 800
         v
[ThingHTTP]
         |
         | HTTP POST / JSON
         v
[IFTTT Webhooks]
         |
         | Event: toohigh
         v
[IFTTT Notifications]
         |
         v
[스마트폰 푸시 알림]
```

## 3. ThingSpeak 채널 생성

ThingSpeak에서 `My First IoT Channel`을 생성하고 다음과 같이 필드를 설정.

| 항목 | 설정 |
| --- | --- |
| Channel ID | 3565687 |
| Field 1 | 가변저항 |
| Field 2 | LED 상태 |
| Description | 가변저항 값과 스위치로 조절되는 LED 상태 값 모니터링 채널 |

Field 1에는 가변저항값을, Field 2에는 LED의 ON/OFF 상태를 저장.

### ThingSpeak 채널 모니터링 결과

<img src="https://github.com/user-attachments/assets/0ffab2d8-6d31-40f0-b911-302db1b04ead" width="700" alt="ThingSpeak 채널 그래프">

Field 1에 기록된 값 `900`을 그래프와 게이지에서 확인.

## 4. IFTTT Webhooks 설정

### 4-1. Trigger 설정

IFTTT에서 새로운 Applet을 생성하고 **If This** 조건으로 Webhooks를 선택.

- Service: Webhooks
- Trigger: Receive a web request with a JSON payload
- Event Name: `toohigh`

이벤트 이름은 ThingHTTP에서 호출하는 Webhook URL의 이벤트 이름과 일치해야함.

### 4-2. Notification Action 설정

**Then That**에서 Notifications 서비스 선택.

- Service: Notifications
- Action: Send a notification from the IFTTT app
- AI Action assist: OFF
- High priority: Yes

알림 메시지에는 Webhooks의 `EventName`과 `JsonPayload`를 사용.

```text
The event named "{{EventName}}" occurred on
the Webhooks service
condition: {{JsonPayload}}
```

이를 통해 Webhooks 이벤트가 발생하면 IFTTT 앱에서 알림을 받을 수 있도록 설정.

## 5. ThingHTTP 설정

ThingSpeak의 **Apps → ThingHTTP**에서 IFTTT Webhooks로 HTTP 요청을 전송하도록 설정.

| 항목 | 설정값 |
| --- | --- |
| Name | Too High Value in Pot Field |
| URL | IFTTT Webhooks Trigger URL |
| Method | POST |
| Content Type | application/json |
| HTTP Version | 1.1 |

### HTTP Request Body

```json
{
  "value1": "%%channel_3565687_field_1%%"
}
```

`%%channel_3565687_field_1%%`은 ThingSpeak 채널의 Field 1 값을 참조하는 변수.

따라서 Field 1 값이 `900`이면 IFTTT에 해당 값을 전달하도록 구성했다.

### ThingHTTP 설정 화면

<img src="https://github.com/user-attachments/assets/1a6043ae-ad9f-4425-aa38-64c122bcf2ed" width="500" alt="ThingHTTP 설정 화면">


## 6. React 조건 설정

ThingSpeak의 **Apps → React**에서 가변저항값이 특정 임계치를 초과할 때 ThingHTTP를 실행하도록 설정.

| 항목 | 설정값 |
| --- | --- |
| React Name | variable resistor value becomes too high |
| Condition Type | Numeric |
| Test Frequency | On Data Insertion |
| Channel | My First IoT Channel |
| Field | Field 1 (가변저항) |
| Condition | is greater than 800 |
| Action | ThingHTTP |
| ThingHTTP Name | Too High Value in Pot Field |
| Options | Run action only the first time the condition is met |

즉, 가변저항값이 `800`을 초과하면 React 조건이 만족되어 ThingHTTP가 실행.

### React 설정 화면

<img src="https://github.com/user-attachments/assets/0938874b-d007-4f25-b4ba-ffc413c7b5df" width="500" alt="React 설정 화면">

## 7. API Request 테스트

ThingSpeak의 Write API를 이용하여 테스트 데이터를 전송.

### 테스트 요청

```text
https://api.thingspeak.com/update?api_key=YOUR_WRITE_API_KEY&field1=900&field2=1
```

- `field1=900`: 가변저항값 900
- `field2=1`: LED ON 상태


### 조건 확인

```text
Field 1 = 900
Threshold = 800

900 > 800

→ React 조건 만족
→ ThingHTTP 실행
→ IFTTT Webhooks 이벤트 발생
→ 스마트폰 알림 전송
```

## 8. 실행 결과

### 8-1. ThingSpeak 데이터 확인

API 요청 후 ThingSpeak의 Field 1 Chart와 가변저항 게이지에 `900`이 표시되는 것을 확인할 수 있음.

Field 2에는 LED 상태값 `1`을 전송하여 LED ON 상태를 기록.

### 8-2. IFTTT 알림 확인

가변저항값이 임계치 `800`을 초과하는 경우, React와 ThingHTTP를 통해 IFTTT 알림을 전송하도록 구성.


<img src="https://github.com/user-attachments/assets/51e2ea4d-df0a-4877-854f-393cd949de1e" width="350" alt="IFTTT 스마트폰 알림 수신 화면">

## 9. 실습 결과 및 느낀 점

이번 실습에서는 ThingSpeak와 IFTTT를 연동하여 센서 데이터를 모니터링하고, 특정 조건이 충족되면 스마트폰으로 알림을 전송하는 IoT 시스템을 구성하는 미션이었다.

ThingSpeak의 React 기능으로 임계치를 검사하고, ThingHTTP를 통해 외부 서비스인 IFTTT에 HTTP 요청을 전달하는 과정을 이해할 수 있었다.

또한 ThingSpeak API를 이용하여 가변저항값과 LED 상태를 기록하고, 그래프와 게이지를 통해 데이터가 반영되는 것을 확인했다.

이번 실습을 통해 센서 데이터 수집부터 조건 판단, 외부 서비스 연동 및 사용자 알림까지 이어지는 IoT 시스템의 전체 동작 과정을 학습할 수 있었다.
