# Tuning MySQL: 3 Simple Tweaks

### 1. Innodb per table

- Thực hiện trước bước import db của zabbix vào mysql 

- Sửa hoặc thêm dòng cấu hình này trong `/etc/my.cnf`

    ```innodb_file_per_table=1```

### 2. Let InnoDB use all that memory

- Đối với máy chủ chỉ chạy MySQL thì nên để tham số  `innodb_buffer_pool_size` chiếm 80% tổng dung lượng memory

    TH: Máy chủ bạn có 32GB, Đặt ngữong 25GB

    `innodb_buffer_pool_size = 25600M`

- Đối vơi máy chủ aio zabbix server thì để khoảng 70%(phần này cần theo dõi thêm)

### 3. Let InnoDB multitask

- Lợi ích của việc sửa dụng InnoDB multitask: 

    Bạn có thể gặp các nút cổ chai từ nhiều luồng cố gắng truy cập nhóm bộ đệm cùng một lúc. Bạn có thể kích hoạt nhiều buffer pools để giảm thiểu sự tranh chấp này.

- Trong ví dụ của chúng tôi về máy chủ 32 GB có `innodb_buffer_pool_size=25 GB`, một giải pháp phù hợp có thể là `25600M / 24 = 1.06GB`

    => ```innodb_buffer_pool_instances = 24```

- Nểu để tỉ lệ `innodb_buffer_pool_size/innodb_buffer_pool_instances` hơn kém 1

# Đừng Quên: 

- Khởi động lại ``mysqld``
