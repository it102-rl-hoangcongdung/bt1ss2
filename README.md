# BÁO CÁO PHÂN TÍCH LỖI LOGIC TÍNH CƯỚC GRABRIDE VÀ SỬA LỖI MÃ NGUỒN C

> 👤 **Học viên:** Hoàng Công Dũng | **Mã SV:** PTIT-HCM-009
> 🏫 **Môn học:** it102-rl-hoangcongdung

---

## 📊 Sơ đồ thiết kế hệ thống (Activity Diagram)

> 💡 *Sơ đồ dưới đây được render tự động trực tiếp trên GitHub bằng Mermaid. Bạn cũng có thể tải file **`bt1.drawio`** trong repository này để mở và chỉnh sửa trực tiếp trên [Draw.io (diagrams.net)](https://app.diagrams.net).* 

```mermaid
graph TD
  Start([Bắt đầu chuyến đi]) --> Input[Nhập quãng đường & trạng thái thời tiết]
  Input --> CheckDist{Quãng đường <= 2km?}
  CheckDist -- Đúng --> BaseFare[Cước phí = 12000 VND]
  CheckDist -- Sai --> CalcExtra[Cước phí = 12000 + (distance - 2) * 4500]
  BaseFare --> CheckRain{Trời có mưa không? is_raining == 1?}
  CalcExtra --> CheckRain
  CheckRain -- Có mưa --> AddRain[Tổng cước = Tổng cước * 1.2]
  CheckRain -- Không mưa --> KeepFare[Giữ nguyên tổng cước]
  AddRain --> Display[In tổng cước phí chuyến xe]
  KeepFare --> Display
  Display --> End([Kết thúc])
```

---

## Nhiệm vụ 1: Phân tích lỗi & Lập bảng Test Cases

Sau khi đọc kỹ mã nguồn legacy của hệ thống tính cước GrabRide, mình đã tìm ra 2 lỗi logic cực kỳ nghiêm trọng khiến hệ thống tính sai tiền của khách hàng:

Thứ nhất, tại dòng lệnh kiểm tra thời tiết 'if (is_raining = 1)', lập trình viên đã dùng toán tử gán '=' thay vì toán tử so sánh '==' . Điều này làm cho biến 'is_raining' luôn nhận giá trị 1 bất kể người dùng nhập vào số 0. Hậu quả là mọi chuyến xe trong điều kiện nắng ráo đều bị tính oan 20% phụ phí thời tiết xấu.

Thứ hai, ở nhánh 'else' tính cước cho quãng đường lớn hơn 2 km, câu lệnh 'total_fare = distance * 4500.0' đã nhân trực tiếp tổng số km với 4500. Điều này làm mất trắng phần cước phí cố định 12.000 VNĐ của 2 km đầu tiên, đồng thời tính tiền nhầm cho cả 2 km đầu với đơn giá 4.500 VNĐ. Ví dụ đi 3 km thì giá hiển thị chỉ có 13.500 VNĐ thay vì 16.500 VNĐ đúng chuẩn nghiệp vụ.

Dưới đây là bảng 02 Test Cases đối chứng chi tiết trước và sau khi sửa lỗi:

- Luôn chú ý phân biệt rõ toán tử so sánh '==' và toán tử gân '=' trong điều kiện if.
- Công thức cước lũy tiến phải tách bạch rõ phần phí cơ bản và phần phí vượt trội.

| Trường hợp kiểm thử | Dữ liệu đầu vào | Kết quả sai thực tế | Kết quả đúng mong đợi |
| --- | --- | --- | --- |
| TC01: Nắng ráo, đi ngắn | distance = 1.5 km, is_raining = 0 | 14.400 VND (Bị tính oan 20% phụ phí mưa) | 12.000 VND (Đúng giá trọn gói 2km đầu) |
| TC02: Nắng ráo, đi dài | distance = 3.0 km, is_raining = 0 | 13.500 VND (Sai công thức, mất cước 2km đầu) | 16.500 VND (12000 + (3.0 - 2) * 4500) |
| TC03: Có mưa, đi dài | distance = 4.0 km, is_raining = 1 | 24.000 VND (Dùng công thức sai nhân thêm 1.2) | 24.600 VND ((12000 + 2 * 4500) * 1.2) |

## Nhiệm vụ 2: Sửa lỗi mã nguồn

Mình đã tiến hành viết lại toàn bộ mã nguồn vào tệp 'grab_fare_debug.c' lưu trong thư mục 'session02_debug/' của repository 'it102-rl-hoangcongdung'.

Mã nguồn đã được biên dịch thành công bằng trình biên dịch GCC, xử lý triệt để lỗi gán nhầm biến và chuẩn hóa lại công thức tính cước lũy tiến đúng 100% theo yêu cầu nghiệp vụ.

- Sửa 'if (is_raining = 1)' thành 'if (is_raining == 1)'.
- Sửa 'else { total_fare = distance * 4500.0; }' thành 'else { total_fare = 12000.0 + (distance - 2.0) * 4500.0; }'.
- Đảm bảo tuân thủ đúng các kiểu dữ liệu float và double, định dạng in ra màn hình chính xác.

---

## 📁 Danh sách tệp tin nộp bài trong Repository
- 📝 `bt1.docx`: Báo cáo tài liệu phân tích nghiệp vụ hoàn chỉnh.
- 🎨 `bt1.drawio`: File thiết kế sơ đồ chuẩn theo quy định đề bài (mở trực tiếp bằng [Draw.io](https://app.diagrams.net) hoặc Lucidchart).
- 💻 `session02_debug/grab_fare_debug.c`: Mã nguồn chương trình.
