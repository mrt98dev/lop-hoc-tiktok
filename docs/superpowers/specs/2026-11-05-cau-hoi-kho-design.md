# Thiết kế: Nâng độ khó bộ câu hỏi Lớp Học Chữ

> Ngày: 2026-11-05 · Trạng thái: đã chốt với chủ kênh

## Vấn đề

Bộ câu hỏi 2076 câu (dễ 572 · vừa 984 · khó 520) đang quá dễ: mỗi tiết chỉ có 4 slot "khó" và chính các câu "khó" cũng đoán được từ gợi ý (GIẤY THẤM, SỔ ĐẦU BÀI, CHÉP PHẠT...). Chủ kênh muốn bộ câu "khó đến cực khó" theo cả 5 hướng: kiến thức hiếm, đố mẹo hiểm, thành ngữ/Hán Việt, toán–khoa học khó hơn, lịch sử–địa lý chuyên sâu.

## Quyết định đã chốt

| Quyết định | Chọn |
|---|---|
| Mức độ khó | Giữ 3 mức (dễ/vừa/khó), không thêm mức 4 |
| Chia 10 câu chữ của tiết | **2 dễ / 2 vừa / 6 khó** |
| Thời gian | Giữ 20 giây cho mọi câu; hết giờ mở 1 chữ + 10 giây như cũ |
| Giới hạn đáp án | 14 → **16 chữ cái** |
| Đối tượng câu khó | **Dân chuyên** — chỉ 5–10% khán giả trả lời được; chấp nhận nhiều câu hết giờ không ai đúng |
| Bộ câu dễ cũ | **Xoá bớt** (~420 câu), giữ ~150 câu tiêu biểu cho slot mở màn |
| Câu "khó" cũ | Hạ nhãn ~200 câu thực ra dễ đoán thành mức vừa |
| Số câu mới | ~600 câu khó/cực khó |
| Tổ chức | Cách lai: file chủ đề mới + bổ sung tại chỗ 2 file slot (mẹo, sử) |

## 1. Luật chơi & cấu trúc tiết

Kế hoạch 20 câu mới (`GAME.roundPlan`):

```
1 easy · 2 easy · 3 anh · 4 medium · 5 math2
6 medium · 7 meo · 8 hard · 9 su · 10 math3
11 hard · 12 anh · 13 meo · 14 hard · 15 math4
16 hard · 17 su · 18 hard · 19 math5 · 20 hard (vàng x2)
```

- Mở màn 2 câu dễ, từ câu 8 trở đi gần như toàn câu khó — nhịp leo dốc.
- Slot toán ghi `math2…math5`: `quiz.ts` đã đọc số từ tên slot (`Number(d.slice(4))`) nên tự map đúng cấp toán, không cần sửa logic.
- Câu hỏi vàng (câu 20) luôn là câu khó thật.
- Toán khó hơn: thêm cấp 5 vào `math.ts` (phép 3 bước, số lớn hơn — vd `24 × 3 + 48 : 6`, `(15 + 27) × 4 − 30`), đáp án luôn số dương nguyên. Bốn slot toán leo dốc: cấp 2, 3, 4, 5.

## 2. Nội dung bộ câu

### Xử lý bộ cũ

| Việc | Số câu | Chi tiết |
|---|---|---|
| Xoá câu dễ | ~420 | Di chuyển sang `src/game/cau-hoi/easy-parked/` (thư mục con — `import.meta.glob('./cau-hoi/*.txt')` và `bank.check.ts` đều không nạp). Không mất nội dung; muốn trả lại chỉ cần kéo file lên. Giữ ~150 câu dễ tiêu biểu nhất (PHẤN, VỞ, CÔ GIÁO, BÚT CHÌ...) cho 2 slot mở màn — đủ ~75 tiết không lặp. |
| Hạ nhãn câu "khó" cũ | ~200 | Rà 520 câu khó hiện có, câu nào thực ra đoán được thì đổi nhãn 3 → 2. Còn lại ~320 câu khó thật. |

### Viết mới ~600 câu khó (nhãn 3)

| File | Nội dung | Số câu |
|---|---|---|
| `31-do-hiem.txt` | Đố logic nhiều bước, đố mẹo hiểm | 110 |
| `32-khoa-hoc-kho.txt` | Thiên văn, vật lý, sinh học, hoá — sự thật đã chốt | 90 |
| `33-dia-ly-kho.txt` | Địa lý chuyên sâu Việt Nam + thế giới | 90 |
| `34-lich-su-kho.txt` | Lịch sử sâu (giữ luật: VN tới 1945, không nhân vật tranh cãi) | 80 |
| `35-thanh-ngu-kho.txt` | Thành ngữ, tục ngữ, Hán Việt, chơi chữ | 90 |
| `36-van-hoc-kho.txt` | Văn học, tác giả, thơ | 60 |
| `23-do-meo.txt` (thêm tại chỗ) | Đố mẹo khó hơn | +40 |
| `24-lich-su-su.txt` (thêm tại chỗ) | Lịch sử khó hơn | +40 |

Tổng cộng ~600 câu mới → bộ "khó" ~920 câu, đủ ~150 tiết không lặp (cần tối thiểu 300 cho 50 tiết).

### Chuẩn chất lượng từng câu

- Gợi ý là thông tin **thật** nhưng không lộ đáp án — người biết thì biết ngay, người không biết không đoán được.
- Đáp án 2–16 chữ cái, có 2–5 cách viết khác (tên riêng, cách gọi).
- Lịch sử: chi tiết nguồn chép khác nhau thì KHÔNG đưa vào gợi ý (luật hiện có); không dùng nhân vật còn tranh cãi.
- Khoa học: tránh "kỷ lục" dễ thay đổi; chỉ sự thật ổn định.
- Thành ngữ: đúng từ điển, không chế.
- Không trùng đáp án trong cùng chủ đề; gợi ý không chứa nguyên đáp án (bank.check tự bắt).

Ví dụ mùi vị:

```
BẠCH ĐẰNG | Trận thủy chiến năm 1288, quân dân nhà Trần chặn quân Nguyên trên sông này. | 3
TẢN ĐÀ | Bút danh của nhà thơ Nguyễn Khắc Hiếu. | 3
BIỂN CHẾT | Vùng nước mặn nhất thế giới, nằm giữa Israel và Jordan. | 3
HỊCH TƯỚNG SĨ | Bài văn do Trần Quốc Tuấn viết để động viên quân sĩ. | 3
```

### Hệ quả chấp nhận

Nhiều câu sẽ hết giờ không ai đúng: cô mở 1 chữ + 10 giây, rồi đọc đáp án. Đây là chủ đích (đối tượng "dân chuyên"). Các cơ chế lấp im lặng (chào, chuyện vui, bình chọn, gọi bạn ngủ gật) vẫn hoạt động như cũ.

## 3. Sửa code

| File | Sửa |
|---|---|
| `src/game/config.ts` | `roundPlan` mới (bảng trên); cập nhật chú thích "4 dễ 2 vừa 4 khó" |
| `src/game/math.ts` | `MathLevel` 1–5; thêm sinh phép cấp 5 (3 bước, số lớn hơn); `DIFF[5] = 'hard'` |
| `src/game/bank.check.ts` | `MAX_LETTERS` 14 → 16 |
| `src/game/quiz.check.ts` | Vòng test toán chạy cấp 1–5; cập nhật chú thích cũ ("5 dễ 6 vừa 5 khó") |

Không cần sửa: `questions.ts` (giữ 3 mức), `quiz.ts` (đọc roundPlan + số cấp toán từ tên slot), `GameStage.tsx` (ô chữ tự co theo số chữ), `loi-thoai.ts` (lời cô không nhắc số câu dễ).

## 4. Kiểm thử

- `npx tsx src/game/bank.check.ts` — cú pháp, trùng, gợi ý lộ đáp án, độ dài ≤ 16, đếm dễ/vừa/khó.
- `npx tsx src/game/quiz.check.ts` — chạy nguyên một tiết 20 câu: đúng kế hoạch, điểm, chuỗi, không lặp câu (~71 tiết liền).
- Chạy thử server với `DATA_DIR` tạm, xem một tiết thật.

## 5. Triển khai

2 commit:

1. **Code**: roundPlan mới + toán cấp 5 + giới hạn 16 chữ + test.
2. **Bộ câu**: dời câu dễ vào `easy-parked/`, hạ nhãn câu "khó" cũ, 6 file mới + bổ sung 2 file slot.

## 6. Rủi ro đã cân nhắc

- Sổ lớp `data/so-lop.json` nhớ id câu đã hỏi — id câu bị dời đi chỉ là không bao giờ khớp, vô hại.
- Tiết "ôn bài" (lớp vắng) hỏi lại câu cũ — vẫn hoạt động vì lấy từ sổ đã hỏi.
- Bình chọn chủ đề cuối tiết tự lấy từ mọi chủ đề không phải slot — file mới tự vào danh sách, không cần sửa.
- Test "không lặp câu" chạy ~71 tiết liền (tăng từ 48) — chậm hơn chút, chấp nhận được.

## 7. Ngoài phạm vi

- Không thêm mức độ khó thứ 4, không đổi thời gian trả lời, không đổi luật điểm.
- Không đụng câu tiếng Anh (25-tieng-anh.txt), không đổi lời cô, không đổi giao diện.
