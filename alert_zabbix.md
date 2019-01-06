# Alert - Zabbix - To be the best

Sau gần một năm tìm hiểu zabbix trong môi trường lab, tôi gần đây qua lại với nó và triển khai nó trong môi trường production. Rất nhiều vấn đề gặp phải (nó không còn đẹp đẽ như trong lab), một trong những vấn đề cực kì khiến tôi phải suy nghĩ là hiệu năng và hiệu quả. Sẽ có một series bài như: Cấu hình mysql như thế nào? Đặt các trigger như nào để không spam gây phiền toán? Hay những plugin mà tôi viết thêm ... 
Nhưng hôm nay tôi xin chia sẻ cách cảnh báo zabbix thông qua mail và telegram (2 phần mềm được mọi người sửa dụng mọi lúc mọi nơi và đang là trend của bây giờ)


## Script send mail và telegram

- Trong zabbix thư mục mặc đặt các script cho cảnh báo nằm ở thư mục sau:

    ```
    [root@nhzabbix01 ~]# cat /etc/zabbix/zabbix_server.conf | grep alert
    #	Number of pre-forked instances of alerters.
    #	Full path to location of custom alert scripts.
    # AlertScriptsPath=${datadir}/zabbix/alertscripts
    AlertScriptsPath=/usr/lib/zabbix/alertscripts
    [root@nhzabbix01 ~]# 
    ```

- Tải 2 script mail và telegram của tôi ở đây

## Cấu hình trên dashboard

### Thiết lập Users

Click `Administration > Users` chọn Admin trong cột Alias. Sau khi ấn vô đó kết quả: 

<img src="https://i.imgur.com/T9UOQlb.png">

Ở đây, chuyển sang tab `Media` chọn `add`:

<img src="https://i.imgur.com/hwRIrf8.png">

add thêm telegramID:

<img src="https://i.imgur.com/YOUboWp.png">

Sau khi add xong kết quả: 

<img src="https://i.imgur.com/I9OQxUE.png">

### Thiết lập actions

Click `Configuration > Actions` chọn `Create action`. Tiến hành đặt tên và condition:

<img src="https://i.imgur.com/9w36y84.png">


Click sang tab `Operations` để thiết lập 

- Default operation step duration : `1m`
- Default subject : `Disaster {HOSTNAME}:{TRIGGER.NAME}-status-{TRIGGER.STATUS}`
- Default message : 
    ```
    {TRIGGER.NAME} on {HOSTNAME}
    Status:{TRIGGER.STATUS}
    Severity:{TRIGGER.SEVERITY}
    Values:{ITEM.VALUE1}

    Item Graphic: [{ITEM.ID1}]
    ```
- Operations : Cấu hình send đến Zabbix administrators via Telegram and Gmail 

<img src="https://i.imgur.com/IqKrwxc.png">

<img src="https://i.imgur.com/4lNrRTj.png">

Kết quả như sau:

<img src="https://i.imgur.com/BrVpbu9.png">

Click sang tab `Recovery operations` cấu hình tương tự tab `Operations`

### Kiểm tra lại cấu hình cho việc cảnh báo: 

Để thực hiện việc này ta cần định nghĩa ra các trigger và item test

- Import templates test. 

Click `Configuration > Templates` chọn `Import`

<img src="https://i.imgur.com/NjnnhP7.png">

Tải file xml ở đây và import. 

- Add templates test vào các host

Click `Configuration > Hosts` chọn host cần add template, add template ở đây: 

<img src="https://i.imgur.com/seUTNEx.png">

Template test của mình là `Dummy triggers`

# Thực hiện test cảnh báo trên `zabbix-server`

- Cài đặt zabbix_sender: 

```
 wget https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-sender-4.0.2-1.el7.x86_64.rpm
 rpm -ivh zabbix-sender-4.0.2-1.el7.x86_64.rpm

```

- Đối với lần đâu ta cần thực hiện lệnh này 2 lần liên tiếp:

```
    zabbix_sender \
        --zabbix-server=127.0.0.1 \
        --host="Zabbix server" \
        --key="test.timestamp" \
        --value="${VALUE}"
```

Trong đó: 

- zabbix-server là <IP của zabbix server>
- host="<Hostname agent được khai báo trên dashboard>"

### Mở mail và telegram ra để đón nhận sự tuyệt vời


<img src="https://i.imgur.com/BoWxWoB.png">

<img src="https://i.imgur.com/qYOhYXu.png">

Như vậy đã thành công.





 




