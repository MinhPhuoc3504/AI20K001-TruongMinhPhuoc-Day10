# Báo Cáo Cá Nhân — Lab Day 10: Data Pipeline & Observability

**Họ và tên:** Trương Minh Phước  
**Mã số sinh viên:** 2A202600330  
**Vai trò:** Full-stack Data Pipeline (Nhóm 36)  
**Ngày nộp:** 15/04/2026  
**Độ dài:** ~550 từ

---

## 1. Tôi phụ trách phần nào? (80–120 từ)

Trong dự án lab Day 10 lần này, vì là nhóm một thành viên, tôi chịu trách nhiệm xây dựng toàn bộ hệ thống từ đầu đến cuối. 

**File / module:**
- `etl_pipeline.py`: Thiết kế kiến trúc orchestrator quản lý flow dữ liệu.
- `transform/cleaning_rules.py`: Triển khai 3 quy tắc làm sạch mở rộng và logic sửa lỗi chính sách hoàn tiền.
- `quality/expectations.py`: Tích hợp bộ thư viện kiểm tra chất lượng dữ liệu.
- `docs/`: Biên soạn Data Contract và Runbook để đảm bảo tính quan sát của hệ thống.

**Bằng chứng (commit / comment trong code):**
Tôi đã thực hiện các lượt chạy kiểm chứng thành công với các `run_id`: `sprint1`, `sprint2`, `inject-bad`, và `sprint4-final`. Mọi log và manifest đều được lưu vết trong thư mục `artifacts/`.

---

## 2. Một quyết định kỹ thuật (100–150 từ)

Quyết định kỹ thuật quan trọng nhất của tôi là việc triển khai **Idempotency dựa trên Stable Chunk ID**. Thay vì sử dụng UUID ngẫu nhiên cho mỗi lần nạp, tôi đã băm (hash) nội dung của văn bản kết hợp với mã tài liệu (`doc_id`).

**Lý do:** Điều này cho phép tôi chạy lại pipeline nhiều lần mà không làm trùng lặp (duplicate) dữ liệu trong Vector DB (ChromaDB). Khi phát hiện chính sách hoàn tiền cũ (14 ngày), tôi có thể chạy lệnh "fix" và ChromaDB sẽ tự động thực hiện lệnh `upsert` để thay thế đoạn văn bản lỗi bằng văn bản mới (7 ngày) ngay tại ID đó. Điều này giúp tiết kiệm tài nguyên và đảm bảo tính nhất quán (Consistency) của dữ liệu tri thức.

---

## 3. Một lỗi hoặc anomaly đã xử lý (100–150 từ)

Trong Sprint 3, tôi đã đối mặt với lỗi **Business Hallucination** do dữ liệu bẩn lọt vào. 

**Triệu chứng:** Agent truy xuất song song cả hai chính sách hoàn tiền (7 ngày và 14 ngày), dẫn đến câu trả lời thiếu tin cậy. 
**Phát hiện:** Tôi đã sử dụng script `eval_retrieval.py` và phát hiện cột `hits_forbidden` trả về giá trị **yes** cho lượt chạy `inject-bad`. 
**Xử lý:** Tôi đã kích hoạt Expectation `refund_no_stale_14d_window` với mức độ nghiêm trọng là `halt`. Khi chạy lại pipeline chuẩn, hệ thống đã bắt đúng lỗi này và thực hiện rule làm sạch `apply_refund_window_fix`. Sau khi nạp lại, kết quả đánh giá đã chứng minh `hits_forbidden` chuyển về **no**, xóa bỏ hoàn toàn rủi ro trả lời sai.

---

## 4. Bằng chứng trước / sau (80–120 từ)

**Run ID:** `sprint4-final`  
Trích xuất từ file `artifacts/eval/final_cleanup.csv`:

| query_id | contains_expected | hits_forbidden | comment |
|----------|-------------------|----------------|---------|
| q_refund_window | yes | **no** | Đã loại bỏ 14 ngày |
| q_leave_version | yes | **no** | Đã chọn version 2026 |

Bằng chứng này cho thấy Agent của tôi chỉ "nhìn thấy" dữ liệu đúng theo Data Contract đã cam kết.

---

## 5. Cải tiến tiếp theo (40–80 từ)

Nếu có thêm 2 giờ, tôi sẽ tích hợp **Human-in-the-loop** vào thư mục Quarantine. Thay vì chỉ lưu file CSV, tôi sẽ tạo một giao diện Web đơn giản cho phép Data Owner click "Approve" hoặc "Edit" ngay trên các bản ghi lỗi để tự động đẩy ngược lại vào Ingestion queue cho vòng chạy tiếp theo.
