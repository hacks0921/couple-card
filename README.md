# 누구 카드 쓸까? — 맞벌이 카드 몰아주기 계산기

`index.html` 파일 하나로 동작하는 모바일 웹 프로토타입입니다. 서버·빌드·라이브러리 없이 열면 바로 계산됩니다.

## 1. 열어보기

- 파일을 그대로 브라우저로 열면 됩니다. (로컬에서 더블클릭)
- 배포는 GitHub Pages에 이 파일만 올리면 끝: 저장소 → Settings → Pages → Branch `main` / `/root`.

## 2. 배포 후 3줄 설정 (index.html 맨 아래 `<script>` 안 `CONFIG`)

```js
var CONFIG = {
  PUBLIC_URL: 'https://hacks0921.github.io/couple-card/', // 배포 주소 (끝 슬래시 포함)
  WEBHOOK_URL: '',   // 아래 3번에서 만든 Apps Script 웹앱 URL
  STORAGE_KEY: 'nugu-card-comments-v1'
};
```

- `PUBLIC_URL`을 넣으면 카톡 공유 텍스트·배우자 링크·결과 이미지 하단에 항상 이 주소가 찍힙니다.
- 비워두면 지금 열린 주소를 그대로 씁니다. (`file://`로 열면 링크 공유는 동작하지 않습니다.)

## 3. 한 줄 의견 → 구글 시트 연결 (선택)

1. 새 구글 시트 → 확장 프로그램 → Apps Script → 아래 코드 붙여넣기
2. 배포 → 새 배포 → 유형 '웹 앱' → 실행 계정 '나' → 액세스 '모든 사용자'
3. 나온 웹앱 URL을 `CONFIG.WEBHOOK_URL`에 넣기

```js
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
  var d = JSON.parse(e.postData.contents);
  sheet.appendRow([new Date(), d.name, d.text, d.url, d.ua]);
  return ContentService.createTextOutput(JSON.stringify({ ok: true }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

시트 1행에 `시간 / 닉네임 / 의견 / 페이지 / UA` 순서로 쌓입니다.
`WEBHOOK_URL`이 비어 있으면 의견은 방문자 브라우저의 `localStorage`에만 저장됩니다.

## 4. 계산 기준

- 총급여의 25% 초과분부터 공제: 신용카드 15%, 체크카드·현금영수증 30%
- 신용카드 사용액이 25% 문턱을 먼저 채우고, 남은 문턱은 체크카드가 채우는 순서
- 공제 한도: 총급여 7,000만원 이하 300만원 / 초과 250만원
- 세율: 총급여에서 근로소득공제·인적공제 150만원·4대보험 9%를 뺀 추정 과세표준으로 6.6 / 16.5 / 26.4 / 38.5 / 41.8% 중 선택
- 전통시장·대중교통·문화비 추가 공제는 미반영 (간이 계산)

## 5. 바이럴 장치

| 장치 | 동작 |
| --- | --- |
| 배우자 링크 | 내 숫자를 `?a_sal=&a_cred=&a_deb=`로 담아 카톡 공유. 받은 쪽은 '나' 칸만 채우면 결론이 나옴 (받은 링크의 `a_*`는 상대 = 배우자 칸으로 들어감) |
| 결과 복사 | 카톡에 붙여넣을 요약 텍스트 |
| 이미지 저장 | 1080×1350 PNG 카드(HTML5 Canvas) — 인스타 스토리·카톡 |
| 한 줄 의견 | 로그인 없이 제출 → localStorage + (설정 시) 구글 시트 |
