# Ghi chú phân tích · Day 04

Họ và tên: Đỗ Việt Anh

MSSV:2A202603973

## 1. Case được chọn

- ID case: A1
- Câu trả lời của agent: Agent trả lời đúng phần lớn phạm vi của GitHub Actions và
  Cloudflare Spectrum, nhưng ghi `h02n9572mx9y` resolved lúc `2026-11-07 01:10`
  và `xr3myzj7ms51` resolved lúc `2026-11-07 01:04`.
- Điểm ở lần chạy 1: 85/100

## 2. Bằng chứng trong trace

Judge đối chiếu trace cho thấy hai mốc trên là `updated_at` của incident, không
phải thời điểm update chuyển sang `resolved`. Thời điểm resolved trong các update
lần lượt là `2026-11-04 18:16` và `2026-11-04 17:01`. Vì vậy câu trả lời suy luận
về quan hệ thời gian giữa các incident không đủ chắc chắn.

```text
eval_before.json, case A1:
- h02n9572mx9y: updated_at = 2026-11-07T01:10:18Z
- h02n9572mx9y: update status=resolved, created_at = 2026-11-04T18:16:08Z
- xr3myzj7ms51: updated_at = 2026-11-07T01:04:38Z
- xr3myzj7ms51: update status=resolved, created_at = 2026-11-04T17:01:11Z
```

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: `agent.py` lưu `updated_at` và thời điểm của update
  `status=resolved` khi đọc incident. Trước khi hoàn tất, agent phát hiện câu trả
  lời gán `updated_at` cho resolved, ghi event
  `validation_mislabelled_resolution` và yêu cầu model sửa trong `max_steps`.
- Vì sao chọn thay đổi này: Đây là lỗi factual cụ thể có thể kiểm tra tự động,
  không cần đoán ý định của model. `updated_at` là mốc chỉnh sửa metadata; nó
  không đại diện cho lúc incident chuyển trạng thái.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 94.05 | 82.41 |
| Điểm case đã chọn | 85 | 98 |
| Số model calls | 3 model calls trong trace A1 | 6 model calls trong trace A1 |
| Token, chi phí nếu có | Có trong `eval_before.json` | Có trong `eval_after.json` |

- Lỗi đã giải quyết: A1 tăng từ 85 lên 98; câu trả lời sau cải tiến đã phân biệt
  mốc resolved với `updated_at` tốt hơn.
- Lỗi còn lại hoặc mới xuất hiện: A5 và H8 kết thúc ở `max_steps`, nhưng vẫn
  được chấm; cần tối ưu việc dừng hoặc giảm gọi tool nếu muốn cải thiện điểm.

## 5. Giới hạn

Điểm judge là ước lượng, snapshot không đại diện cho trạng thái hiện tại, và
`eval_before.json` chỉ là một lần chạy nên có thể bị ảnh hưởng bởi timeout hoặc
biến động model/provider. Thay đổi này chỉ kiểm tra mốc thời gian khi tool đã
trả update tương ứng; không chứng minh mọi claim trong câu trả lời đều đúng.

## 6. Bonus

- `agent.py` dùng `tools.schemas` và `tools.system_prompt` khi tools bonus cung
  cấp chúng, nhưng vẫn tương thích với tools bài chính.
- Bonus hỗ trợ 5 status provider, pagination qua `next_offset`, dữ liệu nhiễu,
  lỗi `INVALID_ARGUMENTS`, `RATE_LIMITED` và timeout.
- Retry cho cùng tool/provider bị giới hạn 3 lần; sau đó ghi
  `retry_exhausted` và báo rõ không thể xác nhận trạng thái.
- Đã pass 4 bonus agent tests và toàn bộ 38 test chính.
- Kết quả bonus sau khi chạy lại: 10/10 case được chấm, điểm 94.7/100,
  quy đổi `bonus_points_0_15 = 14.21/15`.
