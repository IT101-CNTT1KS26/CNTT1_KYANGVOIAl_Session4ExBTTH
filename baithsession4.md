# THIẾT KẾ GIẢI PHÁP PROMPT CHAINING & CHAIN-OF-THOUGHT NÂNG CAO — HỆ THỐNG XỬ LÝ KHIẾU NẠI TECHSHOP
 
 ## PHẦN 1: THIẾT KẾ GIẢI PHÁP (SOLUTION DESIGN REPORT)

### 1. Mục tiêu và Bối cảnh bài toán

Tại hệ thống thương mại điện tử **TechShop**, bộ phận Chăm sóc khách hàng (CSKH) mỗi ngày phải tiếp nhận hàng nghìn khiếu nại phức tạp từ khách hàng (từ lỗi vận chuyển, hỏng hóc sản phẩm cho đến các trường hợp khách hàng tức giận, dùng từ mỉa mai hoặc cung cấp thiếu thông tin đơn hàng).

Nếu xử lý bằng một câu lệnh duy nhất (Single Prompt), mô hình rất dễ bị quá tải, suy luận nông cạn, bỏ sót điều kiện chính sách đổi trả hoặc bịa đặt số liệu (Hallucination). Do đó, giải pháp được thiết kế là một **Hệ thống Prompt Chaining khép kín gồm 3 bước tuần tự**, kết hợp kỹ thuật **Chain-of-Thought (CoT)** và **Cơ chế bắt lỗi dữ liệu dị biệt (Anomaly & Error Handling)** nhằm tự động hóa hoàn toàn quy trình từ khâu phân tích đến soạn thảo phản hồi.

### 2. Kiến trúc Luồng Dữ liệu (Data Handoff Flow)

```text
[Khách hàng: Khiếu nại thô & Dữ liệu nhiễu]
                       │
                       ▼
┌──────────────────────────────────────────────┐
│  BƯỚC 1: Phân tích & Suy luận (CoT Engine)   │ ➔ Trích xuất: Mã đơn, Vấn đề, Sắc thái,
│  - Dùng CoT bóc tách nguyên nhân gốc rễ     │   Đánh giá mức độ nghiêm trọng.
└──────────────────────────────────────────────┘
                       │
                       ▼ (Truyền Data Handoff Structured JSON)
┌──────────────────────────────────────────────┐
│  BƯỚC 2: Đối chiếu Chính sách & Tính toán    │ ➔ Áp dụng Rule-based Logic: Mức bồi thường,
│  - Match chính sách bảo hành/đổi trả TechShop│   Phương án xử lý, Phòng ban chịu trách nhiệm.
└──────────────────────────────────────────────┘
                       │
                       ▼ (Truyền Data Handoff Structured JSON)
┌──────────────────────────────────────────────┐
│  BƯỚC 3: Soạn thảo Phản hồi & Xử lý Ngoại lệ │ ➔ Kiểm tra dữ liệu dị biệt / thiếu sót (Fallback),
│  - Tạo email chuyên nghiệp, lọc thông tin    │   Xuất kết quả sạch lỗi cho hệ thống / CSKH.
└──────────────────────────────────────────────┘
                       │
                       ▼
[Đầu ra hoàn chỉnh: Email phản hồi khách hàng & Dữ liệu Dashboard]

```

---

## PHẦN 2: HỆ THỐNG PROMPT THỰC THI (PROMPT CHAINING SYSTEM)

###  Bước 1: Prompt Trích xuất & Phân tích nguyên nhân (Chain-of-Thought)

*Mục tiêu: Buộc AI phải suy luận từng bước (Step-by-step thinking) để nhìn thấu bản chất vấn đề, phát hiện văn phong mỉa mai và phân loại chính xác các trường dữ liệu.*

```text
[Context / Character]
Bạn là Chuyên viên phân tích dữ liệu và hành vi khiếu nại tại TechShop. Nhiệm vụ của bạn là đọc phản hồi thô của khách hàng và thực hiện quá trình suy luận từng bước (Chain-of-Thought) để bóc tách thông tin chính xác.

[Request & Chain-of-Thought Instructions]
Hãy phân tích đoạn phản hồi thô dưới đây theo các bước tư duy sau:
Bước 1: Đọc kỹ văn bản, phân tích các từ ngữ bề mặt và tầng nghĩa ẩn ý (nhận diện nếu có mỉa mai/khen đểu).
Bước 2: Trích xuất các thực thể cốt lõi: Mã đơn hàng (order_id), Tên khách hàng (customer_name), Vấn đề gặp phải (issue_description).
Bước 3: Xác định mức độ nghiêm trọng (severity: Thấp, Trung bình, Cao, Nghiêm trọng) dựa trên thực tế kỹ thuật/vận chuyển.

[Type of Output & Format]
Xuất kết quả dạng JSON thuần túy cấu trúc sau:
{
  "cot_reasoning": "Chuỗi suy luận từng bước của bạn...",
  "order_id": "...",
  "customer_name": "...",
  "issue_description": "...",
  "severity": "..."
}

[Input Data cần xử lý]:
"Shop làm ăn kiểu gì thế! Đơn hàng #TS-8888 tôi đặt tặng sinh nhật con trai, lúc nhận hộp thì móp méo nát bét, mở ra thì màn hình sọc ngang dọc như tranh trừu tượng, gọi tổng đài thì bắt đợi máy nửa tiếng!"

```

---

###  Bước 2: Prompt Đối chiếu Chính sách & Tính toán (Data Handoff & Logic)

*Mục tiêu: Nhận dữ liệu JSON từ Bước 1 (Data Handoff), đối chiếu trực tiếp với quy định đổi trả của TechShop để đưa ra phương án giải quyết chuẩn xác về mặt chính sách doanh nghiệp.*

```text
[Context / Character]
Bạn là Trợ lý Vận hành & Pháp chế chính sách tại TechShop. Bạn nắm giữ toàn bộ Quy chế Đổi trả và Bồi thường của công ty.

[Policy Reference / Quy định TechShop]
- Lỗi vận chuyển làm hỏng vỏ hộp/sản phẩm: Đổi mới 100% trong 7 ngày, miễn phí hoàn toàn phí ship 2 chiều, tặng Voucher 200k đền bù thời gian.
- Lỗi sản phẩm lỗi kỹ thuật/màn hình sọc: Đổi mới hoặc hoàn tiền 100%, kỹ thuật kiểm tra trong 24h.
- Lỗi quá tải tổng đài / thái độ CSKH: Ghi nhận, xin lỗi chính thức và chuyển ưu tiên VIP support.

[Request]
Dựa vào dữ liệu đầu vào (JSON từ bước trước) và Quy định TechShop ở trên, hãy tính toán phương án bồi thường và hành động xử lý tối ưu.

[Type of Output & Format]
Xuất kết quả dưới dạng JSON chuẩn:
{
  "order_id": "...",
  "compensation_action": "...",
  "voucher_reward": "...",
  "department_assigned": "...",
  "sla_hours": 24
}

```

---

###  Bước 3: Prompt Soạn thảo Phản hồi & Xử lý Ngoại lệ (Sanitization & Fallback)

*Mục tiêu: Nhận dữ liệu từ Bước 2, kiểm tra an toàn dữ liệu (nếu có trường bị thiếu hoặc dị biệt như mã đơn hàng không tồn tại/`null`, kích hoạt cơ chế Fallback an toàn thay vì crash hệ thống), sau đó soạn thảo email xin lỗi lịch sự, chuyên nghiệp.*

```text
[Context / Character]
Bạn là Trưởng bộ phận Chăm sóc khách hàng cao cấp tại TechShop, nổi tiếng với sự khéo léo, thấu cảm và chuyên nghiệp trong việc xoa dịu khách hàng khó tính.

[Error Handling & Fallback Rules (Rất quan trọng)]
- Kiểm tra tính hợp lệ của dữ liệu đầu vào: Nếu trường `order_id` có giá trị là `null`, `"Chưa xác thực"` hoặc không đúng định dạng `#TS-XXXX`, **bắt buộc** kích hoạt quy trình Fallback: Yêu cầu nhân viên CSKH thủ công liên hệ xin lại mã đơn hàng, không được tự bịa ra mã đơn trong email gửi khách.
- Lọc bỏ mọi thông tin nội bộ (không đưa các từ khóa SLA, department_assigned vào email khách hàng).

[Request]
Dựa vào thông tin bồi thường và xử lý từ bước trước, hãy soạn một email phản hồi khách hàng với văn phong chân thành, xin lỗi vì sự cố, thông báo rõ chính sách đổi mới, bồi thường voucher và cam kết thời gian xử lý.

[Type of Output & Format]
Kết quả trả về gồm 2 phần rõ ràng:
1. Trạng thái kiểm tra lỗi hệ thống (JSON kiểm tra `system_status`: "OK" hoặc "FALLBACK_REQUIRED").
2. Nội dung Email phản hồi khách hàng hoàn chỉnh.

```

---

## PHẦN 3: KẾT QUẢ CHẠY THỬ THỰC TẾ (TEST RUN SIMULATION)

Dưới đây là kết quả thực thi chuỗi prompt (Prompt Chaining) với kịch bản khách hàng bức xúc ở Bước 1:

### 1. Kết quả đầu ra từ Bước 1 (Trích xuất & CoT)

```json
{
  "cot_reasoning": "Bước 1: Khách hàng bức xúc ('làm ăn kiểu gì thế'), nhắc đến quà sinh nhật con trai nhưng cốt lõi là phản ánh 2 vấn đề lớn: Vỏ hộp móp méo do vận chuyển và màn hình sản phẩm hỏng nặng (sọc ngang dọc), kèm theo sự không hài lòng vì tổng đài chậm trễ. Bước 2: Trích xuất đơn hàng #TS-8888, tên khách hàng không đề cập trực tiếp. Bước 3: Mức độ lỗi nghiêm trọng do sản phẩm không sử dụng được và ảnh hưởng trực tiếp trải nghiệm khách hàng.",
  "order_id": "#TS-8888",
  "customer_name": "Chưa xác thực",
  "issue_description": "Vỏ hộp móp méo, màn hình sản phẩm bị sọc ngang dọc, tổng đài phản hồi chậm.",
  "severity": "Nghiêm trọng"
}

```

### 2. Kết quả đầu ra từ Bước 2 (Đối chiếu chính sách)

```json
{
  "order_id": "#TS-8888",
  "compensation_action": "Thu hồi sản phẩm hỏng, đổi mới 100% sản phẩm khác trong vòng 24h, miễn phí vận chuyển 2 chiều.",
  "voucher_reward": "Voucher trị giá 200.000 VNĐ",
  "department_assigned": "Bộ phận Kho vận & Kỹ thuật",
  "sla_hours": 24
}

```

### 3. Kết quả đầu ra từ Bước 3 (Soạn thảo phản hồi & Bắt lỗi Fallback)

```json
{
  "system_status": "OK",
  "email_response": "Tiêu đề: [TechShop] Chân thành xin lỗi Quý khách về sự cố đơn hàng #TS-8888\n\nKính gửi Quý khách hàng,\n\nTechShop đã tiếp nhận thông tin phản hồi của Quý khách về sự cố vỏ hộp móp méo và lỗi hiển thị màn hình đối với đơn hàng #TS-8888. Chúng tôi vô cùng lấy làm tiếc vì trải nghiệm không trọn vẹn này, đặc biệt khi sản phẩm là món quà sinh nhật dành tặng người thân của Quý khách.\n\nĐể khắc phục ngay lập tức, TechShop xin phép áp dụng phương án xử lý:\n1. Cử nhân viên kỹ thuật phối hợp đơn vị vận chuyển đến thu hồi sản phẩm cũ và đổi mới 100% sản phẩm nguyên vẹn tận nơi trong vòng 24 giờ tới (Miễn phí hoàn toàn).\n2. Gửi tặng Quý khách Voucher trị giá 200.000 VNĐ nhằm chia sẻ sự bất tiện mà Quý khách đã gặp phải.\n\nBộ phận Chăm sóc khách hàng cũng đã ghi nhận góp ý về thời gian phản hồi tổng đài để chấn chỉnh chất lượng dịch vụ tốt hơn. Trân trọng cảm ơn sự thông cảm và đồng hành của Quý khách cùng TechShop!"
}

```