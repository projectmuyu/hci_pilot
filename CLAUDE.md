# 검색 조건 사진 고르기 도구 · 인수인계

`search_system.html`은 실험의 검색 조건에서 참여자가 방 사진을 검색하고, 마음에 드는 사진을 모아, 최종 1장을 고르는 단일 HTML 도구다. 이 문서는 Claude Code가 이 파일을 이어서 작업하기 위한 정리다. 사용자와의 대화와 화면 문구는 모두 **한국어**로 한다.

## 파일

- `search_system.html` 하나. 빌드 없음.
- `pool/pool.csv` + `pool/photos/*.jpg`: 기본 사진 풀(200장). http(s)로 열면(GitHub Pages 등) 페이지가 시작할 때 `syncDefaultPool()`로 이 브라우저의 IndexedDB에 맞춰 넣는다. 파일로 열면(file://) 브라우저가 막아서 건너뛰고, 그때는 `풀 불러오기`(ZIP)로 넣는다. 로컬 확인은 `python -m http.server`로 띄워서 한다.
- 사진 파일 이름은 `R001`~`R200`이고, 번호는 작업 폴더 `index.json`의 번호와 같다. 예전 이름과 출처는 `pool/rename_map.csv`(`file, old_pool_file, original_file, source`)에 있다. 사진 id가 파일 이름에서 나오므로 이름을 바꾸면 그 전 기록은 사진과 연결이 끊긴다.
- 맨 위에 반드시 `<!doctype html>`, `<html lang="ko">`, `<meta charset="utf-8">`가 있어야 한다(없으면 로컬에서 열 때 한글이 깨진다).
- 외부 의존성은 JSZip(cdnjs, ZIP 내보내기·불러오기)과 Google Fonts(IBM Plex Sans KR / Mono)뿐이다. 둘 다 없어도 핵심 기능은 동작한다.

## 화면 구성

여는 주소에 따라 세 가지로 쓴다. 같은 PC·같은 브라우저면 두 창이 localStorage·IndexedDB를 함께 쓰고 `storage` 이벤트로 서로의 변경을 바로 받는다(기록, 진행 상태, 사진 풀). 기기가 다르면 공유되지 않는다(서버 없음).

- `index.html`: 두 화면으로 가는 링크만 있는 첫 페이지.
- `search_system.html?view=search` (검색 화면 전용, `SEARCH_PAGE`): 시작 화면(`renderStartUI`)에서 진행자·참여자 ID를 넣고 시작 → 검색 → 완료 화면의 "연구자에게 알려 주세요."를 누르면 다시 시작 화면. 연구자 패널(점 버튼, Ctrl+Shift+R)은 없다.
- `search_system.html?view=researcher` (연구자 화면 전용, `RESEARCH_PAGE`): 사진 풀 관리·참여자 기록만. `검색 화면 열기 ↗` 링크, 진행 중인 참여자 표시와 `미완료로 끝내기`(두 번 눌러야 함). 이 창은 진행 상태 `P`를 쓰지 않는다(검색 창의 최신 상태를 덮어쓰지 않게).
- 주소에 `view`가 없으면 아래처럼 한 창에서 둘 다 쓰는 예전 방식.

1. **연구자 화면** (기본)
   - 오른쪽 위 `새 참여자 시작` → 참여자 ID 입력 모달(비우면 진행 안 됨) → 검색 화면. 진행 중인 참여자가 있으면 `P03 이어서 진행` 버튼도 나온다.
   - 탭 `사진 풀 관리`: `기본 풀 다시 맞추기`(http(s)일 때만 보임, 직접 고친 태그를 덮어씀), 사진 추가(드래그/파일/폴더, `pool.csv`를 함께 넣으면 태그 자동 적용), 태그 편집(Ctrl/⌘·Shift 다중 선택), 선택·전체 삭제(두 번 눌러야 삭제, Delete 키 지원), 풀 내보내기·불러오기(ZIP: `pool.csv` + `photos/*.jpg`). `pool.csv` 열은 `file, tags, contains`이고 tags는 `;`로 구분한다.
   - 탭 `참여자 기록`: 참여자별 카드(ID 수정 가능, 완료/미완료, 검색어 목록, 담은 사진과 각각을 담은 검색, 최종 사진, 다른 PC에서 불러온 기록은 "불러온 기록" 표시), 진행자가 둘 이상이면 진행자별 보기(`S.archWho`), `기록 불러오기 (ZIP)`(`importArchive`: ZIP 안 `participants.json`을 읽어 `id`+`researcher`가 같은 기록은 건너뛰고 `imported: true`로 합침), `기록 + 최종 사진 (ZIP)`(화면에 보이는 기록만: `participants.csv` + `participants.json` + `진행자_순번_ID/` 사진 폴더). CSV 단독 내보내기 버튼은 없앴다(ZIP 안에 있음).
2. **참여자 검색 화면** (구글 이미지 검색처럼 검색창만 있음)
   - 결과 사진의 하트 = "마음에 든 사진"에 담기/빼기. 사진을 누르면 크게 보기 모달이 열리고(포인터는 돋보기), 모달에서도 담을 수 있다.
   - `마음에 든 사진 N` 버튼 → 모아 보기 → 왼쪽 위 동그라미로 한 장 선택(한 장만, 다시 누르면 해제) → `이 사진으로 결정` → 확인 모달 → 완료 화면. 모아 보기에서도 사진을 누르면 크게 보기이고, 모달에는 하트와 `이 사진 고르기`가 함께 있으며 눌러도 닫히지 않는다. 모달에서 끈 하트는 모달을 닫을 때 목록에 반영한다.
   - 결과 사진에 마우스를 올리면 왼쪽 위에 `✕`가 나타난다(크게 보기 모달에는 `이 사진 안 보기`). 누르면 그 자리에 흐리게 남고 다음 검색부터 나오지 않는다. 담은 사진은 제외할 수 없고 제외한 사진은 담을 수 없다. 제외한 사진이 있으면 상단 `제외한 사진 N`에서 목록을 보고 되돌린다. 세션 동안 유지되고 새 참여자는 초기화된다.
   - 검색한 말이 둘 이상이면 검색창 아래에 칩 줄이 나온다(친 말 그대로 한 칩, 같은 말은 하나, 최근 순, 맞는 사진이 없던 말도 포함). 칩을 켜고 끄면 결과가 바뀌고 검색창에 켜진 칩이 이어 보인다. 2개 이상 켜지면 모든 칩의 모든 단어에 맞는 사진만 보여 주고 채우지 않는다(0장이면 안내). 1개면 직접 친 검색과 같다. 새 검색어를 치면 그 말만 켜진다. 켜진 칩(`S.active`)은 저장하지 않아 새로고침하면 처음 화면으로 돌아간다. `queries`에는 직접 친 검색만 쌓인다.
   - 완료 화면의 "연구자에게 알려 주세요." 문구를 누르면 연구자 화면으로 돌아간다. 다른 화면에서는 Ctrl+Shift+R 또는 오른쪽 아래 작은 점(연구자 패널)으로 나간다.
   - 참여자 화면에는 다운로드 버튼이 없고 담기 개수 제한도 없다(둘 다 의도된 것).

## 상태와 저장소

| 저장소 | 키 | 내용 |
| --- | --- | --- |
| IndexedDB | DB `room-search-pool`, store `photos` | 사진 레코드 `{id, name, tags[], contains[], blob, thumb, addedAt, fromPool?}`. blob은 긴 변 1600px JPEG, thumb은 420px. 기본 풀 사진은 `fromPool: true`, id는 `pool_` + 파일 이름 해시라 다시 맞춰도 기록의 사진 참조가 유지된다 |
| localStorage | `rs_pool_ver` | 마지막으로 맞춘 `pool/pool.csv`의 해시. 바뀌면 다음 시작 때 태그를 갱신하고 새 사진을 받고, 빠진 기본 풀 사진은 지운다(연구자가 직접 넣은 사진은 그대로) |
| localStorage | `rs_pick` | 진행 중 참여자 `P`: `{pid, liked[], hidden[], hideLog[], filterLog[], final, view, active, _sel, startedAt, finishedAt, queries[], likeLog[], archivedId}` |
| localStorage | `rs_archive` | 참여자 기록 배열 `ARCH`. 고치기 전에 `reloadArch()`로 다른 창의 최신 기록부터 읽는다 |
| localStorage | `rs_researcher` | 이 PC의 진행자 이름. 새 기록의 `researcher`, CSV 열, 내보내기 파일 이름에 붙는다 |
| localStorage | `rs_pool_ping` | IndexedDB를 고칠 때마다 바꾸는 신호. 다른 창이 받아 사진 풀을 다시 읽는다 |
| localStorage | `rs_settings`, `rs_sessions` | 예전 버전의 흔적. 지금 화면에서는 쓰지 않는다 |

`contains`는 사진에 보이는 타깃 요소(식물·러그·스탠드 조명·벽 그림·겹친 침구·담요)를 **보이면 다** 적은 표시로, 나중에 요소 포함 비율을 맞추는 데 쓴다. 타깃 요소는 태그(`식물·러그·액자·스탠드·담요`)로도 달려 검색되고, room-pool 스킬의 `build_pool.py`가 이 태그에서 `contains`를 자동으로 채운다. 검색에서 빼지는 않는다(`isExcluded = () => false`). 2026-10-27에 다섯째 요소를 `따뜻한 조명`에서 `겹친 침구·담요`로 바꿨다: `따뜻한조명` 태그는 일반 조명 태그로 남아 있고, `담요`는 침대 이불 위에 담요·스로우·베드러너(호텔식 보조 이불)를 한 겹 더 얹거나 접어 둔 사진에만 단다(소파에 걸친 담요, 쿠션만 있는 것은 제외). `TARGETS`의 `warm` 항목은 예전 contains가 남은 사진 때문에 지우지 않았다.

참여자 기록 한 건 (`ARCH[i]`):

```json
{
  "id": "a…", "no": 1, "label": "P01", "researcher": "의현", "status": "완료 | 미완료",
  "startedAt": "ISO (첫 검색 또는 첫 담기)", "finishedAt": "ISO",
  "queries": [{"q": "아늑한 침실", "at": "ISO"}],
  "liked": [{"id": "…", "name": "J11_의현.jpg", "qn": 1, "q": "아늑한 침실", "likedAt": "ISO"}],
  "final": {"id": "…", "name": "J01_연수.jpg"},
  "likeLog": [{"action": "담기 | 빼기", "id": "…", "name": "…", "at": "ISO", "qn": 2, "q": "야경", "where": "검색 결과 | 크게 보기 | 모아 보기", "active": ["그때 켜진 칩"]}],
  "hidden": [{"id": "…", "name": "R010.jpg", "qn": 2, "q": "야경", "hiddenAt": "ISO"}],
  "hideLog": [{"action": "제외 | 되돌리기", "id": "…", "name": "…", "at": "ISO", "qn": 2, "q": "야경", "where": "검색 결과 | 크게 보기 | 제외 목록", "active": ["그때 켜진 칩"]}],
  "filterLog": [{"action": "켜기 | 끄기", "q": "우드", "active": ["화이트", "우드"], "results": 31, "at": "ISO"}]
}
```

- 최종 결정 순간 `archiveCurrent('완료')`로 자동 저장된다.
- 최종 전에 새 참여자를 시작하면 `resetSession()`이 담았거나 제외한 사진이 있을 때 `archiveCurrent('미완료')`를 부른다.
- `queries`는 검색 버튼/Enter를 누를 때마다 쌓이고, 최종 결정 뒤에는 쌓이지 않는다.
- `liked[].qn/q`는 그 사진을 **마지막으로 담은** 검색이다.

CSV 열: `no, researcher, participant_id, status, started_at, finished_at, query_count, queries, liked_count, liked_with_query, unliked_count, final_query, liked_files, final_file, final_in_liked_order, hidden_count, hidden_with_query, restored_count, filter_count`. 예전 기록에는 `hidden`·`hideLog`·`filterLog`가 없다. 한 칸 안의 여러 값은 ` | `로 잇고, `liked_with_query`는 `사진@검색번호:검색어` 형식이다.

## 검색 동작

- `runSearch(q, seed)`: 검색어를 단어로 나누고(조사 떼기, `-단어`·"○○ 없는/빼고"는 제외어), 사진 태그와 동의어 사전 `SYN`에 맞는 단어 수로 정렬한 뒤 점수순으로 놓는다. 동점은 시드 해시로 섞는다. 한 단어의 원형과 어간은 한 번만 센다. `FILLER_WORDS`(앤·옆·큰·가구·느낌 등)는 읽지 않고, 풀 어디에도 없는 단어는 교집합 조건에서 빼고 `missing`으로 돌려준다. 결과는 모든 단어 일치(`full`) → 일부 일치(`hits`까지) → 채운 사진(일치 12장 미만이면 24장까지) 순이고, `drawResults`가 넘어가는 자리에 제목을, 일치 0장이나 `missing`이 있을 때 안내 문구를 단다. 태그가 검색어를 품으면 1점, 검색어가 태그를 품기만 하면 0.5점(턴테이블 → 테이블)이다. 한 글자 검색어(책·꽃·옷 등)는 태그·동의어와 똑같을 때만 걸린다. `ROOM_WORDS`(방·침실·원룸·자취방·침대·벽·천장·공간·인테리어·사진 등)는 점수에서 빼고, `N평`은 10평 이하면 `좁은`으로 읽고, 끝의 `톤·색·처럼·랑·만·하게` 등은 떼어 읽는다(베이지톤 → 베이지, 소파랑 → 소파).
- 태그 어휘는 `TAG_GROUPS`(분위기, 스타일, 색, 빛·시간대, 소재·질감, 패턴, 가구, 침대, 조명, 소품·장식, 생활감, 공간·전망, 반려동물 — 97개)에 있고, `.claude/skills/room-pool/vocab.json`과 같게 유지한다. 방 종류(침실·원룸 등)는 일부러 태그로 달지 않는다: 그런 검색어로 결과가 걸러지지 않게 하려는 것이다.

## 주요 함수

- 연구자: `render`, `renderPool`, `drawGrid`, `drawEditor`, `addFiles`, `exportPool`, `importPool`, `confirmDelete`, `askParticipantId`, `renderArchive`, `archiveCSV`, `exportArchiveZip`
- 참여자: `renderSearchUI`, `drawResults`, `toggleLike`(likeLog 기록), `renderSavedUI`, `confirmFinal`, `renderDoneUI`, `openPreview`
- 공통: `archiveCurrent`, `resetSession`, `savePick`, `saveArch`, `urlFor`(blob URL 캐시)

## 작업 방식

- 바꾸기 전에 사용자와 구현 방향을 짧게 합의한다. 사용자는 단계별로 요구를 다듬어 가는 편이다.
- 고친 뒤에는 Playwright(헤드리스 Chromium)로 실제 흐름을 돌려 확인한다: 사진 넣기 → 새 참여자 시작(ID) → 검색 여러 번 → 담기/빼기 → 새로고침 → 모아 보기 → 최종 결정 → 연구자 화면 → 참여자 기록·CSV. `node --check`로 스크립트 문법도 확인한다.
- 화면 문구는 짧고 평이한 한국어로 쓰고, 참여자 화면에는 연구 의도가 드러나는 말을 쓰지 않는다.
- 삭제처럼 되돌릴 수 없는 동작은 "한 번 더 누르기" 확인을 둔다(`confirm()`은 쓰지 않는다).

## 남은 일

- [ ] **사진별 CSV** (제안만 함, 사용자 답 대기): 담은 사진 한 장 = 한 줄. 열 `participant_id, photo, query_no, query, liked_at, where, is_final`. 참여자 기록 탭에 내보내기 버튼으로 추가.
- 기록은 브라우저 localStorage에만 있으므로 세션마다 기록 ZIP으로 백업하도록 안내되어 있다. 연구자 3명이 각자 PC에서 진행하고(참여자 P01~P12를 미리 나눠 배정), 나중에 의현이 각자의 ZIP을 `기록 불러오기`로 한 화면에 합친다. 새 기록 번호(`no`)는 같은 진행자의 자기 기록만 보고 매긴다(불러온 기록 제외).
