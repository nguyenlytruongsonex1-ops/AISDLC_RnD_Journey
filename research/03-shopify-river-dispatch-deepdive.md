# Shopify AI-Native SDLC: River / Aquifer / Dispatch — Deep Dive

**Ngày brief:** 2026-09-12  
**Nguồn chính (primary, Shopify Engineering):**
1. [Under the River](https://shopify.engineering/under-the-river) — Javier Moreno / River — 28 May 2026  
2. [Building an agentic harness that outlasts the model](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model) — Zack Deveau — 29 Jul 2026  
3. [How River takes security work from a fix to merge](https://shopify.engineering/river-vulnerability-remediation) — Kaiyi Li / Erin Son — 2 Sep 2026  

**Quy ước:** Mọi số liệu và chi tiết kiến trúc dưới đây đều trích từ các URL trên (hoặc được đánh dấu rõ là secondary). Không suy diễn metric/kiến trúc ngoài nguồn.

---

## 1. Bối cảnh: tại sao Shopify xây River / Aquifer / Dispatch

### Pain trước River (developer productivity)

Theo [Under the River](https://shopify.engineering/under-the-river), trước Spring 2024 Shopify ở trạng thái:

- Nhiều repo, môi trường dev “bespoke”, feedback chậm — friction đã quen sống với.
- Agent local trên laptop/editor của từng engineer có **ceiling**: mỗi session là artifact riêng; cách điều tra flaky test hôm qua “chết” cùng session; agent không học lẫn nhau.
- Quote Tobi (được trích trong bài): *"If every interaction with an agent happens in a private window, the only person who learns anything is the person at the keyboard."* — Tobi Lütke, *Learning on the Shop floor*.

Bet sớm 2024 (hai quyết định “không phải crowd-pleaser”):

1. Trở thành **monorepo company** — một repo cho mọi thứ Shopify ship, gọi là **World**.
2. Build mọi thứ bằng **Nix** — dev, CI, production images trên một reproducible substrate.

Câu họ nói sớm 2024: *"Code is going to be increasingly written with AI, and our infrastructure needs to be the substrate for that."*

Migration đau: CI scale order-of-magnitude, merge queues load-bearing, build cache thành product, test infra rebuild vì mọi change có thể recompile phần lớn graph. Payoff: agent điều hướng cross-zone từ root, skills dạng file, “ask the repo a question”, trace từ log production về commit trong một workspace, môi trường Nix giống nhau ở dev/CI/prod.

Insight then: **agent-friendly ≈ human-friendly** — monorepo, reproducible env, skills viết xuống, CI signal sạch/nhanh vốn đã trên wishlist của humans; agents chỉ làm debt đó **visible**.

Sau khi River ship, các team muốn “River riêng” (PR review, research, migration, compliance, performance…). Họ đã có River nhưng **chưa có platform cho một trăm Rivers** → cần **Aquifer**.

### Pain AppSec → Dispatch

Theo [Dispatch post](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model):

- AppSec phải bảo vệ merchant/buyer data; models ngày càng mạnh hơn nhưng **noise** cũng tăng.
- Một audit: model tìm >30 candidate vulnerabilities; sau validation **mọi** candidate bị downgrade low/medium, false positive, hoặc defense-in-depth.
- Experiment sớm với “security generalist” multi-agent: có true positives nhưng volume “potential/theoretical” cao, fix thường sai/theoretical, non-deterministic; context window đầy vì chase unordered list rồi give up / compaction.
- Exploitability web khó confirm (không có oracle kiểu compiler flag như memory safety) → cần **test oracle** gắn vào integration/unit/functional suite sẵn có.

### Pain remediation (sau khi đã có patch)

Theo [River vulnerability remediation](https://shopify.engineering/river-vulnerability-remediation):

- Hệ thống tạo dependency upgrade / draft fix **không** đủ để đóng vulnerability: repo đổi trong lúc PR chờ review; CI green trên PR ≠ đã xóa vuln khỏi repository head.
- Engineers phải shuttle state giữa code, CI, Slack, tracker — River được dùng để vận hành **remediation loop** quanh diff, không chỉ detection.

---

## 2. Kiến trúc

### 2.1 River — surface agent

| Thuộc tính | Chi tiết (nguồn) |
|---|---|
| Là gì | AI agent sống trong **company Slack** |
| Surface | `@river` trong **public channel only** — **không DM**; mọi conversation thành public Slack transcript |
| Capabilities | Đọc code, chạy tests, mở PRs, query **data warehouse**, xem **production traces**, đôi khi push back plan xấu |
| Median session | 19 phút; median **50 tool calls**/session |
| Vai trò trong hệ | Profile **Interactive** trên Aquifer; “animating spark” làm World thành living system |

**Multiplayer by construction:** một người @-mention → River làm việc (đọc file, query, post partial findings) → người thứ hai join với constraint → River absorb context → thread searchable; pattern được mine lại vào skills/prompts/defaults → agent thông minh hơn **không cần retrain model**.

World chứa code còn chứa: skills, conventions, intent documents, runbooks, **`AGENTS.md`**, zone knowledge — “intelligence layer” compounding. Sau session tốt: pebble / skill update / `AGENTS.md` diff / shared skill mới.

Peers trên cùng substrate (không phải “con của River”): **PR review** profile; **Vanilla** (headless “pi” agent); future agents.

Nguồn: [Under the River](https://shopify.engineering/under-the-river).

### 2.2 Aquifer — harness / platform

**Aquifer** = internal platform chạy AI agents. Substrate gồm: **session**, **harness**, **sandbox**, **gateway**, durable event log, **credentials proxy**, observability pipeline. Design constraint: *cells die, sandboxes die, machines die — the conversation doesn’t.*

#### Lớp chính

| Layer | Vai trò |
|---|---|
| **Session** | Durable identity; append-only event log; **Postgres-backed**; canonical truth |
| **Harness** | Agent loop: đọc history, gọi model, emit tool intents; cheap to recreate, disposable |
| **Sandbox** | Nơi code chạy: filesystem, shell, repo (bash, edit, build, test); disposable; đôi khi warm |
| **Gateway / credentials proxy / observability** | Được liệt kê là phần substrate; gateway dùng chung cho 3 modes |

Harness **sống ngoài** sandbox → ba property:

1. **Safety** — agent loop không cùng blast radius với `rm -rf`
2. **Replaceability** — swap model/runtime/language harness mà không đụng sandbox
3. **Observability** — decision stream nằm phía harness, một chỗ

**Session cells:** khi session live → ephemeral process trên host (Go runtime + harness cùng process group). Idle → exit. Interaction tiếp → cell mới (có thể host khác); session identity + conversation vẫn còn vì work sống trong Postgres. “Cattle, not pets” ở session level.

**Profile = data:** system prompt + skills + extensions + sandbox policy + model defaults; build bằng **Nix**, ship as **bundles**. Agent product mới = bundle mới, không platform mới.

#### Ba modes / consumers (cùng session model, sandbox plane, gateway)

| Mode | Ví dụ | Đặc điểm |
|---|---|---|
| **Interactive** | River | Durable session, live human, long-lived |
| **Automation** | PR review | Durable session, wake bởi external system, thường không có human in loop |
| **Job** | CI, batch | Ephemeral: provision → run → stream → destroy; reuse sandbox plane; session log optional |

Họ khẳng định: mỗi lần thử architecture khác đều tái phát minh một trong ba modes này.

Nguồn: [Under the River](https://shopify.engineering/under-the-river). Tham chiếu Anthropic *Scaling Managed Agents* (April 2026): *"Decouple the brain from the hands."*

### 2.3 World monorepo + Nix — “agent-friendly ≈ human-friendly”

- **World**: một repo cho mọi thứ Shopify ships.
- **Nix**: một substrate reproducible cho dev environments, CI, production images.
- Agent ở **root** repo, navigable cross-zone.
- Skills = written-down knowledge as files, load on demand vào session.
- Cùng pain point cho human & agent: env không reproducible → agent cũng không; repo fragmented → agent không thấy cross-cutting; không ai viết cách làm → không human onboarding hay agent first session học được.

Nguồn: [Under the River](https://shopify.engineering/under-the-river).  
*(Secondary/video: talk “Tectonix: The bedrock of Shopify's Monorepo” trên YouTube mô tả build system Nix cho World — **không** phải eng-blog primary; chỉ related reading.)*

### 2.4 Dispatch — AppSec agentic harness

**Dispatch** = thin **Ruby** client / harness orchestrator nội bộ; abstract complexity của agentic scanning at scale; cho phép tạo category-specific bug hunting agents → chạy an toàn → **test oracle** → mở PRs.

#### Pipeline (first full scan + sau đó diff scan)

1. **Code partitions** (size/scope) + artifacts: API/data model docs tái dùng.
2. Full scan: **Hunting agents** song song theo partition.
3. Data/artifacts lưu custom Rails backend; follow-up = **diff-based** (so sánh commit mới vs cũ) → giảm token spend.
4. Generalist + vulnerability-specific hunters, encode software security + Shopify context.

#### Stages (ordered)

| Stage | Việc làm | Vì sao |
|---|---|---|
| Test bootstrap | Tìm đúng test commands, verify suite chạy được | Verifier/fix author không tốn turn khám phá cách chạy test |
| Architecture documentation | Mô tả data models, APIs, auth patterns, boundaries | Shared artifacts cho downstream + reuse |
| File cataloging | Flat list file liên quan | Focus file quan trọng |
| Partitioning | Group thành partitions → JSON artifact | Related code cùng nhóm; bounded tokens |
| Hunting | 1 Hunter / partition, parallel; skills cross-repo search | Tốc độ; out-of-repo context |
| Verification | Verifiers **sequential**; **khác model** hunter; author + execute tests | Tests = oracle; tránh port/DB/fixture collision |
| Post-processing | Dedup, severity scoring (Ruby runner) | Actionable, consistent |
| Reporting | Join findings + test results | Human-readable + rationale exploitability |
| Remediation | Fix author / finding; branch + PR body | Draft PRs với context, tests, proposed fixes |

#### Partition heuristics (để cost/recall)

- Token count partition ~ **20–30%** context window (chừa chỗ explore + tools).
- Group theo domain/purpose/feature.
- Mỗi partition kèm global deps / shared context.

Benchmark: reintroduce confirmed vulns locally; partitioned vs non-partitioned → partitioned cải thiện accuracy & recall, cost hợp lý.

#### Test oracle (ví dụ IDOR Verifier guidelines)

- Work backwards từ public call sites; search across web clients.
- Fixtures **hai tenants** khác nhau.
- Exercise public stack nhiều nhất có thể — **không** chỉ unit test ở Model layer.
- Phải extract impactful cross-tenant data; booleans/raw IDs → severity Low.
- Đánh giá trên/dưới layer hiện tại (upstream/downstream controls); không stub controls quan trọng.
- Không prove được trong constraints → reject hoặc Low/Medium.

#### Determinism

- Hand agents deterministic scripts / skills gọi scripts khi cần structured I/O (vd. enforce JSON structure).
- Credentials, Git, storage thuộc **deterministic code**, không để agents tự sở hữu.

Nguồn: [Dispatch post](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model).

### 2.5 River remediation loop (AppSec → merge)

Bổ sung kiến trúc vận hành (không thay Dispatch hunt):

1. **Validate live state** — ledger = claim, không phải truth; reconcile vs GitHub PR / tracker / repo.
2. Confirm vuln **vẫn còn** trước khi edit (dependency resolve version; app finding vs repo head; tìm PR khác đang fix).
3. **Repair & iterate on current head** — rebase/replay upgrade, regenerate lockfile bằng tooling của repo; graft vào lockfile hiện tại (tránh revert upgrade lân cận).
4. **Handoff decision, not investigation** — owners quyết product/risk/merge; River mang evidence + câu hỏi nhỏ nhất.
5. **Verify closure** — sau merge: default branch không còn vuln; tracker + ledger agree. “Done” ≠ CI green trên PR còn open.

Ví dụ được kể (một batch): 8 dependency PRs — rebase 7, bỏ 1 vì human đã takeover; 6/7 green; 1 đỏ cần decision. Một finding khác: River tự recommend **drop patch** sau khi developer hỏi về intent lịch sử của feature.

Nguồn: [River vulnerability remediation](https://shopify.engineering/river-vulnerability-remediation).

---

## 3. Map vào SDLC stages

| SDLC stage | River / Aquifer | Dispatch | Ghi chú |
|---|---|---|---|
| **Plan** | Public Slack thread: hỏi plan, River push back plan xấu; multiplayer redirect | Architecture documentation stage tạo shared context APIs/models/auth | Corpus + skills/`AGENTS.md` là “institutional memory” |
| **Code** | Đọc/edit trong sandbox; mở PR; World root + Nix env | Remediation: fix author agents; draft PRs Shopify-tuned | Profile Interactive vs Job |
| **Review** | PR review = Automation profile trên Aquifer | Verification adversarial (model khác hunter) | Humans retain merge authority (remediation post) |
| **Test** | River chạy tests trong session | Test bootstrap + Verifier author/embed tests vào suite có sẵn; tests = oracle | Diff scan giữ coverage rẻ hơn |
| **AppSec** | Remediation loop (dependency + app vulns) qua Slack threads + ledger | Hunt → verify → score → report → PR fix; 80+ apps incl. Shopify Core | Detection (Dispatch) + closure (River) tách rõ trong nguồn |
| **Deploy / Ops** | Query warehouse, production traces; sau merge check repo head + dependency graph | Continuous / diff scans “while we sleep” | Closure = head + tracker + ledger, không chỉ green PR |

---

## 4. Số liệu (chỉ từ nguồn — kèm URL)

### River / Aquifer — [Under the River](https://shopify.engineering/under-the-river)

| Metric | Giá trị | URL |
|---|---|---|
| Tỷ lệ PR merge coauthored bởi River | **1 in 8** merged PRs across Shopify | https://shopify.engineering/under-the-river |
| Sessions / 30 ngày gần bài | **59,918** River sessions | cùng URL |
| Slack channels | **5,170** channels | cùng URL |
| People touched | **7,000+** (~1,200 hơn lúc Tobi post early May) | cùng URL |
| River-coauthored PRs merged (cùng cửa sổ 30 ngày) | **3,536** | cùng URL |
| Median session length | **19 minutes** | cùng URL |
| Median tool calls / session | **50** | cùng URL |
| Nguồn số | bảng `river_sessions` River tự ghi mỗi session | cùng URL |
| Bet / launch timing | Bet early **2024**; River ship ~**2 months** trước bài (May 28, 2026) | cùng URL |

### Dispatch — [Building an agentic harness…](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model)

| Metric | Giá trị | URL |
|---|---|---|
| Apps scanned (complete) | **>80** unique applications (incl. Shopify Core) | https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model |
| Thời gian / volume scans | ~**6 weeks**, **thousands** of scans | cùng URL |
| Findings | **>300** (defense-in-depth → resolved security incidents) | cùng URL |
| Bug-bounty equivalent (conservative) | **>$400,000** | cùng URL |
| Critical (theo severity calculator họ) | **2** findings | cùng URL |
| Full scan cost (public frontier models) | **$50–$300** / app (phụ thuộc model + size) | cùng URL |
| Incremental diff scan cost | **~$5–$50** | cùng URL |
| Partition size heuristic | **~20–30%** context window | cùng URL |
| Audit anecdote (model noise) | **>30** candidates; sau validation **0** giữ mức cao — tất cả downgrade/FP/DiD | cùng URL |
| Eval window models trong harness | **~5 months** evaluating security-tuned frontier models | cùng URL |

### River remediation — [How River takes security work from a fix to merge](https://shopify.engineering/river-vulnerability-remediation)

| Metric | Giá trị | URL |
|---|---|---|
| Dependency workflow, 11 ngày đầu | Backlog open issues ↓ **~70%** | https://shopify.engineering/river-vulnerability-remediation |
| Trong đó | ~**2/3** direct merges; phần còn lại River confirm obsolete / already fixed | cùng URL |
| Security merges qua freshness-gated merge queue | Từ **~10%** → **~80%** since launch | cùng URL |
| Ví dụ parallel threads | 4 threads / ~1 phút, **35** findings across several codebases | cùng URL |
| Batch 8 dependency PRs | Rebase **7**; bỏ **1** (human takeover); **6/7** green | cùng URL |

**Lưu ý secondary:** InfoWorld/ZenML/Magica lặp lại số River; Magica nhấn mạnh đây là self-reported internal use, không phải đo lường độc lập về quality/cost/business impact — **engineering-depth nằm ở 3 bài shopify.engineering**, secondary chủ yếu là commentary/marketing wrap.

---

## 5. Design principles / patterns tái dùng

1. **Harness outlasts the model** — lợi thế bền là harness gắn ecosystem (test oracle, partitioning, cross-model verify, deterministic Git/creds/storage); model swap được. ([Dispatch](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model))
2. **Decouple brain from hands** — harness ≠ sandbox; safety / replaceability / observability. ([Under the River](https://shopify.engineering/under-the-river); Anthropic quote trong bài)
3. **Session must survive** — Postgres append-only log; cells cattle-not-pets. ([Under the River](https://shopify.engineering/under-the-river))
4. **Multiplayer agent by construction** — public Slack only; corpus là compounding asset; private thread = disadvantage. ([Under the River](https://shopify.engineering/under-the-river))
5. **Agent as profile, not platform** — Nix bundles (prompt + skills + policy + defaults). ([Under the River](https://shopify.engineering/under-the-river))
6. **Agent-friendly ≈ human-friendly** — monorepo, Nix, written skills/`AGENTS.md`, clean CI. ([Under the River](https://shopify.engineering/under-the-river))
7. **Skills + AGENTS.md as living memory** — feed patterns từ transcripts về skills/prompts/defaults; không cần retrain. ([Under the River](https://shopify.engineering/under-the-river))
8. **Test oracle** — prove exploitability bằng real integration-style tests trong suite app. ([Dispatch](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model))
9. **Cross-model adversarial verification** — Verifier ≠ Hunter model; sequential verify. ([Dispatch](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model))
10. **Partition for cost/recall** — 20–30% context; domain grouping + shared context files. ([Dispatch](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model))
11. **Diff-based incremental scans** — full scan lần đầu; sau đó chỉ scan changes. ([Dispatch](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model))
12. **Deterministic code owns credentials / Git / storage / JSON shape** — agents populate, scripts enforce. ([Dispatch](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model); [Remediation](https://shopify.engineering/river-vulnerability-remediation) — “workflow prompt isn't a control plane”)
13. **Revalidate before edit; verify closure on default branch** — ledger/PR green ≠ fixed. ([Remediation](https://shopify.engineering/river-vulnerability-remediation))
14. **Handoff is a first-class stage** — agent dừng đúng lúc với evidence + một câu hỏi rõ. ([Remediation](https://shopify.engineering/river-vulnerability-remediation))
15. **Three consumer modes** — Interactive / Automation / Job trên cùng substrate. ([Under the River](https://shopify.engineering/under-the-river))

---

## 6. Pitfalls / tradeoffs Shopify thừa nhận hoặc ngụ ý

| Pitfall / tradeoff | Nguồn |
|---|---|
| Monorepo + Nix migration **painful** (CI scale, merge queues, rebuild test infra, undo team assumptions) dù payoff lớn | [Under the River](https://shopify.engineering/under-the-river) |
| Local/private agents có **ceiling** học tập tổ chức | cùng |
| Xây surface agent trước khi có platform → “hundred Rivers” pressure; phải tách Aquifer | cùng |
| Models tốt hơn hunter cũng tạo **nhiều noise confident hơn**; noise tới developer **worse than no finding** (mất trust AppSec) | [Dispatch](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model) |
| Generalist security agents: TP có nhưng theoretical/non-deterministic; context window chase list | cùng |
| Full-pipeline-every-file = đắt; single generalist-over-repo = rẻ nhưng trade accuracy/recall/quality | cùng |
| Web exploitability khó; cần strict prove rules hoặc downgrade | cùng |
| Prompt ≠ control plane — agent có thể không enumerate đủ dù prompt bảo; guarantees (pagination, dedup, head identity) phải vào **code** | [Remediation](https://shopify.engineering/river-vulnerability-remediation) |
| Patch “đúng kỹ thuật” trên premise stale đốt thời gian + rủi ro production behavior | cùng |
| CI notification race sau force-push (green cho head cũ) — phải discard và chờ verdict thật | cùng |
| Rebase có thể overwrite work human / preempt design — cần biết **khi nào không đụng** | cùng |
| Fix đề xuất có thể **sai product intent** — đúng outcome có khi là drop patch | cùng |
| Aquifer/River vẫn “at the start” / unfinished; numbers “already wrong, upward” | [Under the River](https://shopify.engineering/under-the-river) |
| Public-only River: privacy/context sensitivity là tradeoff ngụ ý (họ chọn transparency để compound learning) | [Under the River](https://shopify.engineering/under-the-river) |

---

## 7. Nếu team nhỏ hơn Shopify muốn học gì — checklist 8–12 bước

Không copy monorepo/Nix/Aquifer full. Lấy **protocol + harness habits**:

1. **Viết “AGENTS.md / skills” tối thiểu** cho 1–2 zones quan trọng: commands chạy test, boundaries, ví dụ thật, “không được làm gì” — sống cùng repo, diff được.
2. **Tách brain vs hands sớm**: agent loop (API keys, decision log) ngoài sandbox/container chạy code; đừng retrofit sau.
3. **Chọn một multiplayer surface** (public channel / shared ticket thread) cho agent quan trọng — transcript searchable; cấm DM-only nếu muốn compound learning.
4. **Định nghĩa 1 profile trước** (interactive coding helper *hoặc* PR reviewer *hoặc* AppSec hunter) — đừng xây platform cho 10 agents.
5. **Session durable tối thiểu**: lưu event log (DB/file append) để resume sau crash; process agent disposable.
6. **AppSec: hunt ≠ ship finding** — bắt buộc Verifier (model hoặc checklist khác) + **test chứng minh** trước khi ping developer.
7. **Partition theo domain/token budget** (~1/4–1/3 context) thay vì nhét cả repo vào một prompt; cache architecture notes giữa các lần scan.
8. **Diff-first sau baseline**: full scan một lần; incremental trên PR/main diff để kiểm soát cost ($5–50 vs $50–300 là scale Shopify — team nhỏ lấy *ý tưởng* diff, không copy số).
9. **Deterministic wrappers** cho git, secrets, storage, JSON schema — agent điền, script enforce.
10. **Remediation protocol**: revalidate claim vs HEAD trước khi edit; “done” = default branch + tracker; handoff mang 1 câu hỏi + evidence.
11. **Đo tín hiệu tin cậy**, không chỉ volume: % findings có test proof; % handoff được accept; noise rate tới eng (Shopify: noise worse than silence).
12. **Feed lại thất bại vào skills** (không chỉ model upgrade): mỗi false positive / bad patch → một dòng rule trong skill/`AGENTS.md`.

Ba ý Shopify bảo “steal first” ở remediation post: revalidate before edit; evidence follows code (SHA-bound); handoff deliberate. ([Remediation](https://shopify.engineering/river-vulnerability-remediation))  
Ba priorities Aquifer post: decouple brain/hands; multiplayer by construction; next agent = profile. ([Under the River](https://shopify.engineering/under-the-river))

---

## 8. Strongest quotes (EN giữ nguyên) + nguồn

> “Code is going to be increasingly written with AI, and our infrastructure needs to be the substrate for that.”  
> — early 2024 Shopify bet, [Under the River](https://shopify.engineering/under-the-river)

> “If every interaction with an agent happens in a private window, the only person who learns anything is the person at the keyboard.”  
> — Tobi Lütke, *Learning on the Shop floor*, trích trong [Under the River](https://shopify.engineering/under-the-river)

> “Decouple the brain from the hands.”  
> — Anthropic, *Scaling Managed Agents*, April 2026, trích trong [Under the River](https://shopify.engineering/under-the-river)

> “Cells die, sandboxes die, machines die. The conversation doesn't.”  
> — [Under the River](https://shopify.engineering/under-the-river) (paraphrase gần nguyên văn design constraint)

> “River is one profile on top of Aquifer. … Future agents are peers, not children.”  
> — [Under the River](https://shopify.engineering/under-the-river)

> “The models are getting better, but the most important piece of the system remains the harness.”  
> — [Building an agentic harness that outlasts the model](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model)

> “noise sent to a developer is worse than no finding at all. It wastes time and undermines trust with the AppSec Team.”  
> — cùng bài Dispatch

> “You'll keep this part as new models come and go; this is where the innovation is.”  
> — cùng bài Dispatch (về harness)

> “A good patch isn't the same as a fixed vulnerability.”  
> — [How River takes security work from a fix to merge](https://shopify.engineering/river-vulnerability-remediation)

> “A workflow prompt isn't a control plane … Any rule that determines whether a vulnerability is closed must be enforced in code.”  
> — cùng bài Remediation

> “Two years from now, the agent will not be the interesting part. What's underneath it will be.”  
> — [Under the River](https://shopify.engineering/under-the-river)

---

## 9. Related reading links

### Primary — Shopify Engineering (architecture depth)

- https://shopify.engineering/under-the-river — River, World, Nix, Aquifer, modes, metrics  
- https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model — Dispatch AppSec pipeline  
- https://shopify.engineering/river-vulnerability-remediation — River remediation loop (fix → merge)

### Related Shopify Engineering (agentic nhưng **Sidekick merchant**, không phải River/Aquifer SDLC — đừng gộp kiến trúc)

- https://shopify.engineering/building-production-ready-agentic-systems — Sidekick lessons (2025)  
- https://shopify.engineering/sidekicks-continual-learning-loop — Sidekick continual learning (2026)

### Secondary / commentary (label: không phải primary eng-blog; có thể lặp metric hoặc diễn giải)

- https://www.zenml.io/llmops-database/building-river-a-slack-native-ai-agent-on-durable-infrastructure — ZenML LLMOps case summary  
- https://www.infoworld.com/article/4190178/when-software-developers-and-ai-agents-share-the-learning.html — InfoWorld commentary + `agents.md` tips chung  
- https://magica.com/news/slack-public-channels-ai-agent-context — Magica: chi phí/tradeoff shared agent context; cảnh báo self-reported metrics  
- https://pllu.net/links/2026-07-29-shopify-agentic-harness — link blog tóm tắt Dispatch  
- YouTube: “Tectonix: The bedrock of Shopify's Monorepo” — Nix/World build system (talk, không eng post)

### External concept được Shopify cite

- Anthropic, *Scaling Managed Agents* (April 2026) — “Decouple the brain from the hands”  
- Robert Macfarlane, *Is a River Alive?* — metaphor đóng bài Under the River  
- Tobi Lütke, *Learning on the Shop floor* — private-window ceiling (được trích; URL đầy đủ không nằm trong ba primary posts đã fetch)

---

## Appendix — Phân loại nguồn dùng trong brief

| Nguồn | Loại | Dùng cho |
|---|---|---|
| shopify.engineering/under-the-river | Primary engineering | Kiến trúc River/Aquifer/World + hầu hết River metrics |
| shopify.engineering/building-an-agentic-harness-that-outlasts-the-model | Primary engineering | Dispatch stages, oracle, costs, findings |
| shopify.engineering/river-vulnerability-remediation | Primary engineering | Remediation loop + security merge metrics |
| ZenML / InfoWorld / Magica / pllu | Secondary | Related reading only; không invent thêm số |
| Sidekick eng posts | Primary nhưng **product khác** | Related reading; không map vào River SDLC như thể cùng hệ |

*Brief kết thúc. Mọi số đã gắn URL primary; không có metric bịa.*
