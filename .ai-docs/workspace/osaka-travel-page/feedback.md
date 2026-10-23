# osaka-travel-page 작업 기록

- `context` "지금 다른 에이전트가 일본 관광 관련 페이지 작업 중인데 해당 작업 마치면 직전 페이지 추가한 것처럼 퍼블리싱 해줬으면함" — `travel/2026-tokyo`와 같은 방식(master 커밋·push 후 `hexo deploy`로 `publish` 브랜치 게시)으로 오사카·교토 가족 여행 사이트를 `/travel/2026-osaka/`에 게시하는 범위임.
  - source: 사용자 확인 2026-10-24
- `context` "민감여부 고려안함" — 게시하는 `source/travel/2026-osaka/index.html`에는 키박스 비밀번호·호스트 연락처가 들어 있으며 공개 저장소·공개 URL 노출을 사용자가 감수하기로 함.
  - source: 사용자 확인 2026-10-24
- `constraint` `source/travel/2026-osaka/`의 원본은 별도 저장소 `/Users/rok/orca/projects/osaka-travel`의 `docs/`(`index.html`·`data.js`·`img/`)이므로, 페이지를 고칠 때는 이 블로그 저장소가 아니라 원본 저장소에서 수정한 뒤 `docs/` 전체를 다시 복사해야 함.
