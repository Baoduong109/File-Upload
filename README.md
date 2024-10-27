
# Write-Up File Upload level 1-6: Goal: RCE

## Level 1:

Khởi động docker sau đó truy cập http://localhost:12001 để bắt đầu thực hành level 1.

<img width="715" alt="Ảnh chụp Màn hình 2024-10-28 lúc 00 46 57" src="https://github.com/user-attachments/assets/153488cd-6aef-498f-826a-7906f3afdbc5">

Tiến hành upload 1 file .php với nội dung ```<?php phpinfo(); ?>``` để kiểm tra xem server có chặn các file gây hại hay không.

<img width="709" alt="Ảnh chụp Màn hình 2024-10-28 lúc 00 59 19" src="https://github.com/user-attachments/assets/b8223784-c541-4869-80ea-a43672721937">

Ta thấy Successfully uploaded có nghĩa server hiện tại không hề chặn các file .php. Truy cập vào đường dẫn, nơi lưu trữ file để xem server có "chế biến" file .php hay không.

<img width="700" alt="Ảnh chụp Màn hình 2024-10-28 lúc 00 59 31" src="https://github.com/user-attachments/assets/daa38e93-f7e0-4aa9-b008-414b202a2cd8">

Như đã thấy, server đã hiển thị nôi dung của phpinfo. Để RCE, ta tiến hành upload 1 web shell với nội dung ```<?php system($_GET['cmd']); ?>```. Sau khi upload thành công file shell vừa tạo, ta truy cập vào đường dẫn lưu trữ file kèm theo payload sau: ```?cmd=ls /``` Để có thể đọc được các file trong thư mục root.

<img width="786" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 01 16" src="https://github.com/user-attachments/assets/0cd9c407-cf22-4a99-be85-2ea9948060cc">

Lúc này có thể thấy ta có thể xem được bất kì file nào. Thay đổi ```?cmd=cat /secret.txt``` để đọc flag.

<img width="999" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 01 05" src="https://github.com/user-attachments/assets/62a96cd2-5a45-4bce-b250-f4824de8003f">

Thành công RCE level 1 và lấy được flag.

## Level 2:

Truy cập http://localhost:12002 để bắt đầu level 2.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 01 29" src="https://github.com/user-attachments/assets/036b400c-b429-4d70-857e-4b662bcf55cb">

Tương tự level 1, ta thử upload 1 file .php với nội dung phpinfo() để kiểm tra xem có thể upload được file .php hay không.

<img width="563" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 01 43" src="https://github.com/user-attachments/assets/89bc7d7c-09f8-46fc-bfd0-8b2a1ef13fc0">

Kết quả là không thành công, có vẻ như anh dev đã bắt đầu ngăn chặn các file độc hại. Nhưng thật may mắn là ta pentest white box nên ta có thể xem source code để xem anh dev đã làm gì.

<img width="1051" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 02 39" src="https://github.com/user-attachments/assets/3f78078e-38b7-4734-8674-06d1c43c6541">

Chú ý đến dòng 19 và 20, ta thấy đoạn code đang kiểm tra phần tử đầu tiên sau dấu chấm. Nếu là 'php' thì lập tức kết thúc chương trình và hiển thị "Hack detected".

Lúc này có 2 giả thuyết được đặt ra là sẽ ra sao nếu ta đặt tên file 'info.abc.php' vì đoạn code trên chỉ kiểm tra phần tử đâu tiên sau dấu chấm. Hoặc là ngoài '.php' ra thì 'mod-php' có xử lý đuôi file nào khác tương tự 'php' hay không.

Thực hiện giả thuyết đầu tiên, truy cập vào Burp Suite, vào ```Proxy```, chọn gói tin có method là ```POST```, chuột phải và chọn ```Send to Repeater```. 

<img width="1439" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 06 11" src="https://github.com/user-attachments/assets/d5fad70f-fa6d-42c1-b22c-96b149f579ab">

Sau đó thay đổi đuôi file thành 'info.png.php' và Send.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 12 04" src="https://github.com/user-attachments/assets/296c6091-ce68-48c2-a815-faa2e66ada58">

Ở phía ```Response``` ta thấy Successfully uploaded file, nghĩa là ta đã vượt qua được bộ lọc mà anh dev đã đặt ra, truy cập vào đường dẫn file vừa tạo để xem server có trả về nội dung 'phpinfo' hay không.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 12 17" src="https://github.com/user-attachments/assets/6cc463cf-c1f6-4a9d-b9a2-8d1d60be60ff">

Như ta thấy server đã trả về nội dung của 'phpinfo()', như vậy là giả thuyết 1 thành công. Bây giờ ta có thể upload 1 file shell để RCE.

## Level 3:

Truy cập vào http://localhost:12003 để bắt đầu level 3.

Chắn chắn anh dev đã nâng cấp bộ lọc của mình ta không cần thử các file của level trước, tra vào xem source code xem anh dev đã thay đổi những gì.

<img width="1141" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 08 46" src="https://github.com/user-attachments/assets/05c46437-4670-4347-abec-5440b912a11d">

Nhìn bằng mắt thường ta sẽ thấy không khác gì so với level trước nên hãy để VS Code so sánh xem đoạn code ở level 3 có khác gì so với level 2 hay không.

Đầu tiên chọn file 'index.php' của level 3, chuột phải chọn ```Select for Compare```.

<img width="394" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 09 03" src="https://github.com/user-attachments/assets/6d031055-df9d-4778-9eb3-b75f225118ec">

Sau đó chuột phải vào file 'index.php' của level 2, chọn ```Compare with Selected```.

<img width="451" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 09 23" src="https://github.com/user-attachments/assets/111d491f-3396-4903-b307-7fb2e7653297">

