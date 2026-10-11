# AISDLC_RnD_Journey

Đây là nhật ký **hành trình R&D về AI-native SDLC**: tìm hiểu cách các công ty và giới nghiên cứu đưa AI agent vào toàn bộ vòng đời phát triển phần mềm, không chỉ để viết code nhanh hơn. Mục tiêu dài hạn là chọn và xây dựng đề tài **luận văn thạc sĩ Hệ thống thông tin quản lý (MIS)** theo hướng: *"Don't use AI merely to write code faster; build a governed agentic system that turns business tickets into verified, traceable software changes."* Repo gồm tài liệu tham khảo (case study, playbook, danh mục paper) và một lộ trình đọc theo nhịp cho người mới bắt đầu.

## Cấu trúc thư mục

```
AISDLC_RnD_Journey/
├── README.md
├── reading/                                   # Lộ trình đọc theo nhịp (đọc theo số thứ tự)
│   ├── README.md                              # Kế hoạch 16 bài và trạng thái
│   └── 00-ban-do-va-cach-doc.md               # Bài đầu tiên
└── research/                                  # Tài liệu tham khảo
    ├── 01-case-studies-cong-ty.md             # Case study các công ty làm AI-native SDLC
    ├── 02-shopify-mental-model-co-ban.md      # Giải thích từ số 0 ba bài Shopify Engineering
    ├── 03-shopify-river-dispatch-deepdive.md  # Đào sâu Shopify River / Aquifer / Dispatch
    ├── 04-playbook-ai-native-sdlc.md          # Playbook tổng hợp từ các case
    └── 05-danh-muc-47-paper-luan-van.md       # 47 paper đã kiểm chứng, top must-read, khoảng trống đề tài
```

## Cách đọc

1. Bắt đầu từ [`reading/00-ban-do-va-cach-doc.md`](reading/00-ban-do-va-cach-doc.md), rồi đọc tiếp theo số thứ tự trong [`reading/`](reading/README.md).
2. Dùng các file trong `research/` làm tài liệu tra cứu:
   - Muốn hiểu bức tranh thực tế ở doanh nghiệp: [`research/01`](research/01-case-studies-cong-ty.md), sau đó [`02`](research/02-shopify-mental-model-co-ban.md) (bản dễ) trước [`03`](research/03-shopify-river-dispatch-deepdive.md) (bản sâu).
   - Muốn áp dụng cho team: [`research/04`](research/04-playbook-ai-native-sdlc.md).
   - Muốn tra paper theo mã [Vxx]: [`research/05`](research/05-danh-muc-47-paper-luan-van.md).

> Ghi chú: playbook (`research/04`) có nhắc tên file cũ. Tương ứng: `ai-native-sdlc-case-studies.md` = [`research/01`](research/01-case-studies-cong-ty.md), `shopify-river-dispatch-deepdive.md` = [`research/03`](research/03-shopify-river-dispatch-deepdive.md), `ai-native-sdlc-playbook.md` = [`research/04`](research/04-playbook-ai-native-sdlc.md).

## Trạng thái lộ trình đọc

| # | Chặng | File | Paper | Trạng thái |
|---|---|---|---|---|
| 00 | 0 · Bản đồ | [`00-ban-do-va-cach-doc.md`](reading/00-ban-do-va-cach-doc.md) | [V34], [V35] | ✅ Sẵn sàng |
| 01 | 1 · Vì sao cần AI-native | `01-se-3-0-va-agentic-se.md` | [V36], [V37] | ⏳ Sắp tới |
| 02 | 1 · Vì sao cần AI-native | `02-ai-khong-chi-de-code-nhanh.md` | [V47], [V38] | ⏳ Sắp tới |
| 03 | 2 · Bài toán gốc issue → patch | `03-swe-bench.md` | [V28] | ⏳ Sắp tới |
| 04 | 2 · Bài toán gốc issue → patch | `04-swe-agent-va-openhands.md` | [V01], [V04] | ⏳ Sắp tới |
| 05 | 2 · Bài toán gốc issue → patch | `05-agentless-va-autocoderover.md` | [V03], [V02] | ⏳ Sắp tới |
| 06 | 2 · Bài toán gốc issue → patch | `06-da-tac-tu-metagpt-chatdev.md` | [V05], [V06] | ⏳ Sắp tới |
| 07 | 3 · Điểm số có đáng tin | `07-phe-binh-benchmark.md` | [V29]–[V33] | ⏳ Sắp tới |
| 08 | 4 · Kiểm chứng | `08-test-lam-trong-tai.md` | [V15], [V16], [V18] | ⏳ Sắp tới |
| 09 | 4 · Kiểm chứng | `09-pass-test-chua-phai-dung.md` | [V17], [V19], [V07] | ⏳ Sắp tới |
| 10 | 5 · Truy vết | `10-truy-vet-la-gi.md` | [V08], [V09], [V12] | ⏳ Sắp tới |
| 11 | 5 · Truy vết | `11-truy-vet-bang-llm.md` | [V10], [V11], [V13], [V14] | ⏳ Sắp tới |
| 12 | 6 · Con người trong vòng lặp | `12-nguoi-review-pr-cua-ai.md` | [V20], [V21], [V44]–[V46] | ⏳ Sắp tới |
| 13 | 7 · Governance và an ninh | `13-guardrail-audit-provenance.md` | [V22]–[V27] | ⏳ Sắp tới |
| 14 | 8 · Doanh nghiệp làm thật | `14-case-cong-nghiep.md` | [V39]–[V43] | ⏳ Sắp tới |
| 15 | 9 · Chọn đề tài | `15-khoang-trong-va-de-tai.md` | 8 khoảng trống, ghép KT1+KT2+KT3, phương pháp DSR | ⏳ Sắp tới |

## Về nguồn

Tất cả paper trong danh mục đều đã được kiểm chứng có link (tiêu đề, tác giả, năm, nơi công bố). Bài chưa qua bình duyệt được ghi rõ là **arXiv preprint**. Số liệu trong các bài đọc chỉ lấy từ nguồn đã trích dẫn.
