## 1. Chuẩn bị ##

+ Ubuntu 16.04 LTS

+ CPU: 1 vCPU

+ Memory: 1GB RAM

+ Disk: 2.5-5GB tùy thuộc vào số node monitor

+ Disk performance: 20-200 random IOPs trên mỗi node server monitor.

+ Use NetApp storage (NFS, LUN, vmdk, vhd, vhdx) hoặc local SSD

Cài đặt cơ bản: Cấu hình IP, kiểm tra kết nối internet, firewall, đồng bộ thời gian, update...

## 2. Cài đặt grafana ##

- Tải grafana ver 5.4

        sudo yum install https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.1.4-1.x86_64.rpm

        wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.1.4-1.x86_64.rpm

        sudo yum install initscripts fontconfig

- Install via YUM Repository

Tạo file mới /etc/yum.repos.d/grafana.repo

        vi /etc/yum.repos.d/grafana.repo

Thêm các dòng sau

        [grafana]
        name=grafana
        baseurl=https://packagecloud.io/grafana/stable/el/7/$basearch
        repo_gpgcheck=1
        enabled=1
        gpgcheck=1
        gpgkey=https://packagecloud.io/gpg.key https://grafanarel.s3.amazonaws.com/RPM-GPG-KEY-grafana
        sslverify=1
        sslcacert=/etc/pki/tls/certs/ca-bundle.crt
- Cài đặt grafna 

        sudo yum install grafana

        systemctl daemon-reload
        systemctl start grafana-server
        systemctl status grafana-server
        systemctl enable grafana-server.service
        systemctl restart grafana-server

- Truy cập giao diện quản trị grafana

http://ip_server:3000 

Với tài khoản admin/admin

Truy cập lần đầu sẽ yêu cầu bạn thay đổi password mặc định

![Imgur](https://i.imgur.com/sUcOpjv.png)

## 3. Cài đặt graphite ##

- Cài package cần thiết

    sudo yum install -y http://epel.mirror.constant.com/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm

![Imgur](https://i.imgur.com/6dEnjA0.png)

    sudo yum install -y graphite-web python-carbon

- Cấu hình graphite

    vi /etc/carbon/storage-schemas.conf

Thêm dòng sau

    [default]
    pattern = .*
    retentions = 12s:4h, 2m:3d, 5m:8d, 13m:32d, 1h:1y

+Restart service

    systemctl enable carbon-cache
    systemctl start carbon-cache
    systemctl restart carbon-cache

+Set timezone, SECRET_KEY

    cp /etc/graphite-web/local_settings.py /etc/graphite-web/local_settings.py.bk

    vi /etc/graphite-web/local_settings.py

Chỉnh sửa

    #TIME_ZONE = 'America/Los_Angeles'
    change to 
    TIME_ZONE = 'Asia/Ho_Chi_Minh'
 

    #SECRET_KEY = 'UNSAFE_DEFAULT'
    change to 
    SECRET_KEY = ‘KJKzzxcviouzxcvytwkdd94944asmdf9ads9ds9dspp83jlkjLKJL98798KLOIK’

+Thiết lập CSDL

    PYTHONPATH=/usr/share/graphite/webapp django-admin syncdb --settings=graphite.settings

Nhập password cho user root

![Imgur](https://i.imgur.com/Qbvr9T4.png)

- Cấu hình Apache cho Graphite

        cp /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf.bk
        echo > /etc/httpd/conf.d/welcome.conf

Chỉnh sửa /etc/httpd/conf.d/graphite-web.conf

        vi /etc/httpd/conf.d/graphite-web.conf

        Require all granted
        Order allow,deny
        Allow from all


        sudo chown apache:apache /var/lib/graphite-web/graphite.db

        touch /var/lib/graphite-web/index

        systemctl start httpd  
        systemctl enable httpd

- Truy cập graphite qua giao diện web

http://ip_server

![Imgur](https://i.imgur.com/j0XFslz.png)


# 4. Cài đặt collectd #

    yum install epel-release
    yum install -y collectd collectd-rrdtool rrdtool collectd-web
    yum search collectd
    systemctl enable collectd
    systemctl start collectd
    systemctl restart collectd
    cat /etc/httpd/conf.d/collectd.conf 
    sed -i 's/# Require all granted/Require all granted/g;s/Require local/# Require local/g' /etc/httpd/conf.d/collectd.conf 
    systemctl restart httpd collectd


![Imgur](https://i.imgur.com/XeoS7LR.png)



