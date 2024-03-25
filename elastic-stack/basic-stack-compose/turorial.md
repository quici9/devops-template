# Giải thích các thành phần

## Cấu hình Elasticsearch Cluster

### Nodes (es01, es02, es03)

Các nodes: Đại diện cho các máy instance chạy Elasticsearch. Mỗi node có một vai trò cụ thể trong cụm.

Tính khả dụng cao (High Availability): Cluster multi-node nhằm đảm bảo tính khả dụng của dữ liệu và dịch vụ. Nếu một node gặp sự cố, các node khác vẫn đảm bảo cluster hoạt động bình thường. Càng nhiều node đồng nghĩa với độ bền vững (resiliency) của cluster càng cao.

### Khởi tạo và Khám phá Cụm (discovery.seed_hosts, cluster.initial_master_nodes)

**discovery.seed_hosts**: Danh sách các node ban đầu mà node hiện tại có thể kết nối để tìm các node khác trong cụm. Điều này rất quan trọng khi khởi tạo cụm lần đầu.

**cluster.initial_master_nodes**: Danh sách các node đủ điều kiện để trở thành master node. Master node chịu trách nhiệm về các hoạt động cấp cụm như tạo chỉ mục, điều phối shards và replicas.

### Thiết lập Bộ nhớ (ES_JAVA_OPTS, bootstrap.memory_lock, mem_limit)

**ES_JAVA_OPTS**: Chỉ định các tùy chọn cho Java Virtual Machine (JVM), ở đây thiết lập dung lượng bộ nhớ heap tối thiểu (-Xms) và tối đa (-Xmx) cho Elasticsearch.

**bootstrap.memory_lock**: Cho phép Elasticsearch khóa bộ nhớ, ngăn chặn việc bộ nhớ bị hoán đổi (swap) ra đĩa. Điều này tối ưu hiệu suất.

**mem_limit**: Giới hạn lượng tài nguyên bộ nhớ mà container có thể sử dụng. Giúp ngăn Elasticsearch sử dụng hết tài nguyên trên máy chủ vật lý.

### Tính bền vững dữ liệu (volumes)

**volumes** liên kết một thư mục bên trong container (*/usr/share/elasticsearch/data*) với một thư mục trên máy chủ vật lý. Dữ liệu Elasticsearch được lưu trong các thư mục này sẽ được giữ lại ngay cả khi container bị khởi động lại hoặc dừng hoạt động.

### Elasticsearch Health Check

**healthcheck**: Gửi yêu cầu HTTP đến cổng 9200 của Elasticsearch. Nếu Elasticsearch phản hồi thành công, container được coi là *healthy* (khỏe mạnh).

## Cấu hình Kibana

### Connection to Elasticsearch (ELASTICSEARCH_HOSTS, ELASTICSEARCH_USERNAME, ELASTICSEARCH_PASSWORD)

**ELASTICSEARCH_HOSTS**: Xác định địa chỉ Elasticsearch mà Kibana kết nối đến. Chú ý ở đây, Kibana sử dụng tên service es01 với cổng 9200 (cổng mặc định của Elasticsearch). Docker compose có khả năng giải quyết tên hostname trong network nội bộ, tạo điều kiện thuận lợi cho việc liên kết dịch vụ.

**ELASTICSEARCH_USERNAME**: Tên người dùng được sử dụng để xác thực với Elasticsearch. Ở đây, nó sử dụng tài khoản hệ thống dành riêng cho Kibana kibana_system.

**ELASTICSEARCH_PASSWORD**: Mật khẩu ứng với người dùng kibana_system.

### Memory Settings (mem_limit)

**mem_limit**: Xác định giới hạn tài nguyên bộ nhớ tối đa mà container Kibana có thể sử dụng. Điều này giúp ngăn Kibana chiếm dụng tài nguyên quá mức trên máy chủ vật lý, đảm bảo hoạt động trơn tru cho các dịch vụ khác.

### Kibana Health Check

**healthcheck**: Gửi yêu cầu HTTP đến cổng 5601 (cổng mặc định của Kibana) và kiểm tra mã phản hồi HTTP 302 Found. Phản hồi này là dấu hiệu Kibana đang hoạt động và chuyển hướng HTTP requests.

## Cấu hình Logstash

### Inputs, Filters, Outputs (Cấu trúc Logstash Pipeline)

**Inputs**: Xác định nguồn dữ liệu mà Logstash sẽ lấy (ví dụ: files, databases, message queues). Các input plugin phổ biến bao gồm file, jdbc, kafka, beats, etc.

**Filters**: Chịu trách nhiệm xử lý dữ liệu. Các filter có thể được dùng để trích xuất dữ liệu có cấu trúc từ logs, thêm fields mới, sửa đổi dữ liệu, hoặc lọc các events không mong muốn. Một số filters phổ biến gồm grok, mutate, json.

**Outputs**: Chỉ định các điểm đến sẽ nhận dữ liệu. Elasticsearch là output phổ biến của Logstash. Một số outputs khác bao gồm file, kafka, redis.

### Kết nối tới Elasticsearch (ELASTICSEARCH_HOSTS, ...)

**ELASTICSEARCH_HOSTS**: Xác định địa chỉ của cụm Elasticsearch mà Logstash gửi dữ liệu đến. Ở đây, Logstash sử dụng tên service es01 với cổng 9200.

**ELASTICSEARCH_USERNAME** và **ELASTICSEARCH_PASSWORD**: Tên người dùng và mật khẩu để Logstash xác thực với Elasticsearch.

### Logstash Pipeline (pipeline directory)

**./logstash/pipeline**: Thư mục chứa các file định nghĩa pipeline của Logstash. Mỗi file mô tả cụ thể cách Logstash xử lý một loại dữ liệu đầu vào khác nhau.

**pipelines.yml**: File cấu hình nơi bạn liệt kê các pipeline mà Logstash sẽ tải khi khởi động.

Ví dụ Logstash Pipeline

```docker
input {
  file {
    path => "/var/log/my_app.log"
  }
}

filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:loglevel} %{GREEDYDATA:message}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://es01:9200"]
    user => "elastic"
    password => "${ELASTIC_PASSWORD}"
    index => "my_app_logs-%{+YYYY.MM.dd}" 
  }
}
```

### Memory Settings (LS_JAVA_OPTS, mem_limit)

**LS_JAVA_OPTS**: Xác định các tùy chọn cho Java Virtual Machine (JVM), ở đây thiết lập dung lượng bộ nhớ heap tối thiểu (-Xms) và tối đa (-Xmx) cho Logstash.

**mem_limit**: Giới hạn lượng tài nguyên bộ nhớ mà container Logstash có thể sử dụng nhằm đảm bảo sự ổn định của máy chủ.
