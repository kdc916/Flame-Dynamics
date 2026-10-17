# maxVFX Flame Dynamics v1.3.1 — Ember Motion Patch

기준: 사용자가 업로드한 `화염 시뮬레이션.html` → v1.1 → v1.2 → v1.3 손가락 화염 모드 → **v1.3.1 Ember 동작 조정**.

## 주요 기능
- `손으로만` 기본 모드, 모닥불, 불의 벽 / 그리기·바람·지우개 / 컬러 팔레트·무지개·직접 색상 / 컬러 유체 이송 / 붓·폭발·화면 저장.
- Ember 생성: 실제 GPU 고온장 면적·연료량에 따라 발생 확률 조절. v1.3.1에서 수명이 짧아진 만큼 생성률 계수 1.55→1.75로 소폭 상향.
- Ember 수명: v1.3에서는 입자별 약 1.18~3.13초, v1.3.1 기본은 약 0.32~0.65초(처음부터 마지막까지의 이론상 수명, 화면 경계로 더 빨리 소멸 가능).
- Ember 움직임: 실제 `uVelocity`와 `uCurl`를 GPU 입자 업데이트에서 함께 샘플링해 유동과 회전에 반응. 일관된 시간적 흔들림을 더해 프레임별 거친 랜덤 지터 제거. 음수 수직 유속 폭주로 역방향 낙하하지 않도록 아래 방향 제한.
- 설정 패널: `Ember 수명`(0.5~1.8×, 기본 1.0), `Ember 난류 영향`(0~3×, 기본 1.65). 후자는 기존 `일렁임` 값과 함께 반영.
- v1.3에서 제거했던 화염 합성 노이즈 문양은 되살리지 않았습니다. Ember의 시간적 난류는 입자 위치에만 사용합니다.

## 시작
- `index.html` 단일 파일을 WebGL2 호환 브라우저에서 직접 열거나 APK 설치 후 실행. 외부 네트워크 의존성 없음.
- Android APK: `com.maxvfx.flamedynamics`, versionName `1.3.1`, versionCode `131`, Android 8.0+(SDK26), 테스트용 WebView 래퍼.
- v1.3.0 APK와 동일한 개발 서명 인증서로 사인했으므로 해당 패키지에서 버전 업그레이드를 목표로 함.
- 소스 ZIP에는 보안상 개발용 개인 서명키(JKS)를 넣지 않았습니다. Android 재빌드 시 해당 키가 없으면 새 키가 만들어져 기존 설치본에 대한 업데이트가 불가능합니다.

## 코드
- `index.html`, `web/index.html`: 동일한 실행 소스 (WebGL2 + GLSL 단일 파일).
- `android/build_apk.py` + `android/apk_v2.py`: 개발용 APK 빌드/서명 (Java, Python `cryptography` 필요).
- `tests/export_shaders.cjs`: JS에 포함된 실제 GLSL 24개 추출.
- `tests/egl_compile.py`, `tests/egl_regression.py`: Mesa EGL GLES3 셰이더 컴파일·물리 및 Ember 회귀 수치검증.
- `tests/ui_smoke.py`: Chromium + 모의 WebGL2 DOM/UI 검사. 실제 GPU 전체 앱 출력과는 구분해야 함.
- `archive/v1.3.0.html`, `archive/v1.2.0.html`: 이전 HTML 소스 보관.

## 테스트 수행
```
node -e "const s=require('fs').readFileSync('index.html','utf8');new Function(s.match(/<script>([\s\S]*?)<\/script>/)[1]);console.log('JS PASS')"
node tests/export_shaders.cjs
python tests/egl_compile.py
python tests/egl_regression.py
python tests/ui_smoke.py
python android/build_apk.py
```

검증: JS 파싱 PASS, **24/24 GLSL 링크 PASS**, 실제 GLES 셀 연산 및 Ember lifetime/curl 반응 PASS, 색상·연료 영역에 따른 Ember 발생 수 PASS, 모의 WebGL UI 이벤트 PASS, APK v1/v2 서명 PASS. Android 실기기 FPS/발열/WebView GPU 출력은 **미검증**.

**범위:** 2D WebGL2 반응형 화염/입자 VFX이며 EmberGen의 3D 볼륨 연소 시뮬레이션은 아닙니다. 게임용 입자 감각 개선을 목표로 합니다.