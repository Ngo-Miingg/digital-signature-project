# Hệ thống Lưu trữ File An toàn bằng Mã hóa Lai

![Python](https://img.shields.io/badge/python-3.9+-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)

Dự án này là một mô phỏng hệ thống Client-Server cho phép upload và download file một cách an toàn. Hệ thống thể hiện việc áp dụng các kỹ thuật mật mã hiện đại, bao gồm **mã hóa lai (Hybrid Encryption)**, **chữ ký số (Digital Signature)**, và **cơ chế xác thực** để đảm bảo tính bảo mật, toàn vẹn và xác thực cho dữ liệu.

## Các tính năng chính

-   **Mô hình Client-Server:** Một Admin (Client) tương tác với hai Node lưu trữ (Server) độc lập.
-   **Mã hóa lai:**
    -   **RSA-2048** được sử dụng để trao đổi khóa phiên một cách an toàn (mã hóa bất đối xứng).
    -   **AES-256** ở chế độ CBC được sử dụng để mã hóa nội dung file (mã hóa đối xứng), mang lại hiệu suất cao.
-   **Chữ ký số:** Sử dụng **RSA-PSS** với hàm băm **SHA-512** để xác thực danh tính của người gửi và đảm bảo các yêu cầu không bị giả mạo.
-   **Kiểm tra toàn vẹn dữ liệu:** Sử dụng **SHA-512** để tạo mã hash cho gói dữ liệu, đảm bảo file không bị thay đổi hay lỗi trong quá trình truyền tải.
-   **Lập trình đồng thời:**
    -   Server sử dụng **đa luồng (multi-threading)** để xử lý nhiều kết nối cùng lúc.
    -   Client sử dụng đa luồng để upload file lên hai server đồng thời, tăng hiệu suất.
-   **Xử lý file nhị phân:** Hỗ trợ upload/download các loại file nhị phân (ảnh, video...) bằng cách mã hóa dữ liệu sang định dạng **Base64**.

## Cấu trúc dự án

```
.
├── client.py           # Giả lập Admin thực hiện upload/download
├── server.py           # Giả lập hai Node lưu trữ, lắng nghe kết nối
├── key_generator.py    # Script để tạo các cặp khóa RSA cần thiết
└── README.md           # File hướng dẫn này
```

## Luồng hoạt động của hệ thống

Hệ thống hoạt động dựa trên một quy trình mật mã chặt chẽ:

1.  **Tạo khóa:** Chạy `key_generator.py` một lần duy nhất để tạo cặp khóa RSA cho Admin và hai Node.
2.  **Quá trình Upload:**
    -   Client tạo một **khóa phiên AES-256** ngẫu nhiên.
    -   File được mã hóa bằng khóa phiên AES này.
    -   Khóa phiên AES sau đó được mã hóa bằng **khóa công khai RSA** của Node đích.
    -   Client dùng **khóa riêng RSA** của mình để ký vào metadata của file.
    -   Server nhận dữ liệu, dùng khóa công khai của Client để **xác thực chữ ký**. Sau đó, dùng khóa riêng của mình để **giải mã khóa phiên AES** và cuối cùng giải mã nội dung file.
3.  **Quá trình Download:**
    -   Client tạo và **ký** vào một yêu cầu download.
    -   Server **xác thực chữ ký** và gửi lại gói dữ liệu đã được mã hóa (lưu từ lúc upload).
    -   Client dùng **khóa phiên AES ban đầu** để giải mã và lưu file.

## Cài đặt và Chạy thử

### Yêu cầu
- Python 3.7+
- Thư viện `cryptography`
- Thư viện `Pillow` (để tự động tạo file ảnh test)

### Các bước cài đặt

1.  **Clone repository về máy:**
    ```bash
    git clone <URL_CUA_REPOSITORY_CUA_BAN>
    cd <TEN_THU_MUC_DU_AN>
    ```

2.  **Cài đặt các thư viện cần thiết:**
    ```bash
    pip install cryptography Pillow
    ```
    *Lưu ý: Nếu bạn không có `Pillow`, `client.py` sẽ tự tạo một file `.txt` để chạy thử nghiệm.*

### Hướng dẫn sử dụng

Thực hiện theo các bước sau trong các cửa sổ terminal riêng biệt.

1.  **Bước 1: Tạo khóa mật mã**
    Chạy script này đầu tiên và chỉ một lần. Nó sẽ tạo các file `_key.pem` cần thiết.
    ```bash
    python key_generator.py
    ```

2.  **Bước 2: Khởi động Server**
    Mở một terminal mới và chạy server. Server sẽ bắt đầu lắng nghe kết nối từ client.
    ```bash
    python server.py
    ```
    Bạn sẽ thấy thông báo hai Node đã sẵn sàng.

3.  **Bước 3: Chạy Client để Upload và Download**
    Mở một terminal thứ ba và chạy client. Script sẽ tự động tìm hoặc tạo file `test_image.jpg`, sau đó thực hiện upload đồng thời lên 2 Node và download tuần tự về từ mỗi Node.
    ```bash
    python client.py
    ```
    Quan sát output ở cả 3 cửa sổ terminal để thấy rõ quá trình tương tác.

## Công nghệ và Thuật toán sử dụng

-   **Ngôn ngữ:** Python 3
-   **Thư viện chính:** `cryptography`, `socket`, `threading`, `json`
-   **Mã hóa bất đối xứng:** RSA-2048 (với đệm OAEP)
-   **Mã hóa đối xứng:** AES-256 (chế độ CBC, đệm PKCS7)
-   **Hàm băm / Chữ ký:** SHA-512, RSA-PSS

## Giấy phép

Dự án này được cấp phép theo Giấy phép MIT. Xem file `LICENSE` để biết thêm chi tiết.
