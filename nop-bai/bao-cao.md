# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Phùng Thanh An |
| MSSV | 2A202603006 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/Mashall-Anie/K4-L3-DAY21-PhungThanhAn-2A202603006-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Lần chạy 3 có F1 cao nhất, 0.7149, và vượt ngưỡng Quality Gate 0.65. Lần chạy 1 có Accuracy cao nhất (0.8780), nhưng F1 của lần chạy 3 cao hơn; vì vậy chọn mô hình theo Accuracy có thể không chọn được mô hình nhận diện lớp thu nhập cao tốt nhất. Lần chạy 2 chỉ đạt F1 0.6051 nên không đạt ngưỡng. So sánh lần 1 và 3, F1 tăng 0.0040 trong khi Accuracy giảm 0.0040. Do lần 2 và 3 thay đổi đồng thời nhiều tham số, chưa thể kết luận riêng tác động của từng tham số; nói chung learning rate thấp thường cần nhiều cây hơn để bù mức cập nhật nhỏ.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Bộ dữ liệu sau khi làm sạch có khoảng 24,8% mẫu thuộc lớp dương (thu nhập trên 50.000 USD) và 75,2% thuộc lớp âm. Nếu mô hình luôn dự đoán “thu nhập thấp”, accuracy vẫn đạt khoảng 75,2% dù bỏ sót toàn bộ người có thu nhập cao; vì vậy accuracy có thể gây hiểu lầm. F1-score của lớp dương là trung bình điều hòa giữa precision và recall, thể hiện cả độ tin cậy của dự đoán dương lẫn khả năng tìm ra các mẫu dương. Quality Gate dùng F1-score cho `target=1` để đo đúng lớp cần phát hiện. Không dùng `average="macro"` hoặc `average="weighted"` vì bài lab yêu cầu đánh giá riêng lớp dương, không gộp điểm của hai lớp thành một chỉ số tổng hợp.


---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết


| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| `gsutil` không tạo được bucket | Python/OpenSSL đi kèm Cloud SDK gây lỗi `GEN_EMAIL` | Dùng `gcloud storage buckets create` thay cho `gsutil`. |
| Không tạo được khóa JSON cho service account | Project đầu chịu policy `iam.disableServiceAccountKeyCreation` từ organization | Chuyển sang project độc lập `labvinday12`, kiểm tra policy hiệu lực rồi tạo key tại đó. |
| DVC cần đường dẫn credential trên GitHub Actions | Cấu hình credential local không được commit và runner không có file key trên laptop | Lưu JSON trong GitHub Secret, ghi ra `/tmp/sa-key.json` trên runner và đặt `credentialpath` cho DVC tại đó. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)


| | f1_score | accuracy |
|---|---:|---:|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:**  Trên cùng tập holdout, sau khi bổ sung 22.361 mẫu huấn luyện, F1 tăng 0.0205 và accuracy tăng 0.0080. Kết quả cho thấy mô hình cải thiện khả năng nhận diện lớp thu nhập cao; đồng thời, commit dữ liệu đã tự kích hoạt thành công cả bốn job và triển khai model mới.

---

