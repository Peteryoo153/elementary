# Firebase 연결 절차

페이지 주소: https://peteryoo153.github.io/elementary/

아래 5단계를 마치면 로그인과 기록 저장이 동작합니다. 콘솔 작업(브라우저)이 필요한 부분과
파일 수정이 필요한 부분을 나눠 적었습니다.

---

## 1. Firebase 프로젝트 만들기

1. https://console.firebase.google.com 접속
2. **프로젝트 추가** → 이름 예: `lis-elementary` → Google 애널리틱스는 꺼도 무방
3. 프로젝트 개요 화면에서 **웹 아이콘(`</>`)** 클릭 → 앱 닉네임 입력 → **앱 등록**
4. 화면에 나오는 `firebaseConfig` 블록을 복사해 둡니다 (2단계에서 사용)

```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "lis-elementary.firebaseapp.com",
  projectId: "lis-elementary",
  storageBucket: "lis-elementary.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcdef"
};
```

> 이 값들은 공개되어도 되는 값입니다. 웹 클라이언트 키라서 브라우저에 그대로 노출되며,
> 실제 보호는 4단계의 **보안 규칙**이 담당합니다.

## 2. config 값 넣기

`index.html` 안쪽 `const firebaseConfig = { ... }` 블록을 1단계에서 복사한 값으로 교체하고
커밋·푸시하면 몇 분 안에 배포됩니다.

## 3. Google 로그인 켜기 + 도메인 승인

1. 콘솔 좌측 **Authentication** → **시작하기**
2. **Sign-in method** 탭 → **Google** 사용 설정 → 지원 이메일 선택 → 저장
3. **Settings** 탭 → **승인된 도메인**에 아래를 추가

```
peteryoo153.github.io
```

> 이 단계를 빠뜨리면 로그인 팝업이 `auth/unauthorized-domain` 으로 실패합니다.

## 4. Firestore 만들고 보안 규칙 적용

1. 콘솔 좌측 **Firestore Database** → **데이터베이스 만들기**
2. 위치는 `asia-northeast3 (서울)` 권장, 모드는 **프로덕션 모드**로 시작
3. **규칙** 탭을 열고 이 저장소의 [`firestore.rules`](firestore.rules) 내용을 통째로 붙여넣은 뒤 **게시**

## 5. 최초 관리자 계정 등록 (중요)

앱은 `teachers` 컬렉션에 문서가 있는 계정만 로그인시킵니다. 그런데 교사 등록 화면은
관리자만 볼 수 있으므로, **첫 관리자 한 명은 콘솔에서 직접 만들어야 합니다.**

Firestore Database → **데이터 시작하기 / 컬렉션 시작**:

| 항목 | 값 |
| --- | --- |
| 컬렉션 ID | `teachers` |
| 문서 ID | 본인 Google 이메일 (예: `stigma153.peter@gmail.com`) |
| 필드 `name` (문자열) | 이름 (예: `유피터`) |
| 필드 `role` (문자열) | `admin` |

저장한 뒤 페이지에서 Google 로그인을 하면 **관리** 탭이 나타나고,
이후 교사·학생 등록은 화면에서 직접 할 수 있습니다.

---

## 데이터 구조 참고

| 컬렉션 | 문서 ID | 필드 |
| --- | --- | --- |
| `teachers` | 이메일 | `name`, `role`(`admin`\|`teacher`) |
| `students` | 자동 | `name`, `grade`(1~5), `active` |
| `records` | 자동 | `type`(`merit`\|`detention`), `studentId`, `studentName`, `grade`, `date`(YYYY-MM-DD), `reason`, `detail`, `teacherEmail`, `teacherName`, `status`(`pending`\|`completed`\|`excused`\|`recorded`), `createdAt` |

## 문제가 생기면

| 증상 | 원인 |
| --- | --- |
| `auth/invalid-api-key` | 2단계 config 값이 아직 플레이스홀더 |
| `auth/unauthorized-domain` | 3단계 승인된 도메인 누락 |
| `등록되지 않은 계정입니다` | 5단계 teachers 문서 없음 / 이메일 철자 불일치 |
| 로그인은 되는데 목록이 비어 있음 | 4단계 규칙 미게시 또는 Firestore 미생성 |
