# Chương 3. CƠ SỞ LÝ THUYẾT

## 3.1. Hệ thống SIEM/Threat Hunting

### 3.1.1. Tổng quan về giải pháp SIEM

SIEM (Security Information and Event Management) là giải pháp bảo mật toàn diện kết hợp giữa quản lý thông tin an ninh (SIM - Security Information Management) và quản lý sự kiện an ninh (SEM - Security Event Management). Mục tiêu cốt lõi của hệ thống SIEM là cung cấp cho đội ngũ vận hành an ninh (SOC) một cái nhìn tập trung, toàn diện và theo thời gian thực về trạng thái an toàn thông tin của toàn bộ hạ tầng mạng tổ chức.

Hệ thống SIEM hoạt động dựa trên các chức năng nền tảng sau:

- Thu thập dữ liệu tập trung (Data Aggregation): Tự động thu thập nhật ký hoạt động (logs) và các sự kiện bảo mật từ đa dạng nguồn trong hệ thống như: thiết bị mạng (Firewall, Router, Switch), hệ điều hành máy chủ và máy trạm (Windows, Linux), các ứng dụng dịch vụ (Web server, Database Server), cũng như các giải pháp an ninh chuyên sâu khác (IDS/IPS, Antivirus).
- Chuẩn hóa dữ liệu (Data Normalization): Chuyển đổi dữ liệu nhật ký từ nhiều định dạng thô khác nhau (như Syslog, CSV, XML, chuỗi văn bản không cấu trúc) về một định dạng cấu trúc chung, thống nhất giúp việc lưu trữ và truy vấn hiệu quả hơn.
- Tương quan dữ liệu sự kiện (Event Correlation): Sử dụng các tập quy tắc (rules) hoặc thuật toán thông minh để liên kết, xâu chuỗi nhiều sự kiện rời rạc từ các nguồn khác nhau nhằm phát hiện ra chuỗi hành vi tấn công phức tạp mà các thiết bị phòng thủ đơn lẻ không thể nhận diện.
- Trực quan hóa và Cảnh báo (Visualization & Alerting): Tổng hợp dữ liệu thành các bảng điều khiển trực quan (Dashboards) hỗ trợ quản trị viên theo dõi và lập tức phát ra cảnh báo (alerts) khi phát hiện dấu hiệu bất thường.

### 3.1.2. Khái niệm và Vai trò của Threat Hunting

Mặc dù hệ thống SIEM truyền thống rất mạnh mẽ trong việc phát hiện các cuộc tấn công dựa trên các dấu hiệu nhận biết đã biết (Signature-based), tuy nhiên các kỹ thuật xâm nhập tinh vi ngày nay (như tấn công APT, mã độc zero-day) thường có khả năng ẩn mình khéo léo dưới lớp vỏ các tiến trình hợp lệ của hệ thống nhằm vượt qua các quy tắc phát hiện tĩnh. Để giải quyết rào cản này, mô hình phòng thủ cần chuyển dịch sang trạng thái chủ động thông qua kỹ thuật Threat Hunting (Săn tìm mối đe dọa)

Threat Hunting được định nghĩa là một quá trình chủ động, liên tục và mang tính giả định, trong đó các chuyên gia phân tích an ninh mạng trực tiếp tìm kiếm xuyên suốt hệ thống để định vị, cô lập và triệt tiêu các mối đe dọa nâng cao tiềm ẩn mà các giải pháp bảo mật tự động hiện tại đã bỏ sót. Thay vì ngồi chờ hệ thống SIEM kích hoạt cảnh báo, một chu trình Threat Hunting thường bắt đầu bằng việc đặt ra các giả thuyết dựa trên thông tin tình báo về mối đe dọa (Cyber Threat Intelligence), các lỗ hổng mới công bố, hoặc các hành vi bất thường vừa được nhận diện trong mạng.

### 3.1.3. Sự hội tụ giữa SIEM và Threat Hunting trong giám sát an ninh hiện đại

Trong hạ tầng SOC hiện đại, SIEM và Threat Hunting không phải là hai giải pháp tách biệt mà là hai thành phần có mối quan hệ tương hỗ chặt chẽ, tạo nên một chu trình phòng thủ khép kín:

- SIEM làm nền tảng cung cấp dữ liệu cho Threat Hunting: Toàn bộ dữ liệu log phong phú được thu thập, chuẩn hóa và lưu trữ tập trung trong Elasticsearch chính là kho nguyên liệu vô giá để các Hunter truy vấn. Khả năng tìm kiếm tốc độ cao và các bộ lọc linh hoạt (Time Filter, KQL) trên Kibana giúp thu hẹp phạm vi điều tra từ hàng triệu sự kiện xuống các dấu vết đáng ngờ một cách nhanh chóng.
- Threat Hunting tối ưu hóa năng lực phát hiện của SIEM: Khi quá trình săn tìm thủ công phát hiện ra một kỹ thuật xâm nhập mới hoặc một chuỗi hành vi bất thường chưa từng được định nghĩa, Hunter sẽ chuyển hóa tri thức này thành các quy tắc phát hiện tự động (Detection Rules) hoặc tích hợp vào các mô hình học máy (Machine Learning) của SIEM. Điều này giúp hệ thống SIEM nâng cao độ chính xác, giảm thiểu tỷ lệ cảnh báo sai (False Positive) và tự động hóa khả năng phòng thủ cho các chu kỳ tiếp theo

## 3.2. Tổng quan về hệ sinh thái ELK Stack và các thành phần cốt lõi

### 3.2.1. Elasticsearch

Elasticsearch là một công cụ tìm kiếm và phân tích phân tán, đóng vai trò là kho lưu trữ dữ liệu trung tâm và cơ sở dữ liệu vector cho Elastic Stack. Hệ thống cung cấp khả năng tìm kiếm và phân tích gần thời gian thực cho mọi loại hình dữ liệu, bao gồm văn bản có cấu trúc và không cấu trúc, dữ liệu chuỗi thời gian (time-series), dữ liệu vector và dữ liệu không gian địa lý.

- Tính phân tán và khả năng mở rộng: Elasticsearch hoạt động dưới dạng một cụm (cluster) gồm một hoặc nhiều máy chủ, được gọi là các nút (node). Khi dữ liệu được thêm vào hệ thống, chúng sẽ được tổ chức trong các chỉ mục (index), đơn vị lưu trữ cơ bản của Elasticsearch

- Cơ chế phân tán: Dữ liệu bên trong một chỉ mục sẽ được chia nhỏ thành các phân vùng (shard) và phân bổ trên nhiều nút (node) khác nhau trong cụm. Kiến trúc này giúp hệ thống xử lý khối lượng dữ liệu lớn và đảm bảo tính sẵn sàng cao (High Availability). Nếu một nút gặp sự cố, dữ liệu vẫn luôn khả dụng nhờ các bản sao (replica)

- Hiệu suất truy vấn: Hỗ trợ đa dạng ngôn ngữ truy vấn, các phép tổng hợp (aggregations) và các tính năng mạnh mẽ để lọc, phân tích dữ liệu chuyên sâu một cách nhanh chóng.

### 3.2.2. Logstash

Logstash là một công cụ thu thập dữ liệu mã nguồn mở với khả năng xử lý luồng theo thời gian thực. Logstash có thể hợp nhất linh hoạt dữ liệu từ nhiều nguồn khác biệt và chuẩn hóa dữ liệu vào các đích đến tùy chọn. Làm sạch và mở rộng khả năng tiếp cận toàn bộ dữ liệu để phục vụ cho đa dạng các bài toán phân tích đầu ra nâng cao và trực quan hóa.

### 3.2.3. Kibana

Kibana cung cấp giao diện người dùng cho tất cả các giải pháp Elastic. Đây là một công cụ mạnh mẽ để trực quan hóa và phân tích dữ liệu, cũng như để quản lý và giám sát Elastic Stack. Mặc dù có thể sử dụng Elasticsearch mà không cần Kibana, nhưng nó là cần thiết cho hầu hết các trường hợp sử dụng và mặc định có khi triển khai.

Kibana hỗ trợ các tính năng sau:

- Khám phá dữ liệu: Tìm kiếm và lọc dữ liệu thô một cách linh hoạt thông qua công cụ Discover.
- Trực quan hóa dữ liệu: Xây dựng các biểu đồ, đồ thị và số liệu tùy chỉnh dễ dàng bằng thao tác kéo thả với công cụ Lens.
- Xây dựng bảng điều khiển (Dashboard): Tổng hợp các biểu đồ thành các bảng điều khiển tương tác để cung cấp cái nhìn tổng quan và toàn diện về dữ liệu.
- Phân tích không gian địa lý: Thực hiện phân tích dữ liệu không gian địa lý và tích hợp bản đồ trực tiếp vào bảng điều khiển.
- Giám sát và cảnh báo: Thiết lập thông báo cho các sự kiện dữ liệu quan trọng, đồng thời theo dõi sự cố thông qua hệ thống cảnh báo (alerts) và quản lý tình huống (cases).
- Quản lý tài nguyên: Quản trị các tài nguyên hệ thống như pipeline xử lý dữ liệu, luồng dữ liệu (data stream), các mô hình học máy đã được huấn luyện (trained models), v.v.

### 3.2.4. Beats

Beats là các công cụ gửi dữ liệu (data shippers) mã nguồn mở, được cài đặt dưới dạng các agent trên máy chủ để gửi dữ liệu vận hành đến Elasticsearch. Elastic cung cấp các

Beats riêng biệt cho từng loại dữ liệu khác nhau, chẳng hạn như nhật ký (logs), số liệu (metrics) và trạng thái hoạt động (uptime).

Beats hiện đã được thay thế bằng Elastic Agent trong hầu hết các trường hợp sử dụng. Khi chuyển sang sử dụng Elastic Agent, các chức năng cốt lõi của Beats vẫn được giữ nguyên nhưng được tích hợp thêm nhiều tính năng mới. Thay vì phải cài đặt nhiều công cụ gửi dữ liệu Beats trên cùng một máy chủ (host) để đáp ứng các yêu cầu khác nhau, chỉ cần cài đặt một Elastic Agent duy nhất là đã có thể thu thập và truyền tải nhiều loại dữ liệu đồng thời.

### 3.2.5. Elastic Agent

Elastic Agent là một giải pháp hợp nhất và duy nhất để bổ sung khả năng giám sát nhật ký (logs), số liệu (metrics) và các loại dữ liệu khác cho một máy host. Công cụ này cũng có khả năng bảo vệ máy chủ khỏi các mối đe dọa bảo mật, truy vấn dữ liệu từ hệ điều hành và chuyển tiếp dữ liệu từ các phần cứng hoặc dịch vụ từ xa.

Mỗi agent sở hữu một chính sách (policy) duy nhất, cho phép dễ dàng bổ sung các gói tích hợp (integrations) cho những nguồn dữ liệu mới, các lớp bảo vệ an ninh và nhiều tính năng khác. Ngoài ra, các bộ xử lý (processors) của Elastic Agent có thể được sử dụng để làm sạch (sanitize) hoặc làm giàu (enrich) dữ liệu.

Có thể sử dụng tính năng Quản lý tập trung (Central management) trong Fleet để giám sát trạng thái của toàn bộ hệ thống Elastic Agent, quản lý các chính sách agent, cũng như nâng cấp các tệp thực thi (binaries) hoặc các gói tích hợp của Elastic Agent.

### 3.2.6. Integrations (các gói tích hợp)

Các gói tích hợp (Integrations) cung cấp một phương pháp đơn giản để kết nối Elastic với các dịch vụ và hệ thống bên ngoài, nhằm nhanh chóng thu thập thông tin chuyên sâu hoặc thực hiện các hành động phản hồi. Các gói này có khả năng thu thập các nguồn dữ liệu mới và thường đi kèm với những tài nguyên được cấu hình sẵn như bảng điều khiển (dashboards), biểu đồ trực quan (visualizations) và các luồng xử lý (pipelines) nhằm trích xuất các trường dữ liệu có cấu trúc từ nhật ký (logs) và sự kiện (events). Điều này giúp việc phân tích và tiếp cận thông tin chuyên sâu chỉ trong vài giây trở nên dễ dàng hơn. Các gói tích hợp luôn có sẵn cho những dịch vụ và nền tảng phổ biến như Nginx hoặc AWS, cũng như nhiều loại dữ liệu đầu vào thông dụng như các tệp nhật ký (log files).

Kibana cung cấp một giao diện web (web-based UI) để bổ sung và quản lý các gói tích hợp. Tại đây, có thể dễ dàng duyệt qua một chế độ xem hợp nhất về các gói tích hợp hiện có, hiển thị đầy đủ cả các bản tích hợp dành cho Elastic Agent và Beats.

### 3.2.7. Elasticsearch ingest pipelines

Elastics Ingest pipeline cho phép thực hiện các phép biến đổi phổ biến trên dữ liệu trước khi lập chỉ mục (indexing). Ví dụ, có thể sử dụng pipeline để xóa bớt các trường (fields) không cần thiết, trích xuất các giá trị từ văn bản thô và làm giàu (enrich) thông tin cho dữ liệu.

Một pipeline bao gồm một chuỗi các tác vụ có thể cấu hình, được gọi là các bộ xử lý (processors). Mỗi bộ xử lý hoạt động theo thứ tự tuần tự, thực hiện những thay đổi cụ thể trên các tài liệu (documents) đầu vào. Sau khi toàn bộ các bộ xử lý đã chạy xong, Elasticsearch sẽ thêm các tài liệu đã được biến đổi này vào luồng dữ liệu (data stream) hoặc chỉ mục (index) đích.

### 3.2.8. Fleet

Fleet là một ứng dụng trên nền tảng web được tích hợp trực tiếp trong Kibana, cung cấp giải pháp để quản lý tập trung toàn bộ hệ thống Elastic Agent. Công cụ này giúp đơn giản hóa việc vận hành, cho phép theo dõi và điều phối hàng chục nghìn tác nhân (agents) từ xa thông qua một giao diện duy nhất.

- Quản lý tập trung với Fleet (Central management with Fleet): Tính năng này loại bỏ việc phải cấu hình thủ công trên từng máy chủ riêng lẻ thông qua việc sử dụng các chính sách tác nhân (agent policies). Khi cần bổ sung các gói tích hợp (integrations) mới, cập nhật cấu hình thu thập dữ liệu hay nâng cấp các tệp thực thi (binaries), chỉ cần thực hiện thao tác trên Kibana. Fleet sẽ tự động đồng bộ và áp dụng các cấu hình mới này xuống hàng loạt Elastic Agent đang đăng ký nhận chính sách. Bên cạnh đó, hệ thống cũng cung cấp bảng điều khiển trung tâm để giám sát trạng thái hoạt động (health status) và nhật ký lỗi của từng agent theo thời gian thực.
- Fleet Server: Fleet Server là một thành phần cơ sở hạ tầng cốt lõi, đóng vai trò là điểm kiểm soát và cầu nối giao tiếp giữa các Elastic Agent và Fleet. Về bản chất kỹ thuật, nó là một tiến trình (process) chạy trực tiếp bên trong một Elastic Agent được chỉ định. Chức năng chính của Fleet Server là thiết lập kết nối an toàn, phân phối các chính sách từ Kibana xuống các agent ở đầu cuối, đồng thời tiếp nhận và định tuyến các báo cáo trạng thái gửi về. Đối với các kiến trúc hệ thống quy mô lớn, có thể triển khai nhiều Fleet Server để phân phối tải, hỗ trợ kết nối đồng thời cho hàng trăm nghìn tác nhân và đảm bảo tính sẵn sàng cao (High Availability)

## 3.3. Các mô hình triển khai

![Hình 3.1. Luồng dữ liệu của Elastic stack (nguồn: Elastic Pipelines)](./assets/hinh-3.1-elastic-pipeline.png)

Hình 3.1. Luồng dữ liệu của Elastic stack. nguồn: Elastic Pipelines

### 3.3.1. Mô hình truyền thống (Beats → Elasticsearch)

Trong mô hình này, các công cụ thu thập dữ liệu đơn nhiệm (như Filebeat, Metricbeat) được cài đặt trực tiếp trên các máy host. Dữ liệu thô thu thập được sẽ được gửi đến cụm Elasticsearch để lập chỉ mục.

- Đặc điểm: Kiến trúc đơn giản, thiết lập nhanh chóng và yêu cầu ít thành phần trung gian.
- Hạn chế: Khi hệ thống mở rộng, việc cập nhật cấu hình hoặc quản lý hàng loạt các tác vụ Beats phân tán trên nhiều máy chủ trở thành một thách thức lớn. Đồng thời, mô hình này không hỗ trợ tốt các phép biến đổi dữ liệu phức tạp trước khi lưu trữ.

### 3.3.2. Mô hình có phân tích trung gian (Beat/Elastic Agent → Logstash → Elasticsearch)

Mô hình này bổ sung Logstash làm một nút thắt xử lý trung gian giữa luồng dữ liệu đầu vào và kho lưu trữ. Dữ liệu từ các máy host (được thu thập bởi Beats hoặc Elastic Agent) sẽ đi vào Logstash để thực hiện định tuyến, lọc và chuyển đổi định dạng.

- Đặc điểm: Cực kỳ linh hoạt, phù hợp với các hệ thống cần xử lý luồng dữ liệu (pipelining) phức tạp, có cấu trúc log không đồng nhất, hoặc cần phân phối dữ liệu ra nhiều đích đến (outputs) khác nhau ngoài Elasticsearch.
- Hạn chế: Yêu cầu cung cấp và duy trì thêm tài nguyên phần cứng để vận hành cụm Logstash, làm tăng độ phức tạp của hạ tầng mạng và chi phí bảo trì.

### 3.3.3. Mô hình hiện đại tích hợp Ingest Pipeline (Elastic Agent/Fleet → Elasticsearch Ingest Pipeline → Elasticsearch)

Đây là mô hình được nhóm lựa chọn để triển khai chính thức. Kiến trúc này đại diện cho thế hệ mới nhất của Elastic, tập trung vào khả năng quản trị hợp nhất và tối ưu hóa hiệu suất xử lý:

- Đặc điểm: Sử dụng một Elastic Agent duy nhất trên mỗi máy host thay vì nhiều Beats riêng lẻ để thu thập mọi loại dữ liệu (logs, metrics). Toàn bộ hệ thống này được quản lý, theo dõi và cập nhật cấu hình tự động từ xa thông qua ứng dụng Fleet tích hợp sẵn trên Kibana. Dữ liệu từ Elastic Agent được gửi trực tiếp đến Elasticsearch. Thay vì dùng hệ thống bên ngoài như Logstash, dữ liệu sẽ chạy qua Elastics Ingest pipeline ngay bên trong Elasticsearch. Các bộ xử lý (processors) tại đây sẽ đảm nhận việc làm sạch, trích xuất các trường thông tin và làm giàu dữ liệu (enrichment) trước khi đưa vào chỉ mục (index).
- Ưu điểm cốt lõi: Mô hình này giải quyết triệt để bài toán vận hành bằng cách loại bỏ các thành phần hạ tầng không cần thiết, giảm thiểu độ trễ mạng, tinh gọn kiến trúc hệ thống và tận dụng sức mạnh xử lý mạnh mẽ ngay tại tầng cơ sở dữ liệu của Elasticsearch.

## 3.4. Cơ chế hoạt động chi tiết của mô hình đã lựa chọn (Mô hình hiện đại tích hợp Ingest Pipeline)

### 3.4.1. Thiết lập Fleet Server để quản lý tập trung

Quá trình vận hành bắt đầu bằng việc khởi chạy thành phần Fleet Server để làm trung tâm điều khiển toàn bộ mạng lưới. Khi được cấu hình thông qua giao diện Kibana, Fleet Server sẽ mở các kênh giao tiếp bảo mật và lắng nghe tín hiệu mạng, đóng vai trò như một trạm trung chuyển sẵn sàng tiếp nhận kết nối và điều phối các máy chủ đầu cuối tham gia vào hệ thống giám sát.

### 3.4.2. Tiến hành Enroll các Agents

Khi Fleet Server đã sẵn sàng, các ứng dụng Elastic Agent được cài đặt lên các máy host cần giám sát và sử dụng một mã xác thực (enrollment token) để xin phép kết nối. Fleet Server tiến hành kiểm tra mã xác thực này, ghi danh Agent vào danh sách quản lý trung tâm và chính thức mở ra một kết nối giao tiếp hai chiều an toàn giữa máy chủ đầu cuối và hệ thống.

### 3.4.3. Cài đặt các Integrations lên các Agents

Thông qua kết nối giao tiếp vừa thiết lập, Fleet Server truyền các cấu hình từ gói tích hợp (Integrations) xuống thẳng các Agent để phân công nhiệm vụ. Nhận được lệnh, Agent sẽ biết chính xác cần đọc tệp nhật ký (log) hoặc thông số hệ thống nào, từ đó liên tục thu thập, đóng gói các luồng dữ liệu thô tại máy chủ và gửi về phía cụm cơ sở dữ liệu Elasticsearch.

### 3.4.4. Xử lý dữ liệu thu thập được bằng Ingest Pipeline trên Elasticsearch

Ngay khi luồng dữ liệu thô (thường là các chuỗi văn bản liền mạch, chưa có cấu trúc) được gửi đến Elasticsearch, chúng được định tuyến đi qua hệ thống Ingest Pipeline. Tại đây, một cơ chế tự động sẽ phân tích cú pháp, cắt gọt và bóc tách chuỗi văn bản thành các thông tin độc lập (như địa chỉ IP, thời gian, tên người dùng), biến đổi nguồn dữ liệu lộn xộn ban đầu thành một định dạng sạch sẽ và có cấu trúc đã được quy định trong Integrations đã cài đặt lên Agents.

### 3.4.5. Lưu trữ dữ liệu đã được xử lý bằng Elasticsearch

Dữ liệu đã được chuẩn hóa bao gồm cả văn bản, chuỗi thời gian hay dữ liệu không gian địa lý sẽ được Elasticsearch tổ chức và lập chỉ mục (index). Để đảm bảo tính phân tán và khả năng mở rộng, dữ liệu bên trong một chỉ mục tiếp tục được chia nhỏ thành các phân vùng (shard) và phân bổ đều đặn trên các nút (node) khác nhau thuộc cùng một cụm (cluster).

### 3.4.6. Hiển thị dữ liệu trên Elasticsearch bằng Kibana

Ở công đoạn cuối cùng, giao diện Kibana sẽ thực hiện truy vấn vào kho lưu trữ Elasticsearch để trích xuất thông tin, tự động vẽ các biểu đồ và hiển thị dữ liệu lên Bảng điều khiển (Dashboards). Đồng thời, hệ thống cũng không ngừng đối chiếu kho dữ liệu với các quy tắc bảo mật (Security Rules) để phát hiện mối đe dọa, lập tức xuất ra các cảnh báo (alerts) trên màn hình giúp đội ngũ vận hành nắm bắt tình hình và xử lý sự cố kịp thời.
