# 🛡️  AN TOÀN VÀ BẢO MẬT THÔNG TIN

* **Sinh viên:** Trần Hoàng Xuân Vũ
* **Mã số sinh viên (MSSV):** K235480106080
* **Lớp / Chuyên ngành:** Kỹ thuật Máy tính (TNUT)
* **Công nghệ sử dụng:** C# WPF (.NET 8.0 / .NET 10.0), System.Security.Cryptography

---

## 📌 TỔNG QUAN BÀI TẬP

Kho lưu trữ **`BT_ATBM2_TNUT`** chứa trọn bộ mã nguồn và báo cáo thử nghiệm của 2 bài tập lớn môn **An toàn và Bảo mật thông tin**:

1. **Bài 1:** Hệ thống mã hóa đối xứng **AES-256 (Chế độ CBC + Padding PKCS7)**, tối ưu xử lý luồng dữ liệu kích thước lớn (hàng GB).
2. **Bài 2:** Hệ thống mã hóa bất đối xứng **RSA-2048**, phân tích cấu trúc khóa **PEM / ASN.1 DER**, đánh giá **SHA-256 Avalanche Effect** và giải pháp **Chữ ký số ngẫu nhiên RSASSA-PSS (RFC 8017)**.

---

## 🔒 BÀI 1: MÃ HÓA ĐỐI XỨNG AES-256 (CBC + PKCS7)

### 1. Các tính năng đã thực hiện
* **Sinh khóa bí mật:** Khởi tạo khóa AES-256 bit và Vector khởi tạo IV 128 bit ngẫu nhiên an toàn.
* **Quản lý khóa (Export/Import):** Lưu và nạp khóa bí mật ra/từ file mã hóa `aes_key.txt` dưới dạng Base64.
* **Mã hóa/Giải mã văn bản:** Mã hóa chuỗi văn bản bản rõ (Plaintext) ra bản mã (Ciphertext) ở định dạng Base64 và Hex.
* **Mã hóa/Giải mã luồng File lớn (GB):** Xử lý mã hóa file dung lượng bất kỳ với bộ đệm `CryptoStream` (1 MB Chunk Buffer), tối ưu RAM cố định (< 20 MB), hoàn toàn không tràn bộ nhớ.
* **Đối chứng Byte-by-Byte:** So sánh trực tiếp dung lượng và giá trị từng byte giữa file gốc và file giải mã.

### 2. Kết quả Thử nghiệm & Benchmark
* **Dữ liệu thử nghiệm:** File binary mẫu dung lượng 1.0 GB (1,073,741,824 bytes).
* **Thời gian mã hóa:** ~1.21 giây (Tốc độ trung bình: ~849.79 MB/s).
* **Thời gian giải mã:** ~1.31 giây (Tốc độ trung bình: ~784.07 MB/s).
* **Kết quả đối chứng Byte-by-Byte:** Trùng khớp **100%** (File giải mã hoàn toàn nguyên vẹn so với file gốc).

### 3. Báo cáo Kết luận (Bài 1)
1. **Hiệu năng thuật toán:** AES-256 đạt tốc độ xử lý cực cao nhờ tối ưu hóa phần cứng (AES-NI) trên CPU.
2. **Tính toàn vẹn dữ liệu:** Chế độ mã hóa khối CBC kết hợp PKCS7 Padding đảm bảo dữ liệu giải mã không mất mát bit nào.
3. **Tối ưu quản lý bộ nhớ:** Sử dụng `CryptoStream` đọc/ghi từng khối giúp ứng dụng xử lý các file dung lượng hàng chục GB mượt mà.

---

## 🔑 BÀI 2: MÃ HÓA BẤT ĐỐI XỨNG RSA & CHỮ KÝ SỐ RSASSA-PSS

### 1. Các tính năng đã thực hiện
* **Sinh & Xuất cặp khóa RSA-2048:** Tạo cặp khóa Public Key và Private Key chuẩn định dạng **PKCS#1 PEM**.
* **Phân tích chi tiết cấu trúc PEM & ASN.1 DER:** Bóc tách các tham số toán học cốt lõi từ các thẻ `SEQUENCE` (`0x30`) và `INTEGER` (`0x02`):
  * `n = p × q`: Modulus công khai (2048 bits / 256 bytes).
  * `e`: Public Exponent (65537 = 0x10001).
  * `d`: Private Exponent (2048 bits).
  * `p, q`: Các số nguyên tố khởi tạo.
* **Mã hóa & Giải mã RSA-OAEP SHA-256:** Mã hóa mảng byte dữ liệu <= 190 bytes (ví dụ: Khóa đối xứng AES-256 32 bytes) bằng Public Key và giải mã đối chứng bằng Private Key.
* **Đánh giá tốc độ Benchmark (Public Key vs Private Key):** Đo đạc 500 phép tính lặp giữa 2 thao tác toán học.
* **Hiệu ứng tuyết rơi SHA-256 (Avalanche Effect):** Đánh giá mức độ đảo bit ngẫu nhiên realtime khi biến đổi 1 bit đầu vào.
* **Giải pháp Chữ ký số RSASSA-PSS:** Đề xuất và thực nghiệm tạo nhiều chữ ký số **hoàn toàn khác nhau** trên cùng 1 dữ liệu và cùng 1 Private Key, nhưng vẫn xác thực **hợp lệ 100%** bằng Public Key.

### 2. Báo cáo & Giải đáp bài tập (Bài 2)

#### a) So sánh tốc độ giữa Public Key và Private Key
* **Public Key (e = 65537 = 2^16 + 1):** Số mũ e chỉ chứa đúng 2 bit 1 ở dạng nhị phân, phép tính lũy thừa M^e mod n chỉ mất 17 phép nhân/bình phương.
* **Private Key (d):** Số mũ bí mật d dài 2048 bit ngẫu nhiên chứa khoảng 1024 bit 1, phép tính C^d mod n đòi hỏi hàng ngàn phép nhân modulo phức tạp.
* **Kết luận Benchmark:** Phép toán dùng **Public Key nhanh hơn Private Key từ 20 đến 30 lần** (41,753 ops/s so với 1,764 ops/s).

#### b) Hiệu ứng tuyết rơi SHA-256 (Avalanche Effect)
* **Kết quả:** Khi thay đổi chỉ 1 bit ở chuỗi đầu vào, giá trị băm SHA-256 biến đổi ngẫu nhiên **128 / 256 bits (đạt tỷ lệ ~50%)**.
* **Ý nghĩa:** Khẳng định tính kháng va chạm mạnh mẽ của hàm băm cryptographically secure, ngăn chặn các tấn công dò tìm chuỗi gốc.

#### c) Đề xuất giải pháp Chữ ký số ngẫu nhiên (RSASSA-PSS)
* **Đặt vấn đề:** Chuẩn ký PKCS#1 v1.5 mang tính cố định (Deterministic), cùng 1 dữ liệu + cùng 1 Private Key luôn ra 1 chữ ký giống hệt nhau -> Dễ bị tấn công Replay Attack.
* **Đề xuất giải pháp:** Áp dụng chuẩn chữ ký số **RSASSA-PSS (PKCS#1 v2.1 / RFC 8017)**.
* **Cơ chế:** RSASSA-PSS tự động trộn thêm một chuỗi muối ngẫu nhiên **Random Salt (32 bytes)** vào dữ liệu trước khi băm và ký.
* **Kết quả thực nghiệm:**
  * Ký 3 lần liên tiếp trên cùng dữ liệu cho ra **3 chuỗi Chữ ký số Base64 khác nhau 100%**.
  * Kiểm tra xác thực bằng duy nhất 1 Public Key: Cả 3 chữ ký đều báo **VALID (Hợp lệ 100%)**.

---

## 📁 CẤU TRÚC KHO LƯU TRỮ (REPOSITORY STRUCTURE)
Bài 1: 

<img width="1918" height="1009" alt="image" src="https://github.com/user-attachments/assets/c494dbbf-f066-479a-840b-e42a25edb9fa" />

<img width="1918" height="1005" alt="image" src="https://github.com/user-attachments/assets/b22373bb-b31b-4845-9b3e-83e079030244" />


<img width="1918" height="978" alt="image" src="https://github.com/user-attachments/assets/acb913a4-314e-4e98-9cab-dc1a1ecee0e2" />


<img width="1911" height="1005" alt="image" src="https://github.com/user-attachments/assets/5d4a224b-6858-42a6-9937-10090a06f6b9" />


<img width="1918" height="1003" alt="image" src="https://github.com/user-attachments/assets/ec016084-04e5-4ff2-a4f8-30090b64440a" />



Bài 2:
<img width="1915" height="963" alt="dd2a311e-7dcc-45fb-bf4d-9ad7a08c0e8e" src="https://github.com/user-attachments/assets/75a660d9-0a08-4e55-84ea-0fcaa6b1bb55" />

<img width="1918" height="973" alt="b6eb9201-ff91-4645-9432-91b3a6ed0e63" src="https://github.com/user-attachments/assets/1cb15ff0-acb9-4379-9308-298276921b3f" />

<img width="1918" height="973" alt="202e2b48-9141-4303-a58d-cc1df34c1444" src="https://github.com/user-attachments/assets/ca843c3f-58b8-4d99-8657-862dbba62ce0" />

<img width="1918" height="982" alt="d1c68d73-6ee9-4ecd-9200-ce3c692f9618" src="https://github.com/user-attachments/assets/d679d7dc-49dd-4598-9edc-6b3bcf8e6c95" />

<img width="1918" height="982" alt="94607f86-e1b6-4ec5-8562-8017077b4c7e" src="https://github.com/user-attachments/assets/e192f7cd-8b61-4520-8271-8cb5bf4d0bd5" />


<img width="1918" height="976" alt="0657c642-c63c-4966-964e-bc4521d3016f" src="https://github.com/user-attachments/assets/4d0732da-95ea-451d-b65d-64fb5409cde0" />



```text
BT_ATBM2_TNUT/
├── BaiTap1_AES/                  # Dự án Bài 1 (Mã hóa AES-256 WPF)
│   ├── MainWindow.xaml           # Giao diện Cyber Dark Mode Bài 1
│   ├── MainWindow.xaml.cs        # Xử lý CryptoStream & Benchmark File GB
│   └── BaiTap1_AES.csproj
├── BaiTap2_RSA/                  # Dự án Bài 2 (RSA & Chữ ký số RSASSA-PSS WPF)
│   ├── MainWindow.xaml           # Giao diện 6 Tab Bài 2
│   ├── MainWindow.xaml.cs        # Xử lý RSA, ASN.1, SHA-256 & RSASSA-PSS
│   └── BaiTap2_RSA.csproj
└── README.md                     # Báo cáo tổng hợp dự án



