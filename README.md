# WiFi-Cracking

> **Chỉ dành cho học tập, phòng lab, CTF hoặc kiểm thử mạng/thiết bị thuộc quyền sở hữu hay được cho phép rõ ràng.** Một số công cụ trong nguồn có thể gây mất kết nối hoặc liên quan kiểm tra mật khẩu, WPS và mô phỏng điểm truy cập giả; không sử dụng trái phép.

## Cấu trúc ZIP (README ở thư mục gốc)

```text
README.md
01_TOAN_BO_LENH_GOC.txt
02_HUONG_DAN_CHI_TIET_TUNG_LENH.md
```

| Tệp | Nội dung | Khi nào đọc |
|---|---|---|
| `README.md` | Mục tiêu, phạm vi, yêu cầu, mục lục, nguyên tắc an toàn | Đọc đầu tiên |
| `01_TOAN_BO_LENH_GOC.txt` | Gộp toàn bộ **37 dòng lệnh** từ 9 chủ đề, có chú thích theo nhóm; không biến thành script thực thi | Muốn tìm lệnh nhanh |
| `02_HUONG_DAN_CHI_TIET_TUNG_LENH.md` | Giải thích **từng dòng lệnh**, gồm tác dụng, tham số, cách chạy, kết quả và lưu ý | Muốn học và tra cứu chi tiết |

## 9 chủ đề của dự án

| # | Chủ đề | Nội dung tóm tắt |
|---|---|---|
| 1 | Enable Monitor Mode | Quản lý chế độ quan sát trên card WiFi |
| 2 | Scan for WiFi Networks | Đọc thông tin BSSID, kênh, tên và mức tín hiệu |
| 3 | Deauth a Client to Force Reconnect | Tìm hiểu khung deauthentication và rủi ro gián đoạn |
| 4 | Targeted Capture of Handshake | Lọc và lưu dữ liệu trao đổi xác thực trong lab |
| 5 | Crack the Captured Handshake | Đánh giá ứng viên mật khẩu từ dữ liệu thử nghiệm |
| 6 | Cracking Handshake Faster with Hashcat | Định dạng dữ liệu và chế độ Hashcat cũ/mới |
| 7 | PMKID Attack | Tổng quan dữ liệu PMKID và khả năng tương thích |
| 8 | WPS PIN Attack with Reaver | Giải thích rủi ro WPS PIN và công cụ liên quan |
| 9 | Automated WiFi Attack Tools | Wifite, Airgeddon, Fluxion, WiFi Pumpkin 3 |

## Điều kiện, giới hạn và cách đọc

- **Môi trường tham khảo:** Kali Linux hoặc bản Linux phù hợp; card WiFi và driver hỗ trợ chức năng cần dùng; công cụ chỉ nên lấy từ nguồn đáng tin cậy.
- **Phạm vi:** chỉ dùng trong bài lab/thiết bị được cho phép; hạn chế ảnh hưởng đến người khác, không ghi hoặc thu thập thông tin người ngoài lab.
- **Dấu `<...>`:** ký hiệu tham số cần điền của bản gốc, không phải chuỗi nhập trực tiếp. Các địa chỉ MAC minh họa không phải mục tiêu xác định.
- **Tệp và thiết bị:** `wlan0`, `wlan0mon`, `capture.cap`, `capture-01.cap`, `rockyou.txt` là tên minh họa; không bảo đảm hiện có trên máy.
- **Tính tương thích:** lệnh nguồn có thể cũ (đặc biệt Hashcat `-m 2500`, `cap2hccapx`, `hcxdumptool --enable_status=1`, cài Python qua `setup.py`). **Giữ đúng nguyên bản thay vì tự sửa lệnh.**
- **Cách đọc:** xem `02_HUONG_DAN_CHI_TIET_TUNG_LENH.md`; mỗi lệnh có 5 mục riêng: *Tác dụng*, *Cấu trúc / tham số*, *Cách chạy trong lab*, *Kết quả có thể thấy*, *Lưu ý*.

## Tham khảo

- [Aircrack-ng](https://www.aircrack-ng.org/)
- [Hashcat](https://hashcat.net/hashcat/)
- [TryHackMe](https://tryhackme.com/) / [Hack The Box](https://www.hackthebox.com/) / [OverTheWire](https://overthewire.org/)

*Dự án dành cho nghiên cứu có trách nhiệm; bản repo này chỉ chứa tài liệu, không chứa công cụ hay dữ liệu mạng thực tế.*
