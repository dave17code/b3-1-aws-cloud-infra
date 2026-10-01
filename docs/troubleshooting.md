# 통신 장애 트러블슈팅 보고서

- 실험: 실습 EC2의 HTTP 인바운드 차단·복구
- 대상: 서울 리전 / b3-1-web / b3-1-web-sg
- 시각·시간대·요청 구분자: 각 증빙 출력에 기록

## 1. 증상

HTTP 80 인바운드 규칙 제거 후 외부 curl 요청이 연결 시간 초과로 실패했다.

[HTTP 규칙 제거 화면](evidence/05-sg-blocked.png)
[외부 실패 출력](evidence/06-external-blocked.txt)

## 2. 가설

HTTP 80 인바운드 허용 규칙 제거로 새 외부 연결이 차단되었을 가능성이 있다.
Nginx 장애와 구분하기 위해 내부·외부 응답을 비교한다.

## 3. 검증

SSH 접속은 정상이며, Nginx는 active 상태이고 localhost 요청은 HTTP 200 OK와 Hello Cloud를 반환했다.

[차단 중 서비스 상태·내부 응답·로그](evidence/07-internal-during-block.txt)

접근 로그에 요청이 없다는 사실만으로 원인을 확정하지 않고,
SG 변경 내용과 복구 후 응답까지 함께 비교했다.

## 4. 조치

HTTP / TCP 80 / 0.0.0.0/0 규칙을 복구하고,
SSH TCP 22의 접속지 IPv4 /32 제한을 유지했다.

[복구한 SG 규칙](evidence/08-sg-restored.png)

## 5. 결과

HTTP 규칙 복구 후 외부 curl에서 HTTP 200 OK를 확인했고, 브라우저에 Hello Cloud가 표시되었으며, 접근 로그에서 같은 CASE_ID의 요청과 200 응답을 확인했다.

최종 원인 판단: 차단 중에도 서버 내부 응답은 정상이었고 HTTP 인바운드 규칙 복구 후 외부 접속이 회복되어, 원인을 보안 그룹의 HTTP 80 허용 규칙 제거로 판단했다.

[외부 복구 출력](evidence/09-external-restored.txt)
[해당 요청의 접근 로그](evidence/10-access-log.txt)

## 6. 재발 방지

- SG 변경 전후에 HTTP·SSH 허용 규칙을 확인한다.
- SSH는 현재 접속지 공인 IPv4 /32로 제한한다.
- localhost와 외부 HTTP를 각각 검사한다.
- 새 연결로 재검증하고 변경 시각과 결과를 남긴다.
