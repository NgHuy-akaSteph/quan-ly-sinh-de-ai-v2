# Hướng dẫn Setup và Chạy Project

## 1. Kiểm tra Môi trường Hiện tại

### ✅ Đã có sẵn:
- **Python 3.14.6** ✓ (yêu cầu: >= 3.10)
- **Node.js v24.19.0** ✓ (yêu cầu: >= 18)
- **File `.env`** ✓ (đã tồn tại)
- **node_modules** ✓ (frontend dependencies đã cài)

### ❌ Còn thiếu:
- **Python virtualenv** chưa tạo (thư mục `dev/`)
- **Python packages** chưa cài (FastAPI, SQLAlchemy, etc.)
- **MySQL client** không có (nhưng không bắt buộc vì dùng TiDB Cloud)

## 2. Cấu hình Database

### Hiện tại project đang dùng:
**TiDB Cloud** (MySQL-compatible cloud database)

### Thông tin từ `.env` hiện tại:
```
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=quan_ly_sinh_de_ai_v2
DB_PASSWORD=zXdZ5bfA39P5Rr8r
DB_NAME=quan_ly_sinh_de_ai_v2
DB_SSL=false
```

**Lưu ý:** Đây là config cho **MySQL local** (127.0.0.1). Nhưng trong `setup.sh` và `setup.bat` lại có config cho **TiDB Cloud**:
```
DB_HOST=gateway01.ap-southeast-1.prod.alicloud.tidbcloud.com
DB_PORT=4000
DB_USER=338z5oDUxdCvTYx.root
DB_PASSWORD=zXdZ5bfA39P5Rr8r
DB_NAME=quan_ly_sinh_de_ai_v2
DB_SSL=true
```

### ❓ Câu hỏi quan trọng:
**Bạn muốn dùng database nào?**

**Option 1: MySQL Local (127.0.0.1)** - File `.env` hiện tại
- ✅ Ưu điểm: Không phụ thuộc internet, data local
- ❌ Yêu cầu: Phải cài MySQL 8.0 trên máy
- ❌ Yêu cầu: Phải tạo database `quan_ly_sinh_de_ai_v2`
- ❌ Yêu cầu: Phải tạo user `quan_ly_sinh_de_ai_v2` với password

**Option 2: TiDB Cloud** - Config trong setup scripts
- ✅ Ưu điểm: Không cần cài gì, đã có sẵn trên cloud
- ✅ Ưu điểm: Database đã được tạo sẵn
- ❌ Yêu cầu: Cần internet
- ❓ Vấn đề: Không biết TiDB Cloud account này còn hoạt động không

### Khuyến nghị:
**Thử TiDB Cloud trước** (vì script setup mặc định dùng nó), nếu không connect được thì chuyển sang MySQL local.

## 3. Cấu hình Google Gemini API Keys

### API Keys hiện tại trong `.env`:
```
GEMINI_API_KEY_SINGLE="YOUR_GEMINI_API_KEY"
GEMINI_API_KEY_TRUEFALSE="YOUR_GEMINI_API_KEY"
GEMINI_API_KEY_SHORT="YOUR_GEMINI_API_KEY"
```

### ❓ Vấn đề:
- **Format key sai**: Gemini API key thường có format `AIza...` (bắt đầu bằng `AIza`)
- Key hiện tại `AQ.Ab8RN6...` không giống format chuẩn của Google
- Có thể keys này đã hết hạn hoặc không hợp lệ

### 🔧 Cách lấy Gemini API Key mới:
1. Truy cập: https://aistudio.google.com/app/apikey
2. Đăng nhập Google account
3. Click **"Create API Key"**
4. Copy key (format: `AIzaSy...`)
5. Paste vào file `.env`:
   ```
   GEMINI_API_KEY_SINGLE="YOUR_GEMINI_API_KEY"
   GEMINI_API_KEY_TRUEFALSE="YOUR_GEMINI_API_KEY"
   GEMINI_API_KEY_SHORT="YOUR_GEMINI_API_KEY"
   ```

### Tại sao cần 3 keys?
- **SINGLE**: Sinh câu hỏi trắc nghiệm (1 đáp án đúng)
- **TRUEFALSE**: Sinh câu hỏi Đúng/Sai
- **SHORT**: Sinh câu hỏi tự luận ngắn

**Lưu ý:** Bạn có thể dùng chung 1 key cho cả 3 (copy cùng 1 key vào 3 biến)

## 4. Các Bước Setup Chi tiết

### Bước 1: Tạo Python Virtual Environment

```bash
# Tạo virtualenv trong thư mục 'dev'
python3 -m venv dev

# Kích hoạt virtualenv
source dev/bin/activate  # Linux/macOS
# hoặc
dev\Scripts\activate     # Windows
```

**Giải thích:**
- Project dùng thư mục `dev/` (không phải `venv/`) theo logic trong `start.js`
- Virtualenv giúp cô lập dependencies, tránh conflict với system Python

### Bước 2: Cài đặt Python Dependencies

```bash
# Đảm bảo đang trong virtualenv (dev)
pip install -r requirements.txt
```

**Packages sẽ được cài:**
- `fastapi==0.115.12` - Web framework
- `uvicorn==0.34.3` - ASGI server
- `sqlalchemy==2.0.41` - ORM
- `aiomysql==0.2.0` - Async MySQL driver
- `google-genai==1.16.1` - Gemini AI SDK
- `PyJWT==2.10.1` - JWT authentication
- `passlib==1.7.4` - Password hashing
- và các packages khác...

**Thời gian:** ~2-3 phút

### Bước 3: Kiểm tra File `.env`

Mở file `.env` và verify các thông tin:

```bash
# Xem nội dung .env (CHỈ để kiểm tra, KHÔNG share)
cat .env
```

**Checklist:**
- [ ] `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` đúng
- [ ] `DB_SSL=true` nếu dùng TiDB Cloud, `false` nếu MySQL local
- [ ] `GEMINI_API_KEY_*` đã điền (format `AIza...`)
- [ ] `JWT_SECRET_KEY` tồn tại (dùng để mã hóa token)

**Ví dụ `.env` hoàn chỉnh:**
```env
# Application
PORT=3000
APP_URL=http://localhost:3000

# Gemini AI
GEMINI_API_KEY_SINGLE="YOUR_GEMINI_API_KEY"
GEMINI_API_KEY_TRUEFALSE="YOUR_GEMINI_API_KEY"
GEMINI_API_KEY_SHORT="YOUR_GEMINI_API_KEY"

# JWT
JWT_SECRET_KEY=smarttest-change-this-to-random-string-2026
JWT_EXPIRE_MINUTES=52560000
JWT_REFRESH_DAYS=36500

# Database (Option 1: TiDB Cloud)
DB_HOST=gateway01.ap-southeast-1.prod.alicloud.tidbcloud.com
DB_PORT=4000
DB_USER=338z5oDUxdCvTYx.root
DB_PASSWORD=zXdZ5bfA39P5Rr8r
DB_NAME=quan_ly_sinh_de_ai_v2
DB_SSL=true

# Service URLs
EXAM_SERVICE_URL=http://127.0.0.1:8001
AI_SERVICE_URL=http://127.0.0.1:8002
ANALYTICS_SERVICE_URL=http://127.0.0.1:8003
AUTH_SERVICE_URL=http://127.0.0.1:8004
QUANLYTHI_SERVICE_URL=http://127.0.0.1:8005
```

### Bước 4: Khởi động Hệ thống

**Cách 1: Dùng script tự động (Khuyến nghị)**

```bash
# Chạy cả Frontend + Backend
npm run dev
```

**Script này sẽ:**
1. Tự động tạo virtualenv `dev/` nếu chưa có
2. Cài pip dependencies nếu chưa cài
3. Start Frontend Admin (:5173) và Portal (:5174)
4. Start 6 Backend services (8000-8005)

**Cách 2: Chạy riêng từng phần**

```bash
# Terminal 1: Backend services
python start_services.py

# Terminal 2: Frontend Admin
npm run dev:portal  # hoặc: vite

# Terminal 3: Frontend Portal
vite --port 5174
```

### Bước 5: Verify Services Đang Chạy

Sau khi chạy `npm run dev`, check terminal output:

```
🌐 [API Gateway] Khởi động trên Port 8000...
   → Exam Service:      http://127.0.0.1:8001
   → AI Service:        http://127.0.0.1:8002
   → Analytics Service: http://127.0.0.1:8003
   → Auth Service:      http://127.0.0.1:8004
✅ [API Gateway] Sẵn sàng định tuyến!
```

**Kiểm tra từng service:**

```bash
# API Gateway
curl http://localhost:8000/health
# Expected: {"status":"healthy","service":"api-gateway","port":8000}

# Exam Service
curl http://localhost:8001/health
# Expected: {"status":"healthy"}

# AI Service
curl http://localhost:8002/health

# Analytics Service
curl http://localhost:8003/health

# Auth Service
curl http://localhost:8004/health

# QuanLyThi Service
curl http://localhost:8005/health
```

**Nếu service nào trả về 503 hoặc không phản hồi:**
- Check terminal logs để xem lỗi gì
- Thường là lỗi kết nối database hoặc thiếu dependencies

### Bước 6: Truy cập Ứng dụng

Mở browser và truy cập:

1. **Frontend Admin**: http://localhost:5173
2. **Frontend Portal** (Thí sinh): http://localhost:5174
3. **API Gateway**: http://localhost:8000
4. **API Docs** (Swagger UI): 
   - Gateway: http://localhost:8000/docs
   - Exam Service: http://localhost:8001/docs
   - AI Service: http://localhost:8002/docs
   - Analytics: http://localhost:8003/docs
   - Auth: http://localhost:8004/docs
   - QuanLyThi: http://localhost:8005/docs

## 5. Xử lý Lỗi Thường Gặp

### Lỗi 1: `ModuleNotFoundError: No module named 'fastapi'`

**Nguyên nhân:** Python packages chưa cài hoặc chạy sai Python interpreter

**Giải pháp:**
```bash
# Đảm bảo virtualenv đang active
source dev/bin/activate  # Linux/macOS
dev\Scripts\activate     # Windows

# Cài lại dependencies
pip install -r requirements.txt
```

### Lỗi 2: Database connection error

**Lỗi thường thấy:**
```
[Database] Could not verify/create database
aiomysql.err.OperationalError: (2003, "Can't connect to MySQL server")
```

**Nguyên nhân:** 
- MySQL/TiDB không chạy
- Sai thông tin kết nối trong `.env`
- TiDB Cloud account hết hạn

**Giải pháp:**

**Nếu dùng TiDB Cloud:**
- Check internet connection
- Verify TiDB cluster còn hoạt động (login vào TiDB Cloud console)
- Thử ping: `ping gateway01.ap-southeast-1.prod.alicloud.tidbcloud.com`

**Nếu dùng MySQL Local:**
1. Cài MySQL 8.0: https://dev.mysql.com/downloads/mysql/
2. Start MySQL service:
   ```bash
   # Linux
   sudo systemctl start mysql
   
   # macOS
   brew services start mysql
   
   # Windows
   net start MySQL80
   ```
3. Tạo database và user:
   ```sql
   mysql -u root -p
   
   CREATE DATABASE quan_ly_sinh_de_ai_v2 CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   CREATE USER 'quan_ly_sinh_de_ai_v2'@'localhost' IDENTIFIED BY 'zXdZ5bfA39P5Rr8r';
   GRANT ALL PRIVILEGES ON quan_ly_sinh_de_ai_v2.* TO 'quan_ly_sinh_de_ai_v2'@'localhost';
   FLUSH PRIVILEGES;
   ```
4. Update `.env`:
   ```
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_USER=quan_ly_sinh_de_ai_v2
   DB_PASSWORD=zXdZ5bfA39P5Rr8r
   DB_NAME=quan_ly_sinh_de_ai_v2
   DB_SSL=false
   ```

### Lỗi 3: Gemini API Key invalid

**Lỗi:**
```
google.genai.errors.ClientError: API key not valid
```

**Giải pháp:**
1. Lấy API key mới từ https://aistudio.google.com/app/apikey
2. Update vào `.env`
3. Restart services

### Lỗi 4: Port already in use

**Lỗi:**
```
OSError: [Errno 98] Address already in use
```

**Nguyên nhân:** Port 5173, 5174, 8000-8005 đã bị process khác chiếm

**Giải pháp:**
```bash
# Tìm process đang dùng port
lsof -i :8000  # Linux/macOS
netstat -ano | findstr :8000  # Windows

# Kill process
kill -9 <PID>  # Linux/macOS
taskkill /PID <PID> /F  # Windows
```

### Lỗi 5: CORS error trên browser

**Lỗi:** `Access-Control-Allow-Origin` missing

**Nguyên nhân:** CORS middleware chưa config đúng

**Giải pháp:** Check `backend/gateway/main.py` - đã có CORS `allow_origins=["*"]`, nên không nên gặp lỗi này. Nếu vẫn lỗi, clear browser cache.

## 6. Database Auto-seeding

**Tin tốt:** Khi Exam Service khởi động lần đầu, nó sẽ **tự động**:

1. Tạo các bảng (tables) nếu chưa có
2. Nạp dữ liệu mẫu (seed data):
   - Danh mục môn học (Toán, Văn, Anh)
   - Cấp độ tư duy
   - Loại hình câu hỏi
   - Khối lớp
   - Thành phần năng lực

**Logic:** Xem `backend/exam_service/main.py` → `lifespan()` function

**Vì vậy bạn KHÔNG CẦN:**
- Chạy migration scripts thủ công
- Import SQL files
- Tự tạo bảng

Chỉ cần **database rỗng**, service sẽ lo phần còn lại!

## 7. Test Login

### Test Backend API:

```bash
# Test login endpoint
curl -X POST http://localhost:8005/api/exam/portal/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin123"}'
```

**Expected response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "user": {
      "username": "admin",
      "fullName": "Administrator"
    }
  }
}
```

### Test Frontend:

1. Truy cập http://localhost:5173 (Admin)
2. Thử login với credentials mặc định (nếu có)
3. Check Network tab trong DevTools để xem API calls

## 8. Kiểm tra AI Generation

### Test qua Swagger UI:

1. Truy cập: http://localhost:8002/docs
2. Tìm endpoint `/generate`
3. Click **"Try it out"**
4. Điền input:
   ```json
   {
     "subject": "Toán",
     "grade": "Lớp 10",
     "topic": "Hàm số bậc nhất",
     "difficulty": "medium",
     "questionType": "multiple_choice"
   }
   ```
5. Click **Execute**
6. Xem response - nếu trả về câu hỏi → Gemini API đang hoạt động

### Test qua Frontend:

1. Vào **Ngân hàng câu hỏi**
2. Click **Sinh câu hỏi bằng AI**
3. Điền thông tin và click **Sinh câu hỏi**
4. Nếu có câu hỏi xuất hiện → AI service OK

## 9. Monitoring và Logs

### Xem logs real-time:

```bash
# Nếu chạy bằng npm run dev
# Logs sẽ hiện trực tiếp trên terminal

# Nếu chạy riêng từng service
tail -f logs/exam_service.log  # (nếu có log files)
```

### Check service status:

```bash
curl http://localhost:8000/api/microservices/status
```

**Response:**
```json
{
  "success": true,
  "data": {
    "services": [
      {"id": "gateway", "status": "healthy", "port": 8000},
      {"id": "exam", "status": "healthy", "port": 8001},
      {"id": "ai", "status": "healthy", "port": 8002},
      ...
    ]
  }
}
```

## 10. Tắt Hệ thống

```bash
# Nếu chạy bằng npm run dev
# Nhấn Ctrl+C trong terminal

# Services sẽ tự động shutdown gracefully
```

**Verify tắt hết:**
```bash
# Không còn process nào lắng nghe các ports
lsof -i :8000  # Should return nothing
```

## Tóm tắt Các Lệnh Quan trọng

```bash
# 1. Setup (chỉ chạy 1 lần)
python3 -m venv dev
source dev/bin/activate
pip install -r requirements.txt

# 2. Chạy hệ thống
npm run dev

# 3. Test services
curl http://localhost:8000/health
curl http://localhost:8001/health

# 4. Truy cập web
# Admin: http://localhost:5173
# Portal: http://localhost:5174
# API Docs: http://localhost:8000/docs
```

## Câu hỏi cần trả lời

Để tôi hướng dẫn chính xác hơn, bạn vui lòng cho tôi biết:

1. **Database:** Bạn muốn dùng TiDB Cloud (đã có sẵn) hay MySQL local (phải cài và setup)?
2. **Gemini API Key:** Bạn có API key riêng không, hay cần tôi hướng dẫn tạo mới?
3. **Mục đích:** Bạn chạy để test chức năng nào cụ thể? (để tôi hướng dẫn test case cụ thể)
4. **Lỗi hiện tại:** Nếu bạn đã thử chạy và bị lỗi, log lỗi là gì?
