Báo cáo ngắn — Kết quả trên instance `m7i-flex.large`:
- Thời gian huấn luyện (LightGBM benchmark): ~1.43s; mô hình đạt AUC-ROC = 0.9081, cho thấy chất lượng dự đoán tốt trên CPU.
- Tốc độ inference: latency ~0.2203 ms/record và throughput ~1,726,761 rows/s (với batch lớn) — đáp ứng tốt nhiều kịch bản không cần GPU.
- So sánh với GPU: GPU (ví dụ T4) thường nhanh hơn rõ rệt cho việc huấn luyện và inference các mô hình deep‑learning lớn; nhưng với thuật toán tree‑based như LightGBM, CPU hiệu quả và cạnh tranh về thời gian/chi phí.
- Lý do phải dùng CPU: tài khoản AWS chưa có quota GPU hoặc yêu cầu tăng quota đang bị trì hoãn/từ chối, nên chuyển sang `m7i-flex.large` để đảm bảo có kết quả kịp thời và ổn định.
- Kết luận: phương án CPU là lựa chọn thực tế cho bài lab (chi phí, triển khai nhanh, AUC và throughput tốt cho LightGBM); nếu cần xử lý mô hình neural lớn hoặc inference thời gian thực ở quy mô lớn thì nên chuyển sang GPU khi quota cho phép.

---
### Giải thích chi tiết lựa chọn hạ tầng và kết quả chi phí
#### 1. Tại sao không sử dụng GPU (`g4dn.xlarge`) hoặc máy lớn (`r5.2xlarge`)?
* **Hạn chế về Quota:** Tài khoản AWS mới mặc định bị giới hạn mức định mức (Service Quotas) cho các dòng máy GPU On-Demand bằng **0**. Việc yêu cầu tăng quota thường mất thời gian phê duyệt, không khả thi cho tiến độ bài lab.
* **Hạn chế về quyền truy cập:** Các dòng máy lớn như `r5.2xlarge` thường không nằm trong danh mục ưu tiên khởi tạo cho tài khoản trải nghiệm, dễ dẫn đến lỗi "Instance Limit Exceeded".

#### 2. Lý do lựa chọn `m7i-flex.large`
* **Khả năng đáp ứng tức thì:** Đây là dòng máy thuộc phân khúc General Purpose thế hệ mới, có sẵn quota và không yêu cầu thủ tục phê duyệt đặc biệt.
* **Tối ưu cho thuật toán Tree-based:** Với 2 vCPU và **8GB RAM**, máy đảm bảo xử lý mượt mà bộ dữ liệu Credit Card  bằng Pandas mà không bị tràn bộ nhớ. 

#### 3. Giải trình về việc chưa có dữ liệu chi phí (AWS Billing)
Mặc dù hệ thống đã vận hành được hơn 1 giờ, bảng điều khiển hiển thị **"No data"** là do đặc thù kỹ thuật của AWS:
* **Quy trình khởi tạo (24h Setup):** Theo thông báo từ AWS Console: *"Since this is your first visit... please check back in 24 hours"*. Hệ thống cần thời gian để thiết lập đường ống dữ liệu cho lần đầu truy cập.
* **Độ trễ xử lý (Billing Latency):** Dữ liệu từ NAT Gateway, ALB và EC2 được xử lý theo lô. Tần suất cập nhật hóa đơn thường từ **8 - 12 tiếng**.
---
