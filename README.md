# Thực hành WAF trên Nginx

Bài thực hành dựng một web application firewall trước OWASP Juice Shop, sau đó so sánh kết quả kiểm thử khi truy cập trực tiếp và khi đi qua WAF.

## Mục tiêu

- Dựng Juice Shop làm ứng dụng backend có chủ đích dùng cho thực hành bảo mật.
- Đặt Nginx, ModSecurity và OWASP Core Rule Set (CRS) trước Juice Shop.
- Dùng cổng `3000` làm đường truy cập trực tiếp để đối chứng.
- Dùng cổng `8080` làm đường truy cập qua WAF.
- Kiểm tra WAF chặn thử nghiệm SQL injection nhưng vẫn cho phép đăng nhập hợp lệ.

## Mô hình triển khai

```text
Truy cập trực tiếp: trình duyệt ── localhost:3000 ──> Juice Shop
Truy cập qua WAF:   trình duyệt ── localhost:8080 ──> Nginx + ModSecurity + OWASP CRS ──> Juice Shop:3000
```

Cấu hình nằm trong [`docker-compose.yml`](docker-compose.yml):

| Thành phần | Cấu hình |
|---|---|
| Backend `web` | Image `bkimminich/juice-shop`, publish cổng `3000:3000` |
| Gateway `waf` | Image `owasp/modsecurity-crs:nginx`, publish cổng `8080:8080` |
| Backend đích | `http://web:3000` |
| ModSecurity | `MODSEC_RULE_ENGINE=On` |
| Mức CRS | `BLOCKING_PARANOIA=1` |

## Yêu cầu

- Docker Desktop đang chạy.
- Terminal mở tại thư mục repo, cùng cấp với `docker-compose.yml`.

## Khởi động và kiểm tra Lab 1

Chạy trong PowerShell hoặc terminal:

```powershell
docker compose config --quiet
docker compose up -d
docker compose ps
```

Mở hai địa chỉ sau. Cả hai phải hiển thị Juice Shop:

- [http://localhost:3000](http://localhost:3000) — truy cập trực tiếp, không đi qua WAF.
- [http://localhost:8080](http://localhost:8080) — truy cập qua Nginx WAF.

Bằng chứng và hướng dẫn Lab 1 nằm trong [`evidence/lab1/README.md`](evidence/lab1/README.md). Thư mục này có kết quả kiểm tra Compose, trạng thái service, cấu hình Nginx/ModSecurity đang chạy và ảnh trình duyệt của hai cổng.

## Thử nghiệm Lab 2

Thực hiện các bước trên Juice Shop đang chạy trong lab này:

1. Mở `http://localhost:3000`, vào **Login**, nhập email `' OR 1=1--` và mật khẩu bất kỳ. Kết quả theo đề: đăng nhập thành công khi truy cập trực tiếp.
2. Mở `http://localhost:8080` và thử lại cùng payload. Kết quả mong đợi: WAF trả `403 Forbidden`.
3. Qua `http://localhost:8080`, đăng nhập hợp lệ bằng `admin@juice-sh.op` / `admin123`. Kết quả mong đợi: đăng nhập thành công.
4. Xem log WAF:

   ```powershell
   docker compose logs --no-color waf
   ```

   Tìm luật `942100` hoặc thông điệp `SQL Injection Attack Detected via libinjection` để đối chiếu lần chặn SQL injection.

| Trường hợp | Kết quả đã ghi nhận |
|---|---|
| SQLi qua cổng 3000 | HTTP 200, xác thực thành công |
| Cùng payload qua cổng 8080 | HTTP 403, bị WAF chặn |
| Đăng nhập đúng qua cổng 8080 | HTTP 200, xác thực thành công |
| Log CRS | Luật `942100`, SQL Injection Attack Detected via libinjection |

Chi tiết và ảnh minh chứng Lab 2 nằm trong [`evidence/lab2/README.md`](evidence/lab2/README.md). Kết quả HTTP được lưu tại [`http-results.json`](evidence/lab2/http-results.json); trích đoạn log thật của luật CRS được lưu tại [`waf-logs.txt`](evidence/lab2/waf-logs.txt). Header chứa cookie/token đã được loại bỏ khỏi log lưu trong repo.

## Tắt lab

Khi hoàn tất, dừng và xóa các container của Compose:

```powershell
docker compose down
```

## Cấu trúc minh chứng

```text
evidence/
├── lab1/
│   ├── README.md
│   ├── compose-check.txt
│   ├── service-and-http-check.txt
│   ├── nginx-runtime-excerpts.txt
│   ├── web-browser.png
│   └── waf-browser.png
└── lab2/
    ├── README.md
    ├── http-results.json
    ├── waf-detection.txt
    ├── waf-logs.txt
    ├── admin-login-direct-3000.png
    ├── sqli-waf-8080-blocked.png
    └── valid-login-admin-waf-8080.png
```
