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
- Khái quát về cấu trúc dữ liệu {{site.data.glossary.rdd}} (RDD), {{site.data.glossary.dataframe}}, và {{site.data.glossary.dataset}}
- Khái quát về dự án tối ưu hoá Catalyst Optimizer và dự án Tungsten
- Khái quát về thiết kế kiến trúc của Spark 2.0
