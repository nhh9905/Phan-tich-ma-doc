# Bài tập phân tích tĩnh mã độc
Phân tích tĩnh mã độc brbbot.exe trong thư mục sample buoi 2.zip (pass giải nén: infected). Xác định:
## 1. Mã độc có bị virustotal gán nhãn độc hại hay không?
- Truy cập vào trang web `virustotal.com`, ta thấy 63/73 vấn đề mã độc

![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/1.png)
## 2. Mã độc có đạt kỹ thuật persistence?
- Mở file mã độc bằng `IDA Pro`, vào mục `Imports` ta thấy nhiều hàm `Windows API`

![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/2.png)
- Ấn `x` để xem các hàm con có chứa hàm windows API. Để ý hàm `RegOpenKeyExA`, ta thấy có đường dẫn `Software\\Microsoft\\Windows\\CurrentVersion\\Run`, chương trình sẽ thực thi khi hệ thống khởi động
- `GetModuleFileNameA`: Trả về đường dẫn của file bao gồm module chỉ định, tức là trả về đường dẫn của file exe
- Ấn `r` để xem char của số đó
- `GetEnvironmentVariableA`: biến `Buffer` lưu đường dẫn của `APPDATA`
- `RegOpenKeyExA`: 

![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/3.png)
![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/4.png)
-> Hàm `sub_140002230` (hàm persistence): lấy tên file + đường dẫn thư mục `appdata`, sau đó nối tên file vào đường dẫn, mở autorun key, ghi đường dẫn mã độc vào `brbbot`, di chuyển file mã độc và file config `brbconfig.tmp` vào `C:\Users\Asus\AppData\Roaming`
## 3. Strings của file mã độc gợi ý điều gì?
- Strings này sẽ gợi ý cho chúng ta có mã độc này sẽ có `persistence`
![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/5.png)
- Có 1 số hàm API liên quan đến crypto, kết nối mạng, API liên quan đến resource
![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/6.png)
- 1 số API có khả năng `persistence`

![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/7.png)
## 4. Dấu hiệu file của mã độc xuất hiện trên máy nạn nhân?
- Dấu hiệu về file: đường dẫn của file `brbbot.exe`
- Dấu hiệu đường dẫn `appdata`
- 1 số hàm API: `WriteFile`, `MoveFile`...
## 5. Dấu hiệu về network của mã độc?
- 1 số string về network

![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/8.png)
- 1 số API liên quan

![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/9.png)
- Ngoài ra có thể dùng `behavior` của `virustotal.com` mà ko cần dùng `IDA`
## 6. Mã độc sử dụng thuật toán gì để giải mã config?
- Truy cập vào hàm `sub_140002C50`, ta thấy một số hàm mã hóa

![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/10.png)
- Để ý hàm `CryptCreateHash` và tìm hiểu các thông số của chúng, tham số thứ 2 `Algid` giúp chúng ta nhận dạng được thuật toán mã hóa
![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/11.png)
- Chuột phải vào `0x8003u` và chọn enums, ta thấy rằng đó là thuật toán `MD5`

![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/12.png)
- `CryptHashData`: tạo mã hash để làm key cho hàm sau
- `CryptDeriveKey`: chú ý đến tham số 2 và 3, tham số 2 nhận dạng được là thuật toán `RC4`, tham số 3 để giải mã file config
- Vào `cyberchef`, ta lấy được key giải mã của file config từ input

![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/13.png)
- Dùng tool `Resource Hacker`, ta lấy được file config và giải mã được chúng

![alt text](https://github.com/nhh9905/Phan-tich-ma-doc/blob/main/Phan%20tich%20tinh/14.png)
Tổng kết: mở file và lấy handle, kích thước file và đọc dữ liệu từ file (v14), truyền tham số v14 vào `CryptDecrypt` để giải mã

## 7. Mã độc thuộc loại mã độc nào?

```
Tổng quan về file mã độc:
- sub_140002230: copy file vào appdata
- sub_140001150: đọc dữ liệu CONFIG từ Resource Hacker, tạo file brbconfig.tmp và ghi dữ liệu vào file config
- sub_140002C50: giải mã config
- sub_1400012E0: parse config
- sub_140001C10: lấy thông tin hệ thống và mã hóa dữ liệu
```