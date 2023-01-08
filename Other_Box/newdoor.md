# BOX NEWDOOR
- [BOX NEWDOOR](#box-newdoor)
  - [1. Box info](#1-box-info)
  - [2. Shell as user](#2-shell-as-user)
  - [3. Shell as root](#3-shell-as-root)
  - [4. Kết luận](#4-kết-luận)

## 1. Box info
- Box NEWDOOR này là một box được thiết kế cho thi thực hành cuối kỳ môn ATM của trường UIT. 
Box có dải địa chỉ là `192.168.19.(120->134)`. Và gồm 3 flag bonus cùng với flag user và flag root. 
- Mục tiêu của chúng ta là khai thác các lỗ hỏng, missconfig,... ở 1 trong số IP trên để có thể lấy được 5 flag và hoàn thành bài thi.
## 2. Shell as user
- Service scanning :
    
    Đầu tiên ta thực hiện `nmap -p- -sC -sV --min-rate 1000 --script banner 192.168.19.120` để scan các dịch vụ trên tất cả các port (`-p-`) cùng với phiên bản(`-sV`) và script default (`-sC`) đặc biệt chỉ định script lấy banner, cuối cùng là sẽ giảm thời gian bằng các gửi 1000 gói tin trong 1 lần (`--min-rate`)
    
    ![](https://i.imgur.com/lRMVvdf.png)

    Sau khi thực hiện thì ta thấy hệ thống CNTT này có port đang ở trình trạng mở như `SSH(22)`, `RPC(111)`, `NFS(2049)` và các port hỗ trợ `RPC`.
    Thì có thể thấy hệ thống này có thể bị lỗ hỏng Mountable NFS share. Nó là một lỗ hỏng có rủi ro rất cao và thường được tìm thấy trong mạng lưới mạng toàn cầu. Nó khá khó phát hiện và giải quyết nên có thể đôi khi dẫn đến việc bị bỏ qua. Lỗ hỏng này cho phép các remote hosts có thể mount vào hệ thống hoặc thư mục qua mạng. Điều này xảy ra khi người quản trị cấu hình bị thiếu sót, từ đó ta có thể vào hệ thống và đọc dữ liệu.
    
- Mount NFS:
     Thực hiện` showmount -e 192.168.19.120` để liệt kê ra các NFS share có sẵn.
     
     ![](https://i.imgur.com/b94qH76.png)

    Thì ta thấy ở đây ta có thể mount vào `/var/nfs/keepass`. Vì vậy ta thực hiện mount nó vào `/mnt` bằng câu lệnh `sudo mount -t nfs 192.168.19.120:var/nfs/keepass /mnt`
    
    ![](https://i.imgur.com/9uMWi8l.png)

    Sau đó thực hiện kiểm tra thư mục `/mnt` vừa mới mount xong thì thấy rằng chúng ta không có quyền truy cập vào tài nguyên này
    
    ![](https://i.imgur.com/gIRmsfi.png)

    Ta quan sát thấy thì thư mục `/mnt` này có `userid = 2022` và `groupid = 2202`. Nhưng tài khoản `zios` của chúng ta thì có `userid và groupid là 1000` vì vậy nên mới bị ngăn chặn truy cập tài nguyên. Ta còn được biết là NFS sẽ không theo dõi user/group trên các máy vì vậy ta chỉ cần tạo một user có `userid =2022` là có thể truy cập được tài nguyên một cách hợp pháp. Ở đây ta thực hiện tạo một user `thehao` mang `userid = 2022` và gọi bash shell để truy cập tài nguyên.
    
    ![](https://i.imgur.com/DKGK7yT.png)

    Sau khi đã có bash shell với `userid là 2022` thì ta đã có thể truy cập vào `/mnt`. Thực hiện đọc file `nfs.flag.txt` để giải quyết challenge `Newdoor lite subflag 1`

    ![](https://i.imgur.com/VmhJgLN.png)

- Initial Foothold:
    
    Tại thư mục mount `/mnt` thì ta thấy ngoài flag ra thì còn file `secure.kdbx`. Đây chính là `KeyPass Password Database` có định dạng nhị phân được dùng để quản lý mật khẩu cho windows. Bây giờ ta thực hiện trích xuất hash password của file `secure.kbdx` này bằng lệnh `keepass2john secure.kbdx > secure hash`.
    
    ![](https://i.imgur.com/YFZ6Gqj.png)

    Vì `keepass2john` tự động thêm tên file vào nên phải xóa ``“secure:”`` trước khi làm bước kế tiếp . Sau khi hoàn tất thì ta dùng `hashcat` để có thể dò password bằng wordlist `rockyou.txt` với câu lệnh 
    `hashcat -m 13400 -a 0 secure.hash '/home/zios/Downloads/rockyou.txt’`

    ![](https://i.imgur.com/dE3LYLP.png)

    Chỉ trong thời gian ngắn đã có thể dò ra được password của KeyPass database là `newholland`. Có được password rồi thì ta cùng xem trong KeyPass database có gì bằng `KeePass2` `(sudo apt install keepass2)`.
    
    ![](https://i.imgur.com/jLzmHhv.png)

    Sau khi đăng nhập thành công và quan sát thì thấy được có 1 dòng với title là `Flag` và 1 dòng với title là `ssh`.
    
    ![](https://i.imgur.com/p8h55ou.png)

    Thực hiện double-click vào title Flag để đọc flag và giải quyết challenge `Newdoor lite subflag 2`
    
    ![](https://i.imgur.com/JqM2lcy.png)

    Tiếp tục thực hiện double-click vào title `ssh` để kiểm tra tài khoản `newdoor` và password của nó.
    
    ![](https://i.imgur.com/2T5T5t4.png)

    Vậy là ta đã có thông tin để kết tạo kết nối ssh đó là `username = newdoor` và `password = 254U&jvT8eWpAgz2` . Thực hiện tạo kết nối ssh và khởi tạo shell với tư cách là một user thường.
    
    ![](https://i.imgur.com/SLZtzUK.png)

    Tiếp theo thì chỉ cần đọc nội dung file user.txt thì có thể lấy được flag user.
    
    ![](https://i.imgur.com/PVBH2xU.png)

## 3. Shell as root

Trong quá trình tìm kiếm thì ta tìm được file docker.sock tại `/run/docker.sock`.

![](https://i.imgur.com/6NoZBDe.png)

Ta có thể xác định được lỗ hỏng này có thể là `Mount Docker Socket`. Đôi khi vì một lý do nào đó cần mount docker socket vào bên trong docker để kết nối với docker daemon thực hiện một số hành động ví dụ như là sysadmin và developers cần debug chương trình bằng cách đọc log của nó từ bên trong container. Đây là một vấn đề nghiêm trọng, nếu như attacker tấn công vào các container này thì attacker có thể dùng các câu lệnh hoặc các thư viện docker tương tác với docker thông qua Unix Domain Socket với nhiều mục đích xấu.

Để thực hiện khai thác ta thực hiện tạo container và mout root với `/host` sau đó `chroot` để cấp quyền root trên đó là có thể khởi tạo shell với quyền root và đọc được flag. Trước tiên kiểm tra các `image` bằng lệnh `docker images` nhưng kết quả cho thấy là không được cấp quyền.

![](https://i.imgur.com/HgvhIqo.png)

Vậy tại user `newdoor` chưa có quyền sử dụng lệnh `docker` này một cách thoải mái. Vì vậy ta cần phải thử thực hiện `mount docker socket` ở một user khác. Ngoài user `newdoor` thì còn một user khác đó chính là `insec`. Kiểm tra tại thư mục `/home/insec` thì thấy có một file SUID `download_file` và `insec.flag.txt`

![](https://i.imgur.com/E5XXqOl.png)

Ta có thể tận dụng file SUID này để đọc `insec.flag.txt` và đọc cả `Private Key` trong thư mục `.ssh` để có thể truy cập vào user `insec` và thực hiện `mount docker socket`.
Để làm được điều đó thì ta cần kiểm tra xem file SUID này thực hiện gì.

![](https://i.imgur.com/VodL7xg.png)

Có thể thấy trong lần chạy đầu tiên thì chương trình yêu cầu ta nhập vào một URL, nhập xong thì nó trả về `not definded`. Trong lần chạy tiếp theo ta thử gửi `signal Interrupt (CTRL+c)` thì thấy nó báo `/opt/download.py`. Để chắc chắn hơn thì ta dùng lệnh `strings download_file` để kiểm tra các chuỗi trong đó.

![](https://i.imgur.com/qDoNJ5Q.png)

Thì ta thấy được là nó thực hiện file `/opt/download.py`. Kiểm tra nội dung file `/opt/download.py` để hiểu rõ hơn hoạt động của nó.

![](https://i.imgur.com/NPcORx6.png)

File này nó cố gắng request đến url và lấy về một filename sau đó parse filename đó ra cho đúng chuẩn sau đó mở filename ra và ghi content của request vào. Sau khi đọc và thử một số input thì có vẻ như không thể dùng nội dung có sẵn này để khai thác. 

Vậy ta nghĩ đến 2 hướng là . Đầu tiên là thêm code để đọc file `insec.flag.txt` nhưng cách này không hiệu quả vì ta không có quyền để làm điều đó. Cách thứ hai là điều hướng sang một chương trình để ta thực hiện đọc file `insec.flag.txt`. Rõ ràng là cách 2 rất khả thi hơn bằng cách thực hiện kỹ thuật `Python library hijacking`.
Vì `Python library` sẽ thực hiện tìm kiếm thông qua `Python Path Environment Variable`. Biến này chứa một danh sách các thư mục nơi python sẽ tìm kiếm module đã nhập. Vì vậy mục tiêu của chúng ta sẽ tạo ra một module trùng tên với module mà chương trình `/opt/download.py` sẽ gọi và thay đổi Python path là có thể gọi được module trùng tên vừa tạo kia.

Bắt đầu thực hiện, đầu tiên ta thấy trong chương trình `/opt/download.py` có 2 module là `re` và `requests`, ta chọn tạo 1 trong 2 module này. Ở đây ta sẽ chọn tạo module `requests.py` ở `/tmp` với nội dung :

![](https://i.imgur.com/A3CLsny.png)

Sau khi tạo xong một module trùng tên với module gốc thì nhiệm vụ tiếp theo là chỉnh `PYTHONPATH`. Lúc này đây ta tạo file module trùng tên này ở thư mục `/tmp` vì vậy ta cần export `PYTHONPATH=/tmp/`. Và lúc này thực hiện chạy file SUID để khai thác.

![](https://i.imgur.com/4oeGUMT.png)

Như đã giải thích ở trên thì trong trường hợp này file SUID sẽ thực hiện câu lệnh `/usr/bin/python2  /opt/download.py`. Tiếp theo thì file `download.py` này sẽ load module `re` và `request` thông qua `PYTHONPATH` nên sẽ load trúng cái module `request.py` trùng tên kia. Vậy là ta đã có thể đọc được `insec.flag.txt` và giải quyết được challenge `Newdoor lite subflag 4`

Tương tự như thế thì ta sẽ đọc Private Key với nội dung module requests.py như sau :

![](https://i.imgur.com/6LPwbO6.png)

Và thực hiện chạy file SUID để khai thác.

![](https://i.imgur.com/LQqE2Ad.jpg)

Bây giờ ta đã có `private key` của user `insec`. Thực hiện tạo một file `rsa.key` và bỏ privatekey này vào, cấp quyển read và write sau đó dùng lệnh `ssh -i rsa.key insec@192.198.19.120` để khởi tạo shell với quyền user `insec`.

![](https://i.imgur.com/CV2Ieff.png)

Quay lại giống như trên, ta sẽ thực hiện tìm kiếm socket

![](https://i.imgur.com/lQnQvA8.png)

Sau đó thực hiện tạo container và mout `root` với `/host` sau đó chroot trên đó bằng lệnh `docker run -it -v /:/host/ ubuntu chroot /host/ bash`

![](https://i.imgur.com/qR5pSda.png)

Sau khi chạy container lên thì bây giờ ta đã có quyền root trên container cũng tương ứng đó là quyền root ở ngoài vì ta đã mount lại rồi. Bây giờ có thể thực hiện các thay đổi như tạo mật khẩu cho root sau đó thoát khỏi container và đăng nhập vào root bằng mật khẩu vừa tạo. 

![](https://i.imgur.com/6AcMjFP.png)

Vậy là có thể thấy ta đã thực hiện leo quyền lên root một cách thành công. Bây giờ chỉ cần đọc nội dung trong file root.txt là có thể lấy được flag root.

![](https://i.imgur.com/NbLDyU3.png)

## 4. Kết luận

- Qua box NEWDOOR này thì ta có thể thấy hệ thống tồn tại khá nhiều lỗi với các mức độ ảnh hưởng khá nghiêm trọng như `weak password, weak configured, PATH variable, Mountable NFS share, Mount Docker Socket`.
- Từ đó ta có thể đưa ra một số khuyến nghị như sau :
    - Thực hiện cấu hình một cách chính xác, hạn chế đối với tất cả các NFS share hoặc chặn luôn các truy cập bên ngoài. Đồng thời dùng các password không phổ biến, có đủ độ phức tạp và thay đổi nó một cách định kỳ
    - Nên cẩn thận trước khi sử dụng các docker image yêu cầu truy cập vào Docker socket ngay cả với quyền read-only vì nó có thể khiến môi trường của bạn gặp phải một số rủi ro khác. Điểm mấu chốt là không gắn docker.socket vào một container nào trừ khi bạn tin vào nguồn gốc vào tính bảo mật của nó.

- Vậy là ta đã hoàn thành được box NEWDOOR này.