# Chương 2. PHÂN TÍCH HỆ THỐNG

## 2.1. Ngữ cảnh hệ thống

Hệ thống được xây dựng trong bối cảnh một mô hình mạng doanh nghiệp giả lập, bao gồm nhiều thành phần và phân vùng mạng nhằm đảm bảo tính bảo mật và khả năng kiểm soát truy cập.

Tại lớp biên, hệ thống sử dụng thiết bị pfSense làm tường lửa để kết nối với Internet và thực hiện phân tách các vùng mạng nội bộ. Mạng được chia thành các vùng như sau:

- DMZ: Chứa các máy chủ cung cấp dịch vụ công khai như Web Server, cho phép truy cập từ Internet.
- Mạng nội bộ người dùng (Internal User): Bao gồm các máy trạm phục vụ hoạt động của người dùng trong hệ thống.
- Mạng cơ sở dữ liệu (Database): Lưu trữ dữ liệu quan trọng và chỉ cho phép truy cập từ các thành phần được kiểm soát
- Mạng giám sát (SIEM): Chứa hệ thống ELK Stack dùng để thu thập và phân tích dữ liệu log.

Trong hệ thống này, dữ liệu log được tạo ra từ nhiều nguồn khác nhau như máy chủ ở DMZ, máy trạm và thiết bị mạng (pfSense). Thay vì được lưu trữ riêng lẻ trên từng thiết bị, các endpoint được cài đặt Elastic Agent để thu thập và gửi log về hệ thống trung tâm.

## 2.2. Đặt vấn đề

Trong mô hình hệ thống đã xây dựng, dữ liệu log được tạo ra từ nhiều thành phần khác nhau như máy chủ trong vùng DMZ, máy trạm người dùng và thiết bị mạng. Tuy nhiên, các dữ liệu này thường được lưu trữ phân tán trên từng thiết bị riêng lẻ, gây khó khăn trong việc quản lý và theo dõi

Một số vấn đề chính có thể nhận thấy trong hệ thống:

- Dữ liệu log không được thu thập và lưu trữ tập trung, gây khó khăn trong việc tổng hợp và phân tích
- Việc tìm kiếm và truy vấn log gặp nhiều hạn chế do dữ liệu nằm rải rác ở nhiều nguồn khác nhau
- Thiếu công cụ trực quan hóa khiến việc giám sát hệ thống theo thời gian thực không hiệu quả
- Khó phát hiện các hành vi bất thường như đăng nhập thất bại nhiều lần hoặc quét cổng khi không có cái nhìn tổng thể.
- Việc phân tích log chủ yếu thực hiện thủ công, tốn thời gian và dễ bỏ sót sự cố.

Từ những vấn đề trên, việc xây dựng một hệ thống giám sát log tập trung là cần thiết nhằm nâng cao khả năng theo dõi, phân tích và phát hiện các sự cố an ninh trong hệ thống.

## 2.3. Giải pháp đề xuất

Để giải quyết vấn đề dữ liệu log phân tán, đề tài đề xuất xây dựng hệ thống giám sát log tập trung sử dụng ELK Stack kết hợp với Elastic Agent.

Trong hệ thống, Elastic Agent được cài đặt trên các endpoint để thu thập log và gửi trực tiếp về Elasticsearch. Tại đây, dữ liệu được xử lý thông qua ingest pipeline trước khi lưu trữ. Người quản trị sử dụng Kibana để truy vấn, phân tích và trực quan hóa dữ liệu.

Luồng xử lý dữ liệu:

Elastic Agent/Fleet → Elasticsearch Ingest Pipeline→ Elasticsearch

Giải pháp giúp tập trung hóa log, hỗ trợ tìm kiếm nhanh và nâng cao khả năng phát hiện bất thường trong hệ thống
