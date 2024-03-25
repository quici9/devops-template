# Triển khai Elastic Stack cơ bản

## Tổng quan Elastic Stack (ELK)

### 1. Giới thiệu

**Elastic Stack** (hay **ELK Stack**) là tập hợp các phần mềm mã nguồn mở giúp thu thập, lưu trữ, tìm kiếm, phân tích và trực quan hóa dữ liệu. Nó bao gồm:

- Elasticsearch: Cơ sở dữ liệu phân tán, cung cấp khả năng tìm kiếm và phân tích dữ liệu hiệu quả.
- Logstash: Công cụ thu thập và xử lý dữ liệu từ nhiều nguồn khác nhau.
- Kibana: Giao diện trực quan hóa dữ liệu, giúp người dùng dễ dàng khám phá và phân tích dữ liệu.

### 2. Trường hợp sử dụng

- Đồng bộ dữ liệu từ cơ sở dữ liệu quan hệ PostgresQL: Logstash có thể được sử dụng để trích xuất dữ liệu từ PostgresQL và lưu trữ trong Elasticsearch. Sau đó, Kibana có thể được sử dụng để trực quan hóa và phân tích dữ liệu.
- Tìm kiếm và phân tích: Elasticsearch cung cấp khả năng tìm kiếm và phân tích dữ liệu mạnh mẽ. Kibana cho phép người dùng dễ dàng tạo các biểu đồ, bảng điều khiển và báo cáo để trực quan hóa dữ liệu.

### 3. Lợi ích

- Khả năng mở rộng: Elastic Stack có thể được mở rộng để xử lý lượng dữ liệu lớn.
- Tính linh hoạt: Hỗ trợ nhiều nguồn dữ liệu và định dạng dữ liệu khác nhau.
- Dễ sử dụng: Kibana cung cấp giao diện trực quan giúp người dùng dễ dàng sử dụng.
- Mã nguồn mở: Miễn phí và có thể tùy chỉnh.

## Tại sao nên sử dụng repo này?

Tiết kiệm thời gian và công sức: Repo giúp bạn tiết kiệm thời gian và công sức trong việc triển khai ELK, cho phép bạn tập trung vào việc sử dụng ELK để phân tích dữ liệu.

Dành cho tất cả mọi người: Repo này phù hợp với mọi người, từ người mới bắt đầu đến người có kinh nghiệm sử dụng ELK.

## Yêu cầu tối thiểu

1. Docker và Docker compose
2. Hiểu biết cơ bản về Elastic Stack

## Cách sử dụng repo

1. Clone repo
2. Copy .env.example thành .env và điền các thông tin cần thiết
3. Tạo image Logstash với thư viện driver của postgres

    ```bash
    cd logstash/custom-image
    docker build -t logstash-postgres:0.1 .
    ```

4. Chạy docker compose

    ```docker
    sudo docker compose up -d
    ```

5. Truy cập vào Kibana tại `http://localhost:5601`

## Lưu ý

- Repo này triển khai với mức bảo mật cơ bản, chỉ bao gồm mật khẩu cho elasticsearch và kibana.
- Đặt kích thước heap trong khoảng từ 25% đến 50% RAM khả dụng, không vượt quá 32GB.
