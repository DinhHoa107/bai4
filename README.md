<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/d7922a61-8952-4ecf-9d17-2ca5dca0d1ac" /># Bài tập 04 - Khai thác n8n để tự động đăng bài lên WordPress

**Môn:** Phát triển ứng dụng với mã nguồn mở (TEE0421)  
**Lớp:** 58KTPM  
**Sinh viên:** Đinh Hoa  
**Deadline:** 23h59 ngày 25/05/2026

---

## Mục tiêu

Xây dựng hệ thống tự động đăng bài lên WordPress bằng cách kết hợp:
- **Docker Compose** để triển khai 5 service cùng lúc
- **Cloudflare Tunnel** để public các service ra internet
- **n8n** để tạo workflow automation
- **Telegram Bot + Google Gemini AI** để sinh nội dung bài viết tự động

---

## 1. Cấu hình Docker Compose

File `docker-compose.yml` chứa 5 services:

| Service | Image | Chức năng |
|---|---|---|
| mariadb | mariadb:latest | Database cho WordPress |
| phpmyadmin | phpmyadmin:latest | Quản lý DB trực quan |
| wordpress | wordpress:latest | CMS chính |
| cloudflared | cloudflare/cloudflared:latest | Public ra internet |
| n8n | n8nio/n8n:latest | Automation workflow |

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/7499c26d-916c-4295-b693-bbbaa2f94bea" />
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/883a66f6-4b9e-4a52-ba24-6ad82aa6bcf2" />

Khởi động tất cả services:
```bash
cd ~/wordpress-btvn04
docker compose up -d
```

<img width="1487" height="761" alt="image" src="https://github.com/user-attachments/assets/6eb5069a-089e-4db3-82c2-acb941660ce5" />


---

## 2. Cloudflare Tunnel

Tạo tunnel **btvn04** và cấu hình 3 public hostname routes:

| Subdomain | Domain | Service nội bộ |
|---|---|---|
| wp | taphamdinhhoa.io.vn | http://wordpress:80 |
| pma | taphamdinhhoa.io.vn | http://phpmyadmin:80 |
| n8n | taphamdinhhoa.io.vn | http://n8n:5678 |

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/ebe27d93-4315-4ea6-9828-7da0e70d5d96" />


<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/aecfd829-389d-436c-b5ed-1383fb241d18" />


---

## 3. Kiểm tra DB trước khi cài WordPress

Truy cập **pma.taphamdinhhoa.io.vn** → đăng nhập → chọn **wordpress_db**

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/057ffa8e-422c-4445-92bb-ed262a19900f" />


---

## 4. Cài đặt WordPress

Truy cập **wp.taphamdinhhoa.io.vn** → làm theo hướng dẫn cài đặt:
- Tiêu đề: Đinh Hoa
- Username: dinhhoa
- Email: Dinhhoa1072004@gmail.com

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/edd778b6-d119-4b65-915b-3d843444e315" />


<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/e36e8d49-c4d6-4bab-a127-119fa8f085d0" />


---

## 5. Kiểm tra DB sau khi cài WordPress

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/fa9cd5b0-26d0-444f-a545-4e6a17393771" />


WordPress đã tự động tạo **12 bảng** trong database.

---

## 6. Tạo bài viết thủ công trên WordPress

### Bài viết 1: Giới thiệu bản thân
Nội dung giới thiệu thông tin cá nhân, sở thích, mục tiêu học tập.

<img width="1916" height="1079" alt="image" src="https://github.com/user-attachments/assets/3c9e41f1-a6d4-42a0-9523-50286a53822d" />


### Bài viết 2: Kiến thức môn học
Tổng hợp những kiến thức đã học được trong môn Phát triển ứng dụng với mã nguồn mở.

<img width="1916" height="1079" alt="image" src="https://github.com/user-attachments/assets/468aa119-1eb0-4010-b6fd-88c862089de0" />


---

## 7. Cấu hình n8n Workflow

Truy cập **n8n.taphamdinhhoa.io.vn** → tạo tài khoản → tạo workflow mới.

### Workflow gồm 4 node:
[Telegram Trigger] → [Google Gemini] → [Code JavaScript] → [WordPress: Create Post]
> 📸 **[Chèn ảnh: workflow 4 node đã kết nối đầy đủ]**

### Chi tiết từng node:

**Node 1 - Telegram Trigger:**
- Bot: @DinhHoaAutoPost_bot
- Trigger On: Message

> 📸 **[Chèn ảnh: cấu hình node Telegram Trigger - Connection tested successfully]**

**Node 2 - Google Gemini (Message a Model):**
- Model: gemini-2.5-flash
- Prompt: `{{ $json.message.text }}. Kết quả sinh ra ở định dạng JSON với 2 trường: post_title và post_content (HTML+CSS)`

> 📸 **[Chèn ảnh: cấu hình node Google Gemini với prompt]**

**Node 3 - Code in JavaScript:**
```javascript
const rawText = $input.first().json.content.parts[0].text;
const cleaned = rawText.replace(/```json\n?|\n?```/g, '').trim();
const cleanData = JSON.parse(cleaned);
return {
  title: cleanData.post_title,
  content: cleanData.post_content
};
```

> 📸 **[Chèn ảnh: node Code JavaScript với code đã điền]**

**Node 4 - WordPress: Create a Post:**
- WordPress URL: https://wp.taphamdinhhoa.io.vn
- Title: `{{ $json.title }}`
- Content: `{{ $json.content }}`
- Status: Publish

> 📸 **[Chèn ảnh: cấu hình node WordPress Create a Post]**

---

## 8. Demo kết quả cuối cùng

### Bước 1: Nhắn tin với Telegram Bot

> 📸 **[Chèn ảnh: tin nhắn gửi đến @DinhHoaAutoPost_bot trên điện thoại]**

### Bước 2: n8n xử lý tự động

> 📸 **[Chèn ảnh: n8n Executions - Succeeded, 4 node đều có dấu ✓ xanh]**

### Bước 3: Bài viết tự động xuất hiện trên WordPress

> 📸 **[Chèn ảnh: bài viết mới tự động đăng trên wp.taphamdinhhoa.io.vn]**

---

## Nhận xét & Đánh giá

**Những gì đạt được:**
- Triển khai thành công 5 services bằng Docker Compose
- Public 3 subdomain ra internet qua Cloudflare Tunnel
- Xây dựng workflow automation hoàn chỉnh với n8n
- Tích hợp Telegram Bot + Google Gemini AI để tự động tạo và đăng bài lên WordPress

**Kiến thức thu được:**
- Hiểu sâu hơn về Docker networking - các container giao tiếp với nhau qua tên service
- Cloudflare Tunnel là giải pháp hiệu quả để public service mà không cần mở port router
- n8n là công cụ automation mạnh mẽ, dễ dùng, tích hợp được nhiều service
- Google Gemini API có thể sinh nội dung HTML chất lượng cao từ prompt đơn giản

**Khó khăn gặp phải:**
- Disk server đầy 100% phải dọn dẹp trước khi làm bài
- Token Cloudflare bị lỗi do copy thiếu ký tự
- Gemini trả về JSON bọc trong markdown backtick cần xử lý thêm trong Code node
