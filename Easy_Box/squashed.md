# BOX SQUASHED
- [BOX SQUASHED](#box-squashed)
  - [1. Box info](#1-box-info)
  - [2. Shell as user](#2-shell-as-user)
  - [3. Shell as root](#3-shell-as-root)
  - [4. Kết luận](#4-kết-luận)

## 1. Box info

- Box SQUASHED là một `machine` free trong phần `Retired Machine`. Nó có địa chỉ là `10.10.11.191` và gồm flag user và flag root.
- Mục tiêu của chúng ta là khai thác các lỗ hỏng, missconfig,...để có thể lấy được 2 flag và submit.
- Trong phần `infomation` có cung cấp một số `keywords` liên quan đến machine:

    ![](https://i.imgur.com/sQE6xSF.png)

## 2. Shell as user
- Service scanning :
    Đầu tiên ta thực hiện `nmap -sC -sV -p- -T4 --min-rate 1000 10.10.11.191` để scan các dịch vụ trên tất cả các port (`-p-`) cùng với phiên bản(`-sV`) và script default (`-sC`), cuối cùng là sẽ giảm thời gian bằng các gửi 1000 gói tin trong 1 lần (`--min-rate`) và scan ở cường độ cao hơn mức bình thường (`-T4`).
    
    ![](https://i.imgur.com/FJ9vCri.png)
    
    Sau khi thực hiện thì ta thấy hệ thống này có port đang ở trình trạng mở như `SSH(22)`,`HTTP(80)` , `RPC(111)`, `NFS(2049)` ,`HTTP(5555)`và các port hỗ trợ `RPC`.
    
-	Web Enumeration :
    
    Sau khi scan các service trên hệ thống thì ta có thể thấy nó có service web. Vào trang web và kiểm tra thì thấy trang web này chẳng có gì hữu ích cả.
    
    ![](https://i.imgur.com/mo10Fry.jpg)

    Thử thực hiện `brute force` xem trang web các `directory`  bằng câu lệnh `ffuf -u http://10.10.11.191/FUZZ -w ./SecLists/Discovery/Web-Content/dirsearch.txt -mc all -fs 274` thì thấy cũng chẳng có gì hữu ích cả.
    
    ![](https://i.imgur.com/X8jJgiE.png)

-	Inital Foothold :

    Ta thấy kế quả lúc scan service bằng `nmap` ở trên thì có thể thấy hệ thống này có thể bị lỗ hỏng `Mountable NFS share`. Nó là một lỗ hỏng có rủi ro rất cao và thường được tìm thấy trong mạng lưới mạng toàn cầu. Nó khá khó phát hiện và giải quyết nên có thể đôi khi dẫn đến việc bị bỏ qua. Lỗ hỏng này cho phép các remote hosts có thể mount vào hệ thống hoặc thư mục qua mạng. Điều này xảy ra khi người quản trị cấu hình bị thiếu sót, từ đó ta có thể vào hệ thống và đọc dữ liệu. 
    
    - Mount NFS:
         Thực hiện`showmount -e 10.10.11.191` để liệt kê ra các NFS share có sẵn.
     
         ![](https://i.imgur.com/pW2pXMi.png)

        Thì ta thấy ở đây ta có thể mount vào `/home/ross` và `/var/www/html`. Ta lần lượt thử mount nó vào `/mnt` và kiểm tra
        - Đối với thư mục `/home/ross`:
            Ta thực hiện bằng câu lệnh `sudo mount -t nfs 10.10.11.191:/home/ross /mnt` và kiểm tra thư mục bằng lệnh `find /mnt -ls` 
            
            ![](https://i.imgur.com/SMLueSg.png)

        - Đối với thư mục `/var/www/html`:
            Ta thực câu lệnh `umount /mnt` rồi sau đó hiện bằng câu lệnh `sudo mount -t nfs 10.10.11.191:/var/www/html /mnt`. Dễ thấy rằng chúng ta không có quyền truy cập vào tài nguyên này.
            
            ![](https://i.imgur.com/9QXf0SQ.png)

            
            
     Sau khi kiểm tra 2 `share mount` thì ta có thể thấy tại `/mnt` khi mount với `/home/ross` thì có 2 file đáng ngờ là `Passwords.kdbx` và file `.Xauthority`. 
     
     Trong đó thì file `Passwords.kdbx` là `KeyPass Password Database` có định dạng nhị phân được dùng để quản lý mật khẩu cho windows, ta thử thực hiện trích xuất hash password của file `Passwords.kdbx` này bằng lệnh `keepass2john Passwords.kdbx > password.hash`. Tuy nhiên file này ở version 4 nên không hỗ trợ. Còn `.Xauthority` là tệp cookie được `X11` dùng để cho phép truy cập. Nó sẽ là tiền đề để ta khởi tạo shell với quyền root.
     
     Còn trong `/mnt` khi mount với `/var/www/html` thì ta quan sát thấy thì thư mục `/mnt` này có `userid = 2017` và `groupid bằng với group id của www-data`. Nhưng tài khoản `zios` của chúng ta thì có `userid và groupid là 1000` vì vậy nên mới bị ngăn chặn truy cập tài nguyên. Ta còn được biết là NFS sẽ không theo dõi user/group trên các máy vì vậy ta chỉ cần tạo một user có `userid =2017` là có thể truy cập được tài nguyên một cách hợp pháp. Ở đây ta thực hiện tạo một user `thehao` mang `userid = 2017` và gọi bash shell để truy cập tài nguyên. Bây giờ ta có quyền `read` và `write` trên này. Vậy nên ta sẽ tạo một `reverse shell`.
     
     Thử kiểm tra bằng cách tạo một file `php` thì thấy nó có thể chạy được. 
     
     ![](https://i.imgur.com/NHMiFFB.png)
     
     ![](https://i.imgur.com/fXHSCIG.png)

     Ta thực hiện tạo một `reverse shell` bằng cách tạo 1 file `shell.php` với nội dung như sau :

    ![](https://i.imgur.com/31L1Zut.png)

     Và truy cập file đó bằng url đồng thời thực hiện lắng nghe thì nhận được kết nối 
     
     ![](https://i.imgur.com/R2dy6iJ.png)

    Vậy là ta đã khởi tạo shell với quyền alex thành công. Ta có thể thực hiện upgrade shell TTY để thao tác dễ dàng hơn bằng lệnh `python -c 'import pty; pty.spawn("/bin/bash")'` -> `ctrl+z` -> `stty raw -echo` -> `fg`-> `Enter`.
    
    ![](https://i.imgur.com/ewzgncW.png)

    Bây giờ ta chỉ cần đọc nội dung file `user.txt` là có thể lấy được flag user.
     
## 3. Shell as root

Như đã nói ở trên thì bây giờ là lúc dùng đền file `.Xauthority`. Về cơ bản thì file này lưu trữ session của user mỗi khi đăng nhập. Ta có thể dùng file này để thay thế session của Alex với session của Ross.

Vậy đầu tiên thì ta nên copy file `.Xauthority` đó từ thư mục `/mnt` sang cho `alex` bằng cách tạo server ở `/mnt`

![](https://i.imgur.com/udbx84c.png)

Và dùng `curl http://10.10.14.35:8000/.Xauthority -o /tmp/.Xauthority` để tải file đó về.

![](https://i.imgur.com/0ZdQP1s.png)

Tuy nhiên thì `alex` cũng có 1 `.Xauthority` riêng để xác minh. Bây giờ ta cần thay đổi nó để có thể thay đổi thành session của Ross. Thì mỗi khi kiểm tra xem user có cookie hay không nó sẽ kiểm tra trong biến môi trường `XAUTHORITY`. Bây giờ ta chỉ cần tạo đường dẫn trong biến môi trường này để đánh lừa và dẫn tới `/tmp/.Xauthority`. 

Sau đó dùng câu lệnh để chụp màn hình `xwd -root -screen -silent -display :0 > ~/img.xwd`.

Tiếp theo đó thì đưa file về máy local của chúng ta và chuyển sang file dạng `png` để có thể quan sát một cách dễ dàng bằng câu lệnh `convert screenshot.xwd screenshot.png`

![](https://i.imgur.com/KFvEdoG.png)

Ta chỉ cần dùng password trên và dùng lệnh `su` để đăng nhập vào `root`. Vậy là có thể đọc được flag root.

![](https://i.imgur.com/O3F6i0d.png)

## 4. Kết luận

- Qua box SQUASHED này thì ta có thể thấy hệ thống tồn tại khá nhiều lỗi với các mức độ ảnh hưởng khá nghiêm trọng như `weak password, weak configured, PATH variable, Mountable NFS share`.
- Từ đó ta có thể đưa ra một số khuyến nghị như sau :
    - Thực hiện cấu hình một cách chính xác, hạn chế đối với tất cả các NFS share hoặc chặn luôn các truy cập bên ngoài. Đồng thời dùng các password không phổ biến, có đủ độ phức tạp và thay đổi nó một cách định kỳ

- Vậy là ta đã hoàn thành được box SQUASHED này.