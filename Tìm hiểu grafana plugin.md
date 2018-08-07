<h1>Grafana - Graphite - Collectd</h1>

<h2> 1. Sơ đồ chung </h2>
    
![](https://i.imgur.com/iR9to5t.png)

<h2> 2. File config collectd.conf </h2>

**collectd.conf** chứa toàn bộ những câu lệnh cấu hình để lấy thông số của chính server đó để lên server graphite.

Phải config đúng thì mới lấy được metric hiển thị lên graphite và khi khởi động lại service collectd không bị lỗi.

**2.1. Setting cho toàn bộ cơ chế chung của collectd - Setting tĩnh**

***a, Global settings for the daemon***

**Hostname    "localhost"**

+  Hiển thị tên thư mục chứa tất cả các thông tin lấy ở server và hiển thị trong graphite.

**FQDNLookup   true**

+ Giá trị true để phân giải tên máy chủ

**BaseDir     "${prefix}/var/lib/collectd"** 

+ Thiết lập thư mục cơ sở. Đây là thư mục bên dưới mà tất cả các tệp RRD được tạo ra. Có thể có nhiều thư mục con hơn được tạo ra. Đây cũng là thư mục làm việc cho daemon.
 
**PIDFile     "${prefix}/var/run/collectd.pid"** 

+ Đặt thư mục để ghi các tập tin PID để. Tập tin này sẽ bị ghi đè khi nó tồn tại và bị xóa khi chương trình dừng lại.

**PluginDir   "${exec_prefix}/lib/collectd"** 

+ Đường dẫn đến các plugin (shared objects) của collectd

**TypesDB     "/opt/collectd/share/collectd/types.db"** 

+ Đặt một hoặc nhiều tệp có chứa mô tả bộ dữ liệu

types.db - Thông số thiết lập dữ liệu cho hệ thống thống kê daemon thu thập collectd


**AutoLoadPlugin false|true**

+ Khi được đặt thành false (mặc định), mỗi plugin cần phải được config load một cách rõ ràng, sử dụng câu lệnh LoadPlugin được ghi ở trên. Nếu gặp phải một khối <Plugin ...> và không có một callback xử lý cấu hình cho plugin này đã được khai báo, một cảnh báo sẽ được ghi lại và block sẽ bị bỏ qua.
+ Khi thiết lập true, câu lệnh LoadPlugin rõ ràng không bắt buộc. Mỗi khối <Plugin ...> hoạt động như thể nó đã được ngay trước bởi một câu lệnh LoadPlugin. Các khối LoadPlugin vẫn được yêu cầu cho các plugin nhưng không cung cấp bất kỳ cấu hình nào.

**CollectInternalStats false|true**

+ Khi được đặt thành true, các thu thập khác nhau về daemon thu thập sẽ được thu thập, với "collectd" là name plugin. Mặc định là false.

**Interval     10** 

+ Định cấu hình khoảng thời gian để truy vấn lại các plugin. Các giá trị nhỏ hơn dẫn đến một hệ thống laod cao hơn để cung cấp thông tin cho collectd, trong khi các giá trị cao hơn dẫn đến thống kê không được chi tiết.

**MaxReadInterval 86400**

+ Một plugin đọc tăng gấp đôi khoảng giữa các truy vấn sau mỗi lần không thành công để có được dữ liệu.

Tùy chọn này giới hạn giá trị lớn nhất của khoảng thời gian. Giá trị mặc định là 86400.

**Timeout         2**

+ Xem xét một danh sách giá trị "miss" khi không có bản tin cập nhật nào đã được đọc hoặc nhận cho các lần lặp lặp lại. 
+ Theo mặc định, collectd xem xét một danh sách giá trị miss khi không nhận được cập nhật cho hai lần khoảng cập nhật. Vì thiết lập này sử dụng lặp, thời gian cho phép tối đa mà không cần cập nhật phụ thuộc vào thông tin lặp vòng chứa trong mỗi danh sách giá trị. 


**ReadThreads     5**

+ Số luồng để đọc các plugin. Giá trị mặc định là 5, có thể muốn tăng giá trị này nếu có nhiều hơn 5 plugin cần nhiều thời gian để đọc. Chủ yếu là những plugin mà sử dụng network I/O. Việc đặt giá trị này cao hơn số lần gọi lại đã đọc đã cài đặt không được khuyến khích.

**WriteThreads    5**

+ Số luồng để bắt đầu gửi danh sách giá trị để write các plugin. Giá trị mặc định là 5, nhưng có thể muốn tăng giá trị này nếu có nhiều hơn 5 plugin mà có thể mất nhiều thời gian để write.

**WriteQueueLimitHigh 1000000**

**WriteQueueLimitLow   800000**

+ Số các metrics được đọc bởi các luồng đọc và sau đó đưa vào một hàng đợi để được xử lý bởi . Nếu một trong các plugin write chậm (network timeouts, I/O saturation of the disk) hàng đợi này sẽ tăng lên. Để tránh chạy vào các vấn đề bộ nhớ trong trường hợp như vậy, có thể giới hạn kích thước của hàng đợi này.

+ Theo mặc định, không có giới hạn và bộ nhớ có thể phát triển vô hạn, nên đặt giá trị này vào một giá trị khác không.

+ Có thể thiết lập các giới hạn bằng cách sử dụng WriteQueueLimitHigh và WriteQueueLimitLow. 

+ Nếu WriteQueueLimitHigh được đặt bằng không và WriteQueueLimitLow không được đặt, giá trị mặc định sẽ bằng một nửa WriteQueueLimitHigh.

+ Nếu không muốn ngẫu nhiên các giá trị khi kích thước hàng đợi giữa LowNum và HighNum, hãy đặt WriteQueueLimitHigh và WriteQueueLimitLow với cùng một giá trị.

**b, Logging**

Thiết lập các cấu hình về log của quá trình thao tác với collectd trên server.

 - **LoadPlugin syslog**
 	+  LogLevel debug|info|notice|warning|err
 
 	Đặt log-level. Để thiết lập để thông báo, sau đó tất cả các sự kiện với thông báo mức độ nghiêm trọng, cảnh báo, hoặc err sẽ được gửi đến syslog daemon.

		<Plugin syslog>
			LogLevel debug|info|notice|warning|err
		</Plugin>

- **LoadPlugin logfile**

		<Plugin logfile>
			LogLevel info
			File STDOUT
			Timestamp true
			PrintSeverity false
		</Plugin>


	+ LogLevel debug|info|notice|warning|err


	Đặt log-level. Để thiết lập để thông báo, sau đó tất cả các sự kiện với thông báo mức độ nghiêm trọng, cảnh báo, hoặc err sẽ được ghi vào logfile

	+ Thiết lập các tập tin để ghi vào log-message. Các **stdout** đặc biệt và **stderr** có thể được sử dụng để ghi vào đầu ra tiêu chuẩn. Điều này chỉ có giá trị khi collectd đang chạy ở chế độ foreground hoặc non-daemon.

  + Timestamp true|false 

	Đặt trước tất cả dòng log lấy theo thời gian hệ thống, mặc định hiển thị là true

	+ PrintSeverity true|false

	Khi được kích hoạt, tất cả các dòng đều được đặt trước bởi mức độ nghiêm trọng của log message
+ **LoadPlugin log_logstash**

	Hoạt động giống như plugin logfile nhưng định dạng các thông điệp như các sự kiện JSON cho logstash để phân tích cú pháp và nhập vào.

		<Plugin log_logstash>
			LogLevel info
			File "${prefix}/var/log/collectd.json.log"
		</Plugin>
	+ LogLevel info

	Đặt log-level. Để thiết lập để thông báo, sau đó tất cả các sự kiện với thông báo mức độ nghiêm trọng, cảnh báo, hoặc err sẽ được ghi vào logfile

	+ File 

	Xét đường dẫn để write log messages

**c, Một số Config plugin thường sử dụng**

**- Plugin df**

Lấy các thông tin từ lệnh df thực hiện trên server

	<Plugin df>
		Device "/dev/hda1"
		Device "192.168.0.2:/mnt/nfs"
		MountPoint "/home"
		FSType "ext3"
		IgnoreSelected false
		ReportByDevice false
		ReportReserved false
		ReportInodes false
		ValuesAbsolute true|false
		ValuesPercentage false
	</Plugin>
+ Device : Lựa chọn phân vùng disk
+ MountPoint: Cùng disk được mount
+ FSType: Định dạng file system
+ IgnoreSelected true|false: Nếu được đặt thành true, tất cả các phân vùng ngoại trừ những đối tượng phù hợp với bất kỳ các yêu cầu nào được thu thập. Theo mặc định, chỉ các phân vùng được chọn mới được thu thập. Nếu không có lựa chọn tất cả các phân vùng được chọn.
+ ReportByDevice: Report bằng cách sử dụng tên thiết bị.
+ ReportInodes true|false: Bật hoặc tắt những báo cáo về dùng lượng trống, sử dụng của disk ở node.
+ ValuesAbsolute true|false: Bật hoặc tắt những báo cáo về dùng lượng trống, sử dụng của disk ở 1K blocks.
+ ValuesPercentage false|true: Bật hoặc tắt những báo cáo về dùng lượng trống, sử dụng của disk trên đơn vị %.

**- LoadPlugin disk**

Plugin thu thập thông tin về cách sử dụng đĩa vật lý và đĩa logic (phân vùng). Các giá trị được thu thập là số octet được ghi và đọc từ đĩa hoặc phân vùng, số lần hoạt động đọc / ghi được phát hành vào đĩa và một "thời gian"  mà các lệnh này phải thực hiện.


	<Plugin disk>
		Disk "/^[hs]d[a-f][0-9]?$/"
		IgnoreSelected false
		UseBSDName false
		UdevNameAttr "DEVNAME"
	</Plugin>

+ Disk: Tên của disk cần theo dõi. Cho dù nó được thu thập hay bỏ qua tùy thuộc vào thiết lập IgnoreSelected. Như với các plugin khác sử dụng chức năng ignorelist của daemon, một chuỗi bắt đầu và kết thúc bằng một dấu gạch chéo được hiểu như một biểu thức chính quy.
+ IgnoreSelected true|false: Các hành động có thể thực hiện: Nếu không có tùy chọn đĩa được cấu hình, tất cả các đĩa được thu thập. Nếu có ít nhất một tùy chọn Disk và không cấu hình IgnoreSelected hoặc thiết lập là false, chỉ những đĩa phù hợp sẽ được thu thập. Nếu IgnoreSelected được đặt thành true, tất cả các đĩa được thu thập ngoại trừ những cái khớp.
+ UseBSDName true|false

+ UdevNameAttr Attribute: Cố gắng ghi đè lên tên của đĩa với giá trị của một thuộc tính udev được chỉ định khi được xây dựng với libudev. Nếu thuộc tính không được định nghĩa cho thiết bị đã cho, tên mặc định sẽ được sử dụng.

**- LoadPlugin memory**

		<Plugin memory>
			ValuesAbsolute true
			ValuesPercentage false
		</Plugin>

Plugin memory cung cấp các tùy chọn cấu hình sau:
	
+ ValuesAbsolute true|false: Bật hoặc tắt report sử dụng bộ nhớ vật lý theo số tuyệt đối, Đơn vị là byte. Mặc định là true.
+ ValuesPercentage false|true: Bật hoặc tắt report sử dụng bộ nhớ vật lý theo phần trăm, ví dụ: phần trăm bộ nhớ vật lý được sử dụng. Mặc định là false.

Điều này rất hữu ích cho việc triển khai collectd trong một môi trường không đồng nhất, trong đó các kích thước của bộ nhớ vật lý khác nhau.


**- Plugin dns**

		<Plugin dns>
			Interface "eth0"
			IgnoreSource "192.168.0.1"
			SelectNumericQueryTypes true
		</Plugin>


 + Interface: Các plugin dns sử dụng libpcap để nắm bắt dns traffic và phân tích nó. Tùy chọn này thiết lập interface cần được sử dụng. Nếu tùy chọn này không được đặt, hoặc thiết lập để "any", plugin sẽ cố gắng để có được các gói tin từ tất cả các giao diện.
 + IgnoreSource IP-address: Bỏ qua các gói tin bắt nguồn địa chỉ IP sau
 + SelectNumericQueryTypes: Được bật theo mặc định, thu thập các loại truy vấn không xác định (được hiển thị dưới dạng số).


**- Plugin interface**

		<Plugin interface>
			Interface "eth0"
			IgnoreSelected false
		</Plugin>

+ Interface: Lựa chọn interface. Theo mặc định các interface này sau đó sẽ được thu thập. Để thu thập chi tiết hơn phải cấu hình IgnoreSelected.
+ IgnoreSelected

Nếu không có cấu hình được đưa ra, interface-plugin sẽ thu thập dữ liệu từ tất cả các interface. Thục tế có nhiều interface đặc biệt như đối với các giao diện loopback và tương tự. 


Có thể sử dụng Interface-option để chọn các interface. 
Tuy nhiên, đôi khi, dễ dàng hơn / muốn thu thập tất cả các giao diện ngoại trừ một vài giao diện. Tùy chọn này cho phép bạn thực hiện điều đó: Bằng cách đặt IgnoreSelected to **true** ảnh hưởng của interface được đảo ngược: Tất cả các interface được chọn đều bị bỏ qua và tất cả các interface khác đều được thu thập.

**- Plugin memcached**
 
Plugin memcached kết nối với một máy chủ memcached, truy vấn một hoặc nhiều trang nhất định và phân tích cú pháp dữ liệu trả về theo đặc tả người dùng.


		<Plugin memcached>
			<Instance "local">
				Host "127.0.0.1"
				Port "11211"
			</Instance>
		</Plugin>

**-Plugin mysql**

Các plugin mysql yêu cầu mysqlclient sẽ được cài đặt. Nó kết nối với một hoặc nhiều cơ sở dữ liệu khi bắt đầu và giữ kết nối càng lâu càng tốt. Khi kết nối bị gián đoạn vì bất cứ lý do gì nó sẽ cố kết nối lại. Các plugin sẽ thông báo trong TH không phát hiện sự mất kết nối.


		<Plugin mysql>
			<Database db_name>
				Host "database.serv.er"
				User "db_user"
				Password "secret"
				Database "db_name"
				MasterStats true
				ConnectTimeout 10
				InnodbStats true
			</Database>
		
			<Database db_name2>
				Alias "squeeze"
				Host "localhost"
				Socket "/var/run/mysql/mysqld.sock"
				SlaveStats true
				SlaveNotifications true
			</Database>
		</Plugin>


Một khối cơ sở dữ liệu định nghĩa một kết nối đến một cơ sở dữ liệu MySQL. Nó chấp nhận một đối số duy nhất xác định tên của cơ sở dữ liệu. MySQL sẽ sử dụng các giá trị mặc định như được ghi trong các phần "mysql_real_connect ()" và "mysql_ssl_set ()" để kết nối.

+ Host Hostname: Name của database server. Default là localhost
+ User Username: Username dùng để kết nối tới database server
+ Password: Password của user dùng để kết nối tới database server
+ Database: Tên của database muốn kết nối tới
+ MasterStats true: Xác định đó là master hay slave
+ ConnectTimeout: Set khoảng thời gian kết nối lại tới mysql server
+ InnodbStats true|false: Nếu được bật, số liệu về công cụ lưu trữ InnoDB được thu thập. Tắt theo mặc định.
+ Alias Alias: Bí danh để sử dụng như người gửi thay vì tên máy chủ khi report.
+ Socket Socket: Chỉ định đường dẫn socket tới máy chủ MySQL. Tùy chọn này chỉ có tác dụng, nếu Host được thiết lập là localhost (mặc định). Nếu không, sử dụng tùy chọn Port ở trên.

**- Plugin ping**

Plugin Ping gửi các gói tin "ping" ICMP tới các host được định cấu hình định kỳ và đo độ trễ của mạng.

		<Plugin ping>
			Host "host.foo.bar"
			Interval 1.0
			Timeout 0.9
			TTL 255
			SourceAddress "1.2.3.4"
			Device "eth0"
			MaxMissed -1
		</Plugin>



+ Host: Đối tượng để ping đến. Tùy chọn này có thể được lặp lại nhiều lần để ping nhiều máy chủ.
+ Interval Seconds: Xét khoảng thời gian giữa các lần gửi gói tin ICMP. Default là 1s
+ Timeout Seconds: Thời gian chờ phản hồi từ máy chủ mà gói tin ICMP đã được gửi đi. Nếu một hồi đáp không nhận được sau giây x Seconds, máy chủ được giả định là down hoặc gói tin sẽ được bỏ. Cài đặt này phải nhỏ hơn cài đặt Interval Seconds để plugin hoạt động chính xác.
+ SourceAddress host: Thiết lập địa chỉ nguồn để sử dụng. host có thể là một địa chỉ mạng số hoặc một tên máy chủ mạng.
+ Device name: Thiết lập thiết bị mạng gửi đi được sử dụng. tên phải chỉ định tên interface (eth0...). Tùy vào interface của hệ điều hành khác nhau mà có hay không có hỗ trợ.
+ MaxMissed Packets: Kích hoạt trigger DNS sau khi máy chủ đã không trả lời các gói Packets. Điều này cho phép sử dụng các dịch vụ DNS động (như dyndns.org) với plugin ping.

Mặc định: -1 (vô hiệu hóa)

**- Plugin processes**

Thu thập các thông tin về  processes của system

		<Plugin processes>
			Process "name"
		</Plugin>

**- Plugin tcpconns**

Plugin tcpconns tính số lượng các kết nối TCP được thiết lập hiện tại dựa trên port local và / hoặc port remote. Vì có thể có rất nhiều kết nối mặc định nếu đếm tất cả các kết nối với một port local. Có thể sử dụng các tùy chọn sau để lựa chọn các port cần thiết:

		<Plugin tcpconns>
			ListeningPorts false
			AllPortsSummary false
			LocalPort "25"
			RemotePort "25"
		</Plugin>

+ ListeningPorts true|false

Nếu được set thành true, thống kê cho tất cả các local port mà socket listen thu thập. Mặc định phụ thuộc vào LocalPort và RemotePort: Nếu không có cổng nào được chọn cụ thể, mặc định listen các port. Nếu các cổng cụ thể (LocalPort và RemotePort) được chọn, tùy chọn này sẽ sai, chỉ có các cổng được chọn sẽ được thu thập trừ khi tùy chọn này được chỉ định port cụ thể.

+ AllPortsSummary true|false: Nếu set thành true, một bản tóm tắt thống kê từ tất cả các kết nối sẽ được thu thập. Tùy chọn này mặc định là false.
+ LocalPort Port:  Đếm kết nối đến một LocalPort Port cụ thể. Điều này có thể được sử dụng để xem có bao nhiêu kết nối được xử lý bởi một daemon cụ thể. 

Bạn phải chỉ định cổng ở dạng số, mailserver đặt port 25.

+ RemotePort Port: Đếm các kết nối đến một cổng từ RemotePort cụ thể. 
 
Điều này hữu ích để xem bao nhiêu một dịch vụ từ xa được sử dụng. 

Điều này hữu ích nhất nếu muốn biết có bao nhiêu kết nối một dịch vụ local đã mở ra cho các dịch vụ remote.


**- Plugin write_graphite**

Plugin write_graphite viết/gửi/đẩy dữ liệu lên Graphite, để có thể hiển thị dạng đồ thị. Plugin kết nối với Carbon, lớp dữ liệu Graphite, thông qua TCP hoặc UDP và gửi dữ liệu qua giao thức "line based" (mặc định sử dụng cổng 2003). Dữ liệu sẽ được gửi trong các khối tối đa 1428 byte để giảm thiểu số lượng các gói tin mạng gửi đi.

	<Plugin write_graphite>
	        <Node "graphite">
	                Host "45.117.81.35"
	                Port "2003"
	                Protocol "tcp"
	                LogSendErrors true
	                Prefix "collectd."
	                StoreRates true
	                AlwaysAppendDS false
	                EscapeCharacter "_"
	        </Node>
	</Plugin>

+ Host Address: Hostname hoặc địa chỉ kết nối đến graphite để gửi metrics đến. Defaults là localhost
+ Port Service: Port để kết nối tới graphite server. Default 2003
+ Protocol String: Giao thức sử dụng để kết nối đến graphite server. Default là TCP
+ LogSendErrors false|true: Nếu thiết lập true (mặc định), ghi lại các lỗi khi gửi dữ liệu tới Graphite. Nếu thiết lập là false, nó sẽ không ghi lại lỗi. Điều này đặc biệt hữu ích khi sử dụng Protocol UDP lỗi ghi nhật ký ghi lại syslog với các thông điệp không cần thiết.
+ Prefix String: Khi thiết lập, String được thêm vào trước tên máy chủ lưu trữ. Dấu chấm và khoảng trắng không được thiếu trong chuỗi này.
+ ReconnectInterval Seconds: 
+ Postfix String: Khi được set chuỗi được thêm vào hostname. Dấu chấm và khoảng trắng không được thiếu trong chuỗi này.
+ EscapeCharacter Char: Carbon sử dụng dấu chấm (.) làm ký tự thoát và không cho phép khoảng trắng trong ký hiệu nhận diện. Tùy chọn EscapeCharacter xác định các ký tự dấu chấm, khoảng trống và ký tự điều khiển được thay bằng. Mặc định để gạch dưới (_).
+ StoreRates false|true: 
+ SeparateInstances false|true: Nếu thiết lập true, trường hợp plugin và chủ thể đường dẫn sẽ nằm trong thành phần đường dẫn của chúng, ví dụ host.cpu.0.cpu.idle. Nếu thiết lập là false (mặc định), plugin gắn thêm instance (và tương tự kiểu và kiểu thể hiện) được đưa vào một thành phần, ví dụ host.cpu-0.cpu-idle.
+ AlwaysAppendDS false|true
+ PreserveSeparator false|true: Nếu đặt thành false (mặc định) là. (dấu chấm) được thay bằng EscapeCharacter. Nếu thiết lập đúng, (dấu chấm) được sử dụng.
+ DropDuplicateFields false|true: Nếu thiết lập đúng, phát hiện và loại bỏ các thành phần trùng lặp trong tên số liệu Graphite. Ví dụ: tên lưu trữ tên host.load.load.shortterm sẽ được rút ngắn thành host.load.shortterm.
