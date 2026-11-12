# Bài 1: Hiểu đúng bản chất "Máy dự đoán từ tiếp theo" (Next Token Prediction) của LLM

## Phần giải thích kiến thức

**LLM (Large Language Model)** không "suy nghĩ" như con người. Về bản chất kỹ thuật, nó hoạt động theo cơ chế **Next Token Prediction**: dựa trên chuỗi chữ đã có, mô hình tính xác suất để chọn ra *token* (mẩu từ) tiếp theo có khả năng xuất hiện cao nhất.

Ví dụ: câu "Con mèo đang ngồi trên..." → "thảm" (85%), "ghế" (10%), "mái nhà" (5%) → AI chọn "thảm".

**Pre-training (huấn luyện trước):** là giai đoạn mô hình đọc hàng tỷ văn bản để học *quy luật xuất hiện của chuỗi ký tự*. Nó học "sau chữ này thường là chữ nào", chứ **không** học để thực thi thuật toán toán học.

| | Máy tính bỏ túi (Calculator) | LLM |
|---|---|---|
| Cơ chế | Thực thi thuật toán, logic chính xác | Dự đoán xác suất token tiếp theo |
| Với `15.480 + 28.750` | Tính ra đúng 44.230 | "Đoán" chuỗi số trông hợp lý nhất |
| Độ chính xác số học | 100% | Không đảm bảo |

---

## Phần 1: Bóc tách nhận thức sai trong báo cáo của Nam

### Sai lầm 1: Cho rằng AI bị "hỏng phần mềm / máy chủ lỗi"

- **Thực tế:** hệ thống không hỏng. Đây là hành vi **bình thường** của LLM. Mô hình hoạt động theo **Next Token Prediction**: nó không "tính" 15.480 + 28.750 mà dự đoán chuỗi ký tự có xác suất cao nhất đứng sau câu hỏi.
- Kết quả `44.130` là một chuỗi số *trông giống* đáp án (cùng độ dài, cùng đầu số `44.`) nên được mô hình chọn, dù sai ở chữ số hàng chục.

### Sai lầm 2: Cho rằng LLM có "bộ vi xử lý" tính toán chính xác 100% vì có hàng trăm tỷ tham số

- **Thực tế:** hàng trăm tỷ tham số chỉ là các trọng số lưu *quy luật thống kê của ngôn ngữ* học được trong giai đoạn **Pre-training**, không phải bộ máy thực thi phép toán (ALU / calculator).
- Dữ liệu Pre-training là văn bản, không phải mọi phép cộng đều xuất hiện trong dữ liệu. Với những con số cụ thể ít gặp như `15.480 + 28.750`, mô hình không có "đáp án nhớ sẵn" đáng tin nên dễ đoán sai.
- Số lượng tham số lớn ≠ khả năng suy luận logic chính xác.

### Giải thích tổng hợp: Vì sao LLM tính sai phép cộng trên?

1. LLM chỉ là **máy dự đoán từ tiếp theo (Next Token Prediction)**, không phải máy tính.
2. Kiến thức của nó đến từ **Pre-training**: học xác suất xuất hiện chuỗi ký tự, không học thuật toán cộng có nhớ.
3. Số được tách thành các token rời rạc, mô hình ghép chuỗi "hợp lý về mặt thống kê" thay vì thực hiện từng bước cộng → dễ sai chữ số.
4. Đáp án đúng là **15.480 + 28.750 = 44.230**, còn `44.130` là kết quả "đoán sai".

> **Kết luận cho Nam:** AI không lỗi. Lỗi nằm ở việc dùng sai công cụ. Cần dùng AI đúng thế mạnh (xử lý ngôn ngữ) và giao phần tính toán cho công cụ chuyên dụng.

---

## Phần 2: Sửa lỗi và Prompting

### Nguyên tắc

Để có kết quả số học chính xác 100%, cần **buộc AI giao phép tính cho công cụ thực thi code (Code Interpreter / Python)** hoặc **ép AI suy luận từng bước** thay vì đoán đáp án.

### Prompt đề xuất 1: Kích hoạt Code Interpreter / Python (khuyến nghị)

```
Hãy sử dụng công cụ Code Interpreter (Python) để tính chính xác phép tính sau,
không được tính nhẩm hoặc đoán kết quả:

15.480 + 28.750

Yêu cầu:
1. Viết và chạy đoạn code Python thực hiện phép cộng này.
2. Hiển thị đoạn code và kết quả trả về từ Python.
3. Nêu kết quả cuối cùng.
```

**Kết quả kỳ vọng:**

```python
print(15480 + 28750)   # → 44230
```

→ Kết quả: **44.230** (chính xác 100% vì do Python thực thi, không phải do AI đoán).

### Prompt đề xuất 2: Ép suy luận từng bước (Chain-of-Thought)

```
Hãy tính 15.480 + 28.750. Không đưa ngay đáp án.
Hãy giải từng bước, cộng theo từng hàng (đơn vị, chục, trăm, nghìn, chục nghìn),
ghi rõ phần nhớ ở mỗi bước, sau đó kiểm tra lại kết quả bằng phép trừ ngược.
```

**Cách AI giải từng bước:**

| Hàng | Phép cộng | Kết quả | Nhớ |
|---|---|---|---|
| Đơn vị | 0 + 0 | 0 | 0 |
| Chục | 8 + 5 | 13 → viết 3 | nhớ 1 |
| Trăm | 4 + 7 + 1 | 12 → viết 2 | nhớ 1 |
| Nghìn | 5 + 8 + 1 | 14 → viết 4 | nhớ 1 |
| Chục nghìn | 1 + 2 + 1 | 4 | 0 |

→ Kết quả: **44.230**. Kiểm tra: 44.230 − 28.750 = 15.480 ✔

### Vì sao 2 prompt này hiệu quả?

| Prompt | Cơ chế |
|---|---|
| Dùng Code Interpreter | Giao phép tính cho **Python**, một công cụ tính toán thật, nên không còn phụ thuộc vào Next Token Prediction. |
| Suy luận từng bước | Chia nhỏ bài toán thành các bước đơn giản mà mô hình dự đoán đúng hơn, kèm bước kiểm chứng ngược. |

---

## Tóm tắt nộp bài

- **Phần 1:** Nam sai vì (1) cho rằng AI hỏng, trong khi đó là bản chất **Next Token Prediction**; (2) cho rằng LLM có bộ xử lý tính toán, trong khi nó chỉ học quy luật chuỗi ký tự ở giai đoạn **Pre-training**.
- **Phần 2:** Dùng prompt yêu cầu **Code Interpreter / Python** (hoặc suy luận từng bước + kiểm tra ngược) để ra kết quả đúng **44.230**.
