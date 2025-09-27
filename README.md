# Cookie + Session Authentication (Express + MongoDB)

Một ví dụ nhỏ về xác thực sử dụng session lưu trên MongoDB và cookie. Dự án minh họa cách đăng ký, đăng nhập, lưu session bằng `express-session` và `connect-mongo`, cùng một route bảo vệ trả về thông tin profile.

## Tính năng

- Đăng ký người dùng (hash password bằng bcrypt)
- Đăng nhập / lưu session trên MongoDB
- Route bảo vệ (`/auth/profile`) truy xuất thông tin người dùng từ session
- Đăng xuất (hủy session và xóa cookie)

## Yêu cầu

- Node.js (>= 14)
- MongoDB chạy trên localhost hoặc URL MongoDB hợp lệ

## Cài đặt

Mở PowerShell ở thư mục dự án (`d:\LTHDV\src\src\cookie_session_auth`) và chạy:

```powershell
npm install
```

> package.json hiện có các phụ thuộc: express, mongoose, express-session, connect-mongo, cookie-parser, bcryptjs

## Chạy ứng dụng

Ứng dụng chính là `app.js`. Chạy bằng Node:

```powershell
node app.js
```

Ứng dụng mặc định lắng nghe trên cổng 3000. (Console sẽ in: "Server running on http://localhost:3000")

### Lưu ý biến môi trường

Hiện tại `app.js` dùng các giá trị cứng:
- MongoDB: `mongodb://127.0.0.1:27017/sessionAuth`
- Session secret: `mysecretkey`
- PORT: `3000`

Nếu bạn muốn dùng biến môi trường thay vì sửa file, cập nhật `app.js` để dùng `process.env.MONGO_URL`, `process.env.SESSION_SECRET` và `process.env.PORT` rồi chạy tương tự.

Bạn có thể tạm thời đặt biến môi trường trong PowerShell trước khi chạy (nếu đã chỉnh `app.js` để đọc env):

```powershell
$env:MONGO_URL = 'mongodb://127.0.0.1:27017/sessionAuth'; $env:SESSION_SECRET = 'your_secret'; $env:PORT = '3000'; node app.js
```

## API

Base URL: http://localhost:3000/auth

1) Đăng ký (register)

- Endpoint: POST /auth/register
- Body (JSON): { "username": "alice", "password": "secret" }

Curl:

```bash
curl -X POST http://localhost:3000/auth/register -H "Content-Type: application/json" -d '{"username":"alice","password":"secret"}'
```

PowerShell (Invoke-RestMethod):

```powershell
Invoke-RestMethod -Method Post -Uri http://localhost:3000/auth/register -ContentType 'application/json' -Body (@{ username='alice'; password='secret' } | ConvertTo-Json)
```

2) Đăng nhập (login)

- Endpoint: POST /auth/login
- Body (JSON): { "username": "alice", "password": "secret" }
- Khi login thành công server sẽ tạo session và gửi cookie (mặc định cookie tên `connect.sid`).

Curl (lưu cookie ra file `cookies.txt`):

```bash
curl -c cookies.txt -X POST http://localhost:3000/auth/login -H "Content-Type: application/json" -d '{"username":"alice","password":"secret"}'
```

Sau đó gửi cookie khi gọi route bảo vệ:

```bash
curl -b cookies.txt http://localhost:3000/auth/profile
```

PowerShell (giữ cookie bằng `-WebSession`):

```powershell
$s = New-Object Microsoft.PowerShell.Commands.WebRequestSession
Invoke-RestMethod -Method Post -Uri http://localhost:3000/auth/login -ContentType 'application/json' -Body (@{ username='alice'; password='secret' } | ConvertTo-Json) -WebSession $s
Invoke-RestMethod -Uri http://localhost:3000/auth/profile -WebSession $s
```

3) Đăng xuất (logout)

- Endpoint: GET /auth/logout
- Hủy session ở server và xóa cookie `connect.sid`

Curl:

```bash
curl -b cookies.txt http://localhost:3000/auth/logout
```

## Cấu trúc dự án

- `app.js` - cấu hình server, session, kết nối MongoDB và mount route `/auth`
- `routes/auth.js` - route đăng ký, đăng nhập, đăng xuất, profile
- `models/User.js` - schema Mongoose cho user, hash password trước khi lưu

## Bảo mật & Ghi chú

- Hiện `SESSION` secret và MongoDB URL được ghi cứng trong `app.js`. Nên thay bằng biến môi trường trong môi trường production.
- `cookie.secure` hiện được đặt `false` (phù hợp khi chạy trên HTTP local). Nếu deploy qua HTTPS, hãy đặt `cookie.secure = true`.
- Mật khẩu được hash bằng `bcryptjs` trước khi lưu.
- Mặc định session store sử dụng `connect-mongo` lưu vào cùng MongoDB; đảm bảo MongoDB được cấu hình bảo mật nếu dùng production.

## Phụ thuộc (từ package.json)

- bcryptjs
- connect-mongo
- cookie-parser
- express (v5 trong package.json)
- express-session
- mongoose

## Gợi ý nâng cấp

- Thêm biến môi trường và `.env` (với `dotenv`) để cấu hình môi trường dễ dàng
- Thêm kiểm tra đầu vào (validation) cho username/password
- Thêm cơ chế rate-limit cho các route đăng nhập
- Thêm script `start` trong `package.json`

## License

Mã nguồn ví dụ — chỉnh sửa tự do cho mục đích học tập.


---

Nếu bạn muốn, tôi có thể: thêm script `start` vào `package.json`, hoặc cập nhật `app.js` để sử dụng biến môi trường — bạn muốn tôi làm gì tiếp theo?