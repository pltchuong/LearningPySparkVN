---
layout: post
number: Chương 1
title: Tìm hiểu về Spark
---

* TOC
{:toc}

Apache Spark là một nền tảng xử lý dữ liệu mã nguồn mở, vốn là một phần trong luận án tiến sĩ của Matei Zaharia ở trường Đại học UC Berkeley. Phiên bản đầu tiên của Spark được phát hành năm 2012. Sau đó vào năm 2013, Zaharia thành lập công ty Databricks và lên làm {{site.data.glossary.cto}}; anh cũng đồng thời là giảng viên của trường Đại học Stanford sau khi rời khỏi MIT. Cũng trong thời gian này, mã nguồn của Spark được trao cho tổ chức phần mềm mã nguồn mở Apache Software Foundation và trở thành một dự án vô cùng phát triển.

Apache Spark nhanh, dễ dùng, cho phép ta xử lý các kiểu dữ liệu vửa đa dạng vừa phức tạp khác nhau, bất kể là dữ liệu có cấu trúc, bán cấu trúc, dữ liệu {{site.data.glossary.streaming}}, hay là áp dụng cho {{site.data.glossary.machine_learning}} và {{site.data.glossary.data_science}} đều được. Đây cũng là một trong những dự án lớn nhất trong giới mã nguồn mở với hơn 1,000 lập trình viên đến từ hơn 250 công ty lớn nhỏ, đã có hơn 300,000 buổi thảo luận được tổ chức ở hơn 570 thành phố trên khắp thế giới.

Trong chương này, ta sẽ học những thành phần cơ bản nhất của Apache Spark. Ta sẽ nói về các khái niệm trong Spark Jobs và Spark APIs, giới thiệu về kiến trúc và tìm hiểu các khả năng của Spark 2.0.

Các chủ đề bao gồm:
- Apache Spark là gì?
- Spark Jobs và Spark APIs
- Khái quát về cấu trúc dữ liệu {{site.data.glossary.resilient_distributed_dataset}} (RDD), {{site.data.glossary.dataframe}}, và {{site.data.glossary.dataset}}
- Khái quát về Chương trình tối ưu hoá câu lệnh truy vấn Catalyst Optimizer và Chương trình tối ưu hoá tài nguyên máy tính Project Tungsten
- Khái quát về thiết kế kiến trúc của Spark 2.0

## Apache Spark là gì?

Apache Spark là một nền tảng truy vấn và xử lý dữ liệu phân tán mã nguồn mở. Nó linh hoạt và có khả năng mở rộng giống như MapReduce nhưng nhanh hơn rất nhiều: hơn 100 lần so với Apache Hadoop nếu đọc dữ liệu trên bộ nhớ và hơn 10 lần nếu đọc dữ liệu từ ổ cứng.

Ta có thể dễ dàng dùng Apache Spark để đọc, chuyển đổi và tổng hợp dữ liệu, cũng như chạy và triển khai các mô hình thống kê dữ liệu phức tạp. Ta có thể dùng Java, Scala, Python, R và SQL để truy xuất các hàm của Spark APIs. Apache Spark có thể được dùng để viết ra cả một ứng dụng hoàn chỉnh, hoặc có thể đóng gói thành thư viện để triển khai trên các hệ thống máy chủ, hoặc cũng có thể dùng cho các phân tích *ngắn gọn* thông qua các {{site.data.glossary.notebook}} (như Jupyter, Spark-Notebook, Databricks, và Apache Zeppelin).

Apache Spark cung cấp hàng loạt công cụ tương đương với các thư viện mà các {{site.data.glossary.data_analyst}}, các {{site.data.glossary.data_scientist}} hoặc các nhà nghiên cứu hay dùng như `pandas` trong Python, `data.frames` hoặc `data.tables` trong R. Mặc dù Spark DataFrame *khá tương đồng* với `pandas` hoặc `data.frames` / `data.tables`, nhưng ta phải biết là sẽ có những khác biệt nhất định nên đừng quá nóng vội. Những ai đã quen với SQL cũng có thể dễ dàng sử dụng ngôn ngữ này. Ngoài ra, đi kèm với Apache Spark là một vài thuật toán, vài mô hình phân tích thống kê đã được viết và tối ưu sẵn, và vài thư viện khác: MLlib và ML dành cho {{site.data.glossary.machine_learning}}, GraphX và GraphFrames dành cho xử lý dữ liệu đồ thị, và Spark Streaming để xử lý dữ liệu {{site.data.glossary.streaming}}. Spark cũng cho phép ta có thể dễ dàng kết hợp các thư viện này với nhau trong cùng một ứng dụng.

Apache Spark có thể chạy ngay trên máy tính cá nhân, nhưng cũng có thể dễ dàng được triển khai theo chế độ độc lập, hoặc thông qua YARN, hoặc qua Apache Mesos - trên hệ thống máy chủ nội bộ cũng được mà trên {{site.data.glossary.cloud}} cũng được. Nó có thể đọc và ghi từ nhiều các nguồn dữ liệu đa dạng khác nhau, đơn cử như HDFS, Apache Cassandra, Apache HBase, và S3:

![]({{ "/assets/images/B05793_01_01.jpg" | relative_url }})

Nguồn: Apache Spark is the Smartphone of Big Data <http://bit.ly/1QsgaNj>

> Xem thêm Apache Spark is the Smartphone of Big Data <http://bit.ly/1QsgaNj>

## Spark Jobs và Spark APIs
Trong phần này, ta sẽ xem qua về Spark Jobs và Spark APIs. Đây là những kiến thức nền tảng cho phần sau, kiến trúc của Spark 2.0.

### Quy trình xử lý dữ liệu
Một ứng dụng Spark sẽ luôn khởi tạo ra một {{site.data.glossary.driver}} trên {{site.data.glossary.master_node}} (để quản lý các {{site.data.glossary.job}}), rồi chỉ đạo các {{site.data.glossary.executor}} chạy trên các {{site.data.glossary.worker_node}} (để thực hiện các {{site.data.glossary.task}}) như mô hình sau:

![]({{ "/assets/images/B05793_01_02.jpg" | relative_url }})

<span class="text-capitalize">{{ site.data.glossary.driver}}</span> sẽ quyết định số lượng và việc phân bổ các {{site.data.glossary.task}} cho các {{site.data.glossary.executor}} dựa trên một mô hình đồ thị được sinh ra cho mỗi {{site.data.glossary.job}}. Chú ý là mỗi {{site.data.glossary.worker_node}} có thể xử lý nhiều {{site.data.glossary.task}} của nhiều {{site.data.glossary.job}} khác nhau.

Mỗi {{site.data.glossary.job}} trong Spark là một chuỗi của các hành động có liên quan chéo đến nhau, được định hình trong một {{site.data.glossary.direct_acyclic_graph}} (DAG) như ví dụ dưới đây được lấy ra từ Spark UI. Với đồ thị này, Spark có thể sắp xếp công việc một cách tối ưu nhất (ví dụ như tính toán ra có bao nhiêu {{site.data.glossary.task}} và cần bao nhiêu {{site.data.glossary.worker_node}} để chạy) và thực thi các {{site.data.glossary.task}} đó:

![]({{ "/assets/images/B05793_01_03.jpg" | relative_url }})

> Xem thêm bài viết này để hiểu hơn về cơ chế hoạt động của {{site.data.glossary.dag_scheduler}} <http://bit.ly/29WTiK8>

### Resilient Distributed Dataset
Apache Spark được xây dựng dựa trên một tập hợp các dữ liệu {{site.data.glossary.immutable}} trong Java Virtual Machine (JVM) gọi là {{site.data.glossary.resilient_distributed_dataset}} (viết tắt là RDD). Một điều đáng chú ý là trong trường hợp của Python, các đối tượng Python sẽ được lưu lại trong các đối tượng JVM này. Điểm này sẽ được bàn luận kỹ hơn về sau khi ta nói về {{site.data.glossary.rdd}} và {{site.data.glossary.dataframe}}. Những đối tượng này khiến cho {{site.data.glossary.job}} nào cũng sẽ được tính toán siêu nhanh. Các đối tượng {{site.data.glossary.rdd}} sẽ được tính toán, lưu tạm và ghi lại tất cả ngay trong bộ nhớ: một mô hình tính toán mang lại kết quả nhanh vượt trội so với các nền tảng phân tán truyền thống khác như Apache Hadoop.

Mặt khác, {{site.data.glossary.rdd}} cung cấp một vài hàm cơ bản (như là `map(...)`, `reduce(...)`, and `filter(...)`, ta sẽ nhắc lại chi tiết hơn trong Chương 2, *{{site.data.glossary.resilient_distributed_dataset}}*) để giữ nguyên tính cơ động và khả năng mở rộng sẵn có của Hadoop, từ đó có thể thực hiện một cơ số các loại tính toán khác. {{site.data.glossary.rdd}} thực hiện việc biến đổi dữ liệu bằng các tiến trình chạy song song với nhau, vừa nhanh vừa tránh mất dữ liệu. Bằng việc ghi chép lại quá trình biến đổi này, {{site.data.glossary.rdd}} sẽ nắm được lịch sử thay đổi của dữ liệu - một đồ thị dạng cây lưu lại chi tiết từng bước chuyển đổi một. Hiệu quả của việc này là {{site.data.glossary.rdd}} sẽ không bị mất dữ liệu trong quá trình tính toán - nếu lỡ một mảnh nhỏ của một {{site.data.glossary.rdd}} bị mất, nó sẽ có đủ thông tin để có thể tự tái tạo lại mảnh dữ liệu đó, thay vì phải chạy đi chỗ khác để tìm kiếm một bản sao của mảnh bị mất.

> Xem thêm bài viết này để hiểu hơn về lịch sử thay đổi của dữ liệu <http://ibm.co/2ao9B1t>

{{site.data.glossary.rdd}} có hai loại tiến trình: *biến đổi* (sẽ trả về con trỏ sang một {{site.data.glossary.rdd}} mới) và *thực thi* (sẽ trả về kết quả cho {{site.data.glossary.driver}} sau khi tính toán); ta sẽ tìm hiểu sâu hơn ở các chương sau.

> Xem thêm Hướng dẫn lập trình Spark ở đây <http://spark.apache.org/docs/latest/programming-guide.html#rdd-operations> để biết thêm về các tiến trình này

Quá trình biến đổi của {{site.data.glossary.rdd}} được gọi là *lười*, ý nói rằng nó không đưa ra kết quả ngay lập tức. Những biến đổi này chỉ được tính toán khi có một tiến trình thực thi được gọi, và kết quả sẽ được trả về cho {{site.data.glossary.driver}}. Kết quả của việc hoãn thực thi này là hiệu suất của các câu lệnh truy vấn có thể được tối ưu hoá hơn rất nhiều. {{site.data.glossary.dag_scheduler}} trong Apache Spark là nơi bắt đầu quá trình tối ưu hoá này - đây là chương trình chuyên để sắp xếp các biến đổi này theo từng {{site.data.glossary.stage}} như hình trên. Và vì các tiến trình trong {{site.data.glossary.rdd}} được chia ra làm *biến đổi* và *thực thi*, nên {{site.data.glossary.dag_scheduler}} có thể thoải mái tối ưu câu truy vấn mà không cần phải xáo trộn toàn bộ dữ liệu (vốn là thao tác tốn tài nguyên nhất)

Để biết thêm thông tin về {{site.data.glossary.dag_scheduler}} và việc tối ưu hoá, vui lòng tham khảo thêm cuốn *High Performance Spark*, Chương 5 *Effective Transformations*, phân *Narrow vs. Wide Transformations* <https://smile.amazon.com/High-Performance-Spark-Practices-Optimizing/dp/1491943203>.

### DataFrames
Cũng như {{site.data.glossary.rdd}}, {{site.data.glossary.dataframe}} là một tập hợp dữ liệu {{site.data.glossary.immutable}} được phân tán trên khắp các {{site.data.glossary.dag_scheduler}} trong một hệ thống máy chủ. Điểm khác biệt là dữ liệu trong {{site.data.glossary.dataframe}} được sắp xếp theo tên theo cột.

> Ai quen thuộc với `pandas` trong Python hoặc `data.frames` trong R sẽ thấy định nghĩa này hoàn toàn tương tự

{{site.data.glossary.dataframe}} được thiết kế để việc xử lý được lượng dữ liệu lớn còn dễ dàng hơn nữa. Lập trình viên giờ đã có thể dựng lên được cấu trúc của dữ liệu, cho phép trừu tượng hoá dữ liệu ở mức độ cao hơn; hiểu theo nghĩa khác thì {{site.data.glossary.dataframe}} tương đương với khái niệm bảng trong thế giới cơ sở dữ liệu quan hệ. {{site.data.glossary.dataframe}} có các API đặc biệt để thao tác với dữ liệu được phân tán, khiến Spark thân thiện hơn với nhiều chuyên gia khác chứ không chỉ mỗi giới {{site.data.glossary.data_engineer}}.

Một trong những lợi ích lớn nhất mà {{site.data.glossary.dataframe}} mang lại là Spark đầu tiên sẽ dựng lên một {{site.data.glossary.logical_plan}}, sau đó {{site.data.glossary.cost_optimizer}} sẽ tính toán ra một {{site.data.glossary.physical_plan}} và sinh ra các đoạn mã tối ưu, cuối cùng Spảk sẽ chạy những đoạn mã này. Không như {{site.data.glossary.rdd}} khi mà Python chạy chậm hơn đáng kể so với Java hoặc Scala, {{site.data.glossary.dataframe}} mang lại hiệu năng tương đương bất kể là ngôn ngữ nào.

### Datasets
Được giới thiệu từ Spark 1.6, mục tiêu của Spark {{site.data.glossary.dataset}} là đưa ra một API vừa dễ áp dụng trên dữ liệu, vừa đạt hiệu suất tốt, vừa thừa kế bộ máy tuyệt vởi của Spark SQL. Đáng tiếc là vào thời điểm quyển sách này được viết ra, {{site.data.glossary.dataset}} chỉ có trên Scala hoặc Java. Một khi PySpark hỗ trợ, chúng tôi sẽ cập nhật trong lần tái bản sau.

### Catalyst Optimizer
Spark SQL là một trong những thành phần trong Spark có kỹ thuật phức tạp nhất, vì đây là khởi nguồn sức mạnh của các câu lệnh truy vấn và {{site.data.glossary.dataframe}}. Trung tâm của Spark SQL là chương trình tối ưu hoá câu lệnh truy vấn Catalyst Optimizer. Dựa trên ý tưởng của {{site.data.glossary.functional_programming}}, chương trình này được thiết kế ra với hai mục tiêu: Tích hợp các phương pháp tối ưu hoá mới, các chức năng mới vào Spark SQL và cho phép lập trình viên có thể mở rộng thêm (như là thêm vào các quy tắc riêng biệt tuỳ nguồn dữ liệu, hỗ trợ kiểu dữ liệu mới, v.v...)

![]({{ "/assets/images/B05793_01_04.jpg" | relative_url }})

> Xem thêm *Deep Dive into Spark SQL's Catalyst Optimizer* <http://bit.ly/271I7Dk> và *Apache Spark DataFrames: Simple and Fast Analysis of Structured Data* <http://bit.ly/29QbcOV>

### Project Tungsten

Tungsten là tên một dự án tổng hợp của bộ máy xử lý trong Apache Spark. Dự án này tập trung vào việc cải thiện để các thuật toán trong Spark sử dụng bộ nhớ và vi xử lý hiệu quả hơn, sử dụng tối đa hiệu năng của các phần cứng hiện đại.

Dự án này gồm:
- Loại bỏ chi phí quản lý của các đối tượng trong JVM, loại bỏ luôn bộ thu gom rác trong Java, từ đó tối ưu việc sử dụng bộ nhớ.
- Tận dụng bộ nhớ phân cấp để thiết kế các thuật toán và cấu trúc dữ liệu phù hợp.
- Tận dụng các trình biên dịch mới để tự sinh ra các đoạn mã, từ đó tối ưu việc sử dụng vi xử lý.
- Loại bỏ việc điều phối các hàm ảo, từ đó giảm số lượng gọi các vi xử lý.
- Tận dụng kỹ thuật lập trình bậc thấp (như tải thẳng dữ liệu vào các thanh ghi của vi xử lý) để tăng tốc độ truy cập bộ nhớ, từ đó cho phép bộ máy của Spark tăng hiệu quả của việc biên dịch và chạy các vòng lặp đơn giản.

> Xem thêm

> *Project Tungsten: Bringing Apache Spark Closer to Bare Metal* <https://databricks.com/blog/2015/04/28/project-tungsten-bringing-spark-closer-to-bare-metal.html>

> *Deep Dive into Project Tungsten: Bringing Spark Closer to Bare Metal [SSE 2015 Video and Slides]* <https://spark-summit.org/2015/events/deep-dive-into-project-tungsten-bringing-spark-closer-to-bare-metal/>

> *Apache Spark as a Compiler: Joining a Billion Rows per Second on a Laptop* <https://databricks.com/blog/2016/05/23/apache-spark-as-a-compiler-joining-a-billion-rows-per-second-on-a-laptop.html>
