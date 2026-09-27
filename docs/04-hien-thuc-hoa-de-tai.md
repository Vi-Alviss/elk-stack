# Chương 4. HIỆN THỰC HOÁ ĐỀ TÀI - CẤU HÌNH TRIỂN KHAI

## 4.1. Kiến trúc hệ thống

![Hình 4.1. Mô hình Kiến trúc hệ thống.](/images/i41.png)

Hình 4.1. Mô hình Kiến trúc hệ thống.

## 4.2. Phân tích hệ thống

Dựa trên mô hình kiến trúc tổng thể đã đề xuất ở mục 4.1, hệ thống được triển khai trên môi trường ảo hóa (VirtualBox). Quá trình phân tích hệ thống được chia thành ba phần chính: Phân vùng mạng, Chi tiết các thành phần thiết bị và Chiến lược thu thập dữ liệu.

### 4.2.1. Phân vùng không gian mạng và địa chỉ IP

Hệ thống sử dụng thiết bị pfSense làm bộ định tuyến và tường lửa biên (Edge Firewall/Router). Mạng được chia làm 2 vùng chính là WAN (kết nối Internet) và LAN (mạng nội bộ). Để đảm bảo tính bảo mật và nguyên tắc đặc quyền tối thiểu, vùng LAN ban đầu (dải 192.168.1.0/24) được phân tách thành 4 VLAN riêng biệt.

Tất cả các giao diện mạng của pfSense đóng vai trò là Default Gateway cho các VLAN nội bộ với địa chỉ IP có dạng x.x.x.1/24. Cụ thể:

- **Vùng WAN:** Nhận IP động (DHCP) từ môi trường ảo hóa để giao tiếp với Internet. Giao diện nhận dải mạng mặc định là 10.0.2.15/24.
- **VLAN 10 - Vùng DMZ (192.168.10.0/24):** Vùng phi quân sự, nơi đặt các máy chủ cung cấp dịch vụ ra bên ngoài nhưng bị cách ly một phần với mạng nội bộ để giảm thiểu rủi ro khi bị tấn công. (Gateway: 192.168.10.1).
- **VLAN 20 - Vùng Internal User (192.168.20.0/24):** Vùng mạng dành cho người dùng cuối trong tổ chức. Chứa các máy trạm phục vụ công việc hàng ngày. (Gateway: 192.168.20.1).
- **VLAN 30 - Vùng Internal Database (192.168.30.0/24):** Vùng lưu trữ cơ sở dữ liệu lõi. Đây là vùng có mức độ bảo mật cao nhất, chỉ cho phép truy cập từ các máy chủ chỉ định (ví dụ: Web Server ở DMZ). (Gateway: 192.168.30.1).
- **VLAN 40 - Vùng SIEM / Threat Hunting (192.168.40.0/24):** Vùng quản trị an ninh, nơi đặt hạ tầng ELK Stack nhằm giám sát, thu thập và phân tích toàn bộ log của hệ thống. (Gateway: 192.168.40.1).

### 4.2.2. Chi tiết các nút mạng

Mỗi thiết bị trong hệ thống được gán địa chỉ IP tĩnh có dạng x.x.x.2/24 (tương ứng với từng VLAN). Các thành phần bao gồm:

- **Node: pfSense**
  - Hệ điều hành: FreeBSD (pfSense)
  - Vị trí: Gateway giữa Internet và LAN
  - IP WAN: 10.0.2.15/24
  - IP LAN: 192.168.10.1, 192.168.20.1, 192.168.30.1, 192.168.40.1
  - Tài nguyên: 2 GB RAM, 20 GB disk
  - Vai trò: Kiểm soát lưu lượng, định tuyến giữa các VLAN, và làm nguồn sinh log mạng (Firewall Filter Logs, Port Scanning, Routing events).
- **Node: web**
  - Vị trí: Vùng DMZ
  - Hệ điều hành: Ubuntu Server
  - IP: 192.168.10.2/24
  - Tài nguyên: 4 GB RAM, 30 GB disk, 2 core
  - Vai trò: Chạy dịch vụ Apache, cung cấp ứng dụng kiểm thử bảo mật Web (DVWA).
- **Node: user**
  - Vị trí: Vùng Internal User
  - Hệ điều hành: Ubuntu Desktop
  - IP: 192.168.20.2/24
  - Tài nguyên: 4 GB RAM, 30 GB disk, 2 core
  - Vai trò: Đóng vai trò là máy khách, dùng để mô phỏng các hành vi bình thường của nhân viên hoặc thực hiện các kịch bản tấn công (ping dò quét mạng, tải mã độc EICAR) nhằm kích hoạt tính năng EDR.
- **Node: database**
  - Vị trí: Vùng Internal Database
  - Hệ điều hành: Ubuntu Server
  - IP: 192.168.30.2/24
  - Tài nguyên: 2 GB RAM, 30 GB disk, 2 core
  - Vai trò: Chạy dịch vụ MySQL, cung cấp cơ sở dữ liệu cho máy web chạy ứng dụng DVWA.
- **Node: siem**
  - Vị trí: Vùng SIEM
  - Hệ điều hành: Ubuntu Server
  - IP: 192.168.40.2/24
  - Tài nguyên: 8 GB RAM, 80 GB disk, 4 core
  - Vai trò: Máy chủ trung tâm của hệ thống, cài đặt Elasticsearch (cơ sở dữ liệu lưu trữ log), Kibana (giao diện dashboard), và Fleet Server (quản lý tập trung các Agent).

### 4.2.3. Chiến lược triển khai Elastic Agent và Integrations

Hệ thống sử dụng mô hình hiện đại quản lý qua Fleet Server. Các endpoint sẽ được cài đặt Elastic Agent để thu thập log máy chủ, log dịch vụ và thực hiện phản hồi (EDR) gửi thẳng về Elasticsearch. Cấu hình Integrations cho từng máy được quy hoạch như sau:

- **Máy web:** Cài đặt trực tiếp Elastic Agent. Sử dụng các Integrations: System (thu thập metric CPU, RAM, OS logs) và Apache (đọc access.log và error.log để theo dõi các nỗ lực tấn công ứng dụng Web như SQLi, dò quét thư mục).
- **Máy user:** Cài đặt trực tiếp Elastic Agent. Sử dụng các Integrations: System và kết hợp thêm Elastic Defend (như đã phân tích ở các kịch bản EDR) để giám sát tệp tin, tiến trình độc hại và thực thi cô lập máy chủ (Isolate Host) khi phát hiện sự cố.
- **Máy database:** Cài đặt trực tiếp Elastic Agent. Sử dụng các Integrations: System và MySQL (giám sát các luồng truy vấn cơ sở dữ liệu, lỗi đăng nhập database).
- **Máy siem:** Cài đặt Elastic Agent với Integrations là Fleet Server và System. Agent này đồng thời được bổ sung Integration pfSense để tiếp nhận luồng Syslog từ tường lửa pfSense gửi về.
- **Thiết bị pfSense:** Do chạy trên nền FreeBSD không hỗ trợ cài đặt trực tiếp Elastic Agent. Do đó, chiến lược thu thập log sẽ được thực hiện gián tiếp. pfSense được cấu hình gửi Syslog đến máy siem. Elastic Agent tại máy siem được bổ sung vào policy Integrations pfSense để mở port lắng nghe luồng Syslog từ tường lửa, sau đó phân tách dữ liệu và đưa vào hệ thống SIEM.

## 4.3. Kịch bản triển khai

### 4.3.1. Thu thập log

Kịch bản này được xây dựng trong môi trường mạng giả lập nhằm minh họa khả năng thu thập dữ liệu đa nguồn và quản lý tập trung của hệ thống ELK Stack. Để chứng minh khả năng này, hệ thống sẽ tiến hành thu thập nhật ký (log) từ hai thành phần trọng yếu, đại diện cho hạ tầng mạng và máy chủ ứng dụng. Trong phạm vi báo cáo, để đánh giá hiệu quả của hệ thống, kịch bản sẽ chỉ trích xuất và phân tích các mẫu log tiêu biểu cho từng loại hành vi:

- **Thiết bị tường lửa kiêm định tuyến biên (pfSense):** Hệ thống sẽ tập trung thu thập nhật ký bộ lọc tường lửa (firewall filter logs) nhằm giám sát lưu lượng mạng ở mức network layer. Báo cáo sẽ trình bày 02 mẫu log điển hình:
  - Mẫu 1 (Chặn Ping): Log ghi nhận và cảnh báo hành động tường lửa ngăn chặn gói tin dò tìm mạng (luồng ICMP Ping) xuất phát từ máy trạm người dùng (Ubuntu Desktop) hướng đến máy chủ Web.
  - Mẫu 2 (Port Scanning): Log phát hiện và ngăn chặn nỗ lực rà quét cổng mạng tự động từ máy trạm người dùng nhằm tìm kiếm các lỗ hổng dịch vụ đang mở.
- **Máy chủ Web (Ubuntu Server + Apache):** Được triển khai để cung cấp ứng dụng kiểm thử bảo mật DVWA. Thông qua Elastic Agent, hệ thống sẽ đọc và gửi về trung tâm hai tệp nhật ký quan trọng nhất của Apache. Báo cáo sẽ trình bày 02 mẫu tấn công ứng dụng điển hình:
  - Mẫu 3 (SQL Injection): Log ghi nhận cuộc tấn công SQL Injection từ máy trạm người dùng vào ứng dụng DVWA (cấu hình ở mức bảo mật Low). Dấu vết payload tấn công sẽ được trích xuất từ nhật ký truy cập (access.log).
  - Mẫu 4 (Dò quét thư mục/Truy cập trái phép): Log ghi nhận nỗ lực dò quét thư mục ẩn trên máy chủ Web, dẫn đến hàng loạt mã lỗi truy cập thất bại (như HTTP 404 Not Found hoặc 403 Forbidden). Dấu vết này sẽ được trích xuất từ nhật ký lỗi (error.log) và nhật ký truy cập, giúp minh họa khả năng giám sát các nỗ lực thăm dò hệ thống trước khi tấn công.

### 4.3.2. Làm giàu log

Mục tiêu của kịch bản này là minh họa vai trò cốt lõi của Ingest Pipeline trong Elasticsearch. Quá trình làm giàu và chuẩn hóa dữ liệu diễn ra hoàn toàn tự động ở background. Do đó, khi truy cập giao diện Discover trên Kibana, hệ thống sẽ mặc định hiển thị ngay các bản ghi đã được phân tách thành các trường thông tin gọn gàng theo chuẩn ECS (Elastic Common Schema). Để minh họa chức năng làm giàu dữ liệu của ELK Stack, kịch bản sẽ tiến hành đối chiếu log thô (raw log) với log đã được làm giàu và hiển thị trên Kibana, dựa trên 4 mẫu log thu thập được từ kịch bản trước.

### 4.3.3. Tìm kiếm dễ dàng

Mục tiêu của kịch bản này là minh họa khả năng tìm kiếm và truy vấn dữ liệu của Elasticsearch thông qua giao diện Discover trên Kibana. Kịch bản sẽ giới thiệu các cơ chế lọc được Kibana hỗ trợ, bao gồm Time filter, KQL (Kibana Query Language) và Free-text search, sau đó tiến hành thực hiện truy vấn lần lượt trên 4 mẫu log thu thập được từ kịch bản 1, nhằm minh họa khả năng định vị chính xác từng sự kiện trong lượng dữ liệu lớn.

### 4.3.4. Phát hiện và phản hồi

Kịch bản phát hiện và phản hồi được xây dựng nhằm minh họa khả năng EDR của Elastic Defend. Máy user được sử dụng làm endpoint kiểm thử. Khi người dùng tải và giải nén tệp kiểm thử EICAR (tệp kiểm thử mã độc giả lập, vô hại, được dùng để kiểm tra khả năng phát hiện của phần mềm chống mã độc mà không cần sử dụng malware thật), tiến trình unzip tạo ra tệp eicar.com trên endpoint. Elastic Defend quan sát hoạt động này thông qua endpoint telemetry, phát hiện tệp EICAR và sinh cảnh báo Malware Prevention Alert trên Kibana.

Sau khi cảnh báo được tạo, hệ thống thực hiện hai hướng phản hồi. Thứ nhất là phản hồi thủ công, trong đó quản trị viên mở alert trên Kibana và chọn hành động Isolate host để cô lập máy user. Thứ hai là phản hồi tự động, trong đó detection rule được cấu hình kèm Elastic Defend response action. Khi rule phát hiện sự kiện phù hợp, hệ thống tự động gửi lệnh isolate đến host liên quan mà không cần thao tác thủ công.

### 4.3.5. Dashboard

Kịch bản dashboard được xây dựng nhằm minh họa khả năng thống kê và trực quan hóa dữ liệu trên Kibana. Thay vì chỉ quan sát từng log riêng lẻ trong Discover, nhóm tạo dashboard để tổng hợp dữ liệu endpoint theo thời gian và theo nguồn phát sinh.

Nguồn dữ liệu được sử dụng là data view `logs-*` kết hợp bộ lọc KQL `event.module: "endpoint"`. Bộ lọc này giúp giới hạn dữ liệu vào các sự kiện endpoint do Elastic Defend thu thập, bao gồm sự kiện tiến trình, tệp tin, mạng và bảo mật. Từ nguồn dữ liệu này, nhóm xây dựng biểu đồ Stacked Bar để thống kê số lượng sự kiện theo ngày và theo host, đồng thời xây dựng biểu đồ Pie để thể hiện tỷ lệ sự kiện theo từng host.

### 4.3.6. Machine Learning

Trong kịch bản Machine Learning, tạo script giả lập an toàn để tạo dữ liệu endpoint phục vụ quá trình học baseline và phát hiện bất thường. Kịch bản Machine Learning được xây dựng nhằm minh họa khả năng phát hiện bất thường của Elastic ML dựa trên dữ liệu endpoint telemetry thu thập từ máy trạm người dùng. Trong kịch bản này, lựa chọn host user làm endpoint kiểm thử chính. Giai đoạn đầu, script được chạy ở chế độ baseline ở host user để tạo số lượng file nhỏ theo từng vòng, có độ dao động ngẫu nhiên nhằm mô phỏng hành vi bình thường của endpoint. Elastic Defend ghi nhận các thao tác này thành endpoint events và gửi về Elasticsearch.

Sau khi có dữ liệu baseline, nhóm cấu hình Elastic ML job trên data view `endpoint-events`, sử dụng metric Count (Event rate) để theo dõi số lượng sự kiện endpoint theo thời gian. Job được cấu hình phân tích theo `host.name` nhằm xác định host phát sinh bất thường. Tiếp theo, script được chạy ở chế độ anomaly để tạo số lượng lớn file trong thời gian ngắn. Sự tăng đột biến này làm số lượng endpoint events vượt khỏi baseline đã học, từ đó Elastic ML có thể phát hiện và hiển thị điểm bất thường trong Anomaly Explorer.

### 4.3.7. Kịch bản bổ sung: Đo hiệu suất thu thập log của ELK Stack

Kịch bản này được xây dựng trong môi trường mạng giả lập nhằm đánh giá hiệu năng thu thập và lập chỉ mục của hệ thống ELK Stack dưới tải log thực tế. Để đo lường chỉ số này, hệ thống sẽ tiến hành sinh nhật ký giả lập (dummy log) liên tục và đồng thời từ ba node Ubuntu trong hạ tầng, bao gồm máy chủ Web (VLAN 10), máy trạm người dùng (VLAN 20) và máy chủ cơ sở dữ liệu (VLAN 30).

Để đảm bảo tính đồng nhất và dễ theo dõi, toàn bộ ba node sẽ sinh ra cùng một định dạng dummy log tùy chỉnh và gửi liên tục về SIEM mà không ngắt quãng. Định dạng này được thiết kế để tương thích với integration System của Elastic Agent, cho phép thu thập mà không cần cấu hình thêm pipeline hay bộ phân tích cú pháp riêng ở giai đoạn này. Elastic Agent đã được triển khai trên từng node sẽ đọc và chuyển tiếp liên tục các bản ghi về Fleet Server trên máy SIEM.

Toàn bộ luồng log sẽ được quan sát trực tiếp trên Kibana trong khoảng thời gian 5 phút nhằm ghi nhận tổng số bản ghi thu được và tốc độ xử lý thực tế của máy SIEM (8 GB RAM, 4 nhân xử lý, 80 GB lưu trữ). Kết quả đo lường này sẽ là cơ sở để chuyển sang giai đoạn tiếp theo, trong đó ingest pipeline của các agent sẽ được cấu hình nhằm làm giàu dummy log theo các trường thông tin tùy chỉnh.

### 4.3.8. Kịch bản bổ sung: Làm giàu custom log

Kịch bản này minh họa khả năng xử lý và chuẩn hóa các loại log chưa được integration hỗ trợ sẵn của hệ thống ELK Stack. Thay vì phụ thuộc vào các bộ phân tích cú pháp có sẵn, hệ thống sẽ tự cấu hình ingest pipeline trực tiếp trên Kibana để phân tích, trích xuất và bổ sung thêm các trường thông tin tùy chỉnh cho dummy log đã được thu thập ở kịch bản trước. Qua đó minh họa tính linh hoạt của ELK Stack trong việc tiếp nhận và xử lý bất kỳ định dạng log nào phát sinh trong thực tế vận hành.

## 4.4. Cấu hình kịch bản

### 4.4.1. Thu thập log, chuẩn hóa log và tìm kiếm dễ dàng

**Cấu hình thu thập log từ pfSense (qua Syslog)**

Do pfSense chạy trên nền FreeBSD nên không thể cài đặt Elastic Agent trực tiếp. Thay vào đó, pfSense được cấu hình để chuyển tiếp log qua giao thức Syslog đến máy SIEM, máy chủ đồng thời chạy ELK Stack và Fleet Server. Trên máy SIEM, Elastic Agent đã được cài đặt sẵn với các Integration Fleet Server và System. Để thu nhận log từ pfSense, Integration pfSense được bổ sung vào policy của agent này, cho phép lắng nghe và tiếp nhận luồng Syslog từ pfSense gửi về, sau đó gửi dữ liệu về Elasticsearch.

![Hình 4.2. Cấu hình Syslog trên pfSense.](/images/i42.png)

Hình 4.2. Cấu hình Syslog trên pfSense.

![Hình 4.3. Cấu hình Syslog trên pfSense.](/images/i43a.png)

![Hình 4.3. Cấu hình Syslog trên pfSense (tiếp theo).](/images/i43b.png)

Hình 4.3. Cấu hình Syslog trên pfSense.

![Hình 4.4. Integration thu nhận Syslog trên Elastic Agent tại máy SIEM.](/images/i44.png)

Hình 4.4. Integration thu nhận Syslog trên Elastic Agent tại máy SIEM.

Để sinh ra các mẫu log phục vụ kịch bản, hai firewall rule được thiết lập trên pfSense với tùy chọn ghi log được bật:

- **Rule chặn Ping (ICMP):** Chặn lưu lượng ICMP từ máy user Ubuntu Desktop hướng đến máy Web Server, đảm bảo mỗi gói tin bị chặn đều được ghi nhận vào firewall filter log.
- **Rule chặn Port Scanning:** Chặn và ghi nhận các nỗ lực rà quét cổng xuất phát từ máy user.

![Hình 4.5. Rule chặn ICMP Ping từ user đến Web Server và Port Scanning trên pfSense.](/images/i45.png)

Hình 4.5. Rule chặn ICMP Ping từ user đến Web Server và Port Scanning trên pfSense.

**Cấu hình thu thập log từ Web Server (Apache)**

Trên máy Ubuntu Server chạy dịch vụ Apache và ứng dụng DVWA, Elastic Agent được triển khai với integration Apache HTTP Server. Integration này được cấu hình đọc hai tệp nhật ký chính:

- Access log: `/var/log/apache2/access.log`
- Error log: `/var/log/apache2/error.log`

![Hình 4.6. Cấu hình integration Apache trên Elastic Agent với đường dẫn tệp access.log.](/images/i46.png)

Hình 4.6. Cấu hình integration Apache trên Elastic Agent với đường dẫn tệp access.log.

**Chuẩn hóa và tìm kiếm log**

Quá trình chuẩn hóa log được thực hiện tự động thông qua Ingest Pipeline tích hợp sẵn trong Elasticsearch, kích hoạt ngay khi dữ liệu được gửi về từ các integration. Khả năng tìm kiếm và truy vấn thông qua Kibana Discover cũng là tính năng có sẵn của ELK Stack. Do đó, cả hai kịch bản này không yêu cầu cấu hình bổ sung ngoài hạ tầng Elasticsearch và Kibana đã được triển khai ở mục 4.2.

### 4.4.2. Cấu hình phát hiện và phản hồi

Tiến hành cấu hình Elastic Defend Overview, thể hiện thông tin tổng quan về integration Elastic Defend được dùng để bảo vệ endpoint.

![Hình 4.7. Giao diện tổng quan Elastic Defend integration.](/images/i47.png)

Hình 4.7. Giao diện tổng quan Elastic Defend integration.

Sử dụng rule mặc định Endpoint Security (Elastic Defend) vì rule này được Elastic cung cấp sẵn để tiếp nhận và hiển thị các cảnh báo do Elastic Defend sinh ra từ endpoint. Khi Elastic Defend phát hiện các sự kiện bảo mật trên endpoint, rule này sẽ tiếp nhận và hiển thị cảnh báo tương ứng trên Kibana, đồng thời hỗ trợ thực hiện các hành động phản hồi như điều tra, cô lập host hoặc xử lý endpoint liên quan.

![Hình 4.8. Detection rule mặc định Endpoint Security (Elastic Defend).](/images/i48.png)

Hình 4.8. Detection rule mặc định Endpoint Security (Elastic Defend).

Cấu hình Elastic Defend cho các Endpoints trong Fleet, dùng để triển khai chính sách bảo vệ cho máy endpoints.

![Hình 4.9. Cấu hình Elastic Defend.](/images/i49.png)

Hình 4.9. Cấu hình Elastic Defend.

Thiết lập Malware protections ở chế độ Prevent, bật Blocklist và Scan files upon modification để phát hiện và ngăn chặn tệp nghi mã độc trên endpoint.

![Hình 4.10. Thiết lập Malware protections.](/images/i410.png)

Hình 4.10. Thiết lập Malware protections.

Khi thử phần response tự động thì sẽ thêm action khi detection rule phát hiện sự kiện phù hợp, hệ thống sẽ tự động thực hiện hành động isolate để cô lập endpoint liên quan khỏi mạng.

![Hình 4.11. Cấu hình response action tự động.](/images/i411.png)

Hình 4.11. Cấu hình response action tự động.

Kiểm tra danh sách Elastic Defend integration policies đã được gán cho các agent policy tương ứng với các máy web, database và user.

![Hình 4.12. Danh sách Elastic Defend integration policies được gán cho các agent policy.](/images/i412.png)

Hình 4.12. Danh sách Elastic Defend integration policies được gán cho các agent policy.

Trang Security Alerts trước khi kiểm thử EDR, chưa ghi nhận cảnh báo.

![Hình 4.13. Trang Security Alerts trước khi thực hiện kịch bản EDR.](/images/i413.png)

Hình 4.13. Trang Security Alerts trước khi thực hiện kịch bản EDR.

Tải tệp kiểm thử anti-malware dùng cho kịch bản EDR ở trang chính thức của EICAR.

![Hình 4.14. Trang tải tệp kiểm thử EICAR.](/images/i414.png)

Hình 4.14. Trang tải tệp kiểm thử EICAR.

Tải EICAR test file, trong đó nhóm lựa chọn tệp EICAR.COM.ZIP để kiểm thử khả năng phát hiện của Elastic Defend.

![Hình 4.15. Lựa chọn tệp EICAR.COM.ZIP.](/images/i415.png)

Hình 4.15. Lựa chọn tệp EICAR.COM.ZIP.

### 4.4.3. Cấu hình dashboard

Truy cập vào mục Security > Dashboards và tạo một dashboard mới. Giao diện ban đầu của dashboard chưa có panel trực quan hóa, do đó cần chọn chức năng Create visualization để bắt đầu tạo biểu đồ. Đây là bước khởi tạo không gian dashboard dùng để thêm các biểu đồ thống kê phục vụ quá trình giám sát.

![Hình 4.16. Giao diện tạo dashboard mới trên Kibana.](/images/i416.png)

Hình 4.16. Giao diện tạo dashboard mới trên Kibana.

Sau khi vào giao diện tạo visualization, hệ thống sử dụng công cụ trực quan hóa Kibana Lens để xây dựng biểu đồ.

![Hình 4.17. Giao diện Kibana Lens.](/images/i417.png)

Hình 4.17. Giao diện Kibana Lens.

Data view được chọn là `logs-*`, vì đây là nơi lưu trữ các log và sự kiện được Elastic Agent gửi về Elasticsearch. Để chỉ thống kê các sự kiện endpoint, bộ lọc Kibana Query Language được cấu hình như sau: `event.module: "endpoint"`. Bộ lọc này giúp dashboard chỉ lấy các sự kiện do Elastic Defend thu thập từ endpoint, chẳng hạn sự kiện tiến trình, tệp tin, mạng và bảo mật. Nhờ đó, dữ liệu hiển thị trên dashboard phản ánh đúng hoạt động của các endpoint thay vì trộn lẫn với toàn bộ log hệ thống.

![Hình 4.18. Cấu hình data view và bộ lọc KQL.](/images/i418.png)

Hình 4.18. Cấu hình data view và bộ lọc KQL.

Biểu đồ đầu tiên được cấu hình dưới dạng Bar chart với chế độ Stacked. Trường `@timestamp` được đặt ở trục ngang để gom nhóm sự kiện theo thời gian, trong khi trục dọc sử dụng Count of records để đếm số lượng sự kiện. Ngoài ra, trường `host.name` được thêm vào phần Breakdown để phân tách số lượng sự kiện theo từng máy. Với cách cấu hình này, mỗi cột biểu diễn tổng số sự kiện endpoint trong một ngày, còn từng phần màu trong cột tương ứng với các host như user, web hoặc database.

![Hình 4.19. Cấu hình biểu đồ Stacked Bar.](/images/i419.png)

Hình 4.19. Cấu hình biểu đồ Stacked Bar.

Biểu đồ thứ hai được cấu hình dưới dạng Pie chart nhằm thể hiện tỷ lệ sự kiện endpoint theo từng host. Trong biểu đồ này, trường `host.name` được sử dụng ở phần Slice by, còn chỉ số thống kê là Count of records. Biểu đồ Pie giúp người quản trị nhanh chóng xác định host nào phát sinh nhiều sự kiện endpoint nhất trong khoảng thời gian quan sát.

![Hình 4.20. Cấu hình biểu đồ Pie.](/images/i420.png)

Hình 4.20. Cấu hình biểu đồ Pie.

### 4.4.4. Cấu hình Machine Learning

Sử dụng tính năng Anomaly Detection của Elastic Machine Learning để phát hiện sự tăng đột biến số lượng sự kiện tệp tin trên endpoint.

![Hình 4.21. Giao diện khởi tạo Anomaly Detection Job trong Kibana.](/images/i421.png)

Hình 4.21. Giao diện khởi tạo Anomaly Detection Job trong Kibana.

Tạo một data view riêng có tên `endpoint-events` trong Kibana. Data view này sử dụng index pattern `logs-endpoint.events.*-*`, tương ứng với các data stream chứa sự kiện endpoint do Elastic Defend thu thập. Index pattern này khớp với 3 nguồn dữ liệu gồm `logs-endpoint.events.file-default`, `logs-endpoint.events.network-default` và `logs-endpoint.events.process-default`.

Việc tạo data view riêng giúp giới hạn phạm vi phân tích vào các sự kiện endpoint như sự kiện tệp tin, tiến trình và mạng, thay vì trộn lẫn với toàn bộ log hệ thống trong `logs-*`. Trường thời gian được chọn là `@timestamp`, cho phép Kibana và Elastic ML phân tích dữ liệu theo chuỗi thời gian. Đây là bước cần thiết trước khi tạo ML job, vì job phát hiện bất thường cần một nguồn dữ liệu có time field rõ ràng để học baseline và đánh giá sự thay đổi số lượng sự kiện theo từng khoảng thời gian.

![Hình 4.22. Tạo data view endpoint-events cho Machine Learning.](/images/i422.png)

Hình 4.22. Tạo data view endpoint-events cho Machine Learning.

Nhóm xây dựng script `elastic_ml_endpoint_simulator.sh` để tạo dữ liệu giả lập phục vụ kịch bản Machine Learning trên host user. Script có hai chế độ hoạt động chính. Chế độ baseline tạo số lượng file nhỏ theo từng vòng, có khoảng nghỉ và độ dao động ngẫu nhiên nhằm mô phỏng hành vi bình thường của endpoint. Chế độ anomaly tạo số lượng lớn file trong thời gian ngắn nhằm tạo ra sự tăng đột biến số lượng endpoint file events. Các file được tạo trong thư mục `/tmp/elastic-ml-baseline` và `/tmp/elastic-ml-anomaly`, giúp nhóm dễ dàng lọc dữ liệu bằng trường `file.path` trên Kibana Discover. Script không thực hiện hành vi nguy hiểm, không can thiệp file hệ thống và chỉ dùng để tạo dữ liệu phục vụ kiểm thử. Trước khi tạo dữ liệu bất thường, chạy script ở chế độ baseline nhằm tạo ra tập dữ liệu hành vi bình thường cho endpoint. Giai đoạn baseline đóng vai trò là dữ liệu học ban đầu để Elastic Machine Learning xác định mức hoạt động thông thường của máy user, cụ thể là số lượng file events phát sinh theo thời gian.

Lệnh chạy baseline được sử dụng như sau:

```bash
./elastic_ml_endpoint_simulator.sh --mode baseline --rounds 61 \
  --min-files-per-round 10 --max-files-per-round 30 --sleep 60 --sleep-jitter 15
```

Trong lệnh trên, tham số `--mode baseline` cho biết script được chạy ở chế độ tạo dữ liệu bình thường. Tham số `--rounds 61` quy định script thực hiện 61 vòng tạo file, tương ứng với khoảng thời gian gần một giờ. Ở mỗi vòng, số lượng file được tạo ngẫu nhiên trong khoảng từ 10 đến 30 file, được cấu hình thông qua `--min-files-per-round 10` và `--max-files-per-round 30`. Tham số `--sleep 60` yêu cầu script nghỉ khoảng 60 giây giữa các vòng, còn `--sleep-jitter 15` bổ sung độ dao động ngẫu nhiên khoảng 15 giây để dữ liệu sinh ra không hoàn toàn đều tuyệt đối. Cách cấu hình này giúp mô phỏng hành vi bình thường của endpoint theo thời gian, tạo ra dữ liệu baseline phù hợp để Elastic Machine Learning học trước khi chạy kịch bản anomaly.

![Hình 4.23. Chạy script ở chế độ baseline trên máy user.](/images/i423.png)

Hình 4.23. Chạy script ở chế độ baseline trên máy user.

Trong Discover, chọn data view `endpoint-events` và sử dụng bộ lọc KQL để chỉ lấy các sự kiện tệp tin do script giả lập tạo ra: `event.category: "file" and file.path: *elastic-ml*`. Khi kết quả truy vấn hiển thị các đường dẫn như `/tmp/elastic-ml-baseline/...`, có thể xác nhận Elastic Defend đã ghi nhận đúng file events từ máy user.

![Hình 4.24. Kiểm tra dữ liệu baseline trong Discover.](/images/i424.png)

Hình 4.24. Kiểm tra dữ liệu baseline trong Discover.

Sau khi xác nhận dữ liệu đúng, lưu truy vấn Discover thành một saved Discover session để ML job sử dụng đúng tập dữ liệu đã lọc thay vì phân tích toàn bộ endpoint events.

![Hình 4.25. Lưu Discover session cho dữ liệu Machine Learning.](/images/i425.png)

Hình 4.25. Lưu Discover session cho dữ liệu Machine Learning.

Tạo job từ saved Discover session vừa lưu. Loại job được chọn là Multi-metric nhằm phát hiện bất thường dựa trên số lượng sự kiện theo thời gian.

![Hình 4.26. Lựa chọn khoảng thời gian baseline cho ML job.](/images/i426.png)

Hình 4.26. Lựa chọn khoảng thời gian baseline cho ML job.

ML job được cấu hình với detector Count (Event rate), bucket span 5 phút, influencer là `host.name` và không cấu hình split field do kịch bản chỉ tập trung trên máy user. Metric Count (Event rate) cho phép hệ thống đếm số lượng file events trong từng khoảng thời gian và học giá trị bình thường của chuỗi thời gian này.

![Hình 4.27. Cấu hình metric Count (Event rate) cho ML job.](/images/i427.png)

Hình 4.27. Cấu hình metric Count (Event rate) cho ML job.

Ở bước Job details, đặt Job ID là `endpoint_file_event_user`, group là `endpoint-ml` và mô tả là "Detect anomalies in simulated endpoint file events on host user".

![Hình 4.28. Cấu hình thông tin chi tiết cho ML job.](/images/i428.png)

Hình 4.28. Cấu hình thông tin chi tiết cho ML job.

Sau khi kiểm tra cấu hình ở bước Validation và Summary, job được tạo và xử lý dữ liệu baseline ngay lập tức.

![Hình 4.29. Tổng quan cấu hình ML job trước khi khởi tạo.](/images/i429.png)

Hình 4.29. Tổng quan cấu hình ML job trước khi khởi tạo.

Sau khi tạo job, Elastic Machine Learning tiến hành xử lý dữ liệu baseline trong khoảng thời gian đã chọn. Machine Learning job đã học xong hành vi bình thường của endpoint, sẵn sàng được mở lại datafeed để theo dõi dữ liệu mới và phát hiện bất thường trong giai đoạn chạy anomaly.

![Hình 4.30. Trạng thái ML job sau khi xử lý dữ liệu baseline.](/images/i430.png)

Hình 4.30. Trạng thái ML job sau khi xử lý dữ liệu baseline.

### 4.4.5. Cấu hình kịch bản đo hiệu năng và kịch bản làm giàu custom log

**Thu thập log**

*Thiết kế định dạng dummy log*

Trước tiên, cần thiết kế định dạng dummy log tùy chỉnh thống nhất cho cả ba node. Định dạng được lựa chọn dựa trên hai tiêu chí: tương thích với integration System của Elastic Agent để không cần cấu hình thêm bộ phân tích cú pháp ở giai đoạn thu thập, và chứa đủ các trường thông tin có ý nghĩa để phục vụ cho kịch bản làm giàu log ở giai đoạn sau. Định dạng dummy log được thiết kế như sau:

- `level`: mức độ của sự kiện, nhận một trong ba giá trị là INFO (hoạt động bình thường), WARN (cảnh báo cần chú ý) và ERROR (lỗi nghiêm trọng).
- `host`: định danh của máy sinh log, nhận một trong ba giá trị là web-server (máy chủ Web), user-station (máy trạm người dùng) và db-server (máy chủ cơ sở dữ liệu).
- `msg`: nội dung mô tả sự kiện, là chuỗi văn bản tự do mô tả hành vi hoặc trạng thái hệ thống tại thời điểm ghi log.
- `code`: mã trạng thái đi kèm sự kiện, nhận một trong ba giá trị là 200 (thành công), 401 (xác thực thất bại) và 503 (dịch vụ không khả dụng).

Định dạng dummy log mẫu:

```
DUMMY level=INFO host=web-server msg="Service health check OK" code=200
DUMMY level=WARN host=web-server msg="High CPU usage detected" code=503
DUMMY level=ERROR host=web-server msg="Authentication failed for user admin" code=401
```

*Triển khai sinh log trên ba node*

Sau khi có định dạng thống nhất, tiến hành cấu hình script sinh log trên từng node. Script được thiết kế để chạy liên tục dưới dạng một systemd service, đảm bảo dummy log được gửi về SIEM không ngắt quãng trong suốt quá trình đo lường.

Script tạo log cho các máy (giá trị `xxx` của trường Host đại diện cho một trong ba giá trị là web-server, user-station và db-server):

```bash
#!/bin/bash

HOST="xxx"

LEVELS=("INFO" "WARN" "ERROR")
MESSAGES=(
    "Service health check OK|200"
    "High CPU usage detected|503"
    "Authentication failed for user admin|401"
    "File accessed /etc/passwd|200"
    "Network connection established|200"
    "Disk I/O threshold reached|503"
    "Configuration reload triggered|200"
    "Unauthorized access attempt|401"
)

while true; do
    LEVEL="${LEVELS[$RANDOM % ${#LEVELS[@]}]}"
    ENTRY="${MESSAGES[$RANDOM % ${#MESSAGES[@]}]}"
    MSG="${ENTRY%|*}"
    CODE="${ENTRY#*|}"
    logger -t DUMMY "level=$LEVEL host=$HOST msg=\"$MSG\" code=$CODE"
    sleep 0.01
done
```

Do quá trình cấu hình tạo service trên cả ba máy là như nhau, báo cáo sẽ trình bày đại diện các bước thực hiện trên máy user.

- **Bước 1: Tạo script**

![Hình 4.31. Script của dummy-log-gen.sh trên máy user.](/images/i431.png)

Hình 4.31. Script của dummy-log-gen.sh trên máy user.

- **Bước 2: Tạo systemd service**

![Hình 4.32. Sử dụng lệnh tạo service dummy-log-gen.service.](/images/i432.png)

Hình 4.32. Sử dụng lệnh tạo service dummy-log-gen.service.

- **Bước 3: Kiểm tra trạng thái của service đã tạo**

![Hình 4.33. Status của service dummy-log-gen.service.](/images/i433.png)

Hình 4.33. Status của service dummy-log-gen.service.

- **Bước 4: Lặp lại các bước trên cho hai máy còn lại**

**Cấu hình làm giàu log**

Tiến hành cấu hình ingest pipeline trực tiếp trên Kibana cho từng node nhằm phân tích, trích xuất và bổ sung các trường thông tin tùy chỉnh từ dummy log đã thu thập. Pipeline này sẽ được áp dụng đồng nhất cho cả ba nguồn log.

Cấu trúc custom ingest pipeline:

```json
{
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": [
          "DUMMY level=%{WORD:dummy.level} host=%{DATA:dummy.host} msg=\"%{DATA:dummy.msg}\" code=%{NUMBER:dummy.code:int}"
        ]
      }
    },
    {
      "script": {
        "lang": "painless",
        "description": "Map host to source IP",
        "source": "Map ipMap = new HashMap(); ipMap.put('web-server', '192.168.10.2'); ipMap.put('user-station', '192.168.20.2'); ipMap.put('db-server', '192.168.30.2'); if (ctx.dummy?.host != null && ipMap.containsKey(ctx.dummy.host)) { ctx.dummy.ip = ipMap.get(ctx.dummy.host); }"
      }
    },
    {
      "script": {
        "lang": "painless",
        "description": "Map level to numeric severity",
        "source": "Map sevMap = new HashMap(); sevMap.put('INFO', 1); sevMap.put('WARN', 2); sevMap.put('ERROR', 3); if (ctx.dummy?.level != null && sevMap.containsKey(ctx.dummy.level)) { ctx.dummy.severity = sevMap.get(ctx.dummy.level); }"
      }
    },
    {
      "set": {
        "field": "dummy.pipeline_processed",
        "value": true
      }
    },
    {
      "set": {
        "field": "event.dataset",
        "value": "dummy.log"
      }
    }
  ]
}
```

- **Bước 1: Truy cập giao diện và thực hiện tạo ingest pipeline**

![Hình 4.34. Danh sách các Ingest Pipelines trên Kibana.](/images/i434.png)

Hình 4.34. Danh sách các Ingest Pipelines trên Kibana.

![Hình 4.35. Giao diện khi tạo Ingest Pipeline.](/images/i435.png)

Hình 4.35. Giao diện khi tạo Ingest Pipeline.

![Hình 4.36. Tạo logs-system.syslog@custom ingest pipeline.](/images/i436.png)

Hình 4.36. Tạo logs-system.syslog@custom ingest pipeline.

![Hình 4.37. Processors của logs-system.syslog@custom ingest pipeline.](/images/i437.png)

Hình 4.37. Processors của logs-system.syslog@custom ingest pipeline.

![Hình 4.38. Ingest pipelines logs-system.syslog@custom được thêm vào policy.](/images/i438.png)

Hình 4.38. Ingest pipelines logs-system.syslog@custom được thêm vào policy.

- **Bước 2: Làm tương tự cho hai máy còn lại**
