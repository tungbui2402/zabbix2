# Zabbix 2
## 1. Tải xuống và cài đặt kho lưu trữ Zabbix
Trước khi cài đặt phần mềm trên máy chủ Ubuntu mới, hãy cập nhật gói và thông tin phụ thuộc:
```
sudo apt update
```
Bây giờ chọn cài đặt Zabbix Server từ các gói

Để kiểm tra, gõ hostnamectl:
```
hostnamectl
```
![image](https://github.com/tungbui2402/zabbix2/assets/129025623/aaa7627c-4e1a-4564-9211-eab905ea874a)

Command:
```
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-1+ubuntu20.04_all.deb
dpkg -i zabbix-release_6.0-1+ubuntu20.04_all.deb
sudo apt update
```
![image](https://github.com/tungbui2402/zabbix2/assets/129025623/35554b33-a4f3-4c90-b97f-0a4aad7c8c52)

![image](https://github.com/tungbui2402/zabbix2/assets/129025623/6f38a8b8-26b3-4751-81e3-ee09a82e806e)


Sau khi cập nhật, xác nhận tham chiếu APT (Advanced Packaging Tool) của mình các phiên bản chính xác:
```
sudo apt search zabbix-server
```
![image](https://github.com/tungbui2402/zabbix2/assets/129025623/a06ee3f3-6ca3-40ac-aed8-f20133149563)


## 2. Cài đặt Zabbix Server, Frontend và Agent
Cài đặt tất cả các thành phần cho MySQL, giao diện người dùng front end và tác nhân:
```
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```
Kiểm tra trạng thái của máy chủ và tác nhân:
```
sudo service zabbix-server status
sudo service zabbix-agent status
```
![image](https://github.com/tungbui2402/zabbix2/assets/129025623/0fc9f42d-34df-45c9-a1b4-10e5a1d161fd)

## 3. Tạo cơ sở dữ liệu ban đầu
Để kiểm tra xem có máy chủ cơ sở dữ liệu MySQL đang chạy hay không, nhập:
```
sudo service mysql status
```
![image](https://github.com/tungbui2402/zabbix2/assets/129025623/22af4d10-3976-40bc-9a10-db209b1bb245)

Nếu gặp lỗi `Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'`

Sau đó, nó thường có nghĩa là MySQL không được cài đặt.

Vì vậy, trong trường hợp có một máy chủ hoàn toàn mới không được cài đặt MySQL theo mặc định, cần cài đặt MySQL:
```
sudo apt install mysql-server
```
Kiểm tra xem nó có đang chạy không.
```
sudo service mysql status
```
Bây giờ nó sẽ chỉ ra rằng MySQL đang hoạt động.

### Tạo người dùng và cơ sở dữ liệu Zabbix
Bây giờ chúng ta có thể đăng nhập vào máy chủ MySQL và thiết lập cơ sở dữ liệu Zabbix và người dùng của nó.
```
mysql -uroot -p

create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'password';
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
quit
```
![image](https://github.com/tungbui2402/zabbix2/assets/129025623/31320e86-f421-4998-9486-383f22b2d15d)

### Nhập lược đồ Zabbix
Bây giờ, nhập lược đồ Zabbix:
```
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```
Đợi khoảng 30 giây để nó kết thúc.

![image](https://github.com/tungbui2402/zabbix2/assets/129025623/d87e2e2b-ac1d-4881-9b78-88b30ac1faf0)

### Chỉnh sửa tệp cấu hình Zabbix Server
Bây giờ trong tệp cấu hình Zabbix Server, hãy tìm và đặt thuộc tính làm mật khẩu mà đã đặt cho người dùng. `DBPassword` `zabbix@localhost`:
```
sudo nano /etc/zabbix/zabbix_server.conf
DBPassword=password
```

Thực hiện một số kiểm tra MySQL nếu muốn:
```
mysql
show databases;
use zabbix;
show tables;
select * from users;
quit
```
### Bắt đầu tiến trình máy chủ Zabbix
Bây giờ MySQL đang chạy, chúng ta có thể bắt đầu tiến trình Zabbix Server.
```
sudo service zabbix-server start
```
Và kiểm tra trạng thái của nó có hoạt động không
```
sudo service zabbix-server status
```
Để đảm bảo rằng Zabbix Server, Agent và Apache khởi động lại trong trường hợp khởi động lại, hãy chạy lệnh này.
```
sudo systemctl enable zabbix-server zabbix-agent apache2
```
![image](https://github.com/tungbui2402/zabbix2/assets/129025623/59fa2279-3db3-495b-a709-4f1eab14001e)

## 4. Đăng nhập và cấu hình Zabbix Server Front End
Chúng ta cũng nên khởi động lại dịch vụ Apache2 vì đã có một số thay đổi đối với cấu hình trong quá trình thiết lập Zabbix cho đến nay.
```
sudo service apache2 restart
```
Bây giờ chúng ta có thể truy cập Máy chủ Zabbix tại http://your-server-ip-address/zabbix
![image](https://github.com/tungbui2402/zabbix2/assets/129025623/8079ab0c-2e64-4a69-bb93-3158acf5a76d)

Nhấn Bước Tiếp theo một vài lần và sau đó nhập mật khẩu cơ sở dữ liệu của bạn khi được yêu cầu.

Và sau đó, cuối cùng, đăng nhập vào cài đặt Zabbix Server mới hoàn thành bằng thông tin đăng nhập mặc định của

Tên đăng nhập : Admin

Mật khẩu : zabbix

![image](https://github.com/tungbui2402/zabbix2/assets/129025623/a29aba63-353e-487f-98c2-cade06e7ded7)

## 5. Cấu hình tên miền cho Zabbix Server

## 6. Định cấu hình SSL cho Zabbix Server Front end
Máy chủ Zabbix chưa bật bất kỳ mã hóa truyền tải nào, vì vậy mọi thông báo được truyền giữa trình duyệt và máy chủ đều ở dạng văn bản thuần túy. Chúng ta nên bảo mật máy chủ của mình càng sớm càng tốt bằng chứng chỉ SSL.

Tạo chứng chỉ bằng các tùy chọn do LetsEncrypt cung cấp. Điều này có thêm lợi ích là miễn phí.

Vì vậy, chúng ta cần ssh vào Zabbix Server và cài đặt Certbot
```
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --apache
```

## 7. Overview So Far
Kiểm tra trạng thái của các dịch vụ.
```
sudo service zabbix-server status
sudo service zabbix-agent status
sudo service apache2 status
sudo service mysql status
```
Tệp cấu hình Zabbix Server
```
sudo nano /etc/zabbix/zabbix_server.conf
```
Tệp cấu hình Zabbix Agent
```
sudo nano /etc/zabbix/zabbix_agentd.conf
```
