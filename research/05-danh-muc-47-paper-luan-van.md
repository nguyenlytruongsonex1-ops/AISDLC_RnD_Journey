# Danh mục tài liệu học thuật: "AI-native SDLC" cho luận văn Thạc sĩ HTTT (MIS)

*Biên soạn: 02/10/2026 (giờ Việt Nam). Cả 47 bài đều đã được kiểm chứng: tiêu đề, tác giả, năm đối chiếu với trang arXiv qua arXiv API; nơi công bố đối chiếu với Crossref/DOI, Semantic Scholar, OpenReview/ICLR, NeurIPS proceedings, IEEE Xplore hoặc ghi chú "accepted" của chính tác giả trên arXiv. Bài nào chưa qua bình duyệt thì ghi "arXiv preprint". Với bài ghi "accepted (theo arXiv)", nơi công bố lấy từ ghi chú của tác giả trên arXiv. Bản chính thức chưa được kiểm tra trong thư viện số của nhà xuất bản.*

## Giới thiệu

Đến 2026, nghiên cứu về "AI-native SDLC" đã đi qua ba giai đoạn. Đầu tiên là chứng minh tác tử LLM có thể biến một issue thành patch (SWE-agent, AutoCodeRover, Agentless, OpenHands). Tiếp theo là phát hiện việc đánh giá "đã giải quyết" thường quá lạc quan, do test yếu, patch "plausible but wrong" và dữ liệu bị nhiễm (contamination). Hiện nay trọng tâm là cách đưa tác tử vào quy trình doanh nghiệp một cách có kiểm soát, gồm review, CI gate, chính sách, truy vết và kiểm toán. Hướng luận văn "biến ticket nghiệp vụ thành thay đổi phần mềm đã kiểm chứng và truy vết được" nằm đúng chỗ giao nhau còn ít người làm: kết hợp tác tử giải issue (cụm 1), truy vết yêu cầu–mã–test (cụm 2), kiểm chứng (cụm 3) và quản trị (cụm 4). Hướng này cũng có thể đánh giá bằng benchmark công khai (cụm 5) và bằng nghiên cứu thực nghiệm hoặc công nghiệp (cụm 7).

Ký hiệu: **[Vxx]** là số thứ tự bài, dùng để tham chiếu chéo ở phần Top must-read và Khoảng trống nghiên cứu.

---

## 1. Tác tử chuyển issue/ticket thành PR (Issue-to-PR agents)

- **[V01] [SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering](https://arxiv.org/abs/2405.15793)** — John Yang et al. — 2024 — NeurIPS 2024.
  Bài đề xuất khái niệm "agent-computer interface" (ACI): một giao diện riêng để tác tử LM duyệt repo, sửa file và chạy test, và cho thấy thiết kế giao diện ảnh hưởng mạnh đến hiệu năng. Với luận văn, ACI là chỗ tự nhiên để gắn chính sách, ghi log và kiểm soát hành động của tác tử.

- **[V02] [AutoCodeRover: Autonomous Program Improvement](https://arxiv.org/abs/2404.05427)** — Yuntong Zhang et al. — 2024 — ISSTA 2024.
  Kết hợp LLM với tìm kiếm mã có cấu trúc (theo lớp/phương thức trên AST) để giải GitHub issue, theo hướng "kỹ nghệ phần mềm" hơn là để tác tử tự do. Đây là ví dụ cho thấy phân tích chương trình truyền thống làm tác tử đáng tin hơn.

- **[V03] [Agentless: Demystifying LLM-based Software Engineering Agents](https://arxiv.org/abs/2407.01489)** — Chunqiu Steven Xia et al. — 2024 — FSE 2025 (Proc. ACM Softw. Eng.; bản chính thức có tên "Demystifying LLM-Based Software Engineering Agents", [DOI](https://doi.org/10.1145/3715754)).
  Một pipeline cố định ba pha (định vị lỗi, sửa, kiểm chứng patch), không để LLM tự quyết hành động, nhưng đạt 32% trên SWE-bench Lite với chi phí khoảng 0,70 USD mỗi issue. Tác giả cũng phát hiện nhiều issue có mô tả thiếu hoặc gây hiểu nhầm. Với hệ thống "có quản trị", pipeline tất định và dễ giải thích kiểu Agentless là baseline rất phù hợp.

- **[V04] [OpenHands: An Open Platform for AI Software Developers as Generalist Agents](https://arxiv.org/abs/2407.16741)** — Xingyao Wang et al. — 2024 — ICLR 2025.
  Nền tảng mã nguồn mở (tên cũ OpenDevin) cho tác tử viết mã, dùng dòng lệnh và trình duyệt trong sandbox, có hỗ trợ đa tác tử và benchmark. Đây là nền tảng thực tế để làm prototype luận văn và chèn thêm lớp policy/audit.

- **[V05] [MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework](https://arxiv.org/abs/2308.00352)** — Sirui Hong et al. — 2023 — ICLR 2024 (oral, [OpenReview](https://openreview.net/forum?id=VtmBAGCN7o)).
  Mã hóa quy trình chuẩn (SOP) thành chuỗi prompt và phân vai PM/kiến trúc sư/kỹ sư/QA để các tác tử kiểm tra kết quả trung gian của nhau. Đây là cách "đưa quy trình tổ chức vào tác tử", rất gần góc nhìn MIS.

- **[V06] [ChatDev: Communicative Agents for Software Development](https://arxiv.org/abs/2307.07924)** — Chen Qian et al. — 2023 — ACL 2024.
  Mô phỏng một "công ty phần mềm" gồm các tác tử trao đổi qua chat chain ở các pha thiết kế, viết mã, kiểm thử, có cơ chế "communicative dehallucination". Bài này đại diện cho trường phái SDLC đa tác tử theo mô hình thác nước, nên dùng làm đối chứng với cách tiếp cận dựa trên issue.

- **[V07] [SpecRover: Code Intent Extraction via LLMs](https://arxiv.org/abs/2408.02232)** — Haifeng Ruan et al. — 2024 — ICSE 2025.
  Suy diễn đặc tả (intent) lặp đi lặp lại từ issue và mã, rồi dùng một reviewer agent để thẩm định patch và đưa ra độ tin cậy kèm giải thích cho lập trình viên. Đây là ý tưởng gần nhất với "thay đổi đã kiểm chứng kèm bằng chứng" trong hướng luận văn.

## 2. Truy vết yêu cầu – mã – test bằng LLM và LLM cho kỹ nghệ yêu cầu

- **[V08] [Natural Language Processing for Requirements Traceability](https://arxiv.org/abs/2405.10845)** — Jin L. C. Guo et al. — 2024 — Chương sách trong *Handbook of Natural Language Processing for Requirements Engineering* (bản arXiv).
  Tổng quan các bài toán truy vết, gồm trace link recovery, trace link maintenance và cách dùng liên kết trong thực tế. Nên đọc để nắm khái niệm và thuật ngữ chuẩn trước khi thiết kế phần truy vết.

- **[V09] [Prompts Matter: Insights and Strategies for Prompt Engineering in Automated Software Traceability](https://arxiv.org/abs/2308.00229)** — Alberto D. Rodriguez et al. — 2023 — RE 2023 Workshops (REW), IEEE.
  Một trong những bài đầu tiên dùng LLM sinh và xếp hạng trace link, kèm bài học về thiết kế prompt. Nhóm của Cleland-Huang là nhóm chủ chốt về traceability.

- **[V10] [LiSSA: Toward Generic Traceability Link Recovery through Retrieval-Augmented Generation](https://ieeexplore.ieee.org/document/11029919)** — Dominik Fuchß et al. — 2025 — ICSE 2025 (Research Track).
  Khung RAG + LLM khôi phục liên kết giữa nhiều loại artefact (yêu cầu–mã, tài liệu–mã, tài liệu kiến trúc–mô hình). Phương pháp vượt SOTA ở các tác vụ liên quan đến mã, nhưng tác giả thừa nhận vẫn chưa đủ tốt để dùng thực tế. Đây là thành phần ứng viên để nối ticket với mã trong hệ thống của luận văn.

- **[V11] [Natural Language-Programming Language Software Traceability Link Recovery Needs More than Textual Similarity](https://arxiv.org/abs/2509.05585)** — Zhiyuan Zou et al. — 2025 — arXiv preprint.
  Chỉ ra rằng chỉ dựa vào độ tương đồng văn bản là không đủ cho truy vết giữa ngôn ngữ tự nhiên và mã. Bài bổ sung tín hiệu phụ (phụ thuộc mã, phản hồi người dùng...) vào HGT và Gemini 2.5 Pro cho bài toán requirements-to-code. Kết quả gợi ý rằng tác tử nên ghi lại tín hiệu cấu trúc như file đã đọc hay hàm đã gọi để truy vết tốt hơn.

- **[V12] [Generative AI for Requirements Engineering: A Systematic Literature Review](https://arxiv.org/abs/2409.06741)** — Haowei Cheng et al. — 2024 — Software: Practice and Experience (2025).
  SLR trên 238 bài (2019–2025). Kết quả: nghiên cứu GenAI cho RE tập trung vào phân tích (30,0%) và khơi gợi yêu cầu (22,1%), còn quản lý yêu cầu (6,8%) ít được quan tâm. Các thách thức hàng đầu là khả năng tái lập (66,8%) và ảo giác (hallucination, 63,4%). Bài này là căn cứ trực tiếp để lập luận rằng "quản lý/truy vết yêu cầu trong pipeline tác tử" là khoảng trống.

- **[V13] [TraceDev: A Traceability-Driven Multi-agent Framework for Requirement-to-Code Development](https://arxiv.org/abs/2607.18886)** — Mingyu Chen et al. — 2026 — ISSTA 2026 (accepted, theo arXiv).
  Năm tác tử (Requirement Refiner, Designer, Developer, Tester, Validator), trong đó Validator duy trì một đồ thị truy vết dị thể giữa yêu cầu, thiết kế và mã để kiểm tra nhất quán. Hệ thống đạt tỉ lệ thành công 53,63% (ETOUR) và 56,82% (SMOS). Đây là công trình gần nhất với ý tưởng "truy vết như một thành phần hạng nhất của pipeline tác tử", nhưng mới áp dụng cho sinh mã từ use case, chưa áp dụng cho sửa repo có sẵn.

- **[V14] [ReqToCode: Embedding Requirements Traceability as a Structural Property of the Codebase](https://arxiv.org/abs/2603.13999)** — Thorsten Schlathölter — 2026 — arXiv preprint (tác giả ghi rõ chưa bình duyệt).
  Đề xuất nhúng "Traceable", tức phần tử mã sinh ra đại diện cho từng yêu cầu, để liên kết yêu cầu–mã–test được kiểm tra ngay lúc build, thay vì khôi phục liên kết về sau. Bài chỉ có ví dụ minh họa, chưa có đánh giá thực nghiệm, nên là cơ hội để luận văn đánh giá ý tưởng "traceability by construction" cho mã do tác tử tạo.

## 3. Kiểm chứng thay đổi của tác tử: test làm oracle, CI gate, patch overfitting, review có con người

- **[V15] [SWT-Bench: Testing and Validating Real-World Bug-Fixes with Code Agents](https://arxiv.org/abs/2406.12952)** — Niels Mündler et al. — 2024 — NeurIPS 2024.
  Benchmark đánh giá khả năng tác tử biến issue thành test tái hiện lỗi. Kết quả cho thấy dùng test sinh ra làm bộ lọc patch giúp tăng gấp đôi precision của SWE-agent. Đây là bằng chứng trực tiếp cho thiết kế "test từ ticket làm cổng kiểm chứng".

- **[V16] [Otter: Generating Tests from Issues to Validate SWE Patches](https://arxiv.org/abs/2502.05368)** — Toufique Ahmed et al. — 2025 — ICML 2025.
  Sinh test từ issue trước khi có patch (theo tinh thần TDD) bằng LLM kết hợp phân tích luật và một planner tự phản tỉnh, đồng thời giới thiệu TDD-Bench-Verified. Bài này phù hợp với bước "acceptance criteria → test" trong pipeline ticket→PR.

- **[V17] [Are "Solved Issues" in SWE-bench Really Solved Correctly? An Empirical Study](https://arxiv.org/abs/2503.15223)** — You Wang et al. — 2025 — ICSE 2026.
  Kỹ thuật PatchDiff (differential patch testing) cho thấy 29,6% patch "plausible" hành xử khác patch gốc, và 7,8% patch được tính là đúng dù trượt bộ test của developer. Tổng hợp lại, tỉ lệ giải quyết bị thổi phồng 6,2 điểm phần trăm. Đây là luận cứ cốt lõi để nói "pass test ≠ đúng" và cần cổng kiểm chứng mạnh hơn.

- **[V18] [Investigating Test Overfitting on SWE-bench](https://arxiv.org/abs/2511.16858)** — Toufique Ahmed et al. — 2025 — arXiv preprint (bản v1 có tên "Is the Cure Still Worse Than the Disease? Test Overfitting by LLMs in Automated Program Repair").
  Nghiên cứu thực nghiệm đầu tiên về test overfitting khi hệ thống giải issue dùng test tự sinh từ issue, thậm chí tinh chỉnh mã và test cùng lúc. Bài cảnh báo rằng dùng test sinh tự động làm oracle cũng có rủi ro, nên phải thiết kế cổng kiểm chứng cẩn thận.

- **[V19] [Understanding Automated Program Repair Agents Through the Lens of Traceability: An Empirical Study](https://arxiv.org/abs/2506.08311)** — Ira Ceka et al. — 2025 — ISSTA 2026 (accepted, theo arXiv).
  Truy vết toàn bộ chuỗi quyết định của 5 tác tử APR trên 500 tác vụ. Kết quả: tác tử xử lý tốt lỗi đơn giản nhưng hay tạo patch dài dòng, overfit, qua được test hiện có mà không sửa nguyên nhân gốc. Sinh test và chọn test hồi quy là nút thắt. Tác giả đề xuất "shift-left" về sinh test, rất khớp với luận văn.

- **[V20] [These Aren't the Reviews You're Looking For: How Humans Review AI-Generated Pull Requests](https://arxiv.org/abs/2605.02273)** — Kacper Duma et al. — 2026 — EASE 2026 (accepted, theo arXiv).
  Trên dữ liệu AIDev, phần lớn PR do AI tạo không được review. Khi có review thì thường do tác tử AI chi phối, còn con người chủ yếu "lái" tác tử thay vì đánh giá mã. Điều này cho thấy giả định "đã có human-in-the-loop" trong thực tế rất mong manh.

- **[V21] [From Industry Claims to Empirical Reality: An Empirical Study of Code Review Agents in Pull Requests](https://arxiv.org/abs/2604.03196)** — Kowshik Chowdhury et al. — 2026 — MSR 2026.
  PR chỉ được review bởi code review agent có tỉ lệ merge 45,20%, thấp hơn 23,17 điểm phần trăm so với PR do người review (68,37%). Phần lớn bình luận của CRA có tỉ lệ tín hiệu/nhiễu thấp. Kết luận là CRA nên bổ trợ chứ không thay thế reviewer, một luận cứ cho thiết kế human-in-the-loop.

## 4. Quản trị, guardrail, kiểm toán, nguồn gốc (provenance) và an ninh của tác tử lập trình

- **[V22] [Security Weaknesses of Copilot-Generated Code in GitHub Projects: An Empirical Study](https://arxiv.org/abs/2310.02059)** — Yujia Fu et al. — 2023 — ACM TOSEM (2025).
  Phân tích 733 đoạn mã do Copilot, CodeWhisperer và Codeium sinh trong các dự án GitHub thật. Có 29,5% đoạn Python và 24,2% đoạn JavaScript chứa điểm yếu bảo mật. Đây là căn cứ cho việc bắt buộc quét bảo mật (SAST) trong cổng kiểm chứng.

- **[V23] [RedCode: Risky Code Execution and Generation Benchmark for Code Agents](https://arxiv.org/abs/2411.07781)** — Chengquan Guo et al. — 2024 — NeurIPS 2024 (Datasets & Benchmarks).
  Benchmark gồm 4.050 ca chạy mã rủi ro (Python/Bash, 25 loại lỗ hổng) cùng tác vụ sinh mã độc hại, dùng để đánh giá mức độ an toàn của tác tử. Có thể dùng làm bộ kịch bản để kiểm thử lớp policy/sandbox của luận văn.

- **[V24] ["Your AI, My Shell": Demystifying Prompt Injection Attacks on Agentic AI Coding Editors](https://arxiv.org/abs/2509.22040)** — Yue Liu et al. — 2025 — arXiv preprint.
  Công cụ AIShellJack (314 payload, 70 kỹ thuật MITRE ATT&CK) cho thấy chỉ cần đầu độc tài nguyên dev như file rule là có thể chiếm quyền Cursor và Copilot, với tỉ lệ thành công tới 84%. Nghĩa là ticket hay tài liệu đưa vào tác tử cũng là bề mặt tấn công, cần chính sách kiểm soát đầu vào.

- **[V25] [Progent: Securing AI Agents with Privilege Control](https://arxiv.org/abs/2504.11703)** — Tianneng Shi et al. — 2025 — arXiv preprint.
  Biểu diễn quyền dưới dạng chính sách ký hiệu (luật trên tên tool và tham số), kiểm tra tất định từng tool call để áp nguyên tắc đặc quyền tối thiểu. Đây là mẫu "policy-as-code" có thể chuyển sang bối cảnh SDLC, ví dụ cấm sửa thư mục bảo vệ hay cấm push trực tiếp.

- **[V26] [Visibility into AI Agents](https://arxiv.org/abs/2401.13138)** — Alan Chan et al. — 2024 — ACM FAccT 2024.
  Phân tích ba nhóm biện pháp tăng "khả kiến" của tác tử: định danh tác tử, giám sát thời gian thực và ghi nhật ký hoạt động. Bài cung cấp khung lý thuyết quản trị cho phần audit trail của luận văn.

- **[V27] [Who Wrote this Code? Watermarking for Code Generation](https://arxiv.org/abs/2305.15060)** — Taehyun Lee et al. — 2023 — ACL 2024.
  Phương pháp SWEET thủy vân mã do LLM sinh bằng cách bỏ qua các đoạn entropy thấp, vì mã có entropy thấp khiến watermark thông thường thất bại. Bài cho thấy giới hạn của truy xuất nguồn gốc ở mức token, từ đó gợi ý ghi provenance ở mức quy trình (metadata commit/PR).

## 5. Benchmark và các phê bình

- **[V28] [SWE-bench: Can Language Models Resolve Real-World GitHub Issues?](https://arxiv.org/abs/2310.06770)** — Carlos E. Jimenez et al. — 2023 — ICLR 2024.
  Benchmark gốc gồm 2.294 issue/PR thật từ 12 repo Python, đánh giá bằng test. Đây là chuẩn chung để đặt prototype của luận văn vào ngữ cảnh.

- *Tài liệu xám (không tính vào 47 bài):* [Introducing SWE-bench Verified](https://openai.com/index/introducing-swe-bench-verified/) — OpenAI, blog, 13/08/2024. Đây là tập con SWE-bench được con người lọc lại. Bản thân nó không phải bài bình duyệt, nhưng được [V17], [V32], [V33] phân tích lại.

- **[V29] [SWE-bench Multimodal: Do AI Systems Generalize to Visual Software Domains?](https://arxiv.org/abs/2410.03859)** — John Yang et al. — 2024 — ICLR 2025.
  Mở rộng sang các issue JavaScript có hình ảnh, giao diện người dùng (617 tác vụ, 17 thư viện). Bài cho thấy hệ thống tối ưu cho Python/SWE-bench khái quát hóa kém, điều cần lưu ý khi ticket nghiệp vụ có ảnh chụp màn hình.

- **[V30] [SWE-bench Goes Live!](https://arxiv.org/abs/2505.23419)** — Linghao Zhang et al. — 2025 — NeurIPS 2025 (Datasets & Benchmarks).
  Benchmark cập nhật liên tục từ issue sau 2024, mỗi tác vụ có Docker riêng, kèm pipeline tự động dựng môi trường. Hiệu năng trên benchmark này thấp hơn đáng kể so với SWE-bench tĩnh. Pipeline có thể tái sử dụng để dựng benchmark nội bộ cho luận văn.

- **[V31] [SWE-Bench Pro: Can AI Agents Solve Long-Horizon Software Engineering Tasks?](https://arxiv.org/abs/2509.16941)** — Xiang Deng et al. — 2025 — arXiv preprint.
  Gồm 1.865 bài từ 41 repo (ứng dụng doanh nghiệp, B2B, công cụ dev), có tập held-out và tập thương mại riêng tư. Các tác vụ dài, sửa nhiều file. Benchmark này gần với bối cảnh doanh nghiệp hơn SWE-bench.

- **[V32] [SWE-Bench+: Enhanced Coding Benchmark for LLMs](https://arxiv.org/abs/2410.06992)** — Reem Aleithan et al. — 2024 — arXiv preprint.
  32,67% patch "thành công" của SWE-Agent+GPT-4 là do lời giải đã có sẵn trong issue (solution leakage), và 31,08% là đáng ngờ do test yếu. Sau khi lọc, tỉ lệ giải quyết giảm từ 12,47% xuống 3,97%. Hơn 94% issue có trước ngày cắt dữ liệu của LLM.

- **[V33] [The SWE-Bench Illusion: When State-of-the-Art LLMs Remember Instead of Reason](https://arxiv.org/abs/2506.12286)** — Shanchao Liang et al. — 2025 — ICSE-SEIP 2026.
  Chỉ từ mô tả issue, mô hình đoán đúng file lỗi tới 76% trên SWE-Bench Verified, nhưng chỉ tới 53% với repo ngoài SWE-bench. Đây là dấu hiệu ghi nhớ hoặc nhiễm dữ liệu, nên luận văn cần đánh giá trên dữ liệu "mới" hoặc nội bộ.

## 6. Survey và bài tầm nhìn

- **[V34] [Large Language Model-Based Agents for Software Engineering: A Survey](https://arxiv.org/abs/2409.02977)** — Junwei Liu et al. — 2024 — ACM TOSEM.
  Khảo sát 124 bài, phân loại theo góc nhìn SE và góc nhìn tác tử, có kho bài cập nhật. Phù hợp làm "bản đồ" cho chương tổng quan.

- **[V35] [LLM-Based Multi-Agent Systems for Software Engineering: Literature Review, Vision and the Road Ahead](https://arxiv.org/abs/2404.04834)** — Junda He et al. — 2024 — ACM TOSEM (2025, số đặc biệt "SE 2030").
  Tổng quan hệ đa tác tử dọc vòng đời SDLC, kèm hai case study và chương trình nghiên cứu hướng tới hệ đa tác tử "đáng tin cậy".

- **[V36] [Towards AI-Native Software Engineering (SE 3.0): A Vision and a Challenge Roadmap](https://arxiv.org/abs/2410.06107)** — Ahmed E. Hassan et al. — 2024 — ACM TOSEM (2026).
  Đặt khái niệm SE 3.0: phát triển lấy ý định làm trung tâm (intent-centric), dựa trên hội thoại giữa người và "AI teammate", với stack Teammate.next, IDE.next, Compiler.next... Bài này là nguồn thuật ngữ "AI-native SE" cho tiêu đề luận văn.

- **[V37] [Agentic Software Engineering: Foundational Pillars and a Research Roadmap](https://arxiv.org/abs/2509.06216)** — Ahmed E. Hassan et al. — 2025 — arXiv preprint.
  Tầm nhìn SASE gồm Agent Command Environment và Agent Execution Environment, cùng các artefact mới như Merge-Readiness Pack (MRP) và Consultation Request Pack. Khái niệm MRP gần như trùng với ý "thay đổi đã kiểm chứng, truy vết được", nhưng mới dừng ở mức khái niệm, chưa được hiện thực hóa và đánh giá.

- **[V38] [Challenges and Paths Towards AI for Software Engineering](https://arxiv.org/abs/2503.22625)** — Alex Gu et al. — 2025 — arXiv preprint.
  Đưa ra phân loại nhiệm vụ AI4SE vượt ra ngoài sinh mã, các nút thắt chính và danh sách hướng nghiên cứu. Hữu ích để lập luận vì sao "kiểm chứng" và "đánh giá" là nút thắt.

## 7. Nghiên cứu thực nghiệm và công nghiệp

- **[V39] [Evaluating Agent-based Program Repair at Google](https://arxiv.org/abs/2501.07531)** — Pat Rondon et al. — 2025 — ICSE-SEIP 2025.
  Tác tử Passerine được thử trên 178 bug nội bộ Google. Patch "plausible" đạt 73% với bug do máy báo và 25,6% với bug do người báo, nhưng patch tương đương ngữ nghĩa chỉ đạt 43% và 17,9%. Phân phối bug doanh nghiệp cũng khác SWE-bench, một luận cứ cho việc phải đánh giá trên dữ liệu nội bộ.

- **[V40] [Agentic Program Repair from Test Failures at Scale: A Neuro-symbolic approach with static analysis and test execution feedback](https://arxiv.org/abs/2507.18755)** — Chandra Maddila et al. — 2025 — IEEE TSE (2026).
  Engineering Agent của Meta dùng phản hồi từ phân tích tĩnh và test, cộng LLM-as-a-Judge, sau đó qua review của con người. Trong 3 tháng, 80% fix được review, và 31,5% số được review đã được land. Đây là mẫu tham chiếu công nghiệp cho kiến trúc "tác tử + cổng kiểm chứng + người duyệt".

- **[V41] [WhatsCode: Large-Scale GenAI Deployment for Developer Efficiency at WhatsApp](https://arxiv.org/abs/2512.05314)** — Ke Mao et al. — 2025 — ICSE-SEIP 2026 ([DOI](https://doi.org/10.1145/3786583.3786877)). **(Đã xác minh.)**
  Báo cáo 25 tháng triển khai trong môi trường có yêu cầu tuân thủ (privacy). Độ phủ kiểm chứng privacy tự động tăng từ 15% lên 53%, hơn 3.000 thay đổi được chấp nhận. Hai mẫu cộng tác người–AI xuất hiện: "one-click rollout" (60%) và "commandeer-revise" (40%). Tác giả kết luận yếu tố tổ chức (ownership, quản trị rủi ro) quan trọng ngang kỹ thuật, một kết luận rất "MIS".

- **[V42] [MarsCode Agent: AI-native Automated Bug Fixing](https://arxiv.org/abs/2409.00899)** — Yizhou Liu et al. — 2024 — arXiv preprint (ByteDance).
  Quy trình gồm lập kế hoạch, tái hiện bug, định vị lỗi, sinh patch và kiểm chứng trong một sản phẩm IDE thương mại. Đây là ví dụ công nghiệp tự gọi mình là "AI-native".

- **[V43] [Automated Unit Test Improvement using Large Language Models at Meta](https://arxiv.org/abs/2402.09171)** — Nadia Alshahwan et al. — 2024 — FSE 2024 Companion (Industry).
  TestGen-LLM chỉ đề xuất test đã qua các bộ lọc (build được, pass ổn định, tăng coverage). 73% đề xuất được kỹ sư chấp nhận đưa vào production. Đây là mẫu "assured LLM-based SE": lọc đầu ra LLM bằng các kiểm tra tất định.

- **[V44] [The Rise of AI Teammates in Software Engineering (SE) 3.0: How Autonomous Coding Agents Are Reshaping Software Engineering](https://arxiv.org/abs/2507.15003)** — Hao Li et al. — 2025 — arXiv preprint.
  Dataset AIDev gồm hơn 456.000 PR của 5 tác tử (Codex, Devin, Copilot, Cursor, Claude Code) trên 61.000 repo. Đây là nguồn dữ liệu miễn phí và đủ lớn cho nghiên cứu thực nghiệm về PR của tác tử mà không cần chạy mô hình.

- **[V45] [On the Use of Agentic Coding: An Empirical Study of Pull Requests on GitHub](https://arxiv.org/abs/2509.14745)** — Miku Watanabe et al. — 2025 — ACM TOSEM (2026).
  Trong 567 PR do Claude Code hỗ trợ, 83,8% được merge, nhưng 45,1% số PR được merge phải chỉnh sửa thêm, đặc biệt ở bug fix và tuân thủ chuẩn dự án. Bài cho thấy tác tử hữu ích nhưng vẫn cần giám sát.

- **[V46] [Where Do AI Coding Agents Fail? An Empirical Study of Failed Agentic Pull Requests in GitHub](https://arxiv.org/abs/2601.15195)** — Ramtin Ehsani et al. — 2026 — MSR 2026 (accepted, theo arXiv).
  Phân tích 33 nghìn PR của tác tử. PR không được merge thường lớn hơn, chạm nhiều file và trượt CI. Phân loại lý do bị từ chối gồm thiếu reviewer, PR trùng lặp, tính năng không mong muốn và tác tử hiểu sai ý (misalignment). Đây là căn cứ cho việc kiểm soát phạm vi và độ khớp với ticket.

- **[V47] [Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity](https://arxiv.org/abs/2507.09089)** — Joel Becker et al. — 2025 — arXiv preprint (METR).
  RCT với 16 lập trình viên giàu kinh nghiệm trên 246 tác vụ. Khi được dùng AI, thời gian hoàn thành tăng 19%, trong khi chính họ ước tính AI giúp nhanh hơn 20%. Kết quả này củng cố luận điểm của luận văn: "đừng chỉ dùng AI để viết mã nhanh hơn".

---

## Top 10 must-read (thứ tự đọc gợi ý)

1. **[V37] [Agentic Software Engineering: Foundational Pillars and a Research Roadmap](https://arxiv.org/abs/2509.06216)**: đọc trước để có khung khái niệm (SE for Humans / SE for Agents, Merge-Readiness Pack). Đây là "ngôn ngữ" để định vị luận văn. Có thể đọc kèm [V36] để lấy thuật ngữ SE 3.0.
2. **[V28] [SWE-bench](https://arxiv.org/abs/2310.06770)**: hiểu bài toán issue→patch và cách đánh giá bằng test. Mọi bài sau đều dựa trên benchmark này.
3. **[V01] [SWE-agent](https://arxiv.org/abs/2405.15793)**: hiểu kiến trúc tác tử và ACI, chỗ sẽ gắn policy và log.
4. **[V03] [Agentless](https://arxiv.org/abs/2407.01489)**: pipeline tất định, rẻ, dễ giải thích. Phù hợp làm khung cho hệ thống "có quản trị" và làm baseline.
5. **[V07] [SpecRover](https://arxiv.org/abs/2408.02232)**: suy diễn đặc tả và reviewer agent, cho ra patch kèm giải thích và độ tin cậy. Đây là bước đầu tiên về "bằng chứng" đi kèm thay đổi.
6. **[V15] [SWT-Bench](https://arxiv.org/abs/2406.12952)**: test sinh từ issue làm bộ lọc giúp tăng gấp đôi precision, cơ sở cho CI gate.
7. **[V17] [Are "Solved Issues" in SWE-bench Really Solved Correctly?](https://arxiv.org/abs/2503.15223)**: cho thấy "pass test" chưa đủ (29,6% patch hành xử khác patch gốc) và đưa ra kỹ thuật PatchDiff có thể tái dùng.
8. **[V10] [LiSSA](https://ieeexplore.ieee.org/document/11029919)**: trạng thái hiện tại của truy vết bằng LLM/RAG, thành phần nối ticket↔mã.
9. **[V13] [TraceDev](https://arxiv.org/abs/2607.18886)**: mẫu đưa đồ thị truy vết vào pipeline đa tác tử, rất gần thiết kế mục tiêu của luận văn.
10. **[V41] [WhatsCode](https://arxiv.org/abs/2512.05314)**: bằng chứng công nghiệp rằng giá trị đến từ cộng tác người–AI và quản trị tổ chức chứ không phải tự động hóa hoàn toàn. Đây là cầu nối sang góc nhìn MIS.

---

## Khoảng trống nghiên cứu (khả thi cho luận văn Thạc sĩ HTTT, không cần GPU lớn)

**KT1. Truy vết đầu-cuối ticket → acceptance criteria → test → diff → PR cho thay đổi do tác tử tạo.**
- *Căn cứ:* Các tác tử giải issue ([V01]–[V04], [V07]) không sinh ra trace link tường minh. Các nghiên cứu TLR ([V10], [V11]) khôi phục liên kết sau sự kiện trên dataset tĩnh, và [V10] tự thừa nhận chưa đủ cho thực tế. [V13] có đồ thị truy vết nhưng chỉ cho sinh mã mới từ use case. [V14] chỉ là đề xuất chưa đánh giá. [V19] cho thấy chỉ khi truy vết được quyết định của tác tử mới phát hiện được patch overfit.
- *Câu hỏi nghiên cứu:* Một pipeline tác tử ghi trace link "by construction" (ticket ID ↔ tiêu chí chấp nhận ↔ test sinh ra ↔ hunk trong diff) có cải thiện độ chính xác truy vết (so với LiSSA làm hậu kiểm) và khả năng reviewer phát hiện thay đổi sai hay không?
- *Phương pháp:* Design Science Research (Hevner et al., MISQ 2004). Xây prototype trên Agentless hoặc OpenHands, đánh giá trên một tập con 50–100 instance của SWE-bench Verified (gắn nhãn trace thủ công), kèm thí nghiệm nhỏ với reviewer.
- *Dựa trên:* [V03], [V04], [V07], [V10], [V13], [V14], [V19].

**KT2. Cổng kiểm chứng tổ hợp, rẻ, để chặn patch "plausible but wrong".**
- *Căn cứ:* [V17] cho thấy 29,6% patch plausible hành xử khác patch gốc. [V32] cho thấy 31,08% patch "pass" là do test yếu. [V15] và [V16] cho thấy test sinh từ issue lọc patch hiệu quả, nhưng [V18] cảnh báo test tự sinh cũng gây overfitting. Chưa có nghiên cứu nào so sánh chi phí và lợi ích của *tổ hợp* các cổng (test từ issue + differential testing + SAST + LLM-as-judge như [V40]) trong vai trò một CI gate.
- *Câu hỏi nghiên cứu:* Tổ hợp cổng nào giảm nhiều nhất tỉ lệ "false resolved" với chi phí (token, thời gian CI) chấp nhận được?
- *Phương pháp:* Thực nghiệm offline trên patch đã công bố của các hệ thống trên leaderboard SWE-bench Verified. Cách này không cần chạy lại tác tử nên tốn rất ít tài nguyên. Đo precision/recall của từng cổng so với nhãn đúng/sai của [V17].
- *Dựa trên:* [V15], [V16], [V17], [V18], [V32], [V40], [V22].

**KT3. Hiện thực hóa và đánh giá "Merge-Readiness Pack" cho reviewer.**
- *Căn cứ:* [V37] đề xuất MRP nhưng chỉ ở mức tầm nhìn. [V20] cho thấy phần lớn PR của AI không được con người review. [V21] cho thấy review chỉ bởi CRA làm tỉ lệ merge thấp hơn. [V45] cho thấy 45,1% PR được merge vẫn phải chỉnh sửa. [V07] cho thấy giải thích kèm patch tạo "tín hiệu" chấp nhận.
- *Câu hỏi nghiên cứu:* Những thành phần bằng chứng nào (trace link, kết quả test từ ticket, diff hành vi, đánh giá rủi ro, lập luận của tác tử) giúp reviewer quyết định đúng hơn và nhanh hơn?
- *Phương pháp:* DSR kết hợp thí nghiệm có kiểm soát với 20–30 kỹ sư Việt Nam, review một bộ PR có cài sẵn một số patch sai lấy từ dữ liệu [V17]. So sánh có MRP và không có MRP.
- *Dựa trên:* [V37], [V07], [V17], [V20], [V21], [V45], [V41].

**KT4. Lớp chính sách (policy-as-code) và nhật ký kiểm toán chuyên cho SDLC.**
- *Căn cứ:* [V25] đưa ra policy ký hiệu cho tool call nhưng đánh giá trên tác tử tổng quát. [V24] cho thấy file rule và tài liệu dev có thể bị lợi dụng chiếm quyền với tỉ lệ thành công tới 84%. [V26] nêu khung định danh, giám sát và log nhưng không cụ thể cho SDLC. [V23] có sẵn kịch bản rủi ro. Còn thiếu cách ánh xạ quy định của tổ chức (thư mục được bảo vệ, không lộ secret, bắt buộc test trước khi commit, phân tách nhiệm vụ) thành policy cho tác tử, và cách đo đánh đổi giữa an toàn và hiệu quả.
- *Câu hỏi nghiên cứu:* Một lớp policy suy ra từ quy trình SDLC của tổ chức chặn được bao nhiêu hành động rủi ro (kịch bản RedCode/AIShellJack), và làm giảm tỉ lệ giải issue bao nhiêu trên một tập con SWE-bench?
- *Phương pháp:* Prototype (chèn vào OpenHands) cộng thực nghiệm. Dùng mô hình qua API nên không cần GPU.
- *Dựa trên:* [V04], [V23], [V24], [V25], [V26].

**KT5. Provenance ở mức quy trình cho thay đổi do AI tạo, phục vụ kiểm toán và tuân thủ.**
- *Căn cứ:* Watermark ở mức token ([V27]) gặp khó vì mã có entropy thấp. Các nghiên cứu về PR của tác tử ([V44], [V46]) nhận diện PR AI chủ yếu qua tài khoản/nhãn của tác tử. [V41] nhấn mạnh môi trường "compliance-relevant" nhưng không đề xuất lược đồ provenance chung. [V26] gợi ý activity logging.
- *Câu hỏi nghiên cứu:* Lược đồ metadata provenance nào (ticket, tác tử/mô hình/phiên bản, prompt, kết quả các cổng, người phê duyệt) đáp ứng yêu cầu kiểm toán nội bộ, ví dụ ở ngân hàng hoặc fintech Việt Nam, với chi phí vận hành chấp nhận được?
- *Phương pháp:* DSR, thiết kế artefact (schema + công cụ gắn vào commit/PR), đánh giá bằng phỏng vấn chuyên gia (kiểm toán CNTT, trưởng nhóm kỹ thuật) và thử nghiệm trên một repo mẫu.
- *Dựa trên:* [V26], [V27], [V41], [V44], [V46].

**KT6. Chất lượng ticket là biến quyết định thành công của tác tử. Bước "làm rõ yêu cầu" trước khi code.**
- *Căn cứ:* [V03] phát hiện nhiều issue có mô tả thiếu hoặc gây hiểu nhầm. [V32] cho thấy lời giải bị lộ sẵn trong issue. [V46] liệt kê "tính năng không mong muốn" và "misalignment" là lý do bị từ chối. [V12] cho thấy quản lý yêu cầu là pha ít được nghiên cứu (6,8%). [V13] có Requirement Refiner nhưng không đo riêng tác động của nó.
- *Câu hỏi nghiên cứu:* Một bước tinh chỉnh ticket bằng LLM (phát hiện mơ hồ, sinh acceptance criteria, đặt câu hỏi làm rõ) có tăng tỉ lệ giải đúng hoặc giảm thay đổi ngoài phạm vi hay không?
- *Phương pháp:* Thực nghiệm A/B trên tập con SWE-bench Verified (issue gốc so với issue đã tinh chỉnh) và case study trên ticket Jira thật của một doanh nghiệp (đã ẩn danh).
- *Dựa trên:* [V03], [V12], [V13], [V16], [V32], [V46].

**KT7. Benchmark nội bộ chống nhiễm dữ liệu để đánh giá pipeline trong bối cảnh doanh nghiệp.**
- *Căn cứ:* [V33] và [V32] chỉ ra hiện tượng ghi nhớ hoặc nhiễm dữ liệu. [V30] và [V31] cho thấy hiệu năng giảm mạnh trên dữ liệu mới hoặc doanh nghiệp. [V39] cho thấy phân phối bug nội bộ khác SWE-bench.
- *Câu hỏi nghiên cứu:* Hiệu năng và độ chính xác ngữ nghĩa của pipeline có quản trị trên 30–50 ticket nội bộ (phát sinh sau ngày cắt dữ liệu của mô hình) khác SWE-bench Verified như thế nào, và lỗi tập trung ở đâu?
- *Phương pháp:* Case study tại một công ty phần mềm Việt Nam. Dựng mini-benchmark theo quy trình của [V30] (Docker và test cho từng ticket), phân tích lỗi định tính theo taxonomy của [V46].
- *Dựa trên:* [V28], [V30], [V31], [V33], [V39], [V46].

**KT8. Chấp nhận và quản trị quy trình tác tử tại doanh nghiệp phần mềm Việt Nam (góc nhìn MIS).**
- *Căn cứ:* [V47] cho thấy lập trình viên giàu kinh nghiệm chậm hơn 19% nhưng vẫn tin mình nhanh hơn. [V41] khẳng định yếu tố tổ chức quan trọng ngang kỹ thuật. Các nghiên cứu thực nghiệm hiện có ([V44]–[V46]) chủ yếu dựa trên GitHub mã nguồn mở. Trong số tài liệu đã rà soát, chưa thấy nghiên cứu nào về bối cảnh outsourcing hoặc doanh nghiệp ở Việt Nam/Đông Nam Á.
- *Câu hỏi nghiên cứu:* Những yếu tố nào (theo khung TOE/UTAUT, cộng với niềm tin, nhận thức rủi ro, yêu cầu tuân thủ) ảnh hưởng đến ý định áp dụng quy trình tác tử *có quản trị* tại doanh nghiệp phần mềm Việt Nam? Năng suất cảm nhận và năng suất đo được lệch nhau ra sao?
- *Phương pháp:* Khảo sát định lượng (PLS-SEM, khoảng 150–250 người) kết hợp phỏng vấn bán cấu trúc. Có thể ghép với một pilot đo log của KT7.
- *Dựa trên:* [V36], [V41], [V44], [V45], [V47].

*Gợi ý kết hợp:* Một luận văn mạnh có thể lấy **KT1 + KT2** làm lõi artefact (DSR), dùng **KT3** làm đánh giá với người dùng, và **KT4/KT5** làm phần quản trị. Đánh giá thực hiện trên tập con SWE-bench Verified cộng với một case study nội bộ (**KT7**).

---

## Hội nghị và tạp chí nên theo dõi

- **SE hàng đầu:** ICSE (đặc biệt track SEIP cho bài công nghiệp: [V33], [V39], [V41]), FSE (kèm Industry track: [V43]), ASE, ISSTA ([V02], [V13], [V19]). Tạp chí: ACM TOSEM ([V22], [V34]–[V36], [V45]), IEEE TSE ([V40]), Empirical Software Engineering.
- **Khai phá dữ liệu PR/repo của tác tử:** MSR (2026 có nhiều bài về AIDev, ví dụ [V21], [V46]) và EASE ([V20]).
- **Kỹ nghệ yêu cầu và truy vết:** IEEE RE Conference và workshop ([V09]), REFSQ, Requirements Engineering Journal.
- **ML/NLP (nơi ra đời nhiều benchmark và tác tử):** ICLR, NeurIPS (kèm Datasets & Benchmarks), ICML, ACL. Benchmark mới thường xuất hiện trên arXiv trước.
- **An ninh và quản trị AI:** IEEE S&P, ACM CCS, USENIX Security, ACM FAccT ([V26]).
- **Venue mới chuyên về AI-native SE:** ACM International Conference on AI-Powered Software (AIware), các workshop về agentic SE (ví dụ workshop "Agentic Software Engineering (SE 3.0)" tại KDD 2026).
- **MIS (để định vị đóng góp HTTT):** ICIS, PACIS (châu Á – Thái Bình Dương, phù hợp bối cảnh Việt Nam), HICSS. Tạp chí: MIS Quarterly, Information Systems Research, JMIS. Về phương pháp DSR: Hevner, March, Park & Ram, ["Design Science in Information Systems Research"](https://doi.org/10.2307/25148625), MIS Quarterly, 2004.
