# Chương 5. TRIỂN KHAI VÀ ĐÁNH GIÁ KẾT QUẢ

## 5.1. Thực nghiệm các kịch bản

### 5.1.1. Thu thập log

**Thực hiện ping từ máy user đến máy web**

Từ máy user (192.168.20.2), thực hiện lệnh ping đến máy web (192.168.10.2) nhằm kích hoạt rule chặn ICMP trên pfSense và sinh ra log tường lửa tương ứng.

![Hình 5.1. Thực hiện lệnh ping từ máy user đến máy web.](/images/i51.png)

**Thực hiện port scanning từ máy user**

Từ máy user, sử dụng công cụ nmap để rà quét các cổng mạng đang mở trên máy web, kích hoạt rule phát hiện port scanning trên pfSense.

![Hình 5.2. Thực hiện port scanning từ máy user bằng nmap.](/images/i52.png)

**Kiểm tra log tường lửa trên Kibana**

Sau khi hai hành động trên được thực hiện, truy cập giao diện Discover trên Kibana để xác nhận log đã được thu thập thành công từ pfSense về hệ thống SIEM.

![Hình 5.3. Log ghi nhận gói tin ICMP Ping bị chặn trên Kibana.](/images/i53.png)

![Hình 5.4. Log ghi nhận nỗ lực port scanning bị chặn trên Kibana.](/images/i54.png)

**Thực hiện tấn công SQL Injection từ máy user**

Từ máy user, truy cập ứng dụng DVWA trên máy web và thực hiện tấn công SQL Injection vào form đăng nhập hoặc trang tìm kiếm người dùng ở mức bảo mật Low, nhằm sinh ra dấu vết tấn công trong access.log của Apache.

![Hình 5.5. Thực hiện tấn công SQL Injection trên ứng dụng DVWA.](/images/i55.png)

**Thực hiện dò quét thư mục trên máy web**

Từ máy user, nhập vào đường dẫn url payload để gửi request truy cập các file có trên máy web, tạo ra chuỗi phản hồi lỗi HTTP 404 và 403 trong access.log và error.log của Apache.

![Hình 5.6. Thực hiện dò quét thư mục thất bại trên máy web.](/images/i56a.png)

![Hình 5.6. Thực hiện dò quét thư mục thành công trên máy web.](/images/i56b.png)

**Kiểm tra log Apache trên Kibana**

Truy cập Kibana để xác nhận các log tấn công đã được Elastic Agent thu thập từ máy web và đưa vào hệ thống.

![Hình 5.7. Log SQL Injection xuất hiện trong access.log trên Kibana.](/images/i57.png)

![Hình 5.8. Log dò quét thư mục trong access.log trên Kibana.](/images/i58.png)

![Hình 5.9. Log dò quét thư mục trong error.log trên Kibana.](/images/i59.png)

**Đánh giá kịch bản**

Kịch bản đã minh họa thành công khả năng thu thập log đa nguồn của hệ thống ELK Stack. Toàn bộ 4 mẫu log từ hai nguồn khác nhau, bao gồm log tường lửa từ pfSense qua cơ chế Syslog gián tiếp và log ứng dụng từ Apache qua Elastic Agent trực tiếp, đều được thu thập và hiển thị tập trung trên Kibana. Điều này xác nhận hệ thống có khả năng tổng hợp dữ liệu từ cả lớp mạng lẫn lớp ứng dụng trong một nền tảng giám sát duy nhất.

---

### 5.1.2. Tự động làm giàu log

**Đối chiếu log thô và log đã làm giàu**

Để minh họa vai trò của Ingest Pipeline, tiến hành đối chiếu giữa log thô nhận được từ nguồn và log đã được xử lý hiển thị trên Kibana Discover, dựa trên 4 mẫu log thu thập được từ kịch bản 5.1.1.

**Mẫu 1 (Chặn Ping):** Đây là một chuỗi Syslog thô dạng CSV, không có cấu trúc rõ ràng. Ingest Pipeline của integration pfSense đã phân tách chuỗi này thành các trường có nghĩa theo chuẩn ECS: `source.ip` (192.168.20.2), `destination.ip` (192.168.10.2), `network.transport` (icmp), `event.action` (block), `pfsense.icmp.type` (request), `observer.ingress.interface.name` (em1.20), `network.vlan.id` (20). Nhờ đó người quản trị có thể truy vấn trực tiếp theo từng trường thay vì phải đọc và phân tích chuỗi CSV thủ công.

![Hình 5.10. Log thô pfSense ghi nhận gói tin ICMP bị chặn.](/images/i510.png)

![Hình 5.11. Log đã được làm giàu và phân tách trường trên Kibana.](/images/i511.png)

**Mẫu 2 (Port Scanning):** Tương tự Mẫu 1, pipeline pfSense đã nhận diện đây là gói TCP và phân tách thêm các trường đặc thù của TCP: `pfsense.tcp.flags` (S), `pfsense.tcp.window` (1024), `destination.port` (1094), `source.port` (38169). Đặc biệt, `pfsense.tcp.flags: S` cho thấy đây là gói SYN đặc trưng của nmap SYN scan, và `pfsense.tcp.window: 1024` là dấu hiệu nhận dạng nmap. Những trường này không tồn tại trong log thô mà được pipeline tự động trích xuất và ánh xạ về chuẩn ECS.

![Hình 5.12. Log thô pfSense ghi nhận nỗ lực port scanning.](/images/i512.png)

![Hình 5.13. Log đã được làm giàu và phân tách trường trên Kibana.](/images/i513.png)

**Mẫu 3 (SQL Injection):** Pipeline của integration Apache đã phân tách chuỗi Combined Log Format này thành các trường riêng biệt: `source.ip` (192.168.20.2), `http.request.method` (GET), `http.response.status_code` (200), `http.response.body.bytes` (1870), `user_agent.name` (Firefox). Đặc biệt, pipeline thực hiện thêm hai bước xử lý quan trọng với URL. Thứ nhất, pipeline giải mã URL encoding trong `url.original`, chuyển `1%27+OR+%271%27%3D%271` thành `id=1'+OR+'1'='1` trong trường `url.query`, giúp payload SQL Injection hiện ra rõ ràng và có thể tìm kiếm trực tiếp. Thứ hai, pipeline tách riêng `url.path` (`/DVWA/vulnerabilities/sqli/`) và `url.query` thành hai trường độc lập, cho phép lọc chính xác theo đường dẫn hoặc theo nội dung tham số. Ngoài ra `http.request.referrer` cũng được trích xuất, cho thấy request xuất phát từ trang DVWA trên địa chỉ 192.168.100.2:8080. Pipeline cũng tự động gán `event.outcome: success` dựa trên mã HTTP 200, xác nhận tấn công SQL Injection đã thực hiện thành công.

![Hình 5.14. Log thô Apache access.log ghi nhận payload SQL Injection.](/images/i514.png)

![Hình 5.15. Log đã được làm giàu và phân tách trường trên Kibana.](/images/i515.png)

**Mẫu 4.1 (Path traversal thành công):** Pipeline của integration Apache đã phân tách định dạng Combined Log Format này thành các trường riêng biệt: `source.ip` (192.168.20.2), `http.request.method` (GET), `url.original` (`/DVWA/vulnerabilities/fi/?page=../../../../../../etc/passwd`), `url.query` (`page=../../../../../../etc/passwd`), `http.response.status_code` (200), `http.response.body.bytes` (2127), `user_agent.name` (Firefox). Ngoài ra pipeline còn tự động xác định `event.outcome` là success dựa trên mã HTTP 200, giúp người quản trị có thể lọc ngay các tấn công thành công mà không cần đọc từng dòng log.

![Hình 5.16. Log thô Apache access.log ghi nhận path traversal.](/images/i516.png)

![Hình 5.17. Log của access.log đã được làm giàu và phân tách trường trên Kibana.](/images/i517.png)

**Mẫu 4.2 (Path traversal thất bại):** Pipeline của integration Apache xử lý định dạng error log khác hoàn toàn so với access log. Từ chuỗi thô này, pipeline trích xuất được: `source.ip` (192.168.20.2), `source.port` (35052), `log.level` (warn), `apache.error.module` (php), `event.type` (error), và giữ nguyên nội dung cảnh báo trong trường `message`. Việc tách `log.level` thành trường riêng cho phép người quản trị lọc nhanh tất cả cảnh báo mức warn hoặc error trên toàn hệ thống mà không cần tìm kiếm theo từ khóa trong chuỗi thô.

![Hình 5.18. Log thô Apache error.log ghi nhận path traversal.](/images/i518.png)

![Hình 5.19. Log của error.log đã được làm giàu và phân tách trường trên Kibana.](/images/i519.png)

**Đánh giá kịch bản**

Kịch bản đã minh họa rõ vai trò của Ingest Pipeline trong việc tự động chuẩn hóa dữ liệu log về chuẩn ECS ngay khi dữ liệu được nhận vào Elasticsearch. Thay vì một chuỗi ký tự thô khó đọc, người quản trị có thể trực tiếp quan sát các trường thông tin có cấu trúc như địa chỉ IP nguồn, cổng đích, phương thức HTTP, mã phản hồi và thông tin host ngay trên giao diện Kibana mà không cần xử lý thêm.

---

### 5.1.3. Search filter

Hệ thống SIEM sử dụng các cú pháp ngôn ngữ KQL kết hợp với bộ lọc thời gian để cô lập chính xác các hành vi tấn công giữa hàng triệu dòng log hệ thống. Dưới đây là chi tiết các câu lệnh truy vấn và ý nghĩa thực tế được áp dụng cho từng kịch bản cụ thể:

**Truy vấn Mẫu 1: Phát hiện hành vi Ping bị chặn bởi Tường lửa**

```
event.dataset: "pfsense.log" and network.transport: "icmp" and event.action: "block"
```

![Hình 5.20. Kết quả truy vấn log ICMP Ping bị chặn trên Kibana Discover.](/images/i520.png)

Ý nghĩa các trường dữ liệu:
- Trường `event.dataset` xác định giá trị `pfsense.log` để định hướng hệ thống SIEM chỉ tìm kiếm trong nguồn dữ liệu tập trung luân chuyển từ thiết bị tường lửa pfSense.
- Trường `network.transport` xác định giá trị `icmp` để lọc riêng giao thức ICMP, đây là giao thức mà lệnh ping sử dụng.
- Trường `event.action` xác định giá trị `block` nhằm trích xuất chính xác các gói tin đã bị luật của tường lửa từ chối và thực hiện hành động chặn.

**Truy vấn Mẫu 2: Phát hiện hành vi Dò quét cổng mạng**

```
event.dataset: "pfsense.log" and pfsense.tcp.flags: "S" and event.action: "block"
```

![Hình 5.21. Kết quả truy vấn log port scanning trên Kibana Discover.](/images/i521.png)

Ý nghĩa các trường dữ liệu:
- Kỹ thuật rà quét cổng phổ biến nhất của công cụ Nmap là TCP SYN Scan. Công cụ này liên tục gửi các gói tin mang cờ SYN, được ghi nhận trong trường `pfsense.tcp.flags` với giá trị là `S`, đến hàng loạt cổng khác nhau trên máy chủ Web nhằm dò tìm cổng mở.
- Do tường lửa pfSense đã cấu hình chặn các cổng không thiết yếu, các gói tin quét này lập tức bị hệ thống từ chối và ghi nhận trạng thái block. Khi quan sát trên giao diện, một chuỗi dữ liệu dày đặc gửi từ cùng một địa chỉ nguồn đến nhiều cổng đích khác nhau trong một khoảng thời gian cực ngắn là bằng chứng của hành vi rà quét.

**Truy vấn Mẫu 3: Phát hiện hành vi Tấn công SQL Injection**

```
event.dataset: "apache.access" and url.query: *OR*
```

![Hình 5.22. Kết quả truy vấn log SQL Injection trên Kibana.](/images/i522.png)

Ý nghĩa các trường dữ liệu:
- Trường `event.dataset` chuyển vùng giám sát sang giá trị `apache.access` để chuyển đổi từ tầng mạng sang tầng ứng dụng, cụ thể là nhật ký truy cập của máy chủ Web Apache chứa ứng dụng lỗi DVWA.
- Trường `url.query` sử dụng từ khóa `OR` được đặt trong dấu `*` đại diện để tìm kiếm các giá trị như OR hoặc chuỗi mã hóa `%20or%20` ở bất kỳ vị trí nào trong URL. Khi kẻ tấn công thực hiện thao túng câu lệnh cơ sở dữ liệu bằng các toán tử logic, các ký tự này sẽ được truyền lên URL dưới dạng tham số truy vấn và bị Elastic Agent bắt trọn.

**Truy vấn Mẫu 4: Phát hiện hành vi Dò quét và Khai thác tập tin hệ thống**

Kịch bản này được phân tách làm hai tầng dữ liệu độc lập để tăng tính toàn vẹn cho quá trình điều tra kỹ thuật số:

*Mẫu 4.1: Phát hiện dấu vết tấn công File Inclusion trên Access Log*

```
event.dataset: "apache.access" and url.original: *etc/passwd*
```

![Hình 5.23. Kết quả truy vấn access.log path traversal trên Kibana.](/images/i523.png)

- Ý nghĩa: Bộ lọc tập trung vào hành vi path traversal để đọc file cấu hình nhạy cảm của hệ điều hành Linux. Việc sử dụng ký tự đại diện dấu sao giúp hệ thống bắt được toàn bộ các biến thể đường dẫn mà kẻ tấn công cố tình chèn vào tham số URL.

*Mẫu 4.2: Phát hiện các lỗi phát sinh do dò quét thư mục ẩn trên Error Log*

```
event.dataset: "apache.error" and message : "PHP warning"
```

![Hình 5.24. Kết quả truy vấn error.log dò quét thư mục trên Kibana.](/images/i524.png)

- Trường `event.dataset` truy xuất vào tập dữ liệu lỗi của máy chủ Web (`apache.error`).
- Trường `message` tìm kiếm chính xác từ khóa "PHP warning". Khi mã nguồn ứng dụng (DVWA) tiếp nhận đường dẫn bất thường và đưa vào các hàm xử lý tập tin (như include), bộ phân giải PHP đã gặp sự cố thực thi và lập tức trả về một cảnh báo. Việc bắt được dòng log này chứng minh hệ thống SIEM có khả năng ghi nhận log trong apache error.log.

**Đánh giá kịch bản**

Thông qua bốn mẫu log trên, hệ thống SIEM đã chứng minh được tính hiệu quả và khả năng giám sát toàn diện ở cả hai cấp độ: tầng mạng (Network Layer - giám sát lưu lượng và phát hiện rà quét qua pfSense) và tầng ứng dụng (Application Layer - phát hiện tấn công SQL Injection và Path Traversal qua Apache). Việc vận dụng linh hoạt ngôn ngữ truy vấn KQL kết hợp với các ký tự đại diện đã cho thấy sức mạnh của Kibana trong việc bóc tách và phân tích dữ liệu lớn. Thay vì phải duyệt thủ công qua hàng triệu dòng nhật ký thô, bộ lọc giúp người quản trị nhanh chóng loại bỏ nhiễu, định vị chính xác dấu vết của kẻ tấn công trong thời gian ngắn.

---

### 5.1.4. Phát hiện và phản hồi

**Phản hồi thủ công**

Máy user giải nén tệp `eicar_com.zip`, tạo ra file `eicar.com` để kích hoạt cơ chế quét và ngăn chặn của Elastic Defend.

![Hình 5.25. Máy user giải nén tệp eicar_com.zip](/images/i525.png)

Kibana Security Alerts ghi nhận cảnh báo sau khi Elastic Defend phát hiện tệp kiểm thử EICAR trên endpoint. Danh sách alert/endpoint event cho thấy cảnh báo bảo mật phát sinh trên host user trong quá trình kiểm thử EDR và sẽ ngăn chặn việc giải nén vừa rồi.

![Hình 5.26. Danh sách cảnh báo trên Kibana](/images/i526.png)

Chi tiết cảnh báo Malware Prevention Alert trên Kibana. Cảnh báo được Elastic Defend sinh ra khi phát hiện tệp kiểm thử EICAR trong quá trình giải nén bằng tiến trình unzip trên máy user.

![Hình 5.27. Chi tiết cảnh báo Malware Prevention Alert.](/images/i527.png)

Tiến hành thao tác phản hồi trên cảnh báo, quản trị viên có thể lựa chọn hành động như điều tra, phản hồi hoặc cô lập host.

![Hình 5.28. Giao diện lựa chọn hành động phản hồi trên cảnh báo EDR](/images/i528.png)

Gửi yêu cầu cô lập endpoint đến Elastic Defend. Sau khi gửi yêu cầu, Kibana hiển thị thông báo cho thấy yêu cầu cô lập đã được gửi thành công đến Elastic Defend Agent trên máy user.

![Hình 5.29. Yêu cầu cô lập endpoint user được gửi thành công đến Elastic Defend.](/images/i529.png)

Sau đó, trong danh sách endpoint, trạng thái của máy user vẫn là Healthy, đồng thời xuất hiện thêm nhãn Isolated. Điều này cho thấy Elastic Agent vẫn hoạt động bình thường và còn kết nối quản lý với Elastic Defend, nhưng lưu lượng mạng thông thường của endpoint đã bị cô lập.

![Hình 5.30. Trạng thái endpoint user sau khi cô lập.](/images/i530.png)

Kiểm tra sau khi isolate máy user: lệnh ping đến Web Server 192.168.10.2 không gửi được gói tin, chứng minh host đã bị cô lập khỏi mạng.

![Hình 5.31. Trạng thái endpoint user sau khi cô lập.](/images/i531.png)

**Phản hồi tự động**

Tiến hành giải nén tệp `eicar_com.zip`.

![Hình 5.32. Máy user giải nén tệp eicar_com.zip.](/images/i532.png)

Elastic Defend đã phát hiện và xử lý tệp EICAR trên máy user, đồng thời máy user đã ở trạng thái bị cô lập.

![Hình 5.33. Chi tiết cảnh báo Malware Prevention Alert.](/images/i533.png)

Sau khi rule phát hiện malware được cấu hình kèm response action, hệ thống tự động thực thi hành động isolate đối với endpoint liên quan. Kết quả trong phần Responses cho thấy cảnh báo Malware Prevention Alert đã thực thi lệnh isolate, trạng thái thực thi hoàn tất và thông báo isolate completed successfully.

![Hình 5.34. Kết quả thực thi phản hồi tự động](/images/i534.png)

Kiểm tra lại trong Endpoints host user đã bị cô lập.

![Hình 5.35. Danh sách Endpoints hiển thị host user ở trạng thái Healthy và Isolated](/images/i535.png)

Kiểm tra sau khi isolate máy user: lệnh ping đến Web Server 192.168.10.2 không gửi được gói tin, chứng minh host đã bị cô lập khỏi mạng.

![Hình 5.36. Kiểm tra kết nối từ host user đến Web Server](/images/i536.png)

**Đánh giá kịch bản**

Kịch bản phát hiện và phản hồi đã minh họa thành công khả năng hoạt động của hệ thống EDR tích hợp trong Elastic Defend. Khi tệp kiểm thử EICAR được giải nén trên máy user, Elastic Defend đã nhanh chóng phát hiện hành vi tạo tệp nghi ngờ và sinh cảnh báo Malware Prevention Alert trên Kibana. Điều này cho thấy hệ thống có khả năng giám sát endpoint theo thời gian thực và ghi nhận đầy đủ các hành vi liên quan đến tệp tin, tiến trình và hoạt động hệ thống.

Ở chế độ phản hồi thủ công, quản trị viên có thể trực tiếp thực hiện hành động Isolate Host thông qua giao diện Kibana. Sau khi isolate, máy user vẫn duy trì trạng thái Healthy do Elastic Agent còn kết nối quản lý với Fleet Server, tuy nhiên các kết nối mạng thông thường đến hệ thống nội bộ đã bị chặn. Kết quả kiểm tra ping thất bại đến Web Server chứng minh hành động cô lập endpoint đã được thực thi hiệu quả. Điều này cho thấy hệ thống không chỉ dừng lại ở khả năng phát hiện mà còn hỗ trợ phản ứng nhanh nhằm hạn chế nguy cơ lây lan trong mạng nội bộ.

Đối với phản hồi tự động, việc tích hợp response action trực tiếp vào detection rule giúp hệ thống tự động cô lập endpoint ngay khi phát hiện hành vi phù hợp mà không cần thao tác từ quản trị viên. Kết quả thực nghiệm cho thấy hành động isolate được thực thi thành công và trạng thái endpoint chuyển sang Isolated gần như ngay lập tức. Cơ chế này giúp giảm thời gian phản ứng trước sự cố, đặc biệt hữu ích trong các tình huống tấn công mã độc thực tế khi tốc độ xử lý là yếu tố quan trọng.

---

### 5.1.5. Dashboard thống kê trong X ngày

Dashboard được xây dựng với hai biểu đồ chính. Biểu đồ thứ nhất là dạng Stacked Bar, thể hiện tổng số sự kiện endpoint theo từng ngày và phân tách theo từng host. Mỗi cột biểu diễn số lượng sự kiện trong một ngày, trong đó từng màu tương ứng với một máy trong hệ thống. Kết quả cho thấy số lượng sự kiện phát sinh không đồng đều giữa các ngày. Một số ngày có lượng sự kiện tăng cao, đặc biệt là ngày có hoạt động kiểm thử và giám sát nhiều hơn. Trong từng cột, máy user, database và web đều có đóng góp vào tổng số sự kiện, giúp xác định được nguồn phát sinh dữ liệu theo từng thời điểm.

Biểu đồ thứ hai là dạng Pie Chart, thể hiện tỷ lệ sự kiện endpoint theo host trong toàn bộ khoảng thời gian thống kê. Kết quả cho thấy máy user chiếm tỷ lệ cao nhất với 51,78%, tiếp theo là database với 29,37% và web với 18,85%. Điều này cho thấy trong khoảng thời gian quan sát, máy user là endpoint phát sinh nhiều sự kiện nhất.

![Hình 5.37. Dashboard thống kê sự kiện endpoint trong X ngày.](/images/i537.png)

![Hình 5.38. Biểu đồ số lượng sự kiện endpoint theo ngày và theo host.](/images/i538.png)

![Hình 5.39. Biểu đồ tỷ lệ sự kiện endpoint theo host.](/images/i539.png)

**Đánh giá kịch bản**

Kịch bản dashboard đã minh họa rõ khả năng trực quan hóa dữ liệu của Kibana trong hệ sinh thái ELK Stack. Thay vì phân tích từng dòng log riêng lẻ trong Discover, dashboard cho phép tổng hợp và hiển thị dữ liệu endpoint dưới dạng biểu đồ trực quan, giúp người quản trị nhanh chóng nắm bắt trạng thái hoạt động của hệ thống.

Biểu đồ Stacked Bar đã thể hiện hiệu quả số lượng sự kiện endpoint theo thời gian và theo từng host. Việc phân tách dữ liệu theo `host.name` giúp dễ dàng xác định máy nào phát sinh nhiều sự kiện trong từng giai đoạn. Qua quá trình thực nghiệm, hệ thống ghi nhận sự gia tăng số lượng sự kiện vào các thời điểm có hoạt động kiểm thử bảo mật hoặc thao tác trên endpoint, phản ánh đúng trạng thái vận hành của môi trường thử nghiệm.

Bên cạnh đó, biểu đồ Pie giúp trực quan hóa tỷ lệ sự kiện endpoint giữa các host trong toàn bộ khoảng thời gian quan sát. Kết quả cho thấy máy user chiếm tỷ lệ cao nhất, phù hợp với thực tế do đây là máy được sử dụng để thực hiện phần lớn các thao tác kiểm thử như SQL Injection, dò quét và EDR. Dashboard vì vậy không chỉ hỗ trợ quan sát tổng quan mà còn giúp xác định nhanh các endpoint có mức độ hoạt động bất thường hoặc phát sinh nhiều sự kiện đáng chú ý.

---

### 5.1.6. Machine Learning

Truy cập vào Anomaly Explorer để kiểm tra trạng thái ban đầu của job trước khi tạo dữ liệu bất thường. Tại thời điểm này, Anomaly Explorer chưa ghi nhận anomaly nào vì hệ thống mới chỉ xử lý dữ liệu baseline, chưa có sự kiện tăng đột biến so với mẫu hành vi bình thường.

![Hình 5.40. Giao diện Anomaly Explorer trước khi có anomaly.](/images/i540.png)

Sau khi job học xong baseline, nhóm khởi động datafeed ở chế độ real-time trước khi chạy dữ liệu anomaly. Cấu hình start datafeed được chọn là Continue from now và No end time (Real-time search), giúp job theo dõi các file events mới phát sinh sau thời điểm baseline.

![Hình 5.41. Khởi động datafeed để theo dõi dữ liệu mới.](/images/i541.png)

Sau khi datafeed được khởi động, nhóm chạy script ở chế độ anomaly trên máy user. Ở chế độ này, script tạo số lượng lớn file trong thời gian ngắn tại thư mục `/tmp/elastic-ml-anomaly`. Hành vi này nhằm mô phỏng một tình huống bất thường trên endpoint, trong đó số lượng thao tác liên quan đến file tăng đột biến so với baseline đã học.

Lệnh chạy anomaly được sử dụng như sau:

```bash
./elastic_ml_endpoint_simulator.sh --mode anomaly --total-files 10000 --keep-files
```

Trong đó, `--mode anomaly` cho biết script chạy ở chế độ tạo dữ liệu bất thường, `--total-files 10000` cấu hình số lượng file cần tạo, và `--keep-files` giữ lại các file sau khi chạy để tránh sinh thêm nhiều sự kiện xóa file không cần thiết.

![Hình 5.42. Chạy script ở chế độ anomaly trên máy user.](/images/i542.png)

Sau khi dữ liệu anomaly được gửi về Elasticsearch và datafeed xử lý, Elastic Machine Learning ghi nhận bất thường trong Anomaly Explorer. Kết quả cho thấy detector count phát hiện số lượng sự kiện thực tế trong một bucket thời gian cao hơn đáng kể so với giá trị thông thường mà model đã học từ baseline. Mặc dù script tạo 10.000 file, số lượng event thực tế được Elastic Defend ghi nhận có thể lớn hơn do một thao tác tạo file có thể phát sinh nhiều endpoint events khác nhau.

![Hình 5.43. Kết quả phát hiện bất thường trong Anomaly Explorer.](/images/i543.png)

**Đánh giá kịch bản**

Kịch bản Machine Learning đã minh họa thành công khả năng phát hiện bất thường của Elastic ML dựa trên hành vi endpoint telemetry. Thay vì phụ thuộc hoàn toàn vào các rule hoặc signature cố định, hệ thống sử dụng cơ chế học baseline để xác định mức hoạt động bình thường của endpoint theo thời gian, từ đó phát hiện các biến động bất thường.

Trong giai đoạn baseline, script giả lập được chạy trên host user nhằm tạo số lượng file events nhỏ và ổn định theo từng vòng. Dữ liệu này đóng vai trò là tập hành vi bình thường để Elastic ML học mô hình hoạt động của endpoint. Việc bổ sung khoảng nghỉ và độ dao động ngẫu nhiên giữa các vòng giúp dữ liệu gần với hành vi thực tế hơn thay vì tạo ra chuỗi dữ liệu quá đều và dễ bị nhận diện là giả lập.

Sau khi baseline được hình thành, script được chuyển sang chế độ anomaly để tạo số lượng lớn file trong thời gian ngắn. Điều này làm số lượng endpoint events tăng đột biến vượt khỏi ngưỡng hành vi bình thường mà ML job đã học trước đó. Kết quả trong Anomaly Explorer cho thấy Elastic ML đã phát hiện thành công các điểm bất thường với anomaly score cao, đồng thời xác định chính xác host phát sinh sự kiện bất thường là máy user.

---

### 5.1.7. Đo hiệu năng thu thập log

Tiến hành enable service `dummy-log-gen.service` trên tất cả các máy.

![Hình 5.44. Enable service dummy-log-gen.service trên máy database.](/images/i544.png)

![Hình 5.45. Enable service dummy-log-gen.service trên máy web.](/images/i545.png)

![Hình 5.46. Enable service dummy-log-gen.service trên máy user.](/images/i546.png)

Sau 5 phút thu thập ta dừng các service và kiểm tra trên Kibana. Quan sát hình ta thấy số log thu thập được trong 5 phút là 15.339, tính trung bình là khoảng 51 record/giây.

![Hình 5.47. Kết quả thu thập dummy log trong 5 phút.](/images/i547.png)

Kiểm tra CPU Usage từ 09:00:00 đến 09:18:00 (đây là khoảng thời gian không thu thập dummy log) trong giao diện Observability. %CPU Usage là 4.4%.

![Hình 5.48. Thông tin về hiệu năng trên Observability của máy siem trước khi thu dummy log.](/images/i548.png)

Kiểm tra CPU Usage từ 09:18:00 đến 09:24:00 (đây là khoảng thời gian thu thập dummy log) trong giao diện Observability. %CPU Usage là 6.2%.

![Hình 5.49. Thông tin về hiệu năng trên Observability của máy siem trong 5 phút khi thu dummy log.](/images/i549.png)

**Đánh giá kịch bản**

Qua quá trình thực hiện kịch bản, hệ thống ELK stack đã ghi nhận được tổng cộng 15.339 bản ghi trong vòng 5 phút với tốc độ xử lý trung bình đạt khoảng 51 bản ghi mỗi giây. Kết quả giám sát hiệu năng cho thấy mức sử dụng CPU chỉ tăng rất nhẹ từ mức 4.4% trước khi chạy kịch bản lên mức 6.2% trong thời gian hệ thống tiến hành thu thập dữ liệu. Sự biến động tài nguyên thấp này chứng tỏ nền tảng hoàn toàn có khả năng đáp ứng và thu thập được số lượng bản ghi mỗi giây lớn hơn rất nhiều so với hiện tại. Tuy nhiên do những giới hạn thực tế về mặt thiết bị phần cứng nên không thể tiến hành các bài kiểm thử với mức tải nặng hơn để đánh giá toàn diện giới hạn tối đa của hệ thống.

---

### 5.1.8. Làm giàu custom log

Kiểm tra raw dummy log ban đầu trên các máy. Raw log thấy trong hình là:

```
May 22 02:21:18 user DUMMY[62886]: level=INFO host=user-station msg="Network connection established" code=200
```

![Hình 5.50. Raw dummy log trên máy user.](/images/i550.png)

**So sánh với dummy log đã được làm giàu**

```json
{
    "_index": ".ds-logs-system.syslog-default-2026.05.14-000002",
    "_id": "-b9_TZ4BAI7SipAARaif",
    "_version": 1,
    "_source": {
        "agent": {
            "name": "user",
            "id": "c4952982-b73b-49bf-a274-ed0a2e759ba0",
            "type": "filebeat",
            "ephemeral_id": "5b3e76a9-002f-40b3-b473-e12b9abcbdb5",
            "version": "8.19.13"
        },
        "process": {
            "name": "DUMMY"
        },
        "log": {
            "file": {
                "path": "/var/log/syslog"
            },
            "offset": 4680720
        },
        "elastic_agent": {
            "id": "c4952982-b73b-49bf-a274-ed0a2e759ba0",
            "version": "8.19.13",
            "snapshot": false
        },
        "message": "level=INFO host=user-station msg=\"Network connection established\" code=200",
        "input": {
            "type": "log"
        },
        "@timestamp": "2026-05-22T02:23:59.991Z",
        "system": {
            "syslog": {}
        },
        "ecs": {
            "version": "8.11.0"
        },
        "data_stream": {
            "namespace": "default",
            "type": "logs",
            "dataset": "system.syslog"
        },
        "host": {
            "hostname": "user",
            "os": {
                "kernel": "6.17.0-23-generic",
                "codename": "noble",
                "name": "Ubuntu",
                "family": "debian",
                "type": "linux",
                "version": "24.04.3 LTS (Noble Numbat)",
                "platform": "ubuntu"
            },
            "containerized": false,
            "ip": [
                "192.168.1.104",
                "fe80::deb1:cdc:5744:6d08",
                "192.168.20.2",
                "fe80::5f20:cf51:bc24:8459"
            ],
            "name": "user",
            "id": "8b021dae96a74c1c89371d590942f9f4",
            "mac": [
                "08-00-27-94-70-BF"
            ],
            "architecture": "x86_64"
        },
        "event": {
            "agent_id_status": "verified",
            "ingested": "2026-05-22T02:24:07Z",
            "timezone": "+00:00",
            "dataset": "system.syslog"
        }
    },
    "fields": {
        "elastic_agent.version": ["8.19.13"],
        "process.name.text": ["DUMMY"],
        "host.os.name.text": ["Ubuntu"],
        "host.name.text": ["user"],
        "host.hostname": ["user"],
        "host.mac": ["08-00-27-94-70-BF"],
        "host.ip": [
            "192.168.1.104",
            "fe80::deb1:cdc:5744:6d08",
            "192.168.20.2",
            "fe80::5f20:cf51:bc24:8459"
        ],
        "agent.type": ["filebeat"],
        "event.module": ["system"],
        "agent.name.text": ["user"],
        "host.os.version": ["24.04.3 LTS (Noble Numbat)"],
        "host.os.kernel": ["6.17.0-23-generic"],
        "host.os.name": ["Ubuntu"],
        "log.file.path.text": ["/var/log/syslog"],
        "agent.name": ["user"],
        "elastic_agent.snapshot": [false],
        "host.name": ["user"],
        "event.agent_id_status": ["verified"],
        "host.id": ["8b021dae96a74c1c89371d590942f9f4"],
        "event.timezone": ["+00:00"],
        "host.os.type": ["linux"],
        "elastic_agent.id": ["c4952982-b73b-49bf-a274-ed0a2e759ba0"],
        "data_stream.namespace": ["default"],
        "host.os.codename": ["noble"],
        "input.type": ["log"],
        "log.offset": [4680720],
        "message": ["level=INFO host=user-station msg=\"Network connection established\" code=200"],
        "data_stream.type": ["logs"],
        "host.architecture": ["x86_64"],
        "process.name": ["DUMMY"],
        "event.ingested": ["2026-05-22T02:24:07.000Z"],
        "@timestamp": ["2026-05-22T02:23:59.991Z"],
        "agent.id": ["c4952982-b73b-49bf-a274-ed0a2e759ba0"],
        "ecs.version": ["8.11.0"],
        "host.containerized": [false],
        "host.os.platform": ["ubuntu"],
        "data_stream.dataset": ["system.syslog"],
        "log.file.path": ["/var/log/syslog"],
        "agent.ephemeral_id": ["5b3e76a9-002f-40b3-b473-e12b9abcbdb5"],
        "agent.version": ["8.19.13"],
        "host.os.family": ["debian"],
        "event.dataset": ["system.syslog"]
    }
}
```

**So sánh Raw Log và Enriched Log**

Khi đối chiếu giữa dòng raw log trên terminal và file JSON đã được lưu trữ trong Elasticsearch, ta có thể thấy rõ quá trình "làm giàu" (enrichment) dữ liệu của ELK stack:

- **Định dạng và cấu trúc:**
  - Raw Log: Là một chuỗi văn bản phẳng (plain text) chưa phân loại rõ ràng, tuân theo định dạng syslog cơ bản (`[Thời gian] [Tên hệ thống] [Tiến trình/PID]: [Nội dung]`).
  - Enriched Log: Được chuyển đổi thành cấu trúc JSON đa cấp, tuân thủ chặt chẽ Elastic Common Schema (ECS). Việc cấu trúc hóa này giúp hệ thống dễ dàng phân tích, lập chỉ mục và truy vấn.

- **Bảo toàn nội dung gốc:**
  - Phần tải trọng chính (payload) của raw log (`level=INFO host=user-station msg="Network connection established" code=200`) không hề bị mất đi mà được giữ nguyên vẹn và ánh xạ vào trường `message` trong file JSON.
  - Tên tiến trình sinh ra log (`DUMMY`) được trích xuất riêng biệt vào trường `process.name`.

- **Lượng thông tin bổ sung (metadata):** Đây là điểm khác biệt lớn nhất. Raw log chỉ có thông tin cục bộ tại thời điểm phát sinh. Trong khi đó, Enriched log được Elastic Agent (Filebeat) tự động gắn thêm một lượng lớn siêu dữ liệu (metadata) hữu ích cho quá trình điều tra:
  - Môi trường hệ thống (`host`): Bổ sung chi tiết về hệ điều hành (Ubuntu 24.04.3 LTS - Noble Numbat), phiên bản kernel (`6.17.0-23-generic`), kiến trúc CPU (`x86_64`), danh sách địa chỉ IP và địa chỉ MAC.
  - Nguồn thu thập (`log`, `agent`): Ghi nhận rõ log này được lấy từ file nào (`/var/log/syslog`), thu thập bởi agent gì (Filebeat), phiên bản bao nhiêu (`8.19.13`), và ID định danh của agent đó.
  - Luồng dữ liệu (`data_stream`): Phân loại log thuộc nhóm dataset nào (`system.syslog`) để dễ dàng định tuyến và quản lý vòng đời dữ liệu (ILM).

- **Chuẩn hóa thời gian:**
  - Thời gian trong raw log (`May 22 02:21:18`) mang tính chất hiển thị cục bộ. Khi đưa vào Elasticsearch, nó được gắn thêm timezone và chuẩn hóa thành định dạng ISO 8601 (`@timestamp: "2026-05-22T02:23:59.991Z"`) để đảm bảo tính đồng nhất khi tra cứu trên hệ thống phân tán.

**Đánh giá kịch bản**

Kịch bản làm giàu custom log đã chứng minh thành công tính linh hoạt và khả năng xử lý mạnh mẽ của hệ thống ELK Stack đối với các định dạng dữ liệu chưa được hỗ trợ sẵn trong thực tế vận hành. Thông qua việc tự cấu hình ingest pipeline trực tiếp trên Kibana, hệ thống đã tự động phân tích và chuyển đổi thành công một dòng raw log ban đầu ở dạng văn bản phẳng thiếu cấu trúc thành một định dạng JSON hoàn chỉnh tuân thủ nghiêm ngặt chuẩn Elastic Common Schema. Kết quả đối chiếu cho thấy các nội dung cốt lõi của log gốc được bảo toàn vẹn toàn trong trường thông điệp và tiến trình, đồng thời hệ thống đã làm giàu dữ liệu một cách hiệu quả bằng cách tự động tích hợp thêm hàng loạt siêu dữ liệu quan trọng như thông tin chi tiết về hệ điều hành, cấu trúc phần cứng, địa chỉ IP, địa chỉ MAC và định danh của bộ thu thập dữ liệu, song song với việc chuẩn hóa mốc thời gian sang định dạng quốc tế. Việc cấu trúc hóa và bổ sung lượng thông tin chuyên sâu này không chỉ nâng cao giá trị của dữ liệu phục vụ cho công tác điều tra giám sát mà còn khẳng định ELK Stack là một giải pháp quản lý log toàn diện, có khả năng tiếp nhận và tối ưu hóa bất kỳ định dạng dữ liệu phát sinh nào mà không bị phụ thuộc vào các bộ phân tích cú pháp mặc định.

---

## 5.2. Đánh giá chung kết quả

Qua các kịch bản thực nghiệm, hệ thống ELK Stack kết hợp với Elastic Agent và Elastic Defend đã hoạt động ổn định và đáp ứng được các mục tiêu chính của đề tài. Hệ thống có khả năng thu thập log tập trung từ nhiều nguồn khác nhau như pfSense, Apache và endpoint, đồng thời hỗ trợ chuẩn hóa dữ liệu thông qua Ingest Pipeline để phục vụ việc tìm kiếm và phân tích trên Kibana.

Các chức năng giám sát và truy vấn log hoạt động hiệu quả thông qua Kibana Discover và ngôn ngữ truy vấn KQL, giúp người quản trị nhanh chóng phát hiện và truy vết các hành vi bất thường trong hệ thống. Ngoài ra, dashboard trực quan hóa dữ liệu đã hỗ trợ việc theo dõi tổng quan hoạt động của các endpoint theo thời gian.

Đối với khả năng EDR, Elastic Defend đã phát hiện thành công tệp kiểm thử EICAR và hỗ trợ phản hồi bằng cơ chế isolate host, cho thấy hệ thống có khả năng phát hiện và hạn chế nguy cơ lây lan trên endpoint. Bên cạnh đó, Elastic Machine Learning cũng bước đầu cho thấy hiệu quả trong việc phát hiện bất thường dựa trên hành vi endpoint telemetry thay vì chỉ phụ thuộc vào các rule cố định.

Kịch bản làm giàu custom log đã chứng minh thành công tính linh hoạt và khả năng xử lý mạnh mẽ của hệ thống ELK Stack đối với các định dạng dữ liệu chưa được hỗ trợ sẵn trong thực tế vận hành. Thông qua việc tự cấu hình ingest pipeline trực tiếp trên Kibana, hệ thống đã tự động phân tích và chuyển đổi thành công một dòng raw log ban đầu ở dạng văn bản phẳng thiếu cấu trúc thành một định dạng JSON hoàn chỉnh tuân thủ nghiêm ngặt chuẩn Elastic Common Schema. Kết quả đối chiếu cho thấy các nội dung cốt lõi của log gốc được bảo toàn vẹn toàn trong trường thông điệp và tiến trình, đồng thời hệ thống đã làm giàu dữ liệu một cách hiệu quả bằng cách tự động tích hợp thêm hàng loạt siêu dữ liệu quan trọng như thông tin chi tiết về hệ điều hành, cấu trúc phần cứng, địa chỉ IP, địa chỉ MAC và định danh của bộ thu thập dữ liệu, song song với việc chuẩn hóa mốc thời gian sang định dạng quốc tế. Việc cấu trúc hóa và bổ sung lượng thông tin chuyên sâu này không chỉ nâng cao giá trị của dữ liệu phục vụ cho công tác điều tra giám sát mà còn khẳng định ELK Stack là một giải pháp quản lý log toàn diện, có khả năng tiếp nhận và tối ưu hóa bất kỳ định dạng dữ liệu phát sinh nào mà không bị phụ thuộc vào các bộ phân tích cú pháp mặc định.

Qua quá trình thực hiện kịch bản, hệ thống ELK stack đã ghi nhận được tổng cộng 15.339 bản ghi trong vòng 5 phút với tốc độ xử lý trung bình đạt khoảng 51 bản ghi mỗi giây. Kết quả giám sát hiệu năng cho thấy mức sử dụng CPU chỉ tăng rất nhẹ từ mức 4.4% trước khi chạy kịch bản lên mức 6.2% trong thời gian hệ thống tiến hành thu thập dữ liệu. Sự biến động tài nguyên thấp này chứng tỏ nền tảng hoàn toàn có khả năng đáp ứng và thu thập được số lượng bản ghi mỗi giây lớn hơn rất nhiều so với hiện tại. Tuy nhiên do những giới hạn thực tế về mặt thiết bị phần cứng nên không thể tiến hành các bài kiểm thử với mức tải nặng hơn để đánh giá toàn diện giới hạn tối đa của hệ thống.

Tuy nhiên, hệ thống vẫn còn một số hạn chế như quy mô triển khai nhỏ, số lượng nguồn log còn hạn chế và Machine Learning mới chỉ được thử nghiệm ở mức cơ bản. Dù vậy, kết quả đạt được cho thấy ELK Stack là một giải pháp phù hợp để triển khai hệ thống SIEM/Threat Hunting trong môi trường học tập, nghiên cứu và các hệ thống doanh nghiệp quy mô nhỏ đến trung bình.

