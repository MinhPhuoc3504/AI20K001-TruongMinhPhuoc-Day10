# Báo Cáo Nhóm — Lab Day 10: Data Pipeline & Data Observability

**Tên nhóm:** Nhóm 36  
**Thành viên:**
| Tên | Vai trò (Day 10) | Email |
|-----|------------------|-------|
| Trương Minh Phước | Pipeline & Quality Owner | minhphuoc3504@gmail.com |

**Ngày nộp:** 15/04/2026  
**Độ dài khuyến nghị:** 600–1000 từ

---

## 1. Pipeline tổng quan (150–200 từ)

Nhóm 36 xây dựng pipeline này nhằm giải quyết bài toán "Sạch hóa tri thức" cho Agent CS & IT Helpdesk. Nguồn dữ liệu thô từ `policy_export_dirty.csv` chứa nhiều dữ liệu rác, trùng lặp và đặc biệt là các chính sách cũ (stale) về hoàn tiền.

**Tóm tắt luồng:**
Hệ thống sử dụng `etl_pipeline.py` để điều phối: Ingest (gán run_id) -> Transform (xử lý PII, chuẩn hóa văn bản, sửa lỗi chính sách) -> Validate (Kiểm tra Quality Gates) -> Embed (Upsert vào ChromaDB sử dụng stable hashing). 

**Lệnh chạy một dòng (copy từ README thực tế của nhóm):**
```bash
python etl_pipeline.py run --run-id sprint4-final
```

---

## 2. Cleaning & expectation (150–200 từ)

Chúng tôi đã bổ sung các quy tắc nghiêm ngặt để đảm bảo dữ liệu không chỉ đúng định dạng mà còn đúng về mặt nghiệp vụ kinh doanh.

### 2a. Bảng metric_impact (bắt buộc — chống trivial)

| Rule / Expectation mới (tên ngắn) | Trước (số liệu) | Sau / khi inject (số liệu) | Chứng cứ (log / CSV / commit) |
|-----------------------------------|------------------|-----------------------------|-------------------------------|
| `refund_no_stale_14d` | 14 ngày (Stale) | 7 ngày (Fixed) | `final_cleanup.csv` |
| `PII Masking` | Chứa mật khẩu | Đã che ***** | `run_sprint2.log` |
| `multi_source_check` | N/A | OK (4 unique docs) | `manifest_sprint2.json` |
| `avg_chunk_length_check` | N/A | OK (avg=94.3 chars) | `manifest_sprint2.json` |

**Rule chính (baseline + mở rộng):**
- **PII Masking:** Tự động phát hiện và che giấu các từ khóa nhạy cảm như "mật khẩu", "password".
- **HR Policy Tagging:** Gắn tag phiên bản 2026 cho các tài liệu nhân sự để Agent ưu tiên truy xuất.
- **Refund Policy Fix:** Logic tự động chuyển 14 ngày về 7 ngày để đồng bộ với website.

---

## 3. Before / after ảnh hưởng retrieval hoặc agent (200–250 từ)

**Kịch bản inject (Sprint 3):**
Chúng tôi mô phỏng một sự cố vận hành bằng cách nạp dữ liệu từ file "dirty" mà không qua bộ lọc sửa lỗi chính sách. Điều này khiến database tồn tại cả thông tin hoàn tiền 7 ngày và 14 ngày.

**Kết quả định lượng:**
- **Trước khi fix:** Agent khi được hỏi về hoàn tiền đã bị nhầm lẫn, trả lời có thể lên tới 14 ngày (hits_forbidden=yes). Điều này vi phạm nghiêm trọng SLA CS.
- **Sau khi fix:** Khi kích hoạt quy tắc `apply_refund_window_fix`, hệ thống đã thực hiện **Upsert**. File `after_inject_bad.csv` báo cáo forbidden đạt 0%. Agent giờ đây chỉ truy xuất được thông tin 7 ngày duy nhất, đảm bảo tính nhất quán của hệ thống.

---

## 4. Freshness & monitoring (100–150 từ)

Hệ thống đặt ngưỡng SLA Freshness là **24 giờ**. Với dữ liệu test được export vào ngày 10/04/2026, pipeline đã bắn cảnh báo **FAIL (Stale Data)** vì độ trễ hiện tại đã vượt quá 100 giờ. Điều này giúp đội ngũ Ops biết rằng cần thực hiện một đợt nạp dữ liệu mới từ hệ thống ERP/Wiki.

---

## 5. Liên hệ Day 09 (50–100 từ)

Toàn bộ dữ liệu sạch sau Day 10 sẽ được phục vụ cho Supervisor và Policy Worker của Day 09 thông qua việc swapping collection. Việc có một pipeline observability giúp các agent ở Day 09 tránh được việc trả lời sai do dữ liệu nguồn bẩn, từ đó giảm tỷ lệ "Refuse to Answer" và tăng độ hài lòng của khách hàng.

---

## 6. Kết quả Tự rà soát kỹ thuật (Self-Peer Review)

Dựa trên checklist trao đổi giữa các nhóm (Slide 42), tôi đã tự kiểm tra các rủi ro:
1. **Rerun 2 lần có duplicate không?** -> Không. Đã kiểm tra qua `chunk_id` hashing. Database giữ nguyên số lượng vector sau khi rerun bản sạch.
2. **Freshness đo ở bước nào?** -> Đo ngay sau bước Ingest dựa trên cột `exported_at` để phản ánh đúng thực tế độ trễ của tri thức.
3. **Record bị flag đi đâu?** -> Toàn bộ được đẩy vào `artifacts/quarantine/` với lý do lỗi cụ thể để Data Owner có thể sửa nguồn.

---

## 7. Rủi ro còn lại & việc chưa làm

- Cần bổ sung cơ chế kiểm duyệt nội dung (Content Moderation) cho các chunk văn bản.
- Chưa tích hợp auto-rollback nếu lượt chạy mới có tỷ lệ Quarantine quá cao.

