---
layout: post
number: Chương 1
title: Tìm hiểu về Spark
---

Apache Spark là một nền tảng xử lý dữ liệu mã nguồn mở, vốn là một phần luận án tiến sĩ của Matei Zaharia ở trường Đại học UC Berkeley. Phiên bản đầu tiên của Spark được phát hành từ năm 2012. Sau đó vào năm 2013, Zaharia thành lập công ty Databricks và lên làm Giám đốc công nghệ; anh cũng đồng thời là giảng viên của trường Đại học Stanford sau khi rời khỏi MIT. Cũng trong thời gian này, mã nguồn của Spark được trao cho tổ chức phần mềm mã nguồn mở Apache Software Foundation và trở thành một dự án vô cùng phát triển.

Apache Spark nhanh, dễ dùng, cho phép ta xử lý các kiểu dữ liệu đa dạng phức tạp khác nhau, từ dữ liệu có cấu trúc, bán cấu trúc, dữ liệu liên tục, cho đến phân tích dữ liệu và khoa học dữ liệu. Đây cũng là một trong những dự án lớn nhất trong giới mã nguồn mở với hơn 1,000 lập trình viên đến từ hơn 250 công ty lớn nhỏ, đã có hơn 300,000 buổi thảo luận được tổ chức ở hơn 570 thành phố trên khắp thế giới.

Trong chương này, ta sẽ học những yếu tố cơ bản nhất để có thể hiểu về Apache Spark. Ta sẽ nói về các khái niệm trong Spark Jobs và Spark APIs, giới thiệu về kiến trúc và tìm hiểu các khả năng của Spark 2.0.

Các chủ đề bao gồm:
- Apache Spark là gì?
- Spark Jobs và Spark APIs
- Khái quát về cấu trúc dữ liệu Resilient Distributed Dataset (RDD), DataFrame, và Dataset
- Khái quát về cơ chế tối ưu hoá Catalyst Optimizer và dự án Tungsten
- Khái quát về thiết kế kiến trúc của Spark 2.0

## Apache Spark là gì?

Apache Spark là một nền tảng truy vấn và xử lý dữ liệu phân tán mã nguồn mở. Nó mang đầy đủ tính linh hoạt và khả năng mở rộng của MapReduce nhưng nhanh hơn rất nhiều: hơn 100 lần so với Apache Hadoop nếu đọc dữ liệu trên bộ nhớ và hơn 10 lần nếu đọc dữ liệu từ ổ cứng.

Ta có thể dễ dàng dùng Apache Spark để đọc, tổng hợp dữ liệu, cũng như chạy và triển khai các mô hình thống kê dữ liệu. Các hàm Spark APIs được viết bằng Java, Scala, Python, R và SQL. Apache Spark có thể được dùng để viết cả một ứng dụng hoàn chỉnh, hoặc có thể đóng gói thành thư viện để triển khai trên các hệ thống máy chủ, hoặc cũng có thể dùng cho các phân tích *ngắn gọn* thông qua các notebooks (như Jupyter, Spark-Notebook, Databricks notebook, và Apache Zeppelin).

Apache Spark cung cấp hàng loạt công cụ tương đương với các thư viện mà các chuyên gia phân tích dữ liệu, các nhà khoa học dữ liệu hoặc các nhà nghiên cứu hay dùng như `pandas` trong Python, `data.frames` hoặc `data.tables` trong R. Mặc dù Spark DataFrame *khá tương đồng* với `pandas` hoặc `data.frames` / `data.tables`, nhưng ta phải biết là sẽ có những khác biệt nhất định nên cũng đừng trông chờ nhiều quá. Những ai đã quen với SQL cũng có thể dễ dàng sử dụng ngôn ngữ này. Ngoài ra, đi kèm với Apache Spark là một vài thuật toán, vài mô hình phân tích thống kê đã được viết và tối ưu sẵn, và vài thư viện khác: MLlib và ML dành cho máy học tự động, GraphX và GraphFrames dành cho xử lý dữ liệu đồ thị, và Spark Streaming để xử lý dữ liệu liên tục. Các thư viện này có thể được kết hợp dễ dàng cùng nhau trong cùng một ứng dụng.

Apache Spark có thể chạy ngay trên máy tính cá nhân, nhưng cũng có thể dễ dàng được triển khai độc lập, hoặc thông qua YARN, hoặc qua Apache Mesos - trên hệ thống máy chủ nội bộ cũng được mà trên mây cũng được. Nó có thể đọc và ghi từ nhiều loại nguồn dữ liệu khác nhau ví dụ như HDFS, Apache Cassandra, Apache HBase, và S3:

![]({{ "/assets/images/B05793_01_01.jpg" | relative_url }})

Nguồn: Apache Spark chính là chiếc máy điện thoại thông minh trong giới Dữ liệu lớn http://bit.ly/1QsgaNj

[Xem thêm bài viết Apache Spark chính là chiếc máy điện thoại thông minh trong giới Dữ liệu lớn ở đây http://bit.ly/1QsgaNj]

## Spark Jobs và Spark APIs
Trong phần này, ta sẽ xem xét khái quát về Spark Jobs và Spark APIs. Đây là những kiến thức nền tảng cho phần sau, kiến trúc của Spark 2.0

### Quy trình xử lý dữ liệu
Một ứng dụng Spark sẽ luôn khởi tạo ra một {{site.data.glossary.driver}} (để nắm các {{site.data.glossary.task}}) trên một {{site.data.glossary.master_node}}, rồi chỉ đạo các {{site.data.glossary.executor}} (để thực hiện các {{site.data.glossary.task}}) chạy trên các {{site.data.glossary.worker_node}} như mô hình sau:

![]({{ "/assets/images/B05793_01_02.jpg" | relative_url }})

Driver sẽ quyết định số lượng và việc phân bổ các task trực tiếp cho các executor dựa trên mô hình đồ thị tương ứng cho mỗi job. Chú ý là worker node có thể xử lý nhiều task của nhiều jobs khác nhau.

Mỗi đầu việc *job* trong Spark là một chuỗi của các hành động liên quan chéo đến nhau, được định hình trong một đồ thị có định hướng không tuần hoàn *direct acyclic graph* (DAG) như ví dụ dưới đây được lấy ra từ Spark UI. Với đồ thị này, Spark có thể sắp xếp công việc một cách tối ưu nhất (ví dụ như tính toán ra có bao nhiêu task và cần bao nhiêu nút để chạy) và thực thi các task đó:

[image]

[Xem thêm bài viết này để hiểu hơn về cơ chế hoạt động của DAG http://bit.ly/29WTiK8]

### Resilient Distributed Dataset
Apache Spark được xây dựng dựa trên một tập hợp các đối tượng Java bất biến *immutable* gọi là Bộ dữ liệu phân tán có khả năng tự phục hồi *Resilient Distributed Datasets* (RDD). Một điều đáng chú ý là Trong trường hợp của Python, các đối tượng trong Python sẽ được


Apache Spark is built around a distributed collection of immutable Java Virtual Machine (JVM) objects called Resilient Distributed Datasets (RDDs for short). As we are working with Python, it is important to note that the Python data is stored within these JVM objects. More of this will be discussed in the subsequent chapters on RDDs and DataFrames. These objects allow any job to perform calculations very quickly. RDDs are calculated against, cached, and stored in-memory: a scheme that results in orders of magnitude faster computations compared to other traditional distributed frameworks like Apache Hadoop.