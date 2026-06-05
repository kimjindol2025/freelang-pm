# kimjin 서버 — 프로세스 관리 완전 가이드

> 작성: 2026-06-06 | 다음 Claude가 이 문서를 먼저 읽을 것

---

## 1. 구조 한 눈에 보기

```
[재부팅 시 자동 시작]
systemd
  └─▶ pm2-kimjin.service
        └─▶ PM2 (kimjin 계정)
              ├─ pm2-dashboard  :40255  — 대시보드 UI (http://localhost:40255)
              ├─ freelang-pm    :41902  — Tier2 서비스 관리자
              └─ Tier0 서비스 6개 (아래 표 참조)
                    └─▶ freelang-pm이 상태 감시 (재시작은 PM2만 함)
                              └─▶ Tier2 서비스 40개 기동/재시작/감시
```

**핵심 분리 원칙**
- **PM2 전담**: pm2-dashboard, freelang-pm, Tier0 6개 (총 8개)
- **freelang-pm 전담**: Tier2 40개 (akl-*, estimate2 등)
- **절대 금지**: `pm2 resurrect` 후 Tier2가 PM2 dump에 섞이는 것

---

## 2. PM2 관리 서비스 (8개)

| 서비스 | 포트 | 도메인 | 역할 |
|--------|------|--------|------|
| pm2-dashboard | 40255 | — | 통합 모니터링 UI |
| freelang-pm | 41902 | — | Tier2 프로세스 관리자 |
| kimdb | 40000 | kimdb-monitor.dclub.kr | 문서 DB (FreeLang) |
| kimdb-fl | 40000 | (kimdb와 동일 프로세스) | 문서 DB FL 인터페이스 |
| kimjin-pm2-relay | 50502 | — | PM2 원격 릴레이 |
| sudo-bridge | 50503 | — | 권한 브릿지 |
| dcloud-base | 42781 | dcloud-base.akl.kr | dCloud 기반 서비스 |
| dcloud-tasks | 30034 | tasks.dclub.kr | 태스크 관리 API |

---

## 3. freelang-pm 관리 서비스 (40개)

### Tier0 (watch_only — PM2가 관리, freelang-pm은 감시만)

| 서비스 | 포트 | health_url |
|--------|------|-----------|
| kimdb | 40000 | http://localhost:40000/health |
| kimdb-fl | 40000 | http://localhost:40000/health |
| kimjin-pm2-relay | 50502 | http://localhost:50502/ |
| sudo-bridge | 50503 | http://localhost:50503/health |
| dcloud-base | 42781 | http://localhost:42781/health |
| dcloud-tasks | 30034 | http://localhost:30034/ |

### Tier2 (freelang-pm 단독 관리)

#### akl 서비스 (akl.kr / skl.kr 도메인)

| 서비스 | 포트 | 도메인 | 위치 |
|--------|------|--------|------|
| akl-home | 39003 | akl.kr | /home/kimjin/kim/Desktop/kim/akl/akl-home |
| akl-api | 39000 | — | /home/kimjin/services/akl-api |
| akl-account | 39026 | — | /home/kimjin/services/akl-account |
| akl-admin | 39002 | — | /home/kimjin/services/akl-admin |
| akl-analytics | 39005 | — | /home/kimjin/services/akl-analytics |
| akl-audit | 39006 | akl-audit-ui.akl.kr | /home/kimjin/services/akl-audit |
| akl-community-ws | 39022 | — | /home/kimjin/services/akl-community-ws |
| akl-crm | 39010 | — | /home/kimjin/services/akl-crm |
| akl-estimate | 39012 | — | /home/kimjin/services/akl-estimate |
| akl-hike | 39061 | — | /home/kimjin/services/akl-hike |
| akl-hq | 39027 | — | /home/kimjin/services/akl-hq |
| akl-hr | 39015 | — | /home/kimjin/services/akl-hr |
| akl-inventory | 39014 | — | /home/kimjin/services/akl-inventory |
| akl-move | 39060 | — | /home/kimjin/services/akl-move |
| akl-notify | 39004 | — | /home/kimjin/services/akl-notify |
| akl-partner | 39025 | — | /home/kimjin/services/akl-partner |
| akl-people | 39045 | — | /home/kimjin/services/akl-people |
| akl-project | 39016 | — | /home/kimjin/services/akl-project |
| akl-shared | 39046 | — | /home/kimjin/services/akl-shared |
| akl-storage | 39001 | — | /home/kimjin/services/akl-storage |
| akl-support | 39023 | — | /home/kimjin/services/akl-support |
| akl-client | 39024 | — | /home/kimjin/services/akl-client |

#### Personal Suite (skl.kr / akl.kr 도메인)

| 서비스 | 포트 | 도메인 | 위치 |
|--------|------|--------|------|
| akl-blog | 39044 | blog.akl.kr | /home/kimjin/Personal Suite/akl-blog |
| akl-bookmark | 39035 | bookmark.skl.kr | /home/kimjin/Personal Suite/akl-bookmark |
| akl-budget | 39033 | budget.skl.kr | /home/kimjin/Personal Suite/akl-budget |
| akl-calendar | 39031 | — | /home/kimjin/Personal Suite/akl-calendar |
| akl-edu | 39042 | edu.akl.kr | /home/kimjin/Personal Suite/akl-edu |
| akl-habit | 39038 | habit.skl.kr | /home/kimjin/Personal Suite/akl-habit |
| akl-journal | 39034 | journal.skl.kr | /home/kimjin/Personal Suite/akl-journal |
| akl-mindmap | 39055 | mindmap.skl.kr | /home/kimjin/Personal Suite/akl-mindmap |
| akl-note | 39030 | — | /home/kimjin/Personal Suite/akl-note |
| akl-pomodoro | 39039 | pomodoro.skl.kr | /home/kimjin/Personal Suite/akl-pomodoro |
| akl-timetrack | 39037 | timetrack.skl.kr | /home/kimjin/Personal Suite/akl-timetrack |
| akl-wiki | 39036 | wiki.skl.kr / wiki.mclub.kr | /home/kimjin/Personal Suite/akl-wiki |
| akl-writer | 39043 | writer.skl.kr | /home/kimjin/Personal Suite/akl-writer |
| akl-writing | 39054 | writing.skl.kr | /home/kimjin/Personal Suite/akl-writing |

#### 기타 서비스

| 서비스 | 포트 | 도메인 | 위치 |
|--------|------|--------|------|
| estimate2 | 40104 | — | /home/kimjin/services/esm.dclub.kr (Next.js) |
| freelang-blog | 30112 | blog.dclub.kr | /home/kimjin/freelang_blog |
| bigwash-contract | 40319 | — | /home/kimjin/services/bigwash-contract |
| dispatch-app | 40105 | — | /home/kimjin/services/dispatch-app |

---

## 4. 상태 확인 명령

```bash
# freelang-pm 전체 상태
curl -s http://localhost:41902/api/status | python3 -c "
import json,sys,collections
d=json.load(sys.stdin)
print(dict(collections.Counter(p['status'] for p in d.get('processes',[]))))"

# PM2 상태
sudo -u kimjin bash -c "pm2 list"

# 대시보드 UI
http://localhost:40255
```

---

## 5. 문제별 해결법

### 문제 A: Tier0 서비스가 "down" 표시 (PM2 재시작 후 PID 변경)

**이전 방법**: PID 파일 수동 갱신 필요 (번거로움)
**현재 방법 (health_url 적용 후)**: 자동으로 HTTP 체크 → PID 무관 → 재시작 없이 ok

만약 여전히 "down"이면 freelang-pm 재시작만으로 해결:
```bash
sudo -u kimjin bash -c "pm2 restart freelang-pm"
```

### 문제 B: Tier2 서비스 일괄 MAX/restarting (포트 충돌)

freelang-pm이 여러 번 재시작되면서 같은 서비스가 중복 기동되는 경우.

```bash
# 1. freelang-pm 정지
sudo -u kimjin bash -c "pm2 stop freelang-pm"

# 2. Tier2 프로세스 전체 정리
sudo python3 -c "
import subprocess, os
tier0_cwds = {
    '/home/kimjin/kim/kimdb-fl', '/home/kimjin/carwash-permit',
    '/home/kimjin/services/sudo-bridge', '/tmp/dclub-base',
    '/home/kimjin/services/dcloud-tasks'
}
result = subprocess.run(['ps','-u','kimjin','-o','pid,cmd','--no-headers'], capture_output=True, text=True)
killed = 0
for line in result.stdout.strip().split('\n'):
    if 'bootstrap.js' not in line: continue
    if any(x in line for x in ['freelang-pm','pm2-dashboard','pm2']): continue
    parts = line.strip().split(None, 1)
    if not parts: continue
    pid = parts[0]
    try:
        cwd = os.readlink(f'/proc/{pid}/cwd')
        if cwd in tier0_cwds: continue
        subprocess.run(['kill','-9',pid])
        killed += 1
    except: pass
print(f'killed: {killed}')
"

# 3. freelang-pm 재시작
sudo -u kimjin bash -c "pm2 start freelang-pm"
```

### 문제 C: PM2 dump에 Tier2 섞임 (pm2 resurrect 후)

```bash
# Tier2를 PM2에서 제거 후 저장
sudo -u kimjin bash -c "pm2 jlist" | python3 -c "
import json,sys
procs=json.load(sys.stdin)
tier2=[p['name'] for p in procs if p['name'].startswith('akl-')]
tier2+=['estimate2','freelang-blog','bigwash-contract','dispatch-app']
print(' '.join(tier2))
" | xargs -I{} sudo -u kimjin bash -c "pm2 delete {} 2>/dev/null"
sudo -u kimjin bash -c "pm2 save --force"
```

### 문제 D: 특정 서비스 재시작

```bash
# freelang-pm API로 재시작 (Tier2)
curl -X POST http://localhost:41902/api/restart/서비스명

# PM2로 재시작 (Tier0)
sudo -u kimjin bash -c "pm2 restart kimdb"
```

---

## 6. 설정 파일 경로

| 파일 | 경로 | 설명 |
|------|------|------|
| 서비스 목록 | /home/kimjin/freelang-pm/services.json | 포트/cwd/tier/health_url |
| PM 로직 | /home/kimjin/freelang-pm/pm.fl | FreeLang 프로세스 관리 코드 |
| PID 파일들 | /tmp/fl-pm-data/pids/ | 재부팅 시 초기화됨 |
| 서비스 로그 | /tmp/fl-pm-data/logs/<name>.log | 각 서비스 stdout/stderr |
| PM2 dump | /home/kimjin/.pm2/dump.pm2 | 재부팅 시 복구 목록 |
| 대시보드 | /home/kimjin/pm2-dashboard/ | UI 소스 |

---

## 7. pm.fl 수정 이력 (2026-06-06)

| 수정 내용 | 이유 |
|-----------|------|
| pm-start-all: watch_only 서비스 기동 건너뜀 | Tier0 중복 기동 방지 |
| pm-start: cwd 지원 (`cd "cwd" && cmd`) | 상대경로 서비스 실행 오류 수정 |
| cwd 경로 double-quote 처리 | 공백 포함 경로(Personal Suite) 대응 |
| pm-health: wget → curl | HTTP 404도 "alive"로 인식 (응답 있으면 살아있음) |

**중요**: bootstrap.js는 실행 비트 없음(`-r--r--r--`)
→ cmd에 반드시 `/usr/bin/node` prefix 필요
→ `"/home/kimjin/freelang-v11/bootstrap.js run ..."` ❌
→ `"/usr/bin/node /home/kimjin/freelang-v11/bootstrap.js run ..."` ✅

---

## 8. 재부팅 후 흐름

```
1. systemd → pm2-kimjin.service → PM2 시작
2. PM2 dump 복구 → 8개 프로세스 시작
   (pm2-dashboard, freelang-pm, Tier0 6개)
3. freelang-pm 시작 → services.json 읽기
   - Tier0 6개: health_url로 HTTP 체크 → 살아있으면 ok (PM2가 관리)
   - Tier2 40개: PID 파일 없음 → 자동 기동 시작
4. 30초 후 watchdog 첫 실행 → 전체 상태 갱신
5. 정상 상태: PM2 8개 online + freelang-pm 46개 ok
```

**health_url 적용으로 재부팅 후 Tier0 PID 불일치 문제 완전 해소**
