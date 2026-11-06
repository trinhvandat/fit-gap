# Thu thập dữ liệu: crawler JD, nhận CV, kho thô và kho tinh – thiết kế

07/11/2026 · Leonard · Bản nháp để thảo luận

File này mô tả cách dữ liệu vào engine: JD cào từ nguồn công khai, CV có đồng ý của ứng viên, và CV công ty đưa vào. Dữ liệu được lưu ở kho thô, xử lý qua hàng đợi, rồi thành dữ liệu tinh trong PostgreSQL. Bảng hồ sơ ứng viên và cách tìm K người dùng kho tinh này, xem [ho-so-ung-vien.md](ho-so-ung-vien.md).

## 1. Ranh giới: cào JD, không cào CV

| Loại dữ liệu | Cào tự động | Lý do |
| --- | --- | --- |
| JD | Có, theo điều khoản từng trang | Tin công khai, công ty muốn nhiều người đọc. Engine cần kho JD đang mở cho luồng ứng tuyển hộ |
| CV | Không | CV trên các trang việc làm chỉ mở cho tài khoản nhà tuyển dụng trả phí, và điều khoản cấm tải hàng loạt. Đem CV đó giới thiệu cho công ty khác để thu phí rất gần với mua bán dữ liệu cá nhân. Kho cào về cũng không có `consent` và `availability` |

CV chỉ vào kho qua các nguồn có đồng ý ở mục 4. Câu hỏi pháp lý liên quan nằm ở [phap-ly.md](phap-ly.md) mục 2 và 6.

## 2. Kiến trúc: tách luồng thu thập và luồng xử lý

```mermaid
flowchart LR
  S1[Crawler JD] --> RAW
  S2[Upload CV có đồng ý<br/>tool, form, PDF LinkedIn] --> RAW
  S3[Import ATS công ty] --> RAW
  RAW[(Kho thô<br/>file gốc, bất biến<br/>khóa theo hash)] --> Q[Hàng đợi việc<br/>PostgreSQL]
  Q --> CG{Đồng ý còn hiệu lực?}
  CG -- có --> P[Worker xử lý<br/>Layer 0–1, gắn canonical_id]
  CG -- không --> X[Bỏ, xóa file thô]
  P --> REF[(Kho tinh<br/>PostgreSQL: JD, hồ sơ)]
```

Luồng thu thập chỉ lấy file về và ghi vào kho thô. Luồng xử lý đọc từ hàng đợi, chạy Layer 0–1 rồi ghi kết quả vào kho tinh. Cổng đồng ý chỉ áp dụng cho CV.

Lý do tách:

- **Xử lý lại mà không phải thu lại.** Danh mục kỹ năng, prompt hoặc model đổi (`catalog_version`, `engine_version`) thì chạy lại worker trên kho thô. Trang gốc có thể đã gỡ tin.
- **Chịu lỗi tốt hơn.** AI lỗi hoặc vượt giới hạn gọi thì việc ở lại hàng đợi để thử lại; crawler không phải chạy lại.
- **Kiểm soát chi phí.** Worker chạy theo lô, giới hạn số lượt AI mỗi giờ, và bỏ bản trùng theo hash trước khi tốn token.
- **Tách quyền truy cập.** Kho thô chứa CV còn đầy đủ thông tin cá nhân, nên chỉ worker được đọc. Kho tinh đã ẩn thông tin cá nhân, đội vận hành và engine tìm kiếm chỉ dùng kho này.
- **Có chỗ đặt cổng đồng ý.** Worker chỉ xử lý CV khi đồng ý còn hiệu lực. Ứng viên rút đồng ý thì xóa ở cả hai kho.

## 3. Crawler JD

### 3.1 Nguồn, theo thứ tự ưu tiên

| # | Nguồn | Cách lấy | Ghi chú |
| --- | --- | --- | --- |
| 1 | Trang tuyển dụng của công ty dùng ATS có API công khai | Greenhouse `boards-api.greenhouse.io/v1/boards/{công_ty}/jobs`, Lever `api.lever.co/v0/postings/{công_ty}` | JSON sạch, các API làm ra để chia sẻ tin tuyển, không cần trình duyệt. Hợp với công ty global/remote mà nhóm B nhắm tới |
| 2 | Trang việc làm IT trong nước (ITviec, TopDev) | Playwright | Đọc `robots.txt` và điều khoản từng trang trước; ưu tiên hỏi hợp tác |
| 3 | Trang việc làm tổng hợp (TopCV, VietnamWorks, CareerViet) | Playwright | Như mục 2; kiểm điều khoản trước khi lưu hàng loạt ([phap-ly.md](phap-ly.md) mục 6) |
| 4 | JD công ty hoặc người dùng tự dán | Form | Nguồn sạch nhất, dùng cho luồng công ty (V2) |
| — | LinkedIn Jobs | Không cào | Điều khoản LinkedIn cấm quét tự động. Công ty tự dán, hoặc người của mình chép tay từng JD |

### 3.2 Quy tắc chạy

- Một bộ đọc (adapter) riêng cho mỗi nguồn; thêm nguồn là thêm adapter, không sửa lõi.
- Giới hạn tốc độ theo từng tên miền, ghi rõ user-agent của mình, tôn trọng `robots.txt`.
- Mỗi lần chạy ghi lại `source_url`, `fetched_at`, `content_hash`. Hash không đổi thì không đưa vào hàng đợi.
- Chạy lại định kỳ để cập nhật trạng thái "còn hiển thị". Đây là dữ liệu cho tính năng 7 (kiểm tra JD còn tuyển, [PRD.md](PRD.md) mục 4). Không kiểm tra được thì ghi "chưa rõ", không ghi "đã đóng".
- Trang đổi giao diện làm adapter hỏng thì báo lỗi và dừng adapter đó, không ghi dữ liệu rỗng vào kho.

## 4. Nhận CV có đồng ý

| Nguồn | Đầu vào | Đồng ý | Giai đoạn |
| --- | --- | --- | --- |
| Tool quét gap miễn phí | PDF/DOCX hoặc văn bản dán vào | Ô bắt buộc cho lượt quét; ô thứ hai để giữ trong kho ([ho-so-ung-vien.md](ho-so-ung-vien.md) mục 5.2) | MVP |
| PDF LinkedIn | File từ nút "Save to PDF" trên profile | Như trên | MVP |
| Form ứng tuyển hộ | CV ứng viên gửi sau khi được nhắn tin | Hai ô đồng ý | Sau khi có giấy phép dịch vụ việc làm |
| ATS hoặc kho CV của công ty | File hoặc export từ ATS | Công ty chịu trách nhiệm về đồng ý của ứng viên với họ; hồ sơ chỉ dùng cho công ty đó, không vào kho chung | V2 |

Mỗi file CV vào kho thô kèm bản ghi đồng ý: nguồn, thời điểm, nội dung ô đã tick, hạn. Không có bản ghi đồng ý thì không đưa vào hàng đợi.

## 5. Lưu trữ

| Lớp | Lưu ở | Nội dung | Chính sách |
| --- | --- | --- | --- |
| Kho thô | Object storage tương thích S3 (MinIO tự host, hoặc S3 của nhà cung cấp cloud trong nước) | File gốc PDF/DOCX/HTML/JSON, khóa theo hash | CV: mã hóa, chỉ worker đọc được, xóa theo yêu cầu và khi hết hạn đồng ý. JD: giữ lâu |
| Kho tinh | PostgreSQL | `jd`, `jd_requirement`, `candidate_profile`, `lead`; thêm pgvector khi cần | Đã ẩn thông tin cá nhân; mỗi bản ghi có `engine_version`, `catalog_version` |
| Hàng đợi | PostgreSQL | Bảng job: loại, khóa file thô, trạng thái, số lần thử, lỗi gần nhất | Xóa job đã xong sau một thời gian |
| Nhật ký AI | PostgreSQL | Model, phiên bản prompt, số token, thời gian của mỗi lượt gọi | Theo thời hạn Luật Trí tuệ nhân tạo yêu cầu ([phap-ly.md](phap-ly.md) mục 4) |

Đặt kho thô chứa CV ở đâu phụ thuộc vào câu đang chờ luật sư: có được đưa dữ liệu cá nhân ra nước ngoài không ([phap-ly.md](phap-ly.md) mục 2). Chưa có câu trả lời thì đặt ở máy chủ trong nước. Chỉ văn bản đã ẩn thông tin cá nhân mới được gửi sang API AI.

## 6. Hàng đợi và worker

MVP không cần Kafka. Dùng bảng job trong PostgreSQL, worker lấy việc bằng `SELECT … FOR UPDATE SKIP LOCKED` (hoặc thư viện có sẵn như pg-boss, JobRunr), nên không thêm hệ thống mới phải vận hành.

Vòng đời một job:

1. `queued`: luồng thu thập vừa ghi file thô.
2. `running`: worker đã nhận; quá thời gian thì trả về `queued`.
3. `done`: đã ghi kho tinh.
4. `failed`: thử quá số lần cho phép (vd. 3); chờ người xem.
5. `skipped`: bản trùng theo hash, hoặc CV không còn đồng ý.

Job phải chạy lại được nhiều lần mà kết quả không đổi: ghi kho tinh theo khóa `content_hash` + `engine_version`, chạy lại thì ghi đè đúng bản ghi đó.

**Xử lý lại hàng loạt:** khi `catalog_version` hoặc `engine_version` đổi, tạo job cho các bản ghi cũ theo lô và theo giới hạn chi phí. Bản ghi cũ vẫn dùng được nhưng có dấu "cũ" cho tới khi xử lý xong.

## 7. Ngôn ngữ

| Phần | Đề xuất | Lý do |
| --- | --- | --- |
| Worker xử lý Layer 0 (đọc PDF/DOCX, CV 2 cột) | Python | Thư viện đọc PDF mạnh nhất: PyMuPDF, pdfplumber, docling. Đây là phần khó nhất của Layer 0 |
| Crawler JD | Python + Playwright | Nhiều trang việc làm trong nước render bằng JS; viết chung Python với worker thì bớt một ngôn ngữ |
| API, tìm kiếm, nghiệp vụ | Ngôn ngữ chính của team (Kotlin/Spring Boot hoặc TypeScript) | Truy vấn SQL, tính điểm, phân quyền, thanh toán |

Hai phần nói chuyện qua hàng đợi trong PostgreSQL, không gọi trực tiếp nhau. Nếu team muốn một ngôn ngữ duy nhất thì chọn Python cho cả hai, vì đọc PDF là phần khó thay nhất.

## 8. Bộ chỉ số

Các ngưỡng là đề xuất, cần team chốt.

| Chỉ số | Ngưỡng đề xuất |
| --- | --- |
| Rò rỉ thông tin cá nhân từ kho thô sang kho tinh hoặc sang API AI | 0 |
| CV được xử lý khi không có bản ghi đồng ý còn hạn | 0 |
| Yêu cầu xóa dữ liệu được xóa ở cả hai kho | 100%, trong thời hạn chính sách quyền riêng tư cam kết |
| Job `failed` trên tổng số job | ≤ 2% |
| Thời gian từ lúc nhận CV tới khi có hồ sơ (p95) | < 60 giây |
| Adapter JD hỏng được phát hiện | Trong lần chạy kế tiếp |

## 9. Thứ tự làm

1. **Bảng job và kho thô**, kèm bản ghi đồng ý cho CV.
2. **Worker Layer 0–1 bằng Python** ghi vào kho tinh; dùng lại prompt của bản demo.
3. **Nhận CV từ tool miễn phí và PDF LinkedIn.**
4. **Adapter JD đầu tiên:** Greenhouse và Lever (JSON, ít rủi ro nhất).
5. **Xóa dữ liệu theo yêu cầu** ở cả hai kho.
6. **Adapter cho trang việc làm trong nước**, sau khi kiểm điều khoản từng trang.

## 10. Câu hỏi cần bàn

Tick khi đã chốt và ghi câu trả lời ngay dưới.

- [ ] Kho thô đặt ở đâu: MinIO tự host hay S3 của nhà cung cấp cloud trong nước?
- [ ] Giữ file CV gốc bao lâu sau khi đã có hồ sơ: tới khi hết hạn đồng ý, hay xóa sớm hơn và chỉ giữ văn bản đã ẩn thông tin cá nhân?
- [ ] Trang việc làm trong nước nào hỏi hợp tác trước thay vì cào?
- [ ] Backend chính viết bằng Kotlin/Spring Boot hay TypeScript?
- [ ] CV từ ATS của công ty: có cần hợp đồng xử lý dữ liệu riêng với từng công ty không? (bổ sung vào [phap-ly.md](phap-ly.md) mục 8)

## Nguồn

- [ho-so-ung-vien.md](ho-so-ung-vien.md), mục 3, 5, 6
- [engine.md](engine.md), mục 2, 4
- [PRD.md](PRD.md), mục 4 (tính năng 7, 14), mục 5
- [phap-ly.md](phap-ly.md), mục 2, 4, 6, 8
