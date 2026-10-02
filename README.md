# 2FASecret

Ứng dụng xác thực hai lớp (TOTP) chạy hoàn toàn trên trình duyệt — một tệp `index.html` duy nhất, không cần
server, không gửi secret đi đâu cả.

## Tính năng

| Khu vực | Mô tả |
| --- | --- |
| **01 / Xử lý hàng loạt** | Dán danh sách secret (hoặc `Tên\|SECRET`) → sinh mã TOTP cho từng dòng. |
| **02 / Không gian xác thực** | 20 ô secret (thêm/bớt tuỳ ý), mỗi ô có **Ghi chú** riêng, mã TOTP tự cập nhật, đồng hồ đếm ngược, tuỳ chọn 6/7/8 chữ số và chu kỳ 30s/60s. |
| **03 / Sao lưu & Khôi phục** | 🆕 Xuất toàn bộ **secret + ghi chú** ra tệp `.txt` để cất giữ, và nhập lại từ tệp backup cũ. |

Toàn bộ phép tính HMAC‑SHA1 / RFC 6238 chạy offline trong trình duyệt. Dữ liệu chỉ được lưu vào
`localStorage` khi bạn tự bật tuỳ chọn **Nhớ secret**.

## Sao lưu (Backup)

Mở mục **03 / SAO LƯU & KHÔI PHỤC** → panel bên trái, hoặc bấm nút **⤓ Sao lưu** trên thanh công cụ.

* **Định dạng `otpauth://` (mặc định)** — chuẩn quốc tế, nhập được vào Google Authenticator, Authy,
  Aegis, Bitwarden… Ghi chú được tách thành `issuer` + tên tài khoản:

  ```
  # 2FA.LIVE BACKUP v1 (otpauth)
  # Thoi gian: 2026-10-02T04:20:00.000Z
  # So luong: 3 secret
  otpauth://totp/Google:alice@gmail.com?secret=JBSWY3DPEHPK3PXP&issuer=Google&digits=6&period=30&algorithm=SHA1
  otpauth://totp/Facebook?secret=GEZDGNBVGY3TQOJQGEZDGNBVGY3TQOJQ&digits=6&period=30&algorithm=SHA1
  ```

* **Định dạng text (dễ đọc)** — mỗi dòng `Ghi chú|Secret`, ô không có ghi chú chỉ ghi secret:

  ```
  Google · alice@gmail.com|JBSWY3DPEHPK3PXP
  GEZDGNBVGY3TQOJQGEZDGNBVGY3TQOJQ
  ```

Bấm **Tạo bản sao lưu** để xem trước, **Tải tệp .txt** để lưu về máy, hoặc **Sao chép** để dán vào
trình quản lý mật khẩu. Ở định dạng `otpauth`, chữ số và chu kỳ riêng của từng ô (nếu có) cũng được giữ lại.

> ⚠️ Tệp backup chứa secret nên bất kỳ ai có tệp đều tạo được mã 2FA của bạn — hãy cất ở nơi an toàn
> (USB riêng, trình quản lý mật khẩu, ổ đĩa mã hoá).

## Khôi phục (Import / Restore)

Panel bên phải của mục **03**, hoặc nút **⤒ Khôi phục** trên thanh công cụ. Dán trực tiếp vào ô nhập,
**chọn tệp**, hoặc **kéo‑thả tệp** vào panel.

Định dạng được nhận diện tự động:

| Định dạng | Ví dụ |
| --- | --- |
| `otpauth://` | `otpauth://totp/Tên?secret=ABC...&issuer=...&digits=6&period=30` |
| `Ghi chú\|Secret` | `Facebook · Long\|JBSWY3DPEHPK3PXP` |
| `Ghi chú\|Secret\|123456` | Dòng copy từ nút **Kèm ghi chú** (mã OTP ở cuối được bỏ qua) |
| Secret thuần | `JBSWY3DPEHPK3PXP`, `jbsw y3dp ehpk 3pxp`, có/không dấu `=` đệm |
| `otpauth-migration://` | Mã xuất từ **Google Authenticator** (giải mã protobuf, hỗ trợ nhiều tài khoản) |
| JSON | Tệp JSON chứa `{secret, name/note/issuer}` hoặc dữ liệu `localStorage` cũ của chính app này |

Tuỳ chọn khi khôi phục:

* **Chế độ**: `Gộp vào danh sách` (mặc định) hoặc `Thay thế toàn bộ` (có hỏi xác nhận).
* **Bỏ qua secret trùng**: secret đã có sẽ không thêm lại, nhưng vẫn có thể bổ sung ghi chú còn thiếu.
* **Giữ ghi chú đang có**: không ghi đè ghi chú hiện tại bằng ghi chú trong tệp backup.

Ô nhập hiển thị trước số lượng nhận diện được (`Nhận diện: 5 secret · 2 trùng · sẽ thêm 3`) và báo rõ
những dòng không hiểu, nên bạn luôn biết mình sắp nhập gì.

## Mẹo sử dụng

* **Dán nhiều dòng vào một ô** ở mục 02: tự động chia ra các ô tiếp theo và **tách luôn ghi chú**
  (`Tên|SECRET`, `otpauth://…` đều được hiểu).
* Nút **Chia vào các ô ↓** ở mục 01 làm điều tương tự cho danh sách đã dán.
* Ô nào được import với `digits`/`period` riêng sẽ được tô sáng ở cột số thứ tự; đổi bộ chọn
  *Số chữ số* / *Chu kỳ* trên thanh công cụ sẽ áp dụng lại cho toàn bộ các ô.
* Nhấn vào mã 6 số để copy nhanh.

## Chạy thử

Mở trực tiếp `index.html` bằng trình duyệt (double‑click), hoặc:

```bash
python3 -m http.server 8080   # rồi mở http://localhost:8080/index.html
```

Không cần cài đặt, build hay kết nối mạng.
