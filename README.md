# Bài tập 04 - Khai thác n8n để tự động đăng bài lên WordPress

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

> 📸 **[Chèn ảnh: nội dung file docker-compose.yml]**

Khởi động tất cả services:
```bash
cd ~/wordpress-btvn04
docker compose up -d
```

> 📸 **[Chèn ảnh: kết quả `docker ps` - 5 container STATUS Up]**

---

## 2. Cloudflare Tunnel

Tạo tunnel **btvn04** và cấu hình 3 public hostname routes:

| Subdomain | Domain | Service nội bộ |
|---|---|---|
| wp | taphamdinhhoa.io.vn | http://wordpress:80 |
| pma | taphamdinhhoa.io.vn | http://phpmyadmin:80 |
| n8n | taphamdinhhoa.io.vn | http://n8n:5678 |

> 📸 **[Chèn ảnh: Cloudflare dashboard - 3 routes Published application routes]**

> 📸 **[Chèn ảnh: docker logs cloudflared - Registered tunnel connection x4]**

---

## 3. Kiểm tra DB trước khi cài WordPress

Truy cập **pma.taphamdinhhoa.io.vn** → đăng nhập → chọn **wordpress_db**

> 📸 **[Chèn ảnh: phpMyAdmin - wordpress_db trống, thông báo "Không tìm thấy bảng nào"]**

---

## 4. Cài đặt WordPress

Truy cập **wp.taphamdinhhoa.io.vn** → làm theo hướng dẫn cài đặt:
- Tiêu đề: Đinh Hoa
- Username: dinhhoa
- Email: Dinhhoa1072004@gmail.com

> 📸 **[Chèn ảnh: form cài đặt WordPress đã điền thông tin]**

> 📸 **[Chèn ảnh: trang quản trị WordPress sau khi cài xong]**

---

## 5. Kiểm tra DB sau khi cài WordPress

> 📸 **[Chèn ảnh: phpMyAdmin - wordpress_db có 12 bảng (wp_posts, wp_users, wp_options...)]**

WordPress đã tự động tạo **12 bảng** trong database.

---

## 6. Tạo bài viết thủ công trên WordPress

### Bài viết 1: Giới thiệu bản thân
Nội dung giới thiệu thông tin cá nhân, sở thích, mục tiêu học tập.

> 📸 **[Chèn ảnh: bài viết "Giới thiệu về bản thân" trên WordPress]**

### Bài viết 2: Kiến thức môn học
Tổng hợp những kiến thức đã học được trong môn Phát triển ứng dụng với mã nguồn mở.

> 📸 **[Chèn ảnh: bài viết "Kiến thức học được từ môn Phát triển ứng dụng với mã nguồn mở"]**

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
