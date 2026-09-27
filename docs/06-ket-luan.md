# Chương 6. KẾT LUẬN

## 6.1. Kết quả đạt được

Đề tài đã nghiên cứu và triển khai thành công hệ thống SIEM/Threat Hunting sử dụng ELK Stack kết hợp với Elastic Agent và Elastic Defend trong môi trường mạng giả lập. Hệ thống có khả năng thu thập log tập trung từ nhiều nguồn như pfSense, Apache và endpoint, sau đó chuẩn hóa dữ liệu thông qua Ingest Pipeline và hiển thị trên Kibana.

Các kịch bản thực nghiệm như chặn ICMP, port scanning, SQL Injection, path traversal, phát hiện EICAR và phát hiện bất thường bằng Machine Learning đều được ghi nhận và hiển thị thành công trên hệ thống. Ngoài ra, hệ thống cũng hỗ trợ phản hồi sự cố thông qua tính năng isolate host của Elastic Defend.

Thông qua đề tài, nhóm đã hiểu rõ hơn về quy trình triển khai một hệ thống SIEM hiện đại, từ thu thập log, phân tích dữ liệu, trực quan hóa đến phát hiện và phản hồi sự cố an ninh mạng.

## 6.2. Hướng phát triển tương lai

Mặc dù hệ thống đã đạt được các mục tiêu chính của đề tài, vẫn còn nhiều hướng phát triển có thể được mở rộng trong tương lai nhằm nâng cao khả năng giám sát và phòng thủ an ninh mạng.

Trước hết, hệ thống hiện tại mới được triển khai trong môi trường giả lập với quy mô nhỏ. Trong tương lai, có thể mở rộng hệ thống bằng cách triển khai thêm nhiều endpoint, máy chủ và thiết bị mạng để mô phỏng môi trường doanh nghiệp thực tế hơn. Đồng thời, việc xây dựng Elasticsearch Cluster đa node cũng sẽ giúp tăng khả năng chịu tải, tối ưu hiệu năng truy vấn và nâng cao tính sẵn sàng của hệ thống SIEM.

Bên cạnh đó, hệ thống có thể được bổ sung thêm nhiều nguồn log khác ngoài pfSense và Apache, chẳng hạn như Windows Event Logs, Active Directory, VPN Logs, IDS/IPS, Docker, Kubernetes hoặc Cloud Logs. Việc tích hợp thêm các nguồn dữ liệu này sẽ giúp hệ thống có góc nhìn toàn diện hơn về trạng thái an ninh mạng và hỗ trợ tốt hơn cho các hoạt động threat hunting.

Đối với phần EDR, hệ thống hiện mới dừng ở mức phát hiện cơ bản với tệp kiểm thử EICAR. Trong tương lai, có thể nghiên cứu triển khai các kịch bản malware thực tế hơn trong môi trường sandbox nhằm đánh giá sâu hơn khả năng phát hiện hành vi độc hại của Elastic Defend. Ngoài ra, hệ thống cũng có thể được mở rộng thêm các tính năng như process lineage analysis, memory analysis hoặc behavioral detection để hỗ trợ điều tra và phân tích sự cố chuyên sâu hơn.

Về Machine Learning, mô hình hiện tại mới sử dụng metric Count (Event rate) để phát hiện bất thường dựa trên số lượng sự kiện endpoint theo thời gian. Trong tương lai, có thể mở rộng bằng cách kết hợp nhiều đặc trưng khác như network events, process events hoặc user behavior analytics nhằm nâng cao độ chính xác của mô hình và giảm false positive. Ngoài ra, việc kết hợp các mô hình AI/LLM vào quá trình phân tích log và hỗ trợ điều tra bảo mật cũng là một hướng nghiên cứu tiềm năng trong các hệ thống SIEM hiện đại.

Cuối cùng, hệ thống có thể được tích hợp thêm cơ chế SOAR (Security Orchestration, Automation and Response) nhằm tự động hóa quy trình phát hiện, phân tích và phản hồi sự cố an ninh mạng. Việc tự động hóa này sẽ giúp giảm tải cho đội ngũ vận hành, rút ngắn thời gian phản ứng trước sự cố và nâng cao hiệu quả bảo vệ hệ thống trong môi trường thực tế.

## Danh mục tài liệu tham khảo

[1] Elastic, "The Elastic Stack." Available: [https://www.elastic.co/docs/get-started/the-stack](https://www.elastic.co/docs/get-started/the-stack). Accessed: Mar. 2026.

[2] Elastic, "Elastic Security solution & project type overview." Available: [https://www.elastic.co/docs/solutions/security](https://www.elastic.co/docs/solutions/security). Accessed: Mar. 2026.

[3] Elastic, "Elastic Agent and Fleet." Available: [https://www.elastic.co/docs/reference/fleet](https://www.elastic.co/docs/reference/fleet). Accessed: May 2026.

[4] Elastic, "Ingest Pipelines." Available: [https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html). Accessed: May 2026.

[5] Elastic, "Kibana Guide." Available: [https://www.elastic.co/guide/en/kibana/current/index.html](https://www.elastic.co/guide/en/kibana/current/index.html). Accessed: May 2026.

[6] Elastic, "Machine Learning in Elastic." Available: [https://www.elastic.co/guide/en/machine-learning/current/index.html](https://www.elastic.co/guide/en/machine-learning/current/index.html). Accessed: May 2026.

[7] Elastic, "Elastic Defend." Available: [https://www.elastic.co/docs/solutions/security/configure-elastic-defend](https://www.elastic.co/docs/solutions/security/configure-elastic-defend). Accessed: May 2026.

[8] Netgate, "pfSense Documentation." Available: [https://docs.netgate.com/pfsense/en/latest/](https://docs.netgate.com/pfsense/en/latest/). Accessed: May 2026.

[9] Apache Software Foundation, "Apache HTTP Server Documentation." Available: [https://httpd.apache.org/docs/](https://httpd.apache.org/docs/). Accessed: May 2026.

[10] OWASP Foundation, "Damn Vulnerable Web Application (DVWA)." Available: [https://github.com/digininja/DVWA](https://github.com/digininja/DVWA). Accessed: May 2026.

[11] EICAR, "Anti-Malware Testfile." Available: [https://www.eicar.org/download-anti-malware-testfile/](https://www.eicar.org/download-anti-malware-testfile/). Accessed: May 2026.

[12] Nmap Project, "Nmap Reference Guide." Available: [https://nmap.org/book/man.html](https://nmap.org/book/man.html). Accessed: May 2026.
