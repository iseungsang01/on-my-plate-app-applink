# 초대 링크(Android App Links) 호스팅

`planat://group/join?code=...` 커스텀 스킴은 카카오톡 등 메신저에서 **자동으로 눌리는 링크가 되지 않는다.**
그래서 "공유 코드가 따로 실행되지 않는다"는 문제가 생긴다. 이를 해결하려면 **진짜 https 링크**를 쓰고
Android **App Links**로 그 링크를 우리 앱이 가로채게 해야 한다. 도메인은 무료 호스팅(Vercel / GitHub Pages)으로 충분하다.

이 폴더(`web/applink/`)를 **사이트 루트로** 배포하면:

- `https://<호스트>/?code=XXXXXX` → 초대 폴백 페이지(`index.html`)
- `https://<호스트>/.well-known/assetlinks.json` → App Links 검증 파일

App Links가 검증되면 초대 링크를 누르는 순간 **이 페이지가 뜨기 전에 앱이 바로 열린다.**
앱이 없거나 아직 검증 전이면 이 페이지가 폴백으로 보이고, 코드 복사 + 설치 안내를 제공한다.

---

## 배포 (둘 중 하나)

### A. Vercel (권장 — 루트 도메인이 깔끔)
1. https://vercel.com 로그인 → **Add New → Project**.
2. 이 저장소를 연결하고 **Root Directory = `web/applink`** 로 지정(빌드 명령 없음, 정적 파일).
3. 배포되면 호스트는 `https://<프로젝트>.vercel.app`.
4. `https://<프로젝트>.vercel.app/.well-known/assetlinks.json` 이 열리는지 확인.

### B. GitHub Pages (완전 무료, 단 루트 저장소 필요)
- App Links는 `assetlinks.json` 이 **호스트 루트의 `/.well-known/`** 에 있어야 한다.
- 프로젝트 페이지(`<user>.github.io/<repo>`)가 아니라 **사용자/조직 루트 페이지(`<user>.github.io`)** 저장소에
  `web/applink/` 내용을 올려야 `https://<user>.github.io/.well-known/assetlinks.json` 이 뜬다.
- Jekyll이 `.well-known`(점으로 시작) 폴더를 무시하므로 루트에 빈 **`.nojekyll`** 파일이 있어야 한다(이 폴더에 포함됨).

---

## 내가 승상님께 받아야 하는 것 2가지

1. **최종 호스트 이름** (예: `planat-invite.vercel.app` 또는 `<user>.github.io`).
2. **Play 앱 서명 키의 SHA-256 지문**
   - Play Console → 해당 앱 → **테스트 및 출시 › 앱 무결성 › 앱 서명** →
     **"앱 서명 키 인증서"** 의 `SHA-256 인증서 지문` 복사.
   - ⚠️ 업로드 키가 아니라 **앱 서명 키**(Google이 재서명하는 키)여야 실제 설치본과 일치한다.
   - 로컬 키스토어로 직접 서명한다면:
     `keytool -list -v -keystore <keystore> -alias <alias>` 의 SHA256.

이 둘을 주시면 제가:
- `assetlinks.json` 의 `REPLACE_WITH_PLAY_APP_SIGNING_SHA256` 를 실제 지문으로 채우고,
- `AndroidManifest.xml` 에 아래 App Links intent-filter를 추가하고,
- 초대 문구(`GroupMetaBar` 의 `inviteText`)에 `https://<호스트>/?code=...` 링크를 다시 넣습니다.

### 추가할 manifest intent-filter (호스트 확정 후 내가 적용)
```xml
<!-- 진짜 눌리는 초대 링크: https://<호스트>/?code=<public_id> -->
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https" android:host="<호스트>" />
</intent-filter>
```
기존 `planat://group/join` intent-filter는 그대로 두어(설치돼 있으면 여전히 동작) 이중 안전망으로 쓴다.

## 검증 확인
설치 후:
```
adb shell pm get-app-links com.lss.onmyplate.nativeplanner
```
→ 해당 호스트가 `verified` 로 나오면 완료. (미검증이면 `assetlinks.json` 접근/지문/HTTPS를 다시 확인.)
