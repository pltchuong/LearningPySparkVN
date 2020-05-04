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
- Tạo ra {{site.data.glossary.rdd}}
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
