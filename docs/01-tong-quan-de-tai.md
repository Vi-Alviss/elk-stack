# Chương 1. TỔNG QUAN ĐỀ TÀI

## 1.1. Giới thiệu đề tài

Trong kỷ nguyên số hóa thì dữ liệu đã trở thành một trong những tài sản quan trọng của mọi tổ chức, đồng thời cũng là mục tiêu hàng đầu của các cuộc tấn công mạng. Các kỹ thuật xâm nhập như Brute Force, Port Scanning hay các hành vi khai thác lỗ hổng hệ thống ngày càng trở nên tinh vi và khó nhận diện bằng các phương pháp thủ công

Một trong những rào cản lớn đối với đội ngũ vận hành an ninh là sự rời rạc của dữ liệu log từ nhiều thiết bị khác nhau dẫn đến việc thiếu tầm nhìn tổng thể về các mối đe dọa. Việc không thể giám sát và phân tích sự kiện bảo mật một cách tập trung khiến tổ chức gặp nhiều khó khăn trong việc phát hiện, điều tra và phản ứng trước các sự cố an ninh mạng

Để nâng cao khả năng giám sát và phòng thủ, giải pháp SIEM (Security Information and Event Management) đã được sử dụng nhằm thu thập, lưu trữ và phân tích dữ liệu bảo mật một cách tập trung. Trong đó, ELK Stack (Elasticsearch, Logstash, Kibana) là một bộ công cụ mã nguồn mở phổ biến, cung cấp khả năng xử lý, tìm kiếm và trực quan hóa dữ liệu log hiệu quả, hỗ trợ quá trình giám sát an ninh mạng

Vì vậy, đề tài tập trung vào việc nghiên cứu và triển khai hệ thống giám sát an ninh mạng sử dụng ELK Stack kết hợp với Elastic Agent. Hệ thống hướng đến việc thu thập log tập trung, chuẩn hóa dữ liệu, hỗ trợ tìm kiếm, trực quan hóa thông tin bảo mật và phát hiện một số hành vi bất thường trong môi trường mạng

## 1.2. Mục tiêu đề tài

Mục tiêu tổng quát của đề tài là nghiên cứu và triển khai hệ thống giám sát, thu thập và phân tích dữ liệu log tập trung dựa trên nền tảng ELK Stack kết hợp với Elastic Agent. Hệ thống hướng đến việc hỗ trợ người quản trị trong quá trình theo dõi, tìm kiếm, phân tích sự kiện bảo mật và phát hiện các hành vi bất thường trong môi trường mạng.

Các mục tiêu cụ thể bao gồm:

- Xây dựng hệ thống thu thập log tập trung: Triển khai Elastic Agent trên các thiết bị đầu cuối nhằm thu thập dữ liệu log và sự kiện hệ thống một cách tập trung, liên tục và thống nhất.
- Chuẩn hóa và xử lý dữ liệu log: Thực hiện xử lý và chuẩn hóa dữ liệu log nhằm đưa dữ liệu về định dạng phù hợp, giúp hệ thống dễ dàng lưu trữ, tìm kiếm, phân tích và tương quan sự kiện.
- Hỗ trợ tìm kiếm và lọc dữ liệu hiệu quả: Khai thác khả năng tìm kiếm của Elasticsearch và các bộ lọc có sẵn trong Kibana để hỗ trợ truy vấn, phân tích và truy vết sự kiện bảo mật một cách thuận tiện.
- Xây dựng dashboard giám sát trực quan: Thiết kế các dashboard trên Kibana nhằm trực quan hóa dữ liệu log, giúp người quản trị dễ dàng theo dõi tình trạng hệ thống, quan sát các sự kiện nổi bật và nhận diện dấu hiệu bất thường.
- Triển khai giám sát endpoint và EDR: Tích hợp Elastic Agent với các tính năng Endpoint Security nhằm giám sát hoạt động trên thiết bị đầu cuối, theo dõi các tiến trình, hành vi hệ thống và hỗ trợ phát hiện các dấu hiệu đáng ngờ có thể liên quan đến tấn công hoặc mã độc.
- Mở rộng khả năng phát hiện bất thường bằng AI/ML: Nghiên cứu khả năng sử dụng các tính năng phân tích nâng cao và học máy có sẵn trong hệ sinh thái Elastic nhằm hỗ trợ phát hiện các hành vi bất thường, từ đó nâng cao hiệu quả giám sát an ninh mạng trong tương lai.

## 1.3. Phạm vi nghiên cứu

Đề tài tập trung nghiên cứu và triển khai hệ thống giám sát và phân tích log tập trung trong môi trường giả lập với các phạm vi và giới hạn như sau:

- Về nền tảng triển khai: Hệ thống được xây dựng chủ yếu trên hệ điều hành Ubuntu, bao gồm máy chủ SIEM, các máy chủ dịch vụ và các máy trạm trong môi trường thử nghiệm.
- Về chức năng hệ thống: Hệ thống tập trung vào các chức năng chính bao gồm thu thập log tập trung, chuẩn hóa dữ liệu, hỗ trợ tìm kiếm và lọc log, trực quan hóa thông qua dashboard và giám sát endpoint.
- Về phạm vi dữ liệu: Dữ liệu log được thu thập từ các thành phần trong hệ thống như máy chủ, máy trạm và thiết bị mạng, phục vụ cho mục đích phân tích và phát hiện bất thường.
- Về giới hạn nghiên cứu: Đề tài được triển khai trong môi trường giả lập với quy mô nhỏ, chưa áp dụng trong hệ thống thực tế. Các tính năng nâng cao như EDR chuyên sâu và AI/ML chỉ được nghiên cứu và thử nghiệm ở mức cơ bản.
