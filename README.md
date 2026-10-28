# agent-platform 콘솔 데모

[agent-platform](https://github.com/JeongJaeSoon/agent-platform) 웹 콘솔의 읽기 전용 정적 데모다. 서버·DB·모델 없이 브라우저에서 바로 눌러 본다.

**열기: <https://jeongjaesoon.github.io/agent-platform-demo/>**

- 데모 owner(`demo-owner@example.test`)로 로그인된 상태로 열린다.
- 화면은 예시 데이터로 채운 스택에서 녹화한 GET 응답(`fixtures.json`)으로 그린다. 계정·팀·에이전트는 모두 `example.test` 예시다.
- 읽기 전용이다. 저장·실행·다운로드 요청은 어디로도 보내지 않고 "데모에서는 저장되지 않습니다"라고 알린다.
- 녹화하지 않은 화면 조합은 "찾을 수 없음"으로 보일 수 있다.

## 이 저장소에 있는 것

| 파일 | 내용 |
| --- | --- |
| `fixtures.json` | 녹화한 GET 응답. 데모 빌드가 번들에 넣는다 |
| `.gitleaksignore` | gitleaks가 비밀값으로 잘못 읽는 줄의 fingerprint. 아래 설명 참고 |
| `.github/workflows/pages.yml` | 검사·빌드·GitHub Pages 배포 |

콘솔 코드는 여기 없다. 워크플로가 원본 저장소를 `SOURCE_REF` 커밋으로 받아 공개 전 검사(`apps/web/scripts/demo-scan.ts`)를 돌리고, `vite build --mode demo`로 이 `fixtures.json`을 넣어 빌드한 뒤 Pages에 배포한다. main push마다 다시 배포한다.

`SOURCE_REF`는 녹화한 스택과 같은 화면 코드여야 한다. 지금 녹화는 원본 main `77b91d84`로 띄운 스택에서 했고, `SOURCE_REF`는 그 위에 녹화기와 화면 수정(JeongJaeSoon/agent-platform#896)을 머지한 main 커밋이다. #896은 웹 화면과 녹화 스크립트만 바꿔서 API 응답은 녹화한 스택과 같다.

### `.gitleaksignore`

gitleaks의 `generic-api-key` 규칙은 사용량 breakdown 응답의 `"key": "<에이전트 id>"` 줄을 비밀값으로 읽는다. 그 줄만 fingerprint로 넣었다. `demo-scan`은 각 항목이 breakdown 응답 안의 줄이고, 값이 UUID이며, 녹화된 에이전트 목록의 id인지 확인한다. 하나라도 어긋나면 실패한다.

## 다시 녹화하기

오너 Mac에서, 예시 데이터로 채운 스택(4620)을 상대로 한다. 녹화기는 로그인 한 번 말고는 GET만 스택에 넘기고 끝나면 로그아웃한다. `<source>`는 원본 저장소 checkout, `<demo>`는 이 저장소 checkout의 절대 경로다. `<리포트.json>`도 절대 경로로, 이 저장소 밖에 둔다.

```bash
cd <source>
bun install --frozen-lockfile
bun run --cwd apps/web build                       # 녹화에 쓰는 일반 빌드
bun apps/web/scripts/demo-record.ts --api http://127.0.0.1:<API 포트> \
  --accounts <계정 표 파일> --out <demo>/fixtures.json
(cd <demo> && gitleaks dir fixtures.json --redact=100 -f json -r <리포트.json>)
bun apps/web/scripts/demo-scan.ts --fixtures <demo>/fixtures.json --root <demo> --write-ignores <리포트.json>
bun apps/web/scripts/demo-scan.ts --fixtures <demo>/fixtures.json --root <demo>
DEMO_FIXTURES=<demo>/fixtures.json bun run --cwd apps/web build:demo   # 로컬 확인: apps/web/dist-demo
```

마지막 `demo-scan`이 `clean:`을 찍으면 이 저장소를 갱신한다.

1. `pages.yml`의 `SOURCE_REF`를 녹화에 쓴 `<source>` 커밋으로 바꾼다(원본 저장소에 push된 커밋이어야 한다).
2. `fixtures.json`·`.gitleaksignore`·`pages.yml`을 커밋해 main에 push한다.
3. Actions의 `pages` 실행이 같은 검사와 빌드를 다시 돌리고 배포한다. 끝나면 공개 주소에서 확인한다.

`--write-ignores`가 거부하면 그 발견은 예외로 넘길 것이 아니라 들여다볼 대상이다. 녹화를 고쳐 다시 한다.

## 라이선스

[PolyForm Small Business License 1.0.0](LICENSE). 원본 저장소와 같은 source-available 라이선스이며 오픈소스 라이선스가 아니다.
