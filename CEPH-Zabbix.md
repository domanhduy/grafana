# CEPH monitor Zabbix #

## CEPH server ##

		yum install zabbix-sender


		[root@nhcephssd1 ~]# which zabbix_sender
		/usr/bin/zabbix_sender

+ Enable module ceph-zabbix

		ceph mgr module enable zabbix

+ Set zabbix server

		ceph zabbix config-set zabbix_host 172.16.4.54

+ Set ceph-server

		ceph zabbix config-set identifier NH_Ceph_SSD1

+ Set zabbix_sender
+ 
			which zabbix_sender

			ceph zabbix config-set zabbix_sender /usr/bin/zabbix_sender
+ Set port

		ceph zabbix config-set zabbix_port 10051
+ Set interval time

		ceph zabbix config-set interval 60

+ Show conffig
		
		ceph zabbix config-show

+ Check template

		rpm -ql ceph-mgr | grep xml

![](https://i.imgur.com/6g9WeJ9.png)

## Zabbix web interface ##

+ Import zabbix_temaplte.xml

Config trên zabbix web interface

Configuration -> Templates -> Import

![](https://i.imgur.com/rKhfyVM.png)
Add host link template

![](https://i.imgur.com/yPPTulG.png)

## Config database zabbix ##

Truy cập vào mysql

	mysql -u root -p

Sử dụng database zabbix

	 use zabbix;

	select hostid from hosts where name='ceph-mgr Zabbix module';

![](https://i.imgur.com/9WBWr7j.png)

	select itemid, name, key_, type, trapper_hosts  from items where hostid=10284;

![](https://i.imgur.com/YFbZ5HC.png)

Define allow host

	update items set trapper_hosts='172.16.4.54' where hostid=10284;

![](https://i.imgur.com/7cgjnOq.png)

	select itemid, name, key_, type, trapper_hosts  from items where hostid=10284;
	
## Ceph cron job on CEPH server ##

	vi /etc/cron.d/ceph
	*/1 * * * * root ceph zabbix send

Restart service on zabbix

	systemctl restart mariadb
	systemctl restart zabbix-server
	systemctl restart zabbix-agent

## Test zabbix-sender ##
	
	ceph zabbix send

![](https://i.imgur.com/WOKLptZ.png)

## Link tham khảo ##

https://blog.csdn.net/signmem/article/details/78667569

http://kb.nhanhoa.com/display/NHKB01/Monitor+CEPH+with+ZABBIX

## Zabbix-sender, traps host ##

Zabbix sender là tính năng dòng lệnh có thể được sử dụng để gửi performance data tới máy chủ Zabbix để xử lý.

Tính năng này thường được sử dụng trong các user scripts chạy dài để gửi định kỳ kiểm tra tính availability và performance data.

Để gửi các kết quả trực tiếp đến máy chủ hoặc proxy Zabbix, một **trapper item**  phải được cấu hình.


Trapper items chấp nhận dữ liệu tới nó thay vì phải đi query. Nó hữu dụng cho kiểu dữ liệu nào "push" vào zabbix.\

Sử dụng trapper item khi:

+Có một trapper item được setup trong zabbix

+Gửi data vào zabbix


Allow host: Nếu được chỉ định, trapper (trình thu thập) sẽ chỉ chấp nhận dữ liệu đến từ danh sách Allow host.


Lưu ý: Cụm ceph có nhiều con thì phỉa set các tham số vào đúng 1 con ceph -> Check tham số hay lỗi ở file cd /var/log/ceph.audit.log

![](http://prntscr.com/la6bvy)


