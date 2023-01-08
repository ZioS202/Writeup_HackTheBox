# BOX ALUNNO 
- [BOX ALUNNO](#box-alunno)
  - [1. Box info](#1-box-info)
  - [2. Shell as user](#2-shell-as-user)
  - [3. Shell as root](#3-shell-as-root)
  - [4. Kết luận](#4-kết-luận)

## 1. Box info

- Box ALUNNO này là một box được thiết kế cho thi thực hành giữa kỳ môn ATM của trường UIT. 
Box có dải địa chỉ là `192.168.19.(201->210)`. Và gồm 7 flag bonus cùng với flag user và flag root. 
- Mục tiêu của chúng ta là khai thác các lỗ hỏng, missconfig,... ở 1 trong số IP trên để có thể lấy được 9 flag và hoàn thành bài thi.
## 2. Shell as user

-	Service scanning :

    Đầu tiên ta thực hiện lệnh `nmap -p- -T4 –min-rate 10000 –script banner 192.168.19.203` để scan các dịch vụ, lấy banner trên tất cả các port (`-p-`), với cường độ scan lớn hơn bình thường (`-T4`) và gửi 10000 gói tin trong 1 lần để giảm thời gian scan (`--min-rate`).

    ![](https://i.imgur.com/72FCG9h.png)

    Sau khi thực hiện scan thì ta thấy hệ thống CNTT này có các port đang ở tình trạng open như `22/tcp` chạy service SSH, `80/tcp` chạy service http, `9696/tcp` thì chưa biết nhưng ta thấy banner của nó cho ta **`Flag01`**. Vậy là ta đã thực hiện thành công Alunno bonus 1 và biết được cơ bản các dịch vụ chạy trên hệ thống CNTT

-	Web Enumeration :

    Thực hiện đăng nhập thử bằng một tài khoản giả 
    
    ![](https://i.imgur.com/GRSqQNG.png)
    
    Kiểm tra các request bằng burpsuite
    
    ![](https://i.imgur.com/j8s9mgp.png)

    Dễ dàng thấy request này gửi đến Host là `www.alunno.inseclab`. Vậy ta thực hiện mapping ip `192.168.19.203` với tên Host này trong thư mục `etc/hosts`. Nhưng chưa chắc hệ thống CNTT này chỉ duy nhất một domain nên ta chỉ mapping `alunno.inseclab` với địa chỉ ip `192.168.19.203` để tìm thêm các domain khác.
    
    ![](https://i.imgur.com/oijhQkR.png)

    Ta thực hiện lệnh `gobuster vhost -u http://alunno.inseclab -w ~/SecLists/Discovery/DNS/ bitquark-subdomains-top100000.txt -t 100 --no-error` để dò xem còn domain nào khác không thông qua wordlist thông dụng `bitquark-subdomains-top100000.txt`.
    
    ![](https://i.imgur.com/UgLCu3u.png)

    Ta thấy nó đã tìm được domain khác là `api-dev.alunno.inseclab`. Bây giờ ta sẽ thêm domain này và domain lúc trước `www.alunno.inseclab` mapping với địa chỉ ip `192.168.19.203`.
    
    ![](https://i.imgur.com/KniuTXB.png)

    Ta thử truy cập bằng domain `api-dev.alunno.inseclab` thì thấy có được **`Flag02`**. Vậy là ta đã thực hiện được Alunno bonus 2.
    
    ![](https://i.imgur.com/JhsVTTZ.png)

    Tại đây ta thấy có một URL `/admin` . Thử kết quả này với 2 domain đã tìm : 
    - Tại domain `www.alunno.inseclab` 
        Ta thấy xuất hiện trang `django-admin` và cần đăng nhập bằng tài khoản admin mới có thể truy cập được vào database. Bây giờ ta cần tìm tài khoản admin. Thì ta nhớ rằng trang này được làm từ một người có user github là `pinanek23` và có gắn kèm link ở trang Signin. Thử vào tìm qua các repo thì tại repo `WebAppSecProject` ta đã có thể tìm được tài khoản admin mặc định được tạo.
        
        ![](https://i.imgur.com/0u96LB7.png)

        Thử đăng nhập vào bằng tài khoản này thì thấy ta đã có thể truy cập được database. 
        
        ![](https://i.imgur.com/8SHjdhK.png)

        Vậy bây giờ ta thực hiện khám phá các thông tin trong database này thì thấy trong `ACCOUNT` có một member là `flag` với password là **`Flag03`**. Vậy là ta đã hoàn thành Alunno bonus 3.
        
        ![](https://i.imgur.com/aZsoHv2.png)

    - Tại domain `api.alunno.inseclab` 

        Tương tự như domain trên ta cũng thực hiện đăng nhập vào trang `django-admi`n và khám phá thì thấy một subdomain khác được nhắc đến trong phần lesson đó là `f5gi8.alunno.inseclab`
        
        ![](https://i.imgur.com/T3qtv9F.png)

        Vậy bây giờ ta thực hiện mapping thêm domain mới này.

        ![](https://i.imgur.com/iwouTpj.png)

    Sau khi đã mapping thì ta truy cập vào domain mới này kiểm tra thử xem thì ra một trang mới.
    
    ![](https://i.imgur.com/lceGeyK.png)
    
    Ta thử truy cập bằng tài khoản `admin` khi nãy nhưng không hiệu quả. Đoán rằng tài khoản này vẫn có thể ở default nên ta thực hiện tìm kiếm bằng google. Thì thấy trên github của nền tảng này có tài khoản default.
    
    ![](https://i.imgur.com/O8GWgLp.png)

    Đúng như chúng ta đoán thì ta đã có thể dùng tài khoản mặc định này để truy cập vào subdomain này.
    
    ![](https://i.imgur.com/1jCbkcV.png)
    
    Khám phá và đọc thử các file thì thấy một file sẽ thực hiện `system` để đọc file `flag` cho chúng ta.
    
    ![](https://i.imgur.com/jvIjxdx.png)

    Bây giờ ta thực hiện file này thì thấy kết quả trả về là **`Flag04`**. Vậy là ta đã hoàn thành được Alunno bonus 4
    
    ![](https://i.imgur.com/1X3xLjo.png)

-	Inital Foothold :

    Vậy là ta đã tìm được các thông tin cơ bản về hệ thông CNTT. Bây giờ ta sẽ khởi tạo shell với tư cách là một user thường bằng ssh.
    
    Thì ta nhớ lại trong phần member ở domain `www.alunno.inseclab` thì ngoài flag ra còn có các user khác. Mặc dù mật khẩu được lưu dưới dạng hash.
    
    ![](https://i.imgur.com/MsyDilL.png)
    
    Tuy nhiên ta có thể crack được password này thông qua công cụ `hashcat`. Đầu tiên ta copy các password của các user này vào một file `passHash.txt` 
    
    ![](https://i.imgur.com/JRTIqSO.png)

    Sau đó ta dùng `hashcat` để dò password trong list `rockyou.txt` bằng câu lệnh `hashcat -m 10000 -a 0 -o passCrack.txt passHash.txt /usr/share/wordlist/rockyou.txt`.
    
    ![](https://i.imgur.com/HKUlz8p.png)

    Trong đó `-m 10000` là chọn thuật toán hash Django `(PBKDF2-SHA256)`, `-a 0` là chọn cách dò và `-o` là chọn file sẽ ghi ouput vào. 
    Sau khi dò thì ta thấy được các password khớp với mã hash này đã có.
    
    ![](https://i.imgur.com/FFU4scr.png)

    Bây giờ ta thực hiện đăng nhập vào hệ thống bằng ssh với các username và password tương ứng thì ta vào được tài khoản `alunno` với pass là `kaitlynn`
    
    ![](https://i.imgur.com/azH6bwk.png)

    Vậy ta chỉ cần đọc nội dung file user.txt là hoàn thành được flag user.

    ![](https://i.imgur.com/yL6nW2H.png)


## 3. Shell as root

Đầu tiên ta thực hiện tìm kiếm các tập tin có quyền SUID khả nghi bằng lệnh `find / -perm -u=s -type f 2>/dev/null`.

![](https://i.imgur.com/tRdWLZC.png)

Sau khi tìm kiếm thì ta thấy tập tin `icheck` này có vẻ là con đường dẫn đến root. Ta thử thực hiện thì nó yêu cầu các flag 5 6 7 để unlock  vì vậy cần phải tìm các flag 5 6 7 ứng với các Alunno bonus 5 6 7

-	Flag 5 + Flag 6:

    Ta thực hiện tìm kiếm tất cả các tập tin có size bé hơn `100 bytes` (vì thường nội dung flag khá nhỏ) bằng lệnh `find / -type f -size -100c | while read line; do cat $line | grep "Flag"; done.`
    Nhưng nó sẽ có thể tốn rất nhiều thời gian và có thể bị ngắt quảng giữa chừng nên ta thực hiện tìm kiếm bắt đầu ở các thư mục lớn thay vì bắt đầu ở root. 
    - Tìm kiếm tại thư mục `usr`:
    
        ![](https://i.imgur.com/uzO8rUI.png)
        
    - Tìm kiếm tại thư mục `var`:

        ![](https://i.imgur.com/2orRxbd.png)
        
    Vậy là ta có thể tìm thấy được **`Flag 5`** và **`Flag 6`** ứng với Alunno bonus 5 6.

-	Flag 7:
    Kiểm tra tcp listen bằng lệnh `ss -tulwn | grep LISTEN `thì thấy có các port mở như `9696 9697 80 53 22 80`. 
    
    ![](https://i.imgur.com/7Hi5q6k.png)

    Thấy một port mới khi nmap chúng ta scan không thấy đó là `9697`. Ta thử connect vào và nhận được **`flag 7`** ứng với Alunno bonus 7
    
    ![](https://i.imgur.com/yD7TsjZ.png)

Và sau khi có được các flag 5 6 7 ta thực hiện unlock thì thấy `icheck` nó sẽ cố gắng thực hiện lệnh `ping`.

![](https://i.imgur.com/2orXGyN.png)

Nhìn thôi cũng đủ biết nó sẽ là mấu chốt để dẫn ta đến root. Ta có thể dùng lệnh ping này để leo lên root không? Câu trả lời là có. Với lỗi ghi đè PATH variable thì điều đó hoàn toàn có thể.

Đâu tiên ta thực hiện vào thư mục `/tmp` sau đó thực hiện tạo một file `ping` với nội dung là `“/bin/bash”` , tiếp theo thực hiện cấp quyền và gán nó vào PATH variable để ghi đè lệnh ping 

![](https://i.imgur.com/MtEBhj1.png)

Bây giờ quay lại thực hiện `icheck` và kiểm tra thì thấy ta đã có thể lên được quyền root. Bởi vì khi chạy `icheck` thì chương trình sẽ kiểm tra biến môi trường để tìm kiếm đường dẫn đến lệnh `ping`. Ở đây t đã thêm `/tmp` vào biến môi trường nên nó sẽ tìm vào `/tmp` và sẽ lấy lệnh `ping` trong `/tmp` này thay vì lệnh `ping` gốc.
    
![](https://i.imgur.com/3AULWoW.png)

## 4. Kết luận

- Qua box ALUNNO này thì ta có thể thấy hệ thống tồn tại khá nhiều lỗi với các mức độ ảnh hưởng khá nghiêm trọng như `weak password, weak configured, PATH variable`.

- Từ đó ta có thể đưa ra một số khuyến nghị như sau :
    - Dùng password không nằm trong các wordlist thông dụng, có đủ độ phức tạp và nên thay đổi nó một cách định kỳ. Đồng thời những thông tin default cần phải được thay đổi để đảm bảo yếu tố bảo mật.
    - Không nên tạo sẵn các file SUID dưới quyền root, cần phải phân phần quyền một cách hợp lý. Khi dùng các lệnh thì cần phải chỉ rõ đường dẫn.
        **Ví dụ** : Thay vì `ping 8.8.8.8` thì hãy dùng `/usr/bin/ping 8.8.8.8`
        
- Vậy là ta đã hoàn thành được box ALUNNO này.