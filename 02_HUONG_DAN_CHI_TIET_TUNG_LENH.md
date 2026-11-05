# HƯỚNG DẪN CHI TIẾT TỪNG LỆNH — WIFI-CRACKING

> **Phạm vi:** Tài liệu tham khảo phục vụ học và kiểm thử WiFi **trên thiết bị/mạng lab thuộc quyền quản lý hoặc được ủy quyền**. Nội dung không phải quy trình áp dụng trên mạng công cộng hay mạng của người khác.

## Chuẩn bị cho bài lab được phép

Hệ điều hành ví dụ của nguồn là Kali Linux; cần card WiFi/driver phù hợp và công cụ liên quan được cài từ nguồn đáng tin cậy. Trước khi áp dụng bất cứ lệnh nào, xác định phạm vi thử nghiệm, kiểm tra tên giao diện và đường dẫn tệp thực tế. Những thao tác cần quyền quản trị hoặc có ảnh hưởng đến dịch vụ mạng nên thực hiện trên máy lab riêng, không phải máy làm việc chính.

### Thuật ngữ và ký hiệu

| Ký hiệu | Ý nghĩa |
|---|---|
| `BSSID` | Địa chỉ MAC xác định điểm truy cập (AP). |
| `ESSID` | Tên mạng WiFi phát quảng bá. |
| `CH` | Kênh vô tuyến của điểm truy cập. |
| `wlan0` / `wlan0mon` | Tên giao diện WiFi ví dụ trước/sau khi chuyển chế độ. |
| `handshake` / `EAPOL` | Trao đổi phục vụ xác thực/thiết lập khóa trong một số phiên kết nối. |
| `PMKID` | Dữ liệu định danh khóa liên quan một số cơ chế xác thực WPA. |
| `wordlist` | Tệp văn bản chứa các giá trị mật khẩu ứng viên phục vụ kiểm toán. |
| `.cap`, `.pcapng`, `.hccapx`, `.22000` | Tệp ghi lưu lượng/định dạng xử lý; công cụ và phiên bản quyết định khả năng đọc. |

## Danh mục

1. Enable Monitor Mode (6 dòng lệnh)
2. Scan for WiFi Networks (1 dòng lệnh)
3. Deauth a Client to Force Reconnect (2 dòng lệnh)
4. Targeted Capture of Handshake (2 dòng lệnh)
5. Crack the Captured Handshake (3 dòng lệnh)
6. Cracking Handshake Faster with Hashcat (GPU) (6 dòng lệnh)
7. PMKID Attack (No Clients Required!) (3 dòng lệnh)
8. WPS PIN Attack with Reaver (3 dòng lệnh)
9. Automated WiFi Attack Tools (11 dòng lệnh)

## 01. Giới thiệu monitor mode

**Tên file nguồn:** `Enable Monitor Mode` và `Enable Monitor Mode.md`.

Monitor mode cho phép bộ điều hợp tiếp nhận khung WiFi trên kênh phù hợp. Nên phân biệt việc quan sát tín hiệu với quyền truy cập dữ liệu. Phần này tập trung vào kiểm tra giao diện, chuyển chế độ và khôi phục trạng thái.

### 01.01 — `airmon-ng check kill    `

**Lệnh nguyên văn:**

```bash
airmon-ng check kill     # Stop conflicting services
```

**Tác dụng:** Tìm và dừng các tiến trình có thể xung đột với các công cụ WiFi của Aircrack-ng.

**Cấu trúc / tham số:** `airmon-ng` là tiện ích quản lý giao diện; `check kill` kiểm tra và dừng tiến trình gây xung đột. Phần sau dấu `#` chỉ là chú thích của bản gốc.

**Cách chạy trong lab:** Mở terminal trên máy Linux phục vụ kiểm thử và nhập phần lệnh đứng trước `#`. Chỉ chạy khi có thể chấp nhận WiFi hiện tại bị ngắt.

**Kết quả có thể thấy:** Một số dịch vụ quản lý kết nối có thể bị dừng; thiết bị đang dùng mạng sẽ tạm mất kết nối.

**Lưu ý:** Dòng có chú thích tiếng Anh này được giữ nguyên từ file lệnh; lệnh tương đương không có chú thích cũng có mặt ở mục 01.03.

### 01.02 — `airmon-ng start wlan0   `

**Lệnh nguyên văn:**

```bash
airmon-ng start wlan0    # Enables monitor mode (becomes wlan0mon)
```

**Tác dụng:** Chuyển giao diện WiFi sang chế độ monitor để quan sát khung 802.11.

**Cấu trúc / tham số:** `start` bật monitor mode; `wlan0` là tên giao diện ban đầu, còn nội dung sau `#` là chú thích, không phải tham số.

**Cách chạy trong lab:** Kiểm tra giao diện mạng thực tế của thiết bị lab; trên terminal nhập lệnh với đúng tên giao diện tương ứng.

**Kết quả có thể thấy:** Công cụ có thể thông báo giao diện monitor, thường là `wlan0mon` nhưng không phải lúc nào cũng đổi tên.

**Lưu ý:** Cần adapter/driver hỗ trợ; bản có chú thích này và bản 01.04 là hai dòng nguyên gốc khác nhau.

### 01.03 — `airmon-ng check kill`

**Lệnh nguyên văn:**

```bash
airmon-ng check kill
```

**Tác dụng:** Thực hiện lại thao tác kiểm tra và dừng tiến trình gây xung đột.

**Cấu trúc / tham số:** `check` dùng kiểm tra tiến trình; `kill` yêu cầu dừng các tiến trình xung đột.

**Cách chạy trong lab:** Nhập nguyên lệnh trong terminal khi đang quản trị máy thử nghiệm và chấp nhận việc gián đoạn kết nối WiFi.

**Kết quả có thể thấy:** Terminal có thể hiện danh sách tiến trình bị dừng hoặc không có tiến trình phù hợp.

**Lưu ý:** Trùng về chức năng với 01.01 nhưng được giữ lại vì bản gốc có dòng không chú thích.

### 01.04 — `airmon-ng start wlan0`

**Lệnh nguyên văn:**

```bash
airmon-ng start wlan0
```

**Tác dụng:** Bật chế độ monitor cho giao diện WiFi đã chỉ định.

**Cấu trúc / tham số:** `start` bật chế độ; `wlan0` là tên interface ví dụ, phải khớp với máy.

**Cách chạy trong lab:** Chạy trong terminal trên máy lab với adapter tương thích; đổi tên giao diện nếu hệ thống không dùng `wlan0`.

**Kết quả có thể thấy:** Công cụ in thông tin trạng thái và tên giao diện đang ở monitor mode.

**Lưu ý:** Trùng về thao tác với 01.02 nhưng giữ nguyên dòng không có chú thích.

### 01.05 — `iwconfig`

**Lệnh nguyên văn:**

```bash
iwconfig
```

**Tác dụng:** Xem các giao diện WiFi và thông tin cấu hình không dây.

**Cấu trúc / tham số:** `iwconfig` không có đối số trong bản gốc.

**Cách chạy trong lab:** Chạy trực tiếp trong terminal để xác định tên giao diện WiFi của thiết bị của bạn.

**Kết quả có thể thấy:** Hiện tên giao diện, chế độ (managed/monitor) và các thông tin liên quan nếu driver hỗ trợ.

**Lưu ý:** Đây là công cụ cũ; tùy hệ thống có thể chưa được cài đặt.

### 01.06 — `airmon-ng stop wlan0mon`

**Lệnh nguyên văn:**

```bash
airmon-ng stop wlan0mon
```

**Tác dụng:** Đưa giao diện đang ở monitor mode trở về chế độ quản lý thông thường.

**Cấu trúc / tham số:** `stop` tắt monitor mode; `wlan0mon` là tên giao diện ví dụ.

**Cách chạy trong lab:** Sau khi kết thúc phiên kiểm thử lab, dùng tên giao diện monitor thực tế thay cho `wlan0mon` nếu khác.

**Kết quả có thể thấy:** Giao diện monitor được tắt/đưa về chế độ phù hợp; mạng thường có thể cần kết nối lại.

**Lưu ý:** Lệnh này được trích từ nội dung inline của bản gốc, không nằm trong file lệnh lẻ.

---

## 02. Quan sát WiFi trong lab

**Tên file nguồn:** `Scan for WiFi Networks` và `Scan for WiFi Networks.md`.

BSSID là địa chỉ MAC của AP; ESSID là tên hiển thị; CH là kênh; dBm thể hiện mức tín hiệu nhận. Các giá trị hiển thị tùy card và công cụ, không xác nhận quyền sử dụng mạng.

### 02.01 — `airodump-ng wlan0mon`

**Lệnh nguyên văn:**

```bash
airodump-ng wlan0mon
```

**Tác dụng:** Hiển thị các điểm truy cập và thiết bị quan sát được bằng giao diện monitor.

**Cấu trúc / tham số:** `airodump-ng` là công cụ theo dõi khung WiFi; `wlan0mon` là interface đang ở chế độ monitor.

**Cách chạy trong lab:** Trong môi trường thử nghiệm của bạn, chạy lệnh ở terminal sau khi đã kiểm tra interface monitor; nhấn Ctrl+C để dừng.

**Kết quả có thể thấy:** Có thể thấy BSSID (MAC của AP), CH (kênh), ESSID (tên mạng), mức tín hiệu và các trường khác tùy phiên bản.

**Lưu ý:** Chỉ quan sát; không có nghĩa được phép kết nối mạng khác. Tín hiệu tốt không đồng nghĩa thử mật khẩu nhanh.

---

## 03. Cơ chế deauthentication

**Tên file nguồn:** `Deauth a Client to Force Reconnect` và `Deauth a Client to Force Reconnect.md`.

Deauthentication là loại khung quản lý 802.11; việc gửi khung giả mạo có thể làm gián đoạn truy cập. Chỉ phân tích hoặc kiểm thử trên thiết bị lab được cho phép; có thể bị hạn chế bởi PMF/802.11w.

### 03.01 — `aireplay-ng --deauth 10 -a <BSSID> wlan0mon`

**Lệnh nguyên văn:**

```bash
aireplay-ng --deauth 10 -a <BSSID> wlan0mon
```

**Tác dụng:** Mô tả thao tác gửi khung deauthentication tới AP; có thể gây mất kết nối client.

**Cấu trúc / tham số:** `--deauth 10` là số khung theo cú pháp của công cụ; `-a <BSSID>` chỉ AP; `wlan0mon` là interface.

**Cách chạy trong lab:** Đây là lệnh chỉ dành cho AP/client thử nghiệm do bạn sở hữu và đã cho phép tác động. `<BSSID>` là ký hiệu cần thay bằng giá trị trong lab, không phải chuỗi nhập nguyên dấu ngoặc.

**Kết quả có thể thấy:** Nếu phần cứng, chế độ và hệ thống kiểm thử phù hợp, việc gửi khung có thể được ghi nhận; không đảm bảo thiết bị bị ngắt.

**Lưu ý:** Có rủi ro gián đoạn dịch vụ; không lặp hoặc triển khai trên mạng xung quanh. Các AP dùng PMF có thể giảm hiệu quả loại khung này.

### 03.02 — `aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon`

**Lệnh nguyên văn:**

```bash
aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon
```

**Tác dụng:** Ví dụ lệnh deauthentication với địa chỉ BSSID minh họa.

**Cấu trúc / tham số:** `--deauth 10` nêu lượng khung; `-a AA:BB:CC:DD:EE:FF` là địa chỉ ví dụ; `wlan0mon` là interface.

**Cách chạy trong lab:** Ví dụ trong tài liệu là cú pháp tham khảo, không phải thiết bị thực cần tác động. Chỉ mô phỏng trong lab được phép.

**Kết quả có thể thấy:** Kết quả phụ thuộc hỗ trợ driver, AP và cơ chế bảo vệ khung quản lý.

**Lưu ý:** Bản này được giữ riêng với 03.01 vì địa chỉ minh họa là một biến thể gốc.

---

## 04. Lọc và lưu khung xác thực

**Tên file nguồn:** `Targeted Capture of Handshake` và `Targeted Capture of Handshake.md`.

Khi kết nối hợp lệ phát sinh, thiết bị có thể trao đổi khung EAPOL. Bộ ghi capture có thể lọc theo BSSID/kênh; việc tạo tệp .cap không đồng nghĩa dữ liệu đã đủ để phân tích.

### 04.01 — `airodump-ng --bssid <BSSID> --channel <CH> -w capture wlan0mon`

**Lệnh nguyên văn:**

```bash
airodump-ng --bssid <BSSID> --channel <CH> -w capture wlan0mon
```

**Tác dụng:** Lọc theo AP/kênh và ghi các khung WiFi vào tệp dữ liệu thử nghiệm.

**Cấu trúc / tham số:** `--bssid <BSSID>` lọc AP; `--channel <CH>` chọn kênh; `-w capture` đặt tiền tố đầu ra; `wlan0mon` là interface.

**Cách chạy trong lab:** Trên AP lab của bạn, chỉ định BSSID và kênh của thiết bị thử nghiệm, chạy ở terminal; nhấn Ctrl+C để dừng ghi.

**Kết quả có thể thấy:** Có thể tạo `capture-01.cap` và các tệp phụ mang tiền tố `capture`. Bản ghi không nhất thiết chứa handshake hợp lệ.

**Lưu ý:** Không thu thập lưu lượng mạng người khác; một số adapter không có khả năng monitor/channel phù hợp.

### 04.02 — `airodump-ng --bssid AA:BB:CC:DD:EE:FF --channel 6 -w capture wlan0mon`

**Lệnh nguyên văn:**

```bash
airodump-ng --bssid AA:BB:CC:DD:EE:FF --channel 6 -w capture wlan0mon
```

**Tác dụng:** Ví dụ lọc theo địa chỉ AP và kênh minh họa để ghi dữ liệu.

**Cấu trúc / tham số:** `AA:BB:CC:DD:EE:FF` là BSSID mẫu; `6` là số kênh; `-w capture` đặt tiền tố; `wlan0mon` là giao diện.

**Cách chạy trong lab:** Dùng cú pháp ví dụ để đọc tài liệu, chỉ điền thông số AP thuộc lab và không dùng thông số mạng bên ngoài.

**Kết quả có thể thấy:** Sinh tệp ghi capture nếu điều kiện và quyền ghi thư mục cho phép.

**Lưu ý:** Lệnh ví dụ không chứng minh đã bắt được handshake; tránh giả định mọi khung đều giải mã được.

---

## 05. Đánh giá độ mạnh mật khẩu bằng dữ liệu lab

**Tên file nguồn:** `Crack the Captured Handshake` và `Crack the Captured Handshake.md`.

Aircrack-ng có chức năng kiểm tra ứng viên từ wordlist trên dữ liệu handshake phù hợp. Chỉ thực hiện với mật khẩu và dữ liệu do bạn sở hữu hoặc đã được phép kiểm toán.

### 05.01 — `aircrack-ng -w /usr/share/wordlists/rockyou.txt -b <BSSID> capture.cap`

**Lệnh nguyên văn:**

```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b <BSSID> capture.cap
```

**Tác dụng:** Kiểm tra các ứng viên mật khẩu trong một danh sách đối với bản ghi handshake lab.

**Cấu trúc / tham số:** `-w` nhận đường dẫn wordlist; `-b <BSSID>` lọc AP; `capture.cap` là tệp thu thập đã tồn tại.

**Cách chạy trong lab:** Chỉ dùng với tệp handshake/wordlist thuộc hệ thống được phép đánh giá. Trước khi chạy, bảo đảm có tệp tại đường dẫn tương ứng.

**Kết quả có thể thấy:** Công cụ đọc tệp rồi so khớp ứng viên; có thể không tìm ra kết quả dù cú pháp đúng.

**Lưu ý:** Tên `capture.cap` khác `capture-01.cap` ở biến thể kế tiếp; bản gốc dùng cả hai tên.

### 05.02 — `aircrack-ng -w /usr/share/wordlists/rockyou.txt -b <BSSID> capture-01.cap`

**Lệnh nguyên văn:**

```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b <BSSID> capture-01.cap
```

**Tác dụng:** Cùng thao tác thử wordlist nhưng dùng tệp đầu vào `capture-01.cap`.

**Cấu trúc / tham số:** `-w /usr/share/wordlists/rockyou.txt` là tệp ứng viên; `-b <BSSID>` chọn AP; `capture-01.cap` là file capture.

**Cách chạy trong lab:** Trong lab, bảo đảm file capture và wordlist hiện hữu; thay BSSID bằng AP được quyền kiểm tra.

**Kết quả có thể thấy:** Kết quả phụ thuộc chất lượng bản ghi và việc ứng viên có tồn tại trong wordlist.

**Lưu ý:** Đường dẫn `rockyou.txt` không chắc có sẵn hoặc đã giải nén; không nên hiểu tên file là dữ liệu có thật.

### 05.03 — `aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF capture-01.cap`

**Lệnh nguyên văn:**

```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF capture-01.cap
```

**Tác dụng:** Ví dụ đầy đủ về kiểm tra tệp capture với BSSID minh họa.

**Cấu trúc / tham số:** `AA:BB:CC:DD:EE:FF` là BSSID mẫu thay cho `<BSSID>`; các phần còn lại tương tự 05.02.

**Cách chạy trong lab:** Chỉ coi đây là mẫu cú pháp đã có sẵn trong tài liệu và áp dụng với dữ liệu đánh giá do bạn sở hữu/được phép.

**Kết quả có thể thấy:** Nếu tìm thấy ứng viên hợp lệ, công cụ có thể đưa ra thông báo kết quả; thường cũng có trường hợp không tìm thấy.

**Lưu ý:** Đây là biến thể ví dụ riêng của bản gốc, không phải mật khẩu hay AP thật.

---

## 06. Định dạng dữ liệu và xử lý bằng Hashcat

**Tên file nguồn:** `Cracking Handshake Faster with Hashcat (GPU)` và `Cracking Handshake Faster with Hashcat (GPU).md`.

Hashcat hỗ trợ đánh giá danh sách ứng viên trên dữ liệu WPA. Nguồn có đồng thời lệnh định dạng `.hccapx`/mode 2500 cũ và định dạng 22000 mới hơn; chúng không hoàn toàn thay thế nhau.

### 06.01 — `cap2hccapx capture.cap capture.hccapx`

**Lệnh nguyên văn:**

```bash
cap2hccapx capture.cap capture.hccapx
```

**Tác dụng:** Chuyển một bản ghi `.cap` sang định dạng `.hccapx` cũ.

**Cấu trúc / tham số:** `cap2hccapx` là tiện ích chuyển đổi; đối số thứ nhất `capture.cap` là đầu vào, `capture.hccapx` là đầu ra.

**Cách chạy trong lab:** Chỉ chạy trên tệp capture thuộc lab nếu tiện ích này có trong hệ thống; bảo đảm tên đầu vào khớp.

**Kết quả có thể thấy:** Nếu chuyển đổi được, có tệp `.hccapx`; có thể thất bại nếu thiếu dữ liệu hoặc không có công cụ.

**Lưu ý:** Định dạng cũ và tiện ích này có thể không còn được hỗ trợ trong bản phân phối hiện tại.

### 06.02 — `hashcat -m 2500 capture.hccapx rockyou.txt --force`

**Lệnh nguyên văn:**

```bash
hashcat -m 2500 capture.hccapx rockyou.txt --force
```

**Tác dụng:** Dùng Hashcat với chế độ WPA cũ `2500` để đối chiếu các ứng viên.

**Cấu trúc / tham số:** `-m 2500` chọn định dạng WPA-EAPOL cũ; `capture.hccapx` là đầu vào; `rockyou.txt` là wordlist; `--force` bỏ qua một số kiểm tra/cảnh báo.

**Cách chạy trong lab:** Chỉ là lệnh lưu trữ từ tài liệu cũ; sử dụng trong lab nếu phiên bản Hashcat thực sự còn hỗ trợ chế độ này và có tệp đầu vào.

**Kết quả có thể thấy:** Phiên bản mới có thể từ chối chế độ 2500; cảnh báo tương thích không đồng nghĩa nội dung tệp hỏng.

**Lưu ý:** Không khuyến nghị `--force` cho môi trường thực; lệnh vẫn giữ nguyên để bảo toàn nguồn.

### 06.03 — `hcxpcapngtool -o capture.22000 capture.pcapng`

**Lệnh nguyên văn:**

```bash
hcxpcapngtool -o capture.22000 capture.pcapng
```

**Tác dụng:** Chuyển tệp `.pcapng` thành dữ liệu đầu ra kiểu Hashcat mode 22000.

**Cấu trúc / tham số:** `hcxpcapngtool` đọc `capture.pcapng`; `-o capture.22000` quy định tên tệp kết quả.

**Cách chạy trong lab:** Trong lab, dùng tệp thu đã có theo đúng tên `capture.pcapng` nếu công cụ tương thích.

**Kết quả có thể thấy:** Nếu đầu vào có dữ liệu phù hợp, tệp `capture.22000` có thể được tạo.

**Lưu ý:** Đầu vào không có PMKID/EAPOL phù hợp sẽ không cho ra hash dùng được; tên `.22000` là cách đặt của nguồn.

### 06.04 — `hashcat -m 22000 capture.22000 rockyou.txt --force`

**Lệnh nguyên văn:**

```bash
hashcat -m 22000 capture.22000 rockyou.txt --force
```

**Tác dụng:** Thử wordlist bằng Hashcat với định dạng thống nhất WPA/EAPOL/PMKID.

**Cấu trúc / tham số:** `-m 22000` chọn chế độ mới; `capture.22000` là tệp hash; `rockyou.txt` là wordlist; `--force` bỏ qua một số kiểm tra.

**Cách chạy trong lab:** Chỉ áp dụng cho dữ liệu lab đã chuyển đổi hợp lệ và có môi trường GPU/driver đáp ứng yêu cầu công cụ.

**Kết quả có thể thấy:** Công cụ có thể nhận tệp và kiểm tra ứng viên; không bảo đảm thành công.

**Lưu ý:** Lệnh được giữ nguyên `--force` từ tài liệu nhưng thực tế nên kiểm tra cảnh báo thay vì bỏ qua.

### 06.05 — `cap2hccapx capture-01.cap capture.hccapx`

**Lệnh nguyên văn:**

```bash
cap2hccapx capture-01.cap capture.hccapx
```

**Tác dụng:** Chuyển một biến thể tệp đầu vào `.cap` khác sang `.hccapx` cũ.

**Cấu trúc / tham số:** `capture-01.cap` là tên file do công cụ thu thập có thể tạo; `capture.hccapx` là tên file đích.

**Cách chạy trong lab:** Trong lab, chỉ dùng khi `capture-01.cap` tồn tại và công cụ `cap2hccapx` có sẵn.

**Kết quả có thể thấy:** Tạo tệp `.hccapx` nếu có dữ liệu phù hợp với định dạng cũ.

**Lưu ý:** Giữ riêng với 06.01 vì bản gốc còn có tên `capture.cap`; công cụ cũ có thể không tồn tại.

### 06.06 — `hcxpcapngtool -o capture.22000 capture-01.cap`

**Lệnh nguyên văn:**

```bash
hcxpcapngtool -o capture.22000 capture-01.cap
```

**Tác dụng:** Chuyển trực tiếp `capture-01.cap` thành dữ liệu cho mode 22000.

**Cấu trúc / tham số:** `-o capture.22000` chọn đường dẫn xuất; `capture-01.cap` là đầu vào.

**Cách chạy trong lab:** Chạy trong môi trường lab với tệp capture đã lưu; đường dẫn và nội dung đầu vào cần phù hợp.

**Kết quả có thể thấy:** Có thể tạo tệp `capture.22000` khi tìm thấy dữ liệu mà bộ chuyển đổi hỗ trợ.

**Lưu ý:** Khác với 06.03 ở định dạng/tên file đầu vào (`.cap` thay vì `.pcapng`), cần xem công cụ phiên bản hiện tại.

---

## 07. Tổng quan về PMKID

**Tên file nguồn:** `PMKID Attack (No Clients Required!)` và `PMKID Attack (No Clients Required!).md`.

PMKID liên quan cơ chế xác thực/roaming trên một số hệ thống WiFi. Không phải mọi AP đều cung cấp dữ liệu phù hợp; tên mục là tên của file nguồn, không phải bảo đảm kỹ thuật.

### 07.01 — `hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1`

**Lệnh nguyên văn:**

```bash
hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1
```

**Tác dụng:** Lệnh gốc dùng hcxdumptool thu khung WiFi phục vụ phân tích PMKID/EAPOL.

**Cấu trúc / tham số:** `-i wlan0mon` là interface; `-o pmkid.pcapng` là đường dẫn xuất; `--enable_status=1` là tùy chọn hiển thị trạng thái trong bản cũ.

**Cách chạy trong lab:** Chỉ xem và chạy trong lab vô tuyến cách ly, do một số chế độ của công cụ có thể phát khung chủ động; kiểm tra tài liệu phiên bản cài đặt.

**Kết quả có thể thấy:** Có thể ghi `pmkid.pcapng` nhưng không bảo đảm có PMKID dùng được.

**Lưu ý:** Tham số `--enable_status=1` có thể đã thay đổi/không được nhận ở bản mới; giữ nguyên để đối chiếu nguồn.

### 07.02 — `hcxpcapngtool -o pmkid.22000 pmkid.pcapng`

**Lệnh nguyên văn:**

```bash
hcxpcapngtool -o pmkid.22000 pmkid.pcapng
```

**Tác dụng:** Chuyển tệp capture PMKID/EAPOL sang định dạng được Hashcat hỗ trợ.

**Cấu trúc / tham số:** `-o pmkid.22000` là tệp xuất; `pmkid.pcapng` là tệp thu đầu vào.

**Cách chạy trong lab:** Trong lab, chỉ xử lý capture của hệ thống đã được phép và bảo đảm tệp đầu vào hiện hữu.

**Kết quả có thể thấy:** Tạo tệp 22000 nếu dữ liệu gốc chứa bản ghi phù hợp.

**Lưu ý:** Không phải router nào cũng phát sinh PMKID có thể sử dụng; thiếu kết quả không nhất thiết là lỗi lệnh.

### 07.03 — `hashcat -m 22000 pmkid.22000 rockyou.txt --force`

**Lệnh nguyên văn:**

```bash
hashcat -m 22000 pmkid.22000 rockyou.txt --force
```

**Tác dụng:** Thử các ứng viên mật khẩu đối với dữ liệu `.22000` đã xử lý.

**Cấu trúc / tham số:** `-m 22000` là loại hash; `pmkid.22000` là dữ liệu; `rockyou.txt` là wordlist; `--force` bỏ qua một số cảnh báo.

**Cách chạy trong lab:** Chỉ dùng với dữ liệu/thiết bị kiểm thử được cho phép, khi cả hai tệp đầu vào đã được chuẩn bị và xác nhận hợp lệ.

**Kết quả có thể thấy:** Có thể kiểm tra các ứng viên nhưng không đảm bảo tìm ra khóa.

**Lưu ý:** Tên chủ đề “No Clients Required” không áp dụng cho mọi AP; tránh coi đây là công thức phổ quát.

---

## 08. Rủi ro từ WPS PIN

**Tên file nguồn:** `WPS PIN Attack with Reaver` và `WPS PIN Attack with Reaver.md`.

WPS PIN có thể chịu ảnh hưởng bởi giới hạn thử và trạng thái khóa ở AP. `wash` phục vụ quan sát thông tin WPS, còn `reaver` có thể tác động qua các yêu cầu PIN; phần này chỉ có tính chất học tập trong lab.

### 08.01 — `wash -i wlan0mon`

**Lệnh nguyên văn:**

```bash
wash -i wlan0mon
```

**Tác dụng:** Đọc thông tin quảng bá WPS của các điểm truy cập quan sát được.

**Cấu trúc / tham số:** `wash` là công cụ khảo sát WPS; `-i wlan0mon` chỉ card WiFi monitor.

**Cách chạy trong lab:** Chạy trong lab với giao diện monitor có quyền sử dụng; dừng bằng Ctrl+C khi đã quan sát xong.

**Kết quả có thể thấy:** Có thể hiện BSSID, kênh, thông tin WPS và trạng thái khóa WPS tùy phiên bản.

**Lưu ý:** Có thể có AP không cung cấp WPS hoặc khóa tính năng này; thông tin phát sóng không phải quyền kiểm thử.

### 08.02 — `reaver -i wlan0mon -b <BSSID> -c <CH> -vv`

**Lệnh nguyên văn:**

```bash
reaver -i wlan0mon -b <BSSID> -c <CH> -vv
```

**Tác dụng:** Cú pháp gốc dùng Reaver để kiểm tra cơ chế PIN WPS trên AP.

**Cấu trúc / tham số:** `-i wlan0mon` chọn interface; `-b <BSSID>` chỉ AP; `-c <CH>` là kênh; `-vv` bật log chi tiết.

**Cách chạy trong lab:** Chỉ cân nhắc trong môi trường lab có AP thử nghiệm thuộc quyền quản lý và có sự cho phép; không chạy trên mạng lân cận.

**Kết quả có thể thấy:** Tùy cấu hình, AP có thể bỏ qua yêu cầu, áp dụng giới hạn thử hoặc khóa WPS.

**Lưu ý:** Đây là thao tác có thể tác động tới thiết bị; không cung cấp cách né cơ chế khóa hay tăng số lần thử.

### 08.03 — `reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -vv`

**Lệnh nguyên văn:**

```bash
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -vv
```

**Tác dụng:** Ví dụ cú pháp Reaver với một địa chỉ AP và kênh mẫu.

**Cấu trúc / tham số:** `AA:BB:CC:DD:EE:FF` và `6` chỉ là BSSID/kênh minh họa; `-vv` tăng mức thông tin log.

**Cách chạy trong lab:** Giữ làm ví dụ nguồn để người đọc hiểu cấu trúc tham số; chỉ kiểm tra trong lab và với thiết bị được phép.

**Kết quả có thể thấy:** Không có bảo đảm kết quả; router có thể vô hiệu hóa hoặc khóa PIN WPS.

**Lưu ý:** Phân biệt ví dụ tĩnh với địa chỉ AP thật. Khuyến nghị bảo vệ: tắt WPS PIN nếu không cần thiết.

---

## 09. Công cụ và cách quản lý mã nguồn

**Tên file nguồn:** `Automated WiFi Attack Tools` và `Automated WiFi Attack Tools.md`.

Các dự án Wifite, Airgeddon, Fluxion và WiFi Pumpkin 3 hỗ trợ những bài lab bảo mật khác nhau, bao gồm nội dung có nguy cơ gây gián đoạn hoặc mô phỏng trang giả. Xem kỹ mã nguồn và điều kiện sử dụng trước khi chạy.

### 09.01 — `apt install wifite`

**Lệnh nguyên văn:**

```bash
apt install wifite
```

**Tác dụng:** Cài đặt tiện ích Wifite từ kho gói APT của hệ thống.

**Cấu trúc / tham số:** `apt install` yêu cầu APT cài gói có tên `wifite`.

**Cách chạy trong lab:** Trên bản phân phối có APT và nguồn gói phù hợp, chạy trong terminal với quyền cài đặt được phép.

**Kết quả có thể thấy:** APT sẽ báo gói có tồn tại, phiên bản/những phụ thuộc cần cài hoặc lỗi thiếu quyền.

**Lưu ý:** File gốc không có `sudo`; dòng được giữ nguyên. Cài công cụ không tự cho phép sử dụng trên mạng khác.

### 09.02 — `wifite`

**Lệnh nguyên văn:**

```bash
wifite
```

**Tác dụng:** Mở giao diện dòng lệnh của Wifite.

**Cấu trúc / tham số:** `wifite` không có đối số ở ví dụ gốc, có thể vào chương trình tương tác/tự động hóa.

**Cách chạy trong lab:** Chỉ mở trong lab với các thiết bị vô tuyến đã cô lập; đọc lời nhắc và kiểm tra phạm vi trước khi tương tác.

**Kết quả có thể thấy:** Tùy phiên bản, chương trình sẽ kiểm tra phụ thuộc hoặc hiển thị giao diện và danh sách chức năng.

**Lưu ý:** Wifite có chức năng tấn công tự động; không khởi chạy trên máy gần mạng không được phép.

### 09.03 — `git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git`

**Lệnh nguyên văn:**

```bash
git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git
```

**Tác dụng:** Sao chép mã nguồn dự án Airgeddon từ kho công khai.

**Cấu trúc / tham số:** `git clone` tải một bản sao repository được chỉ định vào thư mục `airgeddon`.

**Cách chạy trong lab:** Trong thư mục làm việc do bạn chọn, dùng Git và có kết nối Internet; hãy đánh giá nguồn trước khi thực thi mã.

**Kết quả có thể thấy:** Nếu tải được, xuất hiện thư mục `airgeddon` chứa tập tin dự án.

**Lưu ý:** Nguồn có thể đổi, repository hoặc phụ thuộc có thể thay đổi; không giả định nội dung đáng tin chỉ vì là mã công khai.

### 09.04 — `cd airgeddon`

**Lệnh nguyên văn:**

```bash
cd airgeddon
```

**Tác dụng:** Đổi thư mục hiện hành vào dự án Airgeddon vừa tải.

**Cấu trúc / tham số:** `cd` là lệnh shell đổi thư mục; `airgeddon` là tên thư mục đích.

**Cách chạy trong lab:** Chạy trong cùng phiên terminal, từ vị trí chứa thư mục `airgeddon`.

**Kết quả có thể thấy:** Đường dẫn hiện hành chuyển vào dự án, hoặc shell báo không tìm thấy thư mục.

**Lưu ý:** Lệnh này không tự chạy chương trình; giữ nguyên thứ tự theo tài liệu gốc.

### 09.05 — `bash airgeddon.sh`

**Lệnh nguyên văn:**

```bash
bash airgeddon.sh
```

**Tác dụng:** Mở script Airgeddon bằng shell Bash.

**Cấu trúc / tham số:** `bash` là trình thông dịch; `airgeddon.sh` là tệp script trong thư mục hiện hành.

**Cách chạy trong lab:** Chỉ thực thi trong lab sau khi đã xem nội dung và quyền hạn script, từ đúng thư mục chứa `airgeddon.sh`.

**Kết quả có thể thấy:** Có thể hiển thị menu, lỗi phụ thuộc hoặc yêu cầu quyền quản trị tùy phiên bản.

**Lưu ý:** Một số chức năng có thể giả mạo AP, gây gián đoạn hoặc đánh lừa người dùng; không triển khai ngoài phạm vi được phép.

### 09.06 — `git clone https://github.com/FluxionNetwork/fluxion.git`

**Lệnh nguyên văn:**

```bash
git clone https://github.com/FluxionNetwork/fluxion.git
```

**Tác dụng:** Sao chép kho mã nguồn dự án Fluxion.

**Cấu trúc / tham số:** `git clone` tải repository tại đường dẫn chỉ định, thường tạo thư mục `fluxion`.

**Cách chạy trong lab:** Chạy trong nơi lưu mã nguồn bằng Git có kết nối mạng; kiểm tra nội dung trước khi dùng.

**Kết quả có thể thấy:** Nếu thành công, có thư mục dự án Fluxion.

**Lưu ý:** Dự án chứa chức năng mô phỏng AP/trang xác thực giả; cài đặt không có nghĩa được phép thu thập thông tin người khác.

### 09.07 — `cd fluxion`

**Lệnh nguyên văn:**

```bash
cd fluxion
```

**Tác dụng:** Di chuyển vào thư mục dự án Fluxion.

**Cấu trúc / tham số:** `cd fluxion` đổi working directory sang thư mục `fluxion`.

**Cách chạy trong lab:** Thực hiện trong cùng phiên terminal tại thư mục mẹ của bản clone.

**Kết quả có thể thấy:** Vị trí terminal thay đổi nếu thư mục tồn tại.

**Lưu ý:** Chỉ thay đổi thư mục, không chạy mô phỏng AP.

### 09.08 — `sudo ./fluxion.sh`

**Lệnh nguyên văn:**

```bash
sudo ./fluxion.sh
```

**Tác dụng:** Khởi chạy script Fluxion với quyền quản trị.

**Cấu trúc / tham số:** `sudo` yêu cầu nâng quyền; `./fluxion.sh` gọi script trong thư mục hiện hành.

**Cách chạy trong lab:** Nếu kiểm tra và tin cậy mã nguồn, chỉ khởi chạy trên máy lab cô lập và nhận biết hệ quả của quyền quản trị.

**Kết quả có thể thấy:** Chương trình có thể hiện menu/cấu hình phụ thuộc hoặc lỗi môi trường.

**Lưu ý:** Không dùng tính năng giả trang đăng nhập để lấy thông tin xác thực từ người dùng thật.

### 09.09 — `git clone https://github.com/P0cL4bs/wifipumpkin3.git`

**Lệnh nguyên văn:**

```bash
git clone https://github.com/P0cL4bs/wifipumpkin3.git
```

**Tác dụng:** Sao chép kho mã nguồn WiFi Pumpkin 3.

**Cấu trúc / tham số:** `git clone` tải repository WiFi Pumpkin 3, thường tạo thư mục `wifipumpkin3`.

**Cách chạy trong lab:** Dùng Git trong môi trường lab có kết nối Internet; rà soát mã nguồn và hướng dẫn dự án.

**Kết quả có thể thấy:** Nếu tải thành công sẽ xuất hiện thư mục mã nguồn.

**Lưu ý:** Dự án/cách phân phối có thể đã thay đổi; không có đảm bảo dòng gốc tiếp tục hoạt động.

### 09.10 — `cd wifipumpkin3`

**Lệnh nguyên văn:**

```bash
cd wifipumpkin3
```

**Tác dụng:** Chuyển tới thư mục dự án WiFi Pumpkin 3.

**Cấu trúc / tham số:** `cd wifipumpkin3` thay đổi thư mục làm việc của terminal.

**Cách chạy trong lab:** Chạy sau khi tải dự án và chắc chắn đang ở thư mục cha chứa `wifipumpkin3`.

**Kết quả có thể thấy:** Shell ở thư mục của dự án nếu thành công.

**Lưu ý:** Không cần đặc quyền để đổi thư mục; không có hoạt động vô tuyến được thực hiện.

### 09.11 — `sudo python3 setup.py install`

**Lệnh nguyên văn:**

```bash
sudo python3 setup.py install
```

**Tác dụng:** Cài đặt WiFi Pumpkin 3 bằng script `setup.py` của nguồn cũ.

**Cấu trúc / tham số:** `sudo` nâng quyền; `python3` là thông dịch Python; `setup.py install` là phương pháp cài Python kiểu cũ.

**Cách chạy trong lab:** Chỉ cân nhắc sau khi rà soát mã dự án trong máy lab; script cài đặt có thể thay đổi hệ thống ở quyền quản trị.

**Kết quả có thể thấy:** Có thể cài các tệp/phụ thuộc hoặc báo lỗi do quy trình `setup.py install` đã cũ.

**Lưu ý:** Giữ nguyên vì xuất hiện trong nguồn; không khuyến nghị cài trực tiếp bằng quyền quản trị trên máy chính.

---


Tài liệu bên ngoài tham khảo: [Aircrack-ng/airmon-ng](https://www.aircrack-ng.org/doku.php?id=airmon-ng), [Hashcat — thông báo thay thế mode 2500](https://hashcat.net/forum/thread-10253-post-53276.html). Tính tương thích phải đối chiếu với phiên bản công cụ thực tế.

©CRE -- QUYET DEV PTIT --