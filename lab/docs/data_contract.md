# Data contract — Lab Day 10

**Nhóm:** Nhóm 36  
**Owner:** Trương Minh Phước  

---

## 1. Nguồn dữ liệu (source map)

| Nguồn | Phương thức ingest | Failure mode chính | Metric / alert |
|-------|-------------------|-------------------|----------------|
| Policy Export CSV | Batch (etl_pipeline.py) | Sai format ngày, duplicate, version cũ | Freshness SLA > 24h, % Quarantine > 10% |

---

## 2. Schema cleaned

| Cột | Kiểu | Bắt buộc | Ghi chú |
|-----|------|----------|---------|
| chunk_id | string | Có | Hash của content + doc_id (Unique) |
| doc_id | string | Có | Mã tài liệu (VD: refund-policy) |
| chunk_text | string | Có | Nội dung tri thức đã qua cleaning |
| effective_date | date | Có | Ngày hiệu lực (ISO YYYY-MM-DD) |
| exported_at | datetime | Có | Thời điểm trích xuất dữ liệu |

---

## 3. Quy tắc quarantine vs drop

- **Quarantine:** Bản ghi thiếu ngày (`missing_effective_date`), lỗi PII nhạy cảm chưa được mask, hoặc nội dung dưới 8 ký tự. Dữ liệu này được đẩy vào `artifacts/quarantine/`.
- **Drop:** Các bản ghi trùng lặp hoàn toàn nội dung (`duplicate_chunk_text`) sẽ bị loại bỏ để tránh gây loãng Vector Space.
- **Approval:** Trương Minh Phước (Team 36) sẽ review quarantine định kỳ để cập nhật lại nguồn.

---

## 4. Phiên bản & canonical

- **Source of truth:** Tệp `policy_refund_v4.txt` là căn cứ cao nhất.
- **Rule of versioning:** Mọi bản ghi có nội dung "14 ngày hoàn tiền" được coi là dữ liệu cũ (Stale) và phải được pipeline tự động chuyển đổi thành "7 ngày" trước khi Serving.
