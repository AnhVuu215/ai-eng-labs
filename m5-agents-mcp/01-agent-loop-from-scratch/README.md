# 01-agent-loop-from-scratch

> M5 · Agents, tool use và MCP — Khối 1 · Tuần 32–33 · Tool calling và vòng lặp agent tự viết

**Trạng thái:** ⬜ Chưa bắt đầu

## Mục tiêu

- Viết vòng lặp agent không framework với 3 tool: search_regulation (chính hệ RAG của bạn), calculate_gpa (code xác định, không để LLM tự tính), get_academic_calendar (dữ liệu giả lập)
- Giới hạn 8 bước; ghi toàn bộ quỹ đạo ra log
- Thử 15 tác vụ như “Em có 1 môn F, GPA 2.1, kỳ sau có bị cảnh báo học vụ không?”

## Tiêu chí đạt

- [ ] đọc log quỹ đạo và giải thích được vì sao agent chọn từng tool; mọi phép tính đi qua tool, không do model tự tính.

## Lý thuyết cần đọc

- [ ] Giao thức tool calling: model trả yêu cầu gọi tool → code chạy tool → gửi kết quả lại
- [ ] Mẫu ReAct (suy nghĩ → hành động → quan sát); điều kiện dừng; giới hạn số bước
- [ ] Context phình to sau nhiều bước; tóm tắt hoặc cắt bớt lịch sử

## Cách chạy

```bash
# TODO
```

## Kết quả đo

| Chỉ số | Giá trị | Ghi chú |
|---|---|---|
|  |  |  |

## Ghi chép / bài học

- 
