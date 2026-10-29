# aws-ubuntu

- 질문을 받으면 `INDEX.md`부터 읽고, 맞는 문서만 연다.
- 문서를 추가·수정·이동하면 같은 작업 안에서 프론트매터와 `INDEX.md`를 갱신한다 (`uv run ~/.claude/skills/docs-tidy/scripts/docs_index.py build --root .`).
- 서버 작업은 `ssh aws-ubuntu-1`로 한다. 서버 상태를 바꾸는 명령(재시작, 삭제, daemon 설정)은 실행 전에 확인받는다.

## 학습 기록 (docs/learn/)

- 서버·도메인·인프라 작업을 하면 `docs/learn/`에 다음 번호(`NN-주제.md`, id `learn-NN-주제`)로 기록하거나, 해당 [예정] 문서를 실제 기록으로 채운다.
- 문서 구성: 용어(한 줄 정의 + 비유) → 실제로 한 일(명령) → 확인 방법 → 겪은 문제.
- 하지 않은 일은 [예정]으로 표시한다. 완료처럼 쓰지 않는다.
- README의 학습 순서 표와 상태를 함께 갱신한다.

## 민감정보 (public 레포)

- 서버 IP, SSH 키 파일 이름, 토큰·비밀번호, `.env` 값, 이메일을 쓰지 않는다. `<SERVER_IP>`, `<KEY>`, `<비밀번호>`로 표시한다.
- 커밋 전에 확인: `grep -rnE '([0-9]{1,3}\.){3}[0-9]{1,3}|\.pem|TOKEN=|PASSWORD=' --include='*.md' .`
