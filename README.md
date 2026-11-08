# fit-gap

Tài liệu sản phẩm cho công cụ Fit & Gap tuyển dụng (tên tạm: Khớp): engine so khớp CV với JD cho thị trường tuyển dụng Việt Nam, phục vụ cả ứng viên (biết thiếu gì, lấp trong bao lâu) và nhà tuyển dụng (ứng viên có hợp không, nên hỏi gì).

## Nội dung

| File | Nội dung |
|---|---|
| [docs/PRD.md](docs/PRD.md) | PRD: bối cảnh, người dùng, tính năng theo giai đoạn, luồng, kiến trúc engine, tiêu chí chất lượng, kinh doanh, đối thủ, chiến lược model, kế hoạch thử nghiệm, rủi ro, layout |
| [docs/engine.md](docs/engine.md) | Engine: nhiệm vụ, kiến trúc AI/code theo layer, quy tắc chấm, yêu cầu từng layer so với bản demo, thời gian lấp gap, bộ chỉ số, thứ tự làm, câu hỏi cần bàn |
| [docs/ho-so-ung-vien.md](docs/ho-so-ung-vien.md) | Hồ sơ ứng viên: lead và hồ sơ, nhận diện một người, cách lưu trong PostgreSQL, tìm K người trên kho, phễu tìm thêm, nguồn làm đầy kho |
| [docs/thu-thap-du-lieu.md](docs/thu-thap-du-lieu.md) | Thu thập dữ liệu: crawler JD, nhận CV có đồng ý, kho thô và kho tinh, hàng đợi xử lý |
| [docs/chien-luoc.md](docs/chien-luoc.md) | Chiến lược: engine chạy như một headhunter, đánh giá từ sáu góc nhìn, ba giai đoạn, cam kết với công ty |
| [docs/phap-ly.md](docs/phap-ly.md) | Pháp lý cần kiểm tra trước khi publish: dữ liệu cá nhân, thương mại điện tử, Luật Trí tuệ nhân tạo, dịch vụ việc làm, checklist theo mốc, câu hỏi cho luật sư |

## Bản gốc

Hai tài liệu trên được xuất từ bản Docs trên claude.ai (bản gốc có sơ đồ tương tác). Bản gốc là nơi chỉnh sửa chính; file trong repo là ảnh chụp tại thời điểm commit.

- PRD: https://claude.ai/artifact/XqpqWdE63ZjWkZbDGHD1Mm
- Pháp lý: https://claude.ai/artifact/U68PBAyyn5JRdJz5XVsEGY
- Layout MVP: https://claude.ai/artifact/1HpuHXNHSYjtYpbyhjG38Y
- Demo engine Layer 0–3: https://claude.ai/artifact/EX6WLqKb6LAnp4qFEZnnem

Các link chỉ người được chia sẻ mới mở được.

## Lưu ý

Tài liệu pháp lý là ghi chú tra cứu nội bộ, không phải ý kiến pháp lý. Kiểm tra lại với luật sư trước mỗi lần publish.
