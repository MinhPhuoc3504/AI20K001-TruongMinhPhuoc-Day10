# Quality report — Lab Day 10 (Nhóm 36)

**run_id:** `sprint4-final`  
**Ngày:** 15/04/2026
**Người kiểm soát:** Trương Minh Phước

---

## 1. Tóm tắt số liệu

| Chỉ số | Trước | Sau | Ghi chú |
|--------|-------|-----|---------|
| raw_records | 10 | 10 | |
| cleaned_records | 6 | 6 | |
| quarantine_records | 4 | 4 | |
| Expectation halt? | FAIL (3b) | PASS | Đã fix stale data |

---

## 2. Before / after retrieval (bắt buộc)

Dữ liệu chứng cứ được lưu tại `artifacts/eval/final_cleanup.csv`.

**Câu hỏi then chốt:** refund window (`q_refund_window`)  
**Trước:** `hits_forbidden=yes` (Agent trả lời sai: 14 ngày).  
**Sau:** `hits_forbidden=no` (Agent trả lời đúng: 7 ngày).

**Merit (khuyến nghị):** versioning HR — `q_leave_version` (`contains_expected`, `hits_forbidden`, cột `top1_doc_expected`)

**Trước:** `hits_forbidden=yes`.  
**Sau:** `top1_doc_expected=yes`. Agent đã trích dẫn đúng tài khoản `hr_policy_v2_2026`.

---

## 3. Freshness & monitor

Kết quả `freshness_check`: **FAIL**.
**Giải thích:** SLA tối đa 24 giờ. dữ liệu thô có timestamp cũ từ 5 ngày trước (>100 giờ). Đây là cảnh báo cần thực hiện ingestion đợt mới.

---

## 4. Corruption inject (Sprint 3)

Chúng tôi đã cố ý tắt logic làm sạch chính sách (`--no-refund-fix`) để kiểm chứng độ nhạy của hệ thống giám sát. Kết quả cho thấy Expectation đã báo lỗi và file Evaluation đã xác nhận sự hiện diện của dữ liệu bẩn (`hits_forbidden=yes`).

---

## 5. Hạn chế & việc chưa làm

- Cần thêm rule kiểm tra encoding văn bản (BOM) tự động.
- Cần mở rộng quy mô bộ câu hỏi đánh giá (Evaluation Dataset).
