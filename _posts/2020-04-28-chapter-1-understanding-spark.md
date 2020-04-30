---
layout: post
number: Chương 1
title: Tìm hiểu về Spark
---

Apache Spark là một nền tảng xử lý dữ liệu mã nguồn mở, vốn là một phần trong luận án tiến sĩ của Matei Zaharia ở trường Đại học UC Berkeley. Phiên bản đầu tiên của Spark được phát hành năm 2012. Sau đó vào năm 2013, Zaharia thành lập công ty Databricks và lên làm {{site.data.glossary.cto}}; anh cũng đồng thời là giảng viên của trường Đại học Stanford sau khi rời khỏi MIT. Cũng trong thời gian này, mã nguồn của Spark được trao cho tổ chức phần mềm mã nguồn mở Apache Software Foundation và trở thành một dự án vô cùng phát triển.

Apache Spark nhanh, dễ dùng, cho phép ta xử lý các kiểu dữ liệu vửa đa dạng vừa phức tạp khác nhau, bất kể là dữ liệu có cấu trúc, bán cấu trúc, dữ liệu {{site.data.glossary.streaming}}, hay là áp dụng cho {{site.data.glossary.machine_learning}} và {{site.data.glossary.data_science}} đều được. Đây cũng là một trong những dự án lớn nhất trong giới mã nguồn mở với hơn 1,000 lập trình viên đến từ hơn 250 công ty lớn nhỏ, đã có hơn 300,000 buổi thảo luận được tổ chức ở hơn 570 thành phố trên khắp thế giới.

Trong chương này, ta sẽ học những thành phần cơ bản nhất của Apache Spark. Ta sẽ nói về các khái niệm trong Spark Jobs và Spark APIs, giới thiệu về kiến trúc và tìm hiểu các khả năng của Spark 2.0.

Các chủ đề bao gồm:
- Apache Spark là gì?
- Spark Jobs và Spark APIs
- Khái quát về cấu trúc dữ liệu {{site.data.glossary.resilient_distributed_dataset}} (RDD), {{site.data.glossary.dataframe}}, và {{site.data.glossary.dataset}}
- Khái quát về dự án tối ưu hoá Catalyst Optimizer và dự án Tungsten
- Khái quát về thiết kế kiến trúc của Spark 2.0

## Apache Spark là gì?

Apache Spark là một nền tảng truy vấn và xử lý dữ liệu phân tán mã nguồn mở. Nó linh hoạt và có khả năng mở rộng giống như MapReduce nhưng nhanh hơn rất nhiều: hơn 100 lần so với Apache Hadoop nếu đọc dữ liệu trên bộ nhớ và hơn 10 lần nếu đọc dữ liệu từ ổ cứng.

Ta có thể dễ dàng dùng Apache Spark để đọc, chuyển đổi và tổng hợp dữ liệu, cũng như chạy và triển khai các mô hình thống kê dữ liệu phức tạp. Ta có thể dùng Java, Scala, Python, R và SQL để truy xuất các hàm của Spark APIs. Apache Spark có thể được dùng để viết ra cả một ứng dụng hoàn chỉnh, hoặc có thể đóng gói thành thư viện để triển khai trên các hệ thống máy chủ, hoặc cũng có thể dùng cho các phân tích *ngắn gọn* thông qua các {{site.data.glossary.notebook}} (như Jupyter, Spark-Notebook, Databricks, và Apache Zeppelin).

Apache Spark cung cấp hàng loạt công cụ tương đương với các thư viện mà các {{site.data.glossary.data_analyst}}, các {{site.data.glossary.data_scientist}} hoặc các nhà nghiên cứu hay dùng như `pandas` trong Python, `data.frames` hoặc `data.tables` trong R. Mặc dù Spark DataFrame *khá tương đồng* với `pandas` hoặc `data.frames` / `data.tables`, nhưng ta phải biết là sẽ có những khác biệt nhất định nên đừng quá nóng vội. Những ai đã quen với SQL cũng có thể dễ dàng sử dụng ngôn ngữ này. Ngoài ra, đi kèm với Apache Spark là một vài thuật toán, vài mô hình phân tích thống kê đã được viết và tối ưu sẵn, và vài thư viện khác: MLlib và ML dành cho {{site.data.glossary.machine_learning}}, GraphX và GraphFrames dành cho xử lý dữ liệu đồ thị, và Spark Streaming để xử lý dữ liệu {{site.data.glossary.streaming}}. Spark cũng cho phép ta có thể dễ dàng kết hợp các thư viện này với nhau trong cùng một ứng dụng.

Apache Spark có thể chạy ngay trên máy tính cá nhân, nhưng cũng có thể dễ dàng được triển khai theo chế độ độc lập, hoặc thông qua YARN, hoặc qua Apache Mesos - trên hệ thống máy chủ nội bộ cũng được mà trên {{site.data.glossary.cloud}} cũng được. Nó có thể đọc và ghi từ nhiều các nguồn dữ liệu đa dạng khác nhau, đơn cử như HDFS, Apache Cassandra, Apache HBase, và S3:

![]({{ "/assets/images/B05793_01_01.jpg" | relative_url }})

Nguồn: Apache Spark chính là chiếc máy điện thoại thông minh trong giới Dữ liệu lớn <http://bit.ly/1QsgaNj>

[Xem thêm bài viết Apache Spark chính là chiếc máy điện thoại thông minh trong giới Dữ liệu lớn ở đây <http://bit.ly/1QsgaNj>]

## Spark Jobs và Spark APIs
Trong phần này, ta sẽ xem qua về Spark Jobs và Spark APIs. Đây là những kiến thức nền tảng cho phần sau, kiến trúc của Spark 2.0.

### Quy trình xử lý dữ liệu
Một ứng dụng Spark sẽ luôn khởi tạo ra một {{site.data.glossary.driver}} trên {{site.data.glossary.master_node}} (để quản lý các {{site.data.glossary.job}}), rồi chỉ đạo các {{site.data.glossary.executor}} chạy trên các {{site.data.glossary.worker_node}} (để thực hiện các {{site.data.glossary.task}}) như mô hình sau:

![]({{ "/assets/images/B05793_01_02.jpg" | relative_url }})

<span class="text-capitalize">{{ site.data.glossary.driver}}</span> sẽ quyết định số lượng và việc phân bổ các {{site.data.glossary.task}} cho các {{site.data.glossary.executor}} dựa trên một mô hình đồ thị được sinh ra cho mỗi {{site.data.glossary.job}}. Chú ý là mỗi {{site.data.glossary.worker_node}} có thể xử lý nhiều {{site.data.glossary.task}} của nhiều {{site.data.glossary.job}} khác nhau.

Mỗi {{site.data.glossary.job}} trong Spark là một chuỗi của các hành động có liên quan chéo đến nhau, được định hình trong một {{site.data.glossary.direct_acyclic_graph}} (DAG) như ví dụ dưới đây được lấy ra từ Spark UI. Với đồ thị này, Spark có thể sắp xếp công việc một cách tối ưu nhất (ví dụ như tính toán ra có bao nhiêu {{site.data.glossary.task}} và cần bao nhiêu {{site.data.glossary.worker_node}} để chạy) và thực thi các {{site.data.glossary.task}} đó:

![]({{ "/assets/images/B05793_01_03.jpg" | relative_url }})

[Xem thêm bài viết này để hiểu hơn về cơ chế hoạt động của {{site.data.glossary.dag}} ở đây <http://bit.ly/29WTiK8>]

## Resilient Distributed Dataset
Apache Spark được xây dựng dựa trên một tập hợp các đối tượng {{site.data.glossary.immutable}} trong Java Virtual Machine (JVM) gọi là {{site.data.glossary.resilient_distributed_dataset}} (viết tắt là RDD). Một điều đáng chú ý là trong trường hợp của Python, các đối tượng Python sẽ được lưu lại trong các đối tượng JVM này. Điểm này sẽ được bàn luận kỹ hơn về sau khi ta nói về {{site.data.glossary.rdd}} và {{site.data.glossary.dataframe}}. Những đối tượng này khiến cho {{site.data.glossary.job}} nào cũng sẽ được tính toán siêu nhanh. Các đối tượng {{site.data.glossary.rdd}} sẽ được tính toán, lưu tạm và ghi lại tất cả ngay trong bộ nhớ: một mô hình tính toán mang lại kết quả nhanh vượt trội so với các nền tảng phân tán truyền thống khác như Apache Hadoop.
