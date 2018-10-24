<h1>Cài đặt Grafana - Graphite - Collectd on Ubuntu 16.04</h1>

## 1. Giới thiệu ##

Grafana là một bộ mã nguồn mở sử dụng trong việc phân tích các dữ liệu thu thập được từ server và hiện thị một các trực quan dữ liệu thu thập được ở nhiều dạng khác nhau.
Các tính năng cơ bản: 

+ Đưa ra cảnh báo Alert: Đưa ra được cảnh báo ở một ngưỡng đã được xác định, có thể đưa ra cảnh báo thông qua Slack, PagerDuty…liên tục đánh giá và gửi thông báo.


+ Tổng hợp dữ liệu: Tổng với nhau để có được phân tích. Grafana hỗ trợ hàng chục cơ sở dữ liệu và có thể mix lại với nhau trong cùng một dashboard.


+ Đa nền tảng: Grafana cung cấp nhiều sự lựa chọn, vì hoàn toàn là mã nguồn mở nên được hỗ trợ bởi một cộng đồng đông đảo người sử dụng có kinh nghiệm. Dễ dàng cài đặt trên bất kỳ nền tảng nào (Linux, windows, MacOS).


+ Mở rộng: Có thể tạo ra trăm bảng điều khiển và plugin trong thư viện chính thức nhờ sự phát triển của cộng đồng sử dụng grafana.


+ Linh hoạt: Tạo bảng điều khiển động có độ tùy biến cao và có thể sử dụng dễ dàng, linh hoạt, kéo thả.

![](https://i.imgur.com/Af8JCrM.png)

**Nguyên tắc hoạt động chung của collectd-graphite-grafana:**

Nhìn vào sơ đồ trên dễ dàng hình dùng được quá trình hoạt động của collectd-graphite-grafana

-Data source Graphite đứng để nhận các thông tin về metrics ở các node server cần monitor đẩy lên thông qua carbon-cache.


-Collectd cài đặt ở các server cần monitor gửi các metrics mà người quản trị yêu cầu thu thập về service carbon-cache.


-	Carbon Metrics 	
	+ Updates/sec (write ops) 
	+ Metrics received/sec (ingress) 
	+ Committed points/sec (datapoints written to disk) 
	+ Creates/sec (new Whisper files) 
	+ CPU (avg across Cache instances) 
	+ Points per Update (avg across Cache instances) 
-	Carbon Memory & CPU
-	Metric keys in Cache (unique metric names) 
-	Datapoints in Cache 
-	Relay Destination Queue Length (datapoints queued per destination) 
-	Average datapoints per key in Cache 
-	Datapoints received during full cache 
-	Average Relay Batch Size (average number of datapoints per relay transmission) 
-	Disk write ops and write time (requires collectd disk plugin) 
-	Metric retrieval time (requires collectd tail plugin and this configuration) 
-	Network traffic (requires collectd interface plugin) 
-	Load (requires collectd load plugin) 
-	CPU (by state, requires collectd cpu plugin)

-Grafana lấy thông tin từ data source từ graphite về để hiển thị dạng trực quan, monitor trực quan. Tùy từng dữ liệu, yêu cầu monitor mà sử dụng một loại biểu đồ hiện thị khác nhau, panel khác nhau sao cho phù hợp nhất.

 hiiiiiii
hihi
 
## 2, Chuẩn bị ##

## 2.1. System Requirements ##

+ Ubuntu 16.04 LTS

+  CPU: 1 vCPU

+ Memory: 1GB RAM

+ Disk: 2.5-5GB tùy thuộc vào số node monitor

+ Disk performance: 20-200 random IOPs trên mỗi node server monitor.

+ Use NetApp storage (NFS, LUN, vmdk, vhd, vhdx) hoặc local SSD

## 2.2. Protocol and Port Requirements ##

Graphite Server IP Graphite

+ Sử dụng plaintext protocol 2003/TCP* : Sử dụng để đẩy metric từ 
Graphite metrics DB.
+ Sử dụng giao thức HTTP* 81/TCP*: Sử dụng để hiển thị, truy vấn APIs
lấy các số liệu định dạng JSON.
+ HTTP* 80/443/TCP*: Sử dụng để truy cập giao diện web của Graphite.

# 3. Cài đặt #

**+ Set địa chỉ IP tĩnh, kiểm tra kết nối ra Internet, update, upgrade các package**


![](https://i.imgur.com/WMqI9M0.png)

**3.1. Install graphite**

**- Cài đặt graphite và các package cần thiết cho graphite**

	sudo apt-get install apache2 libapache2-mod-wsgi graphite-web graphite-carbon

+ Trong quá trình cài đặt "Y" để đồng ý cài đặt các package

+ Cài đặt database cho graphite

Yes để xóa db cũ đi và tạo db mới

No để cài đè lên db cũ (TH graphite đã được cài đặt trước đây trên server).
![](https://i.imgur.com/x4bteSS.png)


Chọn "Yes" để remove và cài mới db


Quá trình cài đặt graphite đang xong.


![](https://i.imgur.com/NLYniOC.png)



**3.2. Configure Graphite**

- Đặt chế độ tự đông bật service graphite-carbon mỗi lần server reboot lại.

+Tìm và chỉnh sửa trong file **sudo nano /etc/default/graphite-carbon**

	CARBON_CACHE_ENABLED=false
	change to
	CARBON_CACHE_ENABLED=true


![](https://i.imgur.com/66gNyFS.png)

- Carbon cache service configuration file bật chế độ ghi log và quản lý ghi log LOGROTATE

+Chỉnh sửa trong file **sudo nano /etc/carbon/carbon.conf**

	ENABLE_LOGROTATION = False
	change to
	ENABLE_LOGROTATION = True

![](https://i.imgur.com/gHGMgYQ.png)

+Tăng số lượng metric tạo ra mỗi phút để hiển thị.

	MAX_CREATES_PER_MINUTE = 50
	change to
	MAX_CREATES_PER_MINUTE = 600

![](https://i.imgur.com/W4yOdgS.png)

- Copy a default storage-aggregation.conf file

		sudo cp /usr/share/doc/graphite-carbon/examples/storage-aggregation.conf.example /etc/carbon/storage-aggregation.conf

![](https://i.imgur.com/6FdgWqr.png)

- Restart service graphite

![](https://i.imgur.com/V8MRYnz.png)

**3.3. Configure graphite-web**

- Cấu hình Graphite web app 

Chỉnh sửa trong file /etc/graphite/local_settings.py

+Đặt key trong quá trình sử dụng web app TCP kết nối.

		#SECRET_KEY = 'UNSAFE_DEFAULT'
		change to 
		SECRET_KEY = ‘KJKzzxcviouzxcvytwkdd94944asmdf9ads9ds9dspp83jlkjLKJL98798KLOIK’

+Chỉnh time zone

		#TIME_ZONE = 'America/Los_Angeles'
		change to 
		TIME_ZONE = 'Asia/Ho_Chi_Minh'

![](https://i.imgur.com/JpzDDNP.png)


- Cài đặt, cấu hình graphite DB và set quyền truy cập cho user

		sudo graphite-manage syncdb
		sudo chown _graphite:_graphite /var/lib/graphite/graphite.db


![](https://i.imgur.com/6TrpLFG.png)



**3.4. Configure Apache**

	sudo a2dissite 000-default
	sudo cp /usr/share/graphite-web/apache2-graphite.conf /etc/apache2/sites-available
	sudo a2ensite apache2-graphite

![](https://i.imgur.com/t82UR0Z.png)


- Edit port listen của Apache từ 80 -8080 để tránh trùng port mới app khác, có thể giữ nguyên nếu có lỗi phải đổi port.

Edit trong file **sudo nano /etc/apache2/ports.conf**

![](https://i.imgur.com/RzMhT8n.png)


Edit port trong của file **/etc/apache2/sites-available/apache2-graphite.conf**

![](https://i.imgur.com/FOh4qen.png)


- Reload service apache

		sudo service apache2 reload

- Cài đặt thành công graphite truy cập qua giao diện web 

![](https://i.imgur.com/xdm4HQP.png)


**3.5. Cài đặt grafana**

+ Chỉnh sửa source get của Ubuntu

Mở file sudo nano **/etc/apt/sources.list**

Thêm source **deb https://packagecloud.io/grafana/stable/debian/ wheezy main**

![](https://i.imgur.com/6uBgE7U.png)

+ Thêm package cloud key để cho phép cài đặt

		sudo curl https://packagecloud.io/gpg.key | sudo apt-key add -

![](https://i.imgur.com/lzRuEfd.png)


+ Update package và cài đặt grafana

		sudo apt-get update
		sudo apt-get install grafana
![](https://i.imgur.com/ZUpQred.png)

**-Cấu hình grafana**

+ Cầu hình grafana truy cập bằng https với chứng thư số từ một CA tin cậy, tự tạo một chứng thư số.

		cd /etc/grafana
		sudo openssl req -x509 -newkey rsa:2048 -keyout cert.key -out cert.pem -days 3650 -nodes

+ Chỉnh sửa file config grafana

Sửa trong file **/etc/grafana/grafana.ini**

		protocol = http -> protocol = https
		http_port = 3000 -> http_port = 443
		#https certs & key file
		;cert_file =
		;cert_key =
		sửa thành
		#https certs & key file
		cert_file = /etc/grafana/cert.pem
		cert_key = /etc/grafana/cert.key

+ Set grafana-server bật khi khởi động server

		sudo setcap 'cap_net_bind_service=+ep' /usr/sbin/grafana-server
		sudo update-rc.d grafana-server defaults 95 10
		sudo service grafana-server start

**-Cài đặt thành công truy cập grafana qua giao diện web**

Truy cập qua đường dẫn trên trình duyệt, đăng nhập với tài khoản mặc định **admin/admin**

**http://ipserver**

![](https://i.imgur.com/hdxOVPW.png)


- Add datasource, tạo dashboard... thông quan giao diện.

![](https://i.imgur.com/kraMtlQ.png)



**3.6. Cài đặt Collectd**


**- Cài đặt trên OS Centos**

	yum -y install libcurl libcurl-devel rrdtool rrdtool-devel rrdtool-prel libgcrypt-devel gcc make gcc-c++

+ Get Collectd, untar it, make it and install

		wget http://collectd.org/files/collectd-5.4.0.tar.gz
		tar zxvf collectd-5.4.0.tar.gz
		cd collectd-5.4.0
		./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var --libdir=/usr/lib --mandir=/usr/share/man --enable-all-plugins
		make
		make install

+ Copy the default init.d script

		cp /root/collectd-5.4.0/contrib/redhat/init.d-collectd /etc/init.d/collectd

+ Set the correct permissions

		chmod +x /etc/init.d/collectd

+ Start the deamon

		service collectd start


**- Cài đặt trên Ubuntu**
+ Cài đặt
		sudo apt-get update
		sudo apt-get install collectd collectd-utils

+ Chỉnh sửa file cấu hình

sudo nano /etc/collectd/collectd.conf

Chi tiết các plugin tham khảo tại

 https://collectd.org/documentation/manpages/collectd.conf.5.shtml

