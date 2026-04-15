# Runbook — Lab Day 10 (incident tối giản)

---

## Symptom

> User / agent thấy gì? (VD: trả lời “14 ngày” thay vì 7 ngày)

- Nhân viên CS báo cáo Agent trả lời sai chính sách hoàn tiền: "Khách có 14 ngày" (Thực tế quy định mới là 7 ngày).
- Khi kiểm tra file `artifacts/eval/after_inject_bad.csv`, cột `hits_forbidden` báo **yes**.

---

## Detection

> Metric nào báo? (freshness, expectation fail, eval `hits_forbidden`)

- **Expectation FAIL:** Quy tắc `refund_no_stale_14d_window` báo vi phạm trong log.
- **Evaluation:** Script `eval_retrieval.py` phát hiện nội dung cấm (stale data) lọt vào top-k kết quả.

---

## Diagnosis

| Bước | Việc làm | Kết quả mong đợi |
|------|----------|------------------|
| 1 | Kiểm tra `artifacts/manifests/*.json` | `no_refund_fix` phải là `false`. Nếu là `true` thì đây là nguyên nhân. |
| 2 | Mở `artifacts/quarantine/*.csv` | Tìm các dòng có reason `stale_refund_window`. |
| 3 | Chạy `python eval_retrieval.py` | Kiểm tra xem `hits_forbidden` đã về `no` chưa. |

---

## Mitigation

- **Rerun Pipeline:** Chạy lại `python etl_pipeline.py run` mà KHÔNG dùng cờ `--no-refund-fix`.
- **Verify Idempotency:** Kiểm tra log để đảm bảo số lượng `embed_upsert` khớp với bản ghi sạch và các vector lỗi cũ đã bị xóa.

---

## Prevention

- **Halt on Fail:** Thiết lập `severity: halt` cho rule chính sách hoàn tiền trong `contracts/data_contract.yaml`.
- **Auto-Eval:** Chạy script đánh giá retrieval sau mỗi lần deploy pipeline dữ liệu mới.
- **Owner Alert:** Gán quyền sở hữu (Owner) cho team AI-Ops để nhận thông báo ngay khi Freshness quá 24h.
