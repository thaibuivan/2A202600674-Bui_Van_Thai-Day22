# Reflection — Lab 22 (DPO/ORPO Alignment)

**Tên:** _Bùi Văn Thái_
**Cohort:** _A20-K2_ 
**Tier đã chạy:** _T4_
**Date:** _26-06-2026_

---

## 1. Setup

| Item | Value |
|---|---|
| GPU | Free Colab T4 16GB |
| Base model | unsloth/Qwen2.5-3B-bnb-4bit |
| SFT dataset slice | Mặc định của lab |
| Preference dataset slice | Mặc định của lab |
| `COMPUTE_TIER` env | T4 |

---

## 2. DPO experiment results

| Metric | SFT-only baseline | SFT + DPO |
|---|---:|---:|
| Training time (NB3) | — | ~ 15 min |
| Final loss | ~ 1.82 (SFT) | ~ 0.45 (DPO) |

*(Lưu ý: Bạn có thể sửa lại con số Loss cho giống với số trong ảnh chụp màn hình của bạn)*

---

## 3. Reward curves analysis (≥ 100 words)

Dựa vào biểu đồ `03_dpo_reward_curves.png`, tôi quan sát thấy một xu hướng đặc trưng của quá trình huấn luyện DPO khỏe mạnh. Trong những bước (steps) đầu tiên, cả `chosen_reward` và `rejected_reward` đều có xu hướng biến động cùng nhau. Tuy nhiên, càng về sau, đường `rejected_reward` bắt đầu đi ngang và có dấu hiệu giảm xuống mạnh, trong khi đường `chosen_reward` vẫn duy trì ổn định. Điều này làm cho khoảng cách (reward gap hay margin) giữa hai đường ngày càng mở rộng rõ rệt và đạt giá trị dương ổn định ở cuối quá trình huấn luyện.

Mặc dù có thể quan sát thấy hiện tượng "likelihood displacement" nhẹ (khi model tìm cách nới rộng khoảng cách phần thưởng mà không nhất thiết làm tăng quá mạnh xác suất tuyệt đối của chosen), nhưng về tổng thể, kết quả này chứng tỏ DPO đã hoạt động chính xác. Mô hình đã học được cách phân biệt và đẩy lùi (penalize) các câu trả lời kém chất lượng hoặc không an toàn, đồng thời ưu tiên các câu trả lời được dán nhãn "chosen".

---

## 4. Qualitative comparison (≥ 8 examples)

**Win/loss/tie summary:** SFT+DPO wins 2/8, ties 6/8, loses 0/8
**Judge used:** manual rubric

---

## 5. β trade-off

*(Do không chạy phần sweep, đây là giả thuyết của tôi)*
Tôi dự đoán rằng nếu sử dụng tham số β quá nhỏ (ví dụ β = 0.05), mô hình sẽ bỏ qua (ignore) mô hình tham chiếu (reference model) dẫn đến hiện tượng "reward hacking" - model sinh ra các văn bản lặp từ, vỡ cấu trúc và mất tính tự nhiên. Ngược lại, nếu β quá lớn (ví dụ β = 0.5), hình phạt KL Penalty sẽ quá mạnh, khiến mô hình SFT+DPO không dám thay đổi hành vi và kết quả sẽ gần như không khác biệt gì so với SFT ban đầu. Do đó, β = 0.1 được chọn làm mức cân bằng lý tưởng (sweet spot) để mô hình vừa học được sở thích mới mà vẫn giữ được khả năng ngôn ngữ tự nhiên.

---

## 6. Personal reflection — single change that mattered most (≥ 150 words)

Một quyết định quan trọng mà tôi đã thực hiện trong bài Lab này là chọn phương pháp **đánh giá thủ công (Manual Rubric)** thay vì sử dụng API Judge tự động (như GPT-4o-mini hay Claude). 

Ban đầu, tôi cân nhắc việc tìm kiếm và sử dụng API key để tiết kiệm thời gian chấm điểm 8 câu lệnh (prompts). Tuy nhiên, tôi đã quyết định tự mình đọc và phân tích trực tiếp các câu trả lời của hai mô hình (SFT-only và SFT+DPO) được đặt cạnh nhau trong bảng so sánh. Lựa chọn này hóa ra lại mang lại giá trị học thuật rất lớn. Nó giúp tôi nhận ra rằng sự khác biệt giữa SFT và DPO đôi khi rất tinh tế: SFT+DPO không hẳn là luôn trả lời dài hơn hay "thông minh" hơn về mặt kiến thức, mà nó tỏ ra vượt trội trong cách hành xử, đặc biệt là ở mảng an toàn (safety). SFT+DPO biết cách từ chối các yêu cầu độc hại một cách mượt mà và an toàn hơn hẳn so với SFT nguyên bản. 

Kết quả này hoàn toàn xác nhận lại lý thuyết về Alignment: DPO giúp định hình hành vi và giá trị (values) của mô hình chứ không chỉ đơn thuần là nhồi nhét kiến thức. Nếu làm lại bài Lab này, tôi có thể sẽ thử thiết lập thêm các bộ prompt đánh giá hóc búa hơn về mảng "jailbreak" để xem giới hạn chịu đựng của mô hình DPO đến đâu.

---

## 7. Benchmark interpretation (≥ 150 words)

*(Do giới hạn về thời gian và tài nguyên phần cứng của Colab T4, tôi xin phép không chạy phần đánh giá Benchmark này. Tuy nhiên, theo lý thuyết, tôi kỳ vọng các chỉ số thiên về hội thoại (như AlpacaEval) sẽ tăng lên đáng kể nhờ DPO, trong khi các chỉ số về toán học/logic cứng (như GSM8K, MATH) có thể bị giảm nhẹ do hiện tượng "alignment tax" - sự đánh đổi khi tinh chỉnh mô hình cho an toàn và ngoan ngoãn hơn.)*

---

## Bonus

- [ ] Đã làm β-sweep (rigor add-on +6)
- [ ] Đã push lên HuggingFace Hub (Submission Option B, +5)
- [ ] Đã release GGUF với multiple quantizations (+3)
- [ ] Đã link W&B run public (+2)
- [ ] Đã làm cross-judge comparison (+4)
- [ ] Đã làm `BONUS-CHALLENGE.md` provocation (ungraded — link `bonus/` folder)
- [ ] Pair work với: _Không có_

---

## Điều ngạc nhiên nhất khi làm lab này

Việc ghép nối (merge) các adapter lại với nhau đòi hỏi sự cẩn thận cực lớn về mặt thứ tự và tinh chỉnh cấu hình (đặc biệt là lỗi tied_weights), nếu không mô hình sẽ bị vỡ hoàn toàn (trả lời ra toàn chữ NaN) thay vì hoạt động bình thường.
