# Hiểu 3 bài Shopify Engineering từ số 0

**Dành cho:** người chưa quen agent / harness / monorepo.  
**Mục tiêu:** build mental model trước — không cần đọc bản đào sâu kỹ thuật.  
**Ba bài gốc:**
1. [Under the River](https://shopify.engineering/under-the-river) — River + Aquifer + World/Nix  
2. [Building an agentic harness that outlasts the model](https://shopify.engineering/building-an-agentic-harness-that-outlasts-the-model) — Dispatch (AppSec)  
3. [How River takes security work from a fix to merge](https://shopify.engineering/river-vulnerability-remediation) — vòng đóng lỗ hổng tới merge  

---

## Tầng 0 — Bức tranh lớn (30 giây)

Shopify đang trả lời một câu hỏi:

> Nếu AI sẽ viết ngày càng nhiều code, **hạ tầng và quy trình** của công ty phải thay đổi thế nào để AI làm việc **an toàn, có kiểm soát, và học được theo thời gian**?

Họ không chỉ “cho engineer dùng ChatGPT”. Họ xây:

| Tên | Vai trò gần đúng | Analogy |
|-----|------------------|---------|
| **World + Nix** | Nhà máy + sổ hướng dẫn giống nhau mọi chỗ | Một nhà bếp chuẩn hóa |
| **River** | Nhân viên AI làm việc trên Slack công khai | Đầu bếp phụ trong bếp mở |
| **Aquifer** | Hệ thống vận hành mọi “nhân viên AI” | Nhà hàng / ca làm việc / bàn giao |
| **Dispatch** | Đội kiểm tra an ninh (AppSec) bằng AI | Thanh tra thực phẩm + lab kiểm chứng |
| **Remediation loop** | Đưa fix từ “đã có bản vá” tới “đã merge thật” | Đóng ticket tới khi hàng lên kệ đúng |

Ba bài = ba góc nhìn cùng một hệ thống: **nền tảng chung → săn lỗi bảo mật → đóng việc tới production**.

---

## Tầng 1 — Các khái niệm nền (hiểu trước khi gặp tên riêng)

Đọc theo thứ tự. Mỗi khái niệm gắn 1 analogy.

### 1. LLM / model = “bộ não nói chuyện”

Model (Claude, GPT, …) giỏi **đoán bước tiếp theo**: viết code, giải thích, đề xuất.  
Nó **không** tự biết repo của bạn, **không** tự chạy test, **không** tự mở PR — trừ khi có thứ khác cấp tay chân.

**Analogy:** Giáo sư rất giỏi nhưng ngồi trong phòng kín. Không có thư viện, lab, chìa khóa kho → chỉ nói lý thuyết.

### 2. Agent = “não + vòng lặp hành động”

**Agent** = model + vòng lặp: nghĩ → gọi tool (đọc file, chạy lệnh, mở PR) → xem kết quả → nghĩ tiếp → … cho tới khi xong hoặc dừng.

**Analogy:** Không chỉ hỏi bác sĩ “bệnh gì?” mà bác sĩ còn **tự lấy máu, xem phim, kê đơn, gọi lại** theo quy trình.

### 3. Tool = “tay chân”

Tool = khả năng cụ thể: đọc code, chạy test, query database, post Slack, tạo PR.

**Analogy:** Dao, chảo, đồng hồ nướng — não không cắt rau được nếu không có dao.

### 4. Sandbox = “phòng thí nghiệm dùng một lần”

Nơi agent **thật sự chạy code**. Có thể hỏng, xóa, tạo lại. Không phải nơi giữ “trí nhớ dài hạn” của cuộc hội thoại.

**Analogy:** Bàn thí nghiệm: đổ hóa chất được. Không phải két sắt hồ sơ bệnh án.

### 5. Harness = “dây cương / khung điều khiển”

**Harness** = phần mềm quanh agent: quyết định khi nào gọi model, tool nào được phép, ghi log, restart, đổi model, gắn CI/test.

Shopify nhấn: **harness sống lâu hơn model**. Model đổi mỗi vài tháng; harness (quy trình, oracle, quyền, log) mới là tài sản bền.

**Analogy:**  
- Model = động cơ xe (đổi được)  
- Harness = khung xe + phanh + GPS + hộp đen  
Động cơ mới không cứu được xe không phanh.

### 6. Session = “cuộc hội thoại bền”

Session = danh tính + lịch sử việc đã làm. Máy có thể tắt; conversation vẫn còn (Shopify lưu trên Postgres).

**Analogy:** Hồ sơ bệnh án. Bác sĩ ca này nghỉ → bác sĩ ca sau vẫn đọc được lịch sử.

### 7. Skills / `AGENTS.md` = “sổ tay nội bộ viết xuống”

Kiến thức cách làm việc của team: lệnh chạy test, quy ước, “đừng làm X”, ví dụ PR tốt. Sống **trong repo**, review như code. Agent (và người mới) đọc được.

**Analogy:** Sổ SOP nhà bếp dán tường — không chỉ nằm trong đầu bếp trưởng.

### 8. Human accountability = “người vẫn chịu trách nhiệm”

Agent đề xuất / draft. **Người** quyết merge và chịu production.

**Analogy:** Copilot máy bay hỗ trợ; cơ trưởng vẫn ký chuyến bay.

### 9. Oracle = “trọng tài khách quan”

Cách **chứng minh** claim đúng/sai bằng tín hiệu cứng — thường là **test chạy được**, không chỉ “model nói nghe có lý”.

**Analogy:** Không tin “món này sạch” vì đầu bếp tự tin — mang mẫu đi lab.

---

## Tầng 2 — Mental model tổng: “Não · Tay · Sổ · Trọng tài · Người”

```
        ┌─────────────┐
        │   Người     │  quyết merge / risk / product
        └──────┬──────┘
               │ handoff
        ┌──────▼──────┐
        │   Harness   │  dây cương, log, quyền, đổi model
        │  (brain)    │
        └──────┬──────┘
               │ tool intents
        ┌──────▼──────┐     ┌──────────────┐
        │   Sandbox   │────▶│ Test / CI    │  oracle
        │   (hands)   │     │ (trọng tài)  │
        └─────────────┘     └──────────────┘
               ▲
               │ đọc
        ┌──────┴──────┐
        │ Repo + skills│  World, AGENTS.md, SOP
        │   (sổ tay)   │
        └─────────────┘
```

Nhớ một câu:

> **Model thông minh hơn ≠ hệ thống tốt hơn.**  
> Hệ thống tốt = harness + sandbox tách nhau + sổ tay viết xuống + oracle + người chịu trách nhiệm.

---

## Tầng 3 — Bài 1: Under the River (River + Aquifer + World)

### Pain họ gặp (bằng lời thường)

1. Nhiều repo, môi trường dev mỗi team một kiểu → chậm, khó onboard (người **và** AI).  
2. Agent chỉ chạy local / cửa sổ riêng → chỉ **một người** học được; lần sau người khác bắt đầu từ không.  
3. Muốn nhiều loại agent (review PR, migration, research…) nhưng chưa có **platform chung**.

### Hai cược nền (early 2024)

| Cược | Nghĩa đơn giản | Analogy |
|------|----------------|---------|
| **World** (monorepo) | Gần như mọi thứ Shopify ship nằm **một repo** | Một thành phố thay vì trăm làng không đường nối |
| **Nix** | Dev / CI / prod dùng **cùng công thức** dựng môi trường | Cùng công thức bánh → tiệm nào cũng nướng ra giống nhau |

Insight then: **chỗ làm agent dễ = chỗ làm người dễ**. Repo rối, env lệch, không ai viết SOP → AI cũng chịu.

### River là gì?

**River** = agent AI sống trong **Slack công khai** của công ty.

- Gọi bằng `@river` trong channel **public** (không DM).  
- Có thể: đọc code, chạy test, mở PR, hỏi data warehouse, xem production traces.  
- Người thứ hai nhảy vào thread → River hấp thụ context → transcript search được → pattern sau đó được **ghi vào skills / `AGENTS.md`**.

**Analogy:** Không thuê gia sư dạy riêng trong phòng kín. Đưa trợ lý vào **bếp mở**: cả team thấy cách làm; hôm sau người khác học được từ sổ tay cập nhật.

Số liệu họ công bố (self-reported): khoảng **1/8** PR merge có River đồng tác giả; ~**60k** session / 30 ngày — để cảm nhận quy mô, không phải bằng chứng độc lập về chất lượng.

### Aquifer là gì?

Sau khi River ra, nhiều team muốn “River của mình”. Họ cần **nền tảng chạy hàng trăm agent**, không phải copy-paste River.

**Aquifer** = platform nội bộ. Tách rõ:

| Lớp | Việc | Analogy |
|-----|------|---------|
| **Session** | Cuộc hội thoại bền (Postgres) | Hồ sơ bệnh án |
| **Harness** | Vòng lặp não, gọi model, emit tool | Bác sĩ + quy trình |
| **Sandbox** | Chỗ chạy code | Phòng mổ / lab |
| **Gateway / creds / observability** | Cổng vào hệ thống, quyền, theo dõi | Lễ tân + thẻ từ + camera |

Ba cách “tiêu thụ” cùng một nền (họ bảo kiến trúc khác cuối cùng cũng về 3 mode này):

| Mode | Ví dụ | Analogy |
|------|--------|---------|
| **Interactive** | River trên Slack, có người | Ca trực có bệnh nhân hỏi |
| **Automation** | Tự review PR khi có event | Chuông cửa → bảo vệ kiểm tra |
| **Job** | Chạy batch trong CI rồi hủy | Ca đêm dọn kho, sáng sạch bóng |

**Agent mới = profile/bundle** (prompt + skills + quyền + model mặc định), không phải xây platform mới.

**Analogy:** Không mở chuỗi nhà hàng mới mỗi món. Cùng bếp + SOP; chỉ đổi **thực đơn / ca** (profile).

### Câu nên nhớ từ bài 1

> Private agent giúp **cá nhân**.  
> Public agent + sổ tay trong repo giúp **tổ chức** học.

---

## Tầng 4 — Bài 2: Dispatch (AppSec harness)

### Pain

Model giỏi săn lỗ hổng cũng **tự tin báo sai** rất nhiều. Đưa noise cho engineer → mất niềm tin AppSec (**tệ hơn là không báo gì**).

Thử “agent bảo mật đa năng” một phát: có đúng vài cái, nhưng nhiều cái lý thuyết / không chứng minh được, và context bị đầy vì đuổi theo danh sách lộn xộn.

### Dispatch làm gì?

**Dispatch** = khung (harness) săn lỗi bảo mật theo pipeline có trật tự — không phải một prompt “hãy tìm hết bug”.

Pipeline (ý tưởng, không cần nhớ hết tên stage):

```
Chia codebase thành “ô” (partition)
        ↓
Hunter AI săn song song từng ô
        ↓
Verifier AI (thường model khác) viết/chạy test để CHỨNG MINH
        ↓
Script lọc trùng + chấm severity
        ↓
Báo cáo + draft PR fix
```

### Các ý then chốt (mental model)

**1. Partition**  
Không nhét cả app vào một lần hỏi. Chia theo domain / kích thước vừa phải (~20–30% cửa sổ context) để vừa đủ chỗ suy nghĩ + tool.

**Analogy:** Không bảo một thanh tra “kiểm cả thành phố hôm nay”. Chia theo phường, mỗi tổ một phường.

**2. Test oracle**  
Claim chỉ “sống” nếu có test (thường kiểu integration, gần hành vi thật) chứng minh khai thác được.

**Analogy:** Không chấp nhận “nghe đồn nhà hàng bẩn” — phải có kết quả lab.

**3. Hunter ≠ Verifier**  
Người tìm và người phản biện **tách vai** (thường khác model). Tránh “tự mình khen mình”.

**Analogy:** Phóng viên điều tra ≠ tòa án xét chứng.

**4. Diff scan sau full scan**  
Lần đầu quét đầy đủ (đắt hơn). Sau đó chỉ quét phần đổi (rẻ hơn nhiều — họ nêu khoảng full ~$50–300 / app, diff ~$5–50, tùy model/size).

**Analogy:** Tổng kiểm định lần đầu; lần sau chỉ kiểm chỗ sửa chữa.

**5. Việc nguy hiểm thuộc script deterministic**  
Git, credentials, lưu trữ, format cứng → **code chắc**, không nhờ agent “nhớ làm đúng”.

**Analogy:** Két sắt và chìa không giao cho thực tập sinh “cứ linh hoạt”.

### Câu nên nhớ từ bài 2

> Model săn bug giỏi hơn vẫn cần **harness + trọng tài test**.  
> Không có oracle → bạn chỉ có danh sách tự tin.

---

## Tầng 5 — Bài 3: River remediation (từ “có fix” tới “đã xong thật”)

### Pain

Phát hiện lỗ hổng / tạo PR nâng dependency **chưa đủ**. Trong lúc chờ review:

- Repo đã đổi  
- PR xanh trên bản cũ ≠ default branch đã sạch  
- Engineer nhảy giữa GitHub, Slack, tracker → mệt và dễ sai

### Mental model: “Done” nghĩa là gì?

| Nhiều người tưởng “xong” | Shopify nói “xong” |
|--------------------------|--------------------|
| CI xanh trên PR đang mở | **Default branch** không còn lỗ hổng |
| Có draft patch | Tracker + sổ cái nội bộ **khớp** với repo head |
| Agent đã đề xuất | Người đã **quyết** merge / drop / chấp nhận risk |

### Vòng remediation (đơn giản hóa)

1. **Kiểm tra hiện trạng thật** — đừng tin sổ cái cũ; đối chiếu GitHub / tracker / repo.  
2. **Xác nhận lỗ hổng còn không** trước khi sửa (có khi đã fix / đã có PR khác).  
3. **Sửa trên head hiện tại** — rebase / tạo lại lockfile đúng tooling repo.  
4. **Handoff** — đưa evidence + **một câu hỏi nhỏ** cho owner (risk/product), không đổ cả đống điều tra.  
5. **Verify closure** — sau merge, check lại nhánh chính.

**Analogy:** Không tuyên bố “cháy đã tắt” vì xe cứu hỏa đã tới. Phải xem **nhà còn khói không**, báo cáo PCCC đã đóng, và sổ theo dõi khớp hiện trường.

Có khi outcome đúng là **bỏ patch** — vì sai product intent — không phải merge bằng mọi giá.

### Câu nên nhớ từ bài 3

> **Prompt workflow ≠ control plane.**  
> Luật “phải kiểm tra head / phải đóng tracker” cần **code enforce**, không chỉ nhờ agent nhớ.

---

## Tầng 6 — Ba bài gắn với nhau thế nào?

```
Bài 1: Xây “thành phố + nhà hàng”
        World/Nix (nền) → Aquifer (vận hành agent) → River (mặt tiền Slack)

Bài 2: Đội thanh tra chuyên biệt trên cùng tinh thần harness
        Dispatch: hunt → prove bằng test → báo cáo → draft fix

Bài 3: Đội vận hành đóng việc tới production
        River remediation: freshness → edit → handoff → closure trên default branch
```

Cùng một bộ ý tưởng xuyên suốt:

1. Tách **não (harness)** và **tay (sandbox)**  
2. Viết kiến thức xuống (**skills / `AGENTS.md`**)  
3. Có **oracle** (test/CI), không chỉ lời model  
4. **Người** chịu merge / risk  
5. “Xong” đo bằng **trạng thái hệ thống thật**, không bằng cảm giác session

---

## Tầng 7 — Glossary siêu ngắn (tra cứu)

| Thuật ngữ | Một câu |
|-----------|---------|
| **Agent** | Model + vòng lặp dùng tool để làm việc |
| **Harness** | Khung điều khiển quanh agent (quyền, log, đổi model, gắn CI) |
| **Sandbox** | Chỗ chạy code, dùng rồi có thể hủy |
| **Session** | Cuộc hội thoại / việc bền theo thời gian |
| **Monorepo (World)** | Nhiều dự án trong một repo lớn |
| **Nix** | Cách dựng môi trường lặp lại được |
| **River** | Agent Slack công khai của Shopify |
| **Aquifer** | Platform chạy nhiều agent (River chỉ là một profile) |
| **Dispatch** | Harness AppSec: săn → chứng minh → báo → draft fix |
| **Partition** | Chia codebase thành phần vừa để agent xử lý |
| **Oracle** | Tín hiệu khách quan (thường là test) để chấp nhận/ bác claim |
| **Remediation** | Đưa từ “biết lỗ hổng / có patch” tới “đã đóng trên nhánh chính” |
| **`AGENTS.md` / skills** | SOP viết trong repo cho người và agent |

---

## Tầng 8 — Lộ trình đọc lại 3 bài gốc (khi đã có model)

1. Đọc **Under the River** như câu chuyện sản phẩm: đau → World/Nix → River → vì sao cần Aquifer. Bỏ qua chi tiết Postgres nếu chóng mặt; giữ mental model Session / Harness / Sandbox.  
2. Đọc **Dispatch** như quy trình nhà máy kiểm định: vì sao không tin hunter một mình; test oracle; partition; diff scan.  
3. Đọc **Remediation** như vận hành ticket: freshness, handoff, định nghĩa “done”.

Nếu một đoạn kỹ thuật quá dày: quay lại **Tầng 2** (sơ đồ Não · Tay · Sổ · Trọng tài · Người) rồi đọc tiếp.

---

## Một analogy tổng kết mang về

Tưởng Shopify đang mở **nhà hàng rất lớn**:

- **World + Nix** = một bếp chuẩn, công thức giống nhau mọi ca  
- **Aquifer** = hệ thống ca trực, hồ sơ order, camera, thẻ quyền  
- **River** = phụ bếp trên quầy mở — cả quán thấy và học được  
- **Dispatch** = thanh tra + lab: nghi ngờ phải có xét nghiệm  
- **Remediation** = không hô “đã sửa” khi mới dán phiếu — phải lên món đúng, sổ đóng, khách nhận hàng thật  

Động cơ bếp (model) sẽ đổi. **Quy trình nhà hàng** (harness) mới là thứ họ đầu tư để sống lâu.
