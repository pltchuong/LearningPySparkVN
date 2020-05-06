---
layout: post
number: Chương 2
title: Resilient Distributed Dataset
---

* TOC
{:toc}

{{site.data.glossary.resilient_distributed_dataset}} (RDD) là một tập hợp các đối tượng bất biến chứa dữ liệu trong {{site.data.glossary.jvm}} cho phép ta thực hiện các thao tác tính toán rất nhanh, chúng chính là *xương sống* của Apache Spark.

Cái tên đã nói lên tất cả, dữ liệu được phân tán; chúng được chia thành những phần nhỏ dựa trên một vài yếu tố và được phân tán ra các {{site.data.glossary.worker_node}}. Như vậy những tính toán cũng được chia nhỏ ra và chạy rất nhanh. Đồng thời, như đã nói ở *Chương 1, Tìm hiểu về Spark*, {{site.data.glossary.resilient_distributed_dataset}} lưu lại toàn bộ quá trình biến đổi trên mỗi phần để tính toán nhanh hơn và đưa ra phương pháp thay thế nếu nhỡ có gì sai sót và phần dữ liệu đó bị mất đi; trong trường hợp đó, {{site.data.glossary.resilient_distributed_dataset}} sẽ tự tính toán lại dữ liệu. Nắm lịch sử thay đổi của dữ liệu là một bước tiến để chống lại việc mất dữ liệu, bổ sung thêm cho việc sao chép dữ liệu.

Các chủ đề trong chương này bao gồm:
- Cơ chế làm việc bên trong {{site.data.glossary.rdd}}
- Khởi tạo {{site.data.glossary.rdd}}
- Phạm vi rộng và phạm vi hẹp
- Tiến trình biến đổi
- Tiến trình thực thi

## Cơ chế làm việc bên trong RDD
Các {{site.data.glossary.rdd}} được xử lý song song với nhau. Đây là lợi thế lớn nhất khi làm việc với Spark: Các biến đổi đều được thực hiện đồng thời khiến tốc độ tăng lên kỷ lục.

Các biến đổi bên trong dữ liệu đều lười. Nghĩa là tất cả những biến đổi này chỉ được thực hiện khi có một tiến trình thực thi được gọi. Điều này cho phép Spark tối ưu hoá quá trình thực thi. Ví dụ, hãy thử xem các bước rất cơ bản dưới đây khi một {{site.data.glossary.data_analyst}} làm quen với dữ liệu:

1. Đếm các giá trị khác biệt trong từng cột.
2. Chọn những giá trị nào bắt đầu với `A`.
3. Hiển thị kết quả ra màn hình.

Những bước trên nghe vô cùng đơn giản, nếu ta chỉ quan tâm đến những thành phần nào bắt đầu với chữ `A`, chả có lý do gì ta phải đếm tất cả các thành phần khác để làm gì. Thế nên, thay vì thực thi theo đúng trình tự đã định, Spark chỉ cần đếm những thành phần nào bắt đầu với `A`, rồi hiển thị kết quả ra màn hình.

Giờ ta thử viết code cho ví dụ này. Đầu tiên, ta yêu cầu Spark lưu tạm lại các giá trị của `A` bằng hàm `.map(lambda v: (v, 1))`, sau đó ta chọn những bản ghi nào bắt đầu với `A` bằng hàm `.filter(lambda val: val.startswith('A'))`. Nếu giờ ta gọi hàm `.reduceByKey(operator.add)`, nó sẽ giảm dần lượng dữ liệu và cộng vào (trong trường hợp này là đếm) số lần xuất hiện của mỗi giá trị. Tất cả những bước này đều nhằm **biến đổi** dữ liệu.

Thứ hai, ta gọi hàm `.collect()` để thực thi tất cả những bước trên. Đây chính là bước thực thi trên dữ liệu ta có - giờ nó mới thực sự đếm những giá trị khác biệt của dữ liệu. Kết quả là, tiến trình thực thi có thể thay đổi thứ tự của quá trình biến đổi, lọc dữ liệu trước khi lưu tạm, cuối cùng tập dữ liệu được thu gọn lại và được đưa vào tổng hợp.

> Đừng lo lắng quá nếu những hàm này có vẻ quá khó hiểu - ta sẽ nói rất kỹ về chùng trong chương này.

## Khởi tạo RDD
Có hai cách để tạo ra một {{site.data.glossary.rdd}} trong PySpark: ta có thể áp dụng hàm `.parallelize(...)` cho một tập hợp (`list` hoặc `array`):

```python
data = sc.parallelize(
    [('Amber', 22), ('Alfred', 23), ('Skye',4), ('Albert', 12),
     ('Amber', 9)])
```

Hoặc ta cũng có thể đọc từ một hoặc nhiều file, nội bộ hoặc bên ngoài:

```python
data_from_file = sc.\
    textFile(
        '/Users/drabast/Documents/PySpark_Data/VS14MORT.txt.gz',
        4)
```

> Ta dùng file `VS14MORT.txt` từ bộ dữ liệu Mortality được tải (vào ngày 31 tháng 7 năm 2016) từ <ftp://ftp.cdc.gov/pub/Health_Statistics/NCHS/Datasets/DVS/mortality/mort2014us.zip>; lược đồ của dữ liệu này nằm ở đây <http://www.cdc.gov/nchs/data/dvs/Record_Layout_2014.pdf>. Sở dĩ ta chọn bộ dữ liệu này là vì: Nó có những dữ liệu bị mã hoá mà ở phần sau của chương này ta sẽ dùng UDF để biến đổi lại. Ta cũng có thể tải file này ở đây <http://tomdrabas.com/data/VS14MORT.txt.gz>

Tham số cuối cùng của hàm `sc.textFile(..., n)` chỉ số phần của dữ liệu sẽ được chia nhỏ ra.

> Có một nguyên tắc là nên chia nhỏ dữ liệu ra làm hai đến bốn phần ra để đọc.

Spark có thể đọc dữ liệu từ vô số hệ thống file khác nhau: hệ thống file trên máy tính cá nhân như NTFS, FAT, hoặc Mac OS Extended (HFS+), hoặc các hệ thống file phân tán như HDFS, S3, Cassandra, và rất nhiều nguồn khác nữa.

> Hãy cẩn trọng với nơi ta sẽ đọc và ghi dữ liệu: đường dẫn không thể chứa các ký tự đặc biệt `[]`. Điều này cũng đúng với các đường dẫn trên Amazon S3 và Microsoft Azure Data Storage.

Các định dạng dữ liệu được hỗ trợ bao gồm: Văn bản chữ, parquet, JSON, bảng Hive, và dữ liệu từ các cơ sở dữ liệu quan hệ thông qua JDBC driver. Chú ý là Spark có thể làm việc trực tiếp với các dữ liệu được nén (như trong ví dụ trên, dữ liệu được nén bằng Gzip).

Tuỳ vào việc dữ liệu được đọc như thế nào, đối tượng trả về sẽ hơi khác nhau một chút. Dữ liệu đọc từ file sẽ nằm trong `MapPartitionsRDD`, trong khi dữ liệu từ `.paralellize(...)` sẽ nằm trong `ParallelCollectionRDD`

### Lược đồ dữ liệu
{{site.data.glossary.rdd}} là cấu trúc dữ liệu *không có lược đồ* (không như {{site.data.glossary.dataframe}}, ta sẽ nói ở chương sau). Cho nên khi đọc dữ liệu, như trong đoạn code ví dụ sau, là hoàn toàn hợp lệ trong Spark khi sử dụng {{site.data.glossary.rdd}}:

```python
data_heterogenous = sc.parallelize([
    ('Ferrari', 'fast'),
    {'Porsche': 100000},
    ['Spain','visited', 4504]
]).collect()
```
Tức là, ta có thể trộn lẫn hầu như tất cả mọi kiểu với nhau: `tuple`, `dict` hoặc `list`, Spark sẽ không phàn nàn gì hết.

Một khi gọi `.collect()` trên dữ liệu này (tức là chạy một tiến trình thực thi để gửi dữ liệu về {{site.data.glossary.driver}}), ta có thể truy cập các dữ liệu này như bình thường trên Python:

```python
data_heterogenous[1]['Porsche']
```

Nó sẽ trả ra kết quả như sau:

```python
100000
```

Hàm `.collect()` trả về {{site.data.glossary.driver}} tất cả các phần tử của {{site.data.glossary.rdd}} dưới dạng `list`

> Ta sẽ bàn thêm về một vài lưu ý khi dùng `.collect()` ở phần sau chương này.

### Đọc dữ liệu từ file
Khi đọc dữ liệu từ một file văn bản, mỗi dòng trong file sẽ tương ứng với một phần tử của {{site.data.glossary.rdd}}

Câu lệnh `data_from_file.take(1)` sẽ trả vê kết quả như sau (hơi khó đọc chút):

![]({{ "/assets/images/B05793_02_01.jpg" | relative_url }})

Để dễ đọc hơn, ta sẽ tạo ra một danh sách các phần tử sao cho mỗi dòng là một danh sách các giá trị.

### Các biểu thức Lambda

Trong ví dụ sau, ta sẽ trích xuất vài thông tin có ý nghĩa từ đống dữ liệu trông có vẻ được mã hoá trong `data_from_file`

> Bạn có thể tham khảo GitHub repository của chúng tôi cho quyển sách này để hiểu kỹ hơn về cách làm này. Ở đây vì không gian hạn chế, chúng tôi chỉ có thể diễn giải một phiên bản rất ngắn gọn, nhất là phần tạo ra regex pattern. Code có thể tim thấy ở đây <https://github.com/drabastomek/learningPySpark/tree/master/Chapter03/LearningPySpark_Chapter03.ipynb>

Đầu tiên, ta cần tạo ra một hàm như sau để bóc tách các bản ghi ra thành những thứ ta có thể sử dụng được:

```python
def extractInformation(row):
    import re
    import numpy as np
    selected_indices = [
         2,4,5,6,7,9,10,11,12,13,14,15,16,17,18,
         ...
         77,78,79,81,82,83,84,85,87,89
    ]
    record_split = re\
        .compile(
            r'([\s]{19})([0-9]{1})([\s]{40})'
            ...
            '([\s]{33})([0-9\s]{3})([0-9\s]{1})([0-9\s]{1})')
    try:
        rs = np.array(record_split.split(row))[selected_indices]
    except:
        rs = np.array(['-99'] * len(selected_indices))
    return rs
```

> Cần phải đặt một cảnh bảo ở đây. Tự định nghĩa ra một hàm Python rất có khả năng sẽ làm chậm chương trình của ta lại vì Spark cần phải liên tục chuyển qua chuyển lại giữa trình thông dịch Python và {{site.data.glossary.jvm}}. Bất cứ lúc nào có thể, ta đều nên dùng các hàm được dựng sẵn trong Spark.

Tiếp, ta cần nhúng các thư viện cần thiết: Thư viện `re` vì ta sẽ dùng các biểu thức chính quy để bóc tách dữ liệu, và thư viện `NumPy` để giúp ta cùng lúc chọn nhiều phần tử.

Cuối cùng, ta tạo ra một đối tượng `Regex` để tách thông tin như mong muốn rồi dùng nó để bóc dữ liệu.

> Ta sẽ không đào sâu vào chi tiết về các biểu thức chính quy ở đây. Tổng hợp các chủ đề liên quan có thể tìm thấy ở đây <https://www.packtpub.com/application-development/mastering-python-regular-expressions>.

Một khi bản ghi đã được bóc tách, ta sẽ biến nó thành một mảng `NumPy` và trả về; nếu bị hỏng ta sẽ trả về một danh sách các giá trị mặc định `-99` để biết là bản ghi này không được bóc tách đúng.

> Ta cũng có thể chủ ý lọc toàn bộ các bản ghi hingr bằng cách sử dụng hàm `.flatMap(...)` và trả về một mảng rỗng `[]` thay vì các giá trị `-99`. Xem chi tiết ở đây <http://stackoverflow.com/questions/34090624/remove-elements-from-spark-rdd>

Giờ, ta sẽ dùng hàm `extractInformation(...)` để chia tách và biến đổi dữ liệu. Chú ý là ta chỉ truyền mỗi tên hàm vào `.map(...)`: các phần tử của {{site.data.glossary.rdd}} từng cái một của từng phần một sẽ đường *truyền* vào hàm `extractInformation(...)`

```python
data_from_file_conv = data_from_file.map(extractInformation)
```

Chạy câu lệnh `data_from_file_conv.take(1)` sẽ đưa ra kết quả như sau (đã được thu gọn lại):

![]({{ "/assets/images/B05793_02_02.jpg" | relative_url }})

## Global versus local scope
Một trong những cái mà ta, một lập trình viên PySpark trong tương lai, phải làm quen đó là đặc tính xử lý song song của Spark. Dù cho ta có quen với Python thế nào đi chăng nữa, muốn viết và chạy code PySpark ta phải thay đổi cách suy nghĩ một chút.

Spark có thể chạy trên hai chế độ: local mode và cluster mode. Code khi chạy Spark với chế độ local mode không khác lắm so với code khi chạy Python: thay đổi chủ yếu chỉ là cú pháp viết code, với chút xoắn não là cả dữ liệu và code có thể được gửi qua lại giữa các worker processes.

Tuy nhiên, mang nguyên chỗ code đó và triển khai lên một cluster có thể sẽ khiến ta vò đầu bứt tai nếu không cẩn thận. Ta buộc phải hiểu cách Spark chạy một job trên cluster.

Trong cluster mode, khi một job được đưa lên để chạy, job sẽ được gửi tới driver (còn gọi là master) node. Driver node sẽ tạo ra một DAG (*xem lại Chương 1, Tìm hiẻu về Spark*) cho job đó và quyết định executor (còn gọi là worker) node nào sẽ thực hiện task cụ thể nào.

Driver sau đó sẽ gửi các chỉ thị đến cho các worker để thực hiện các từng task yêu cầu gửi trả lại kết quả cho driver khi làm xong. Tuy nhiên trước đó, driver phải chuẩn bị từng nội dung từng task: Một tập hợp các giá trị và hàm hiện đang nằm trên driver sẽ được phân phối đến worker để worker làm việc trên các dữ liệu từ RDD.

Tập hợp các giá trị và hàm này được được xem là *tĩnh* trong tình trạng của các executor, tức là, mỗi executor được nhận một *bản sao* của tập hợp các giá trị và hàm từ driver. Nếu, trong khi đang làm viêc, executor cần phải thay đổi các giá trị hoặc ghi đè các hàm, nó sẽ **không** gây ảnh hưởng đến những bản sao trên các executor còn lại cũng như bản gốc trên driver. Việc này có khả năng gây ra một vài tình huống không lường trước được và những bug chỉ xảy ra lúc chạy thật này thi thoảng rất khó đề dò ra.

> Check out this discussion in PySpark's documentation for a more hands-on example: <http://spark.apache.org/docs/latest/programming-guide.html#local-vs-cluster-modes>.
