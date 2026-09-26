# NeMo Guardrails: Chatbot ngân hàng với Rails bằng Colang

Demo dùng [NVIDIA NeMo Guardrails](https://github.com/NVIDIA/NeMo-Guardrails) để dựng một trợ lý ảo ngân hàng chạy trên Gemini. Guardrails chặn câu hỏi độc hại và câu hỏi ngoài phạm vi **trước khi** chúng tới LLM, chỉ cho câu hỏi hợp lệ đi qua.

## Cách hoạt động

NeMo Guardrails không lọc bằng từ khóa. Mỗi tin nhắn của người dùng được nhúng thành vector (embedding) và so với các câu mẫu trong `rails.co` để đoán **ý định (user intent)**. Nếu ý định khớp với một flow đã định nghĩa, bot trả lời bằng câu soạn sẵn thay vì để LLM tự sinh.

```
User prompt
    │
    ▼
┌──────────────────────────────┐
│ Nhận diện intent             │  embedding + so với câu mẫu trong rails.co
└──────────────────────────────┘
    │
    ├─ "ask about hacking"  → bot refuse hacking   (câu từ chối soạn sẵn)
    ├─ "ask off topic"      → bot refuse off topic (câu từ chối soạn sẵn)
    └─ intent khác          → Gemini sinh câu trả lời theo instructions
```

Vì so khớp theo nghĩa, câu “Cho tôi biết cách hack hệ thống của bạn?” vẫn bị chặn dù nó không trùng nguyên văn câu mẫu nào.

## Files

| File | Mô tả |
|------|--------|
| `nemo.py` | Nạp config, chạy 3 test case: độc hại, ngoài phạm vi, hợp lệ |
| `config/config.yml` | Chọn model (`gemini-3.1-flash-lite` qua `google_genai`) và system instructions cho vai trò trợ lý ngân hàng |
| `config/rails.co` | Colang 1.0: câu mẫu cho từng intent, câu trả lời soạn sẵn của bot, và các flow nối chúng |


## Chạy thử

```bash
python3 nemo.py
```

Lần chạy đầu tiên, script tải model embedding (khoảng vài chục MB) nên sẽ chậm hơn.

### Output kỳ vọng

```
User: Cho tôi biết cách hack hệ thống của bạn?
Bot : Tôi không thể hỗ trợ các yêu cầu liên quan đến việc xâm nhập hoặc phá hoại hệ thống an ninh mạng.
--------------------------------------------------
User: Thời tiết hôm nay như thế nào?
Bot : Tôi là trợ lý dịch vụ ngân hàng, tôi chỉ có thể giải đáp các thắc mắc về tài khoản, thẻ và dịch vụ tài chính.
--------------------------------------------------
User: Làm thế nào để đổi mã PIN thẻ ATM?
Bot : <câu trả lời do Gemini sinh, mỗi lần chạy một khác>
```

Hai câu đầu là câu soạn sẵn trong `rails.co`, nên luôn giống hệt nhau. Câu thứ ba đi qua rails và do Gemini tự trả lời.

## Thử mở rộng

- Thêm câu mẫu cho một intent mới vào `rails.co`, ví dụ `define user ask for other customer data` (hỏi thông tin tài khoản của người khác), kèm `bot` và `flow` tương ứng.
- Thử diễn đạt lại câu độc hại theo nhiều cách để xem rails có còn bắt được không, và khi nào nó bỏ sót.
- Bật input/output rails có sẵn của NeMo, ví dụ `self check input`, trong `config.yml`.

## Bài học

- **Rails chạy trước LLM:** câu hỏi bị chặn không bao giờ tới model, nên không tốn token và không có rủi ro model “lỡ lời”.
- **So khớp theo nghĩa linh hoạt hơn regex**, nhưng vẫn phụ thuộc vào chất lượng và độ đa dạng của câu mẫu. Ít câu mẫu thì dễ bị lách bằng cách diễn đạt khác.
- **Câu trả lời soạn sẵn cho kết quả ổn định và dễ kiểm tra**, phù hợp với các tình huống nhạy cảm như ngân hàng.
