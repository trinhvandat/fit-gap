# Chiến lược sản phẩm: engine chạy như một headhunter

08/11/2026 · Leonard · Bản nháp để thảo luận

Engine được hình dung như một headhunter: thấy JD của công ty, tìm ứng viên phù hợp, nộp vào công ty và nhận phí khi tuyển được; lâu dần mới thành đối tác tuyển dụng của từng công ty. File này ghi lại đánh giá mô hình đó từ nhiều góc nhìn và các giai đoạn đề xuất. Mô hình này là "Luồng ứng tuyển hộ" đã chốt ở [PRD.md](PRD.md) mục 5; kho hồ sơ và phễu tìm ứng viên ở [ho-so-ung-vien.md](ho-so-ung-vien.md).

## 1. Mô hình và điều kiện để có tiền

Phí do công ty trả, không thu ứng viên (PRD mục 5 và 8).

**Điều kiện:** công ty chưa ký với mình thì không có nghĩa vụ trả phí. Nộp CV vào một JD công khai khi chưa có thỏa thuận thì theo thông lệ, CV đó bị coi như ứng viên tự nộp; nhiều công ty ghi rõ trong chính sách là không trả phí cho CV agency gửi khi chưa được mời. Vì vậy:

- Phải ký thỏa thuận phí **trước khi** nộp CV.
- JD cào về ([thu-thap-du-lieu.md](thu-thap-du-lieu.md)) không phải đầu vào để nộp CV ngay, mà là **danh sách công ty để đi chào**: JD nào trong kho đang có nhiều người khớp thì chào công ty đó trước.
- Nộp CV cho công ty để thu phí là dịch vụ việc làm, cần giấy phép và ký quỹ ([PRD.md](PRD.md) mục 12, [phap-ly.md](phap-ly.md) mục 5). Trước khi có giấy phép hoặc đối tác có giấy phép thì chưa thu phí từ công ty.

## 2. Đánh giá từ sáu góc nhìn

| Góc nhìn | Lo nhất | Gợi ý |
| --- | --- | --- |
| Trưởng phòng tuyển dụng (người trả tiền) | CV từ agency lạ, chưa ký thì bỏ qua. Đã ký thì chấm agency theo tốc độ, độ đúng người, và không trùng với ứng viên đã có. CV "đã sửa theo JD" dễ bị nghi là tô vẽ | Chào bằng hồ sơ ẩn danh kèm fit report có câu trích. Ghi rõ "chỉnh câu chữ, không thêm dữ kiện" và có sẵn CV gốc khi được hỏi |
| Ứng viên | CV bị nộp lung tung, nhất là vào công ty đang làm; JD giả; bị thu phí | Chỉ nộp khi ứng viên đồng ý từng JD (PRD đã có). Thêm danh sách công ty ứng viên không muốn bị nộp vào. Tin nhắn mời ghi rõ lương và tên công ty, hoặc ít nhất ngành và quy mô. Gap report miễn phí là lý do để họ trả lời |
| Headhunter lâu năm | Tiền nằm ở quan hệ với công ty và ở khâu chốt: đàm phán lương, giữ ứng viên tới ngày đi làm, xử lý khi công ty cũ tăng lương giữ người. Tranh chấp khi ứng viên đã tự nộp hoặc agency khác đã nộp. Bảo hành khi người mới nghỉ trong thời gian thử việc | Engine lo tìm, lọc, viết; người của mình chỉ lo khâu chốt. Ghi nhật ký nộp để chứng minh quyền sở hữu ứng viên; hợp đồng có thời hạn sở hữu (thường 6–12 tháng) và điều khoản bảo hành |
| Pháp lý | Thu phí khi chưa có giấy phép dịch vụ việc làm. Đổi nguồn tìm ứng viên để né điều khoản LinkedIn | Đi qua đối tác có giấy phép hoặc tự xin (PRD mục 12). Luật bảo vệ dữ liệu cá nhân áp lên dữ liệu, không lên nền tảng: xem và liên hệ từng người thì được, cào và lưu khi chưa đồng ý thì không ([ho-so-ung-vien.md](ho-so-ung-vien.md) mục 5.1) |
| Tài chính | Từ lúc nộp tới lúc hết thử việc mất 1–3 tháng; cần vốn lưu động cộng tiền ký quỹ; rủi ro hoàn phí | Đo chi phí cho mỗi người tuyển được (giờ recruiter + chi phí AI). Doanh thu B2B có thể tới sớm hơn freemium và nuôi được tool miễn phí |
| Product và kỹ thuật | Engine mới có chiều "JD → K người" | Thêm chiều "ứng viên → JD đang mở khớp nhất" và "JD cào về → công ty đáng chào". Kho JD cần trường hợp đồng, phí, nhật ký nộp. Kết quả tuyển dụng thật (phỏng vấn, offer, đi làm, qua thử việc) là nhãn tốt nhất để đo engine |

## 3. Ba giai đoạn

| Giai đoạn | Làm gì | Doanh thu |
| --- | --- | --- |
| 0. Chưa có giấy phép | Tool miễn phí làm đầy kho; crawler JD; ứng viên tự nộp bằng CV đã sửa, mình chỉ ghi lại kết quả. Chọn 1–2 nhánh hẹp (mục 4) | Chỉ B2C, nếu bật trả phí |
| 1. Có giấy phép hoặc có đối tác | Từ JD cào về, chọn công ty có nhiều người khớp trong kho; chào bằng hồ sơ ẩn danh; ký thỏa thuận phí khi tuyển được; rồi mới nộp. Ưu tiên các sàn nơi công ty đã công bố phí trước, vì không phải tự đi chào | Phí khi tuyển thành công |
| 2. Phục vụ từng công ty | Công ty đã tuyển thành công vài người qua mình thì chuyển sang hợp đồng trọn gói; nhập kho CV từ ATS của họ; fit report phía công ty (V2) | Hợp đồng trọn gói, B2B |

## 4. Bắt đầu khi chưa có công ty nào

Đến 08/11/2026 chưa có công ty nào sẵn sàng ký phí, nên giai đoạn 0 phải tạo ra lý do để công ty đầu tiên chịu ký.

- **Chọn nhánh hẹp theo số liệu.** Crawler JD cho biết nhánh nào đang có nhiều JD mở (cầu). Tool miễn phí cho biết nhánh nào có nhiều ứng viên vào kho (cung). Chọn nhánh cả hai cùng cao, vd. Backend Java/Go cấp mid.
- **Không nhận đơn lớn ở nhánh kho còn mỏng.** Dồn tool miễn phí và nội dung vào nhánh đã chọn tới khi kho đủ dày.
- **Mang bằng chứng khi đi chào:** "kho đang có N người khớp JD của anh/chị", kèm 3–5 fit report đã ẩn tên.
- **Mục tiêu đầu là kết quả, chưa phải tiền.** 3–5 người tuyển thành công đầu tiên quan trọng hơn mức phí; vài ca đầu có thể nhận phí thấp hoặc thử miễn phí để đổi lấy số liệu và lời giới thiệu.

## 5. Cam kết với công ty

Cam kết "50 CV gửi đi" hay "50 người được tuyển" do sale chốt theo từng khách. Hai cách chênh nhau nhiều: giả định cứ 5–8 CV gửi đi có 1 người được tuyển, thì 50 người được tuyển cần khoảng 250–400 CV đạt chuẩn. Tỷ lệ thật phải đo bằng phễu ([ho-so-ung-vien.md](ho-so-ung-vien.md) mục 7).

**Mặc định đề xuất cho sale khi chưa có số liệu:**

- Thỏa thuận thu phí khi tuyển thành công, không cam kết số lượng.
- Giao CV theo đợt: đợt 1 lấy từ kho trong 60 giây, các đợt sau từ phễu tìm thêm, không chờ đủ mới gửi.
- Trước khi báo giá hay hẹn thời gian cho đơn lớn, cho engine ước lượng: số người khớp trong kho cộng tỷ lệ phễu đã đo. Khi chưa có tỷ lệ phễu thật, ước lượng chỉ dùng nội bộ, chưa đưa cho khách.

**Kho có sẵn ra được bao nhiêu (minh họa, các tỷ lệ là giả định):**

```
10.000 hồ sơ trong kho
 × 20%  là Backend Java        → 2.000
 × 50%  đúng cấp bậc           → 1.000
 × 40%  đang mở (open)         →   400
 × 50%  qua yêu cầu bắt buộc   →   200
 × 60%  đồng ý nộp JD này      →  ~120 CV
```

Kho 10.000 hồ sơ đủ cho một yêu cầu 50 CV; kho 1.000 hồ sơ chỉ ra khoảng 12 người. Phần thiếu bù bằng các nguồn ở [ho-so-ung-vien.md](ho-so-ung-vien.md) mục 6, xếp theo tốc độ: nhóm "Cần hỏi" (1–3 ngày), ứng viên tự tìm tới và giới thiệu (vài ngày), phễu nhắn tin thủ công (1–3 tuần; minh họa: nhắn 1.000 người → 20% trả lời → 40% gửi CV → 50% khớp, ra khoảng 40 CV).

## 6. Engine cần thêm gì

| Việc | Vì sao | Chi phí AI |
| --- | --- | --- |
| Đếm số người khớp theo từng tiêu chí | Cho công ty thấy kho có bao nhiêu người ở từng mức, vd. "tiêu chí hiện tại 12 người; hạ Spring Boot từ 3 xuống 2 năm thì 31 người". Công ty tự quyết nới tiêu chí nào | 0, câu đếm SQL trên `skill_summary` |
| Xếp JD cào về theo số người khớp trong kho | Chọn công ty để đi chào | 0 |
| Ứng viên → các JD đang mở khớp nhất | Bước 3 của luồng ứng tuyển hộ (PRD mục 5) | Theo Layer 2–3 |
| Trường hợp đồng, mức phí, nhật ký nộp trong kho JD | Chỉ nộp JD đã ký; chứng minh quyền sở hữu ứng viên | 0 |
| Ghi kết quả tuyển dụng | Nhãn để đo engine và tỷ lệ phễu để báo giá | 0 |

## 7. Câu hỏi cần bàn

Tick khi đã chốt và ghi câu trả lời ngay dưới.

- [ ] Giấy phép: đi qua đối tác đã có giấy phép hay tự xin (PRD mục 12)? Đây là cửa phải qua trước giai đoạn 1.
- [ ] Mức phí: % lương năm hay số tháng lương? Cần khảo sát phí headhunt IT trong nước (PRD mục 12).
- [ ] Nhánh hẹp đầu tiên: chọn theo số liệu cung và cầu ở mục 4, hay theo quan hệ có sẵn của team?
- [ ] Thời hạn sở hữu ứng viên và điều khoản bảo hành trong mẫu hợp đồng với công ty.
- [ ] Có dùng các sàn mà công ty công bố phí trước không, và sàn nào?
- [x] Chưa có công ty nào sẵn sàng ký phí; bắt đầu từ giai đoạn 0. Ghi nhận 08/11/2026.
- [x] Cam kết số lượng với công ty do sale chốt theo từng khách; mặc định ở mục 5. Ghi nhận 08/11/2026.

## Nguồn

- [PRD.md](PRD.md), mục 5, 8, 12
- [ho-so-ung-vien.md](ho-so-ung-vien.md), mục 4–7
- [thu-thap-du-lieu.md](thu-thap-du-lieu.md), mục 1, 3
- [phap-ly.md](phap-ly.md), mục 2, 5, 6
