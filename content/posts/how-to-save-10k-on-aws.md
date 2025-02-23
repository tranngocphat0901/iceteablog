---
title: "Làm sao tôi giúp công ty cắt giảm $10,000/tháng tiền AWS"
date: 2025-02-22T14:00:00Z
draft: false
tags: ["AWS", "Cost Optimization", "Cloud"]
categories: ["Cloud Architecture", "DevOps"]
author: "Patrick Tran"
summary: "Chia sẻ câu chuyện thực tế về cách giảm $10,000/tháng tiền AWS thông qua tối ưu tài nguyên, giám sát chi phí và thay đổi văn hóa sử dụng."
cover:
  image: "/images/how-to-save-10k-on-aws.jpg"  # Image path
  alt: "Saving 10k USD on AWS Cover Image"  # Alternative text if the image fails to load
  caption: "How to saving cost on AWS Cloud"  # Caption below the image (optional)
  relative: true  # If true, the image is located in the static/ directory
  hidden: false  # If true, the image will not be displayed in the post, only on the post list
---

## Giới Thiệu

Tôi nhớ thời điểm cách đây không lâu, khi công ty bắt đầu mở rộng quy mô sản phẩm trên nền tảng AWS, chi phí điện toán “phình to” như một quả bóng bay mỗi cuối tháng. Ban đầu, nó chỉ là những con số năm, bảy nghìn đô, rồi nhanh chóng leo thang vượt mức 20.000 đến 30.000 đô la mỗi tháng. Lúc đó, cấp trên đề nghị tôi “soi” lại toàn bộ hệ thống để **cắt giảm ít nhất 10.000 đô**. Bằng những bước tối ưu hóa cẩn thận, tôi đã giúp công ty tiết kiệm thành công khoản tiền khổng lồ hàng tháng. Trong bài viết này, tôi sẽ chia sẻ hành trình đó và các “mẹo” áp dụng để bạn có thể học hỏi và triển khai cho chính dự án của mình.

Bài viết này sẽ đi sâu vào các **chiến lược** và **thủ thuật** cụ thể, bao gồm:

1.  **Nhận diện dịch vụ AWS “ngốn tiền”**
    
2.  **Tối ưu hóa EC2**
    
3.  **Tận dụng Spot Instances, Reserved Instances, và Savings Plan**
    
4.  **Tối ưu chi phí lưu trữ S3 và EBS**
    
5.  **Kiểm soát băng thông và Data Transfer**
    
6.  **Tắt tài nguyên không dùng, giảm quy mô tài nguyên môi trường dev/test**
    
7.  **Theo dõi, giám sát chi phí liên tục**
    

Hãy cùng bắt đầu!

## 1\. Nhận diện dịch vụ AWS “ngốn tiền”
Mọi “cuộc săn” đều bắt đầu bằng việc biết rõ **mình đang săn gì**. Trên AWS, việc đầu tiên tôi làm là sử dụng **AWS Cost Explorer** và **AWS Budgets** để xem chi phí bị “đốt” nhiều nhất nằm ở đâu. Khi nhìn vào bảng thống kê, có ba danh mục nổi lên hàng đầu:

1.  **Amazon EC2 (Elastic Compute Cloud)**: Phần “xương sống” cho việc chạy máy chủ, container, và batch processing.
    
2.  **Amazon S3 (Simple Storage Service)**: Lưu trữ rất nhiều dữ liệu và thường bị “phình” khi đội ngũ phát triển quên dọn các bucket cũ.
    
3.  **Data Transfer**: Kinh điển nhất là chi phí liên quan đến truyền dữ liệu giữa các vùng (region), giữa các dịch vụ, hoặc đẩy ra Internet.
    

Ngoài ra, một số **kẻ thầm lặng** khác cũng khiến hóa đơn tăng như: **Amazon RDS**, **AWS Lambda** (khi chạy ở quy mô lớn), và **Amazon CloudFront**. Lần theo dấu vết, tôi thấy rằng phần lớn “thủ phạm” chính là EC2 và S3. Từ đây, tôi lên một **bảng phân tích** để xác định:

*   **EC2**: Chạy những instance nào? Bao nhiêu lâu? CPU/RAM sử dụng trung bình ra sao?
    
*   **S3**: Mức dung lượng, lớp lưu trữ (storage class), tần suất truy cập, số lần PUT/GET.
    

Khi đã có con số cụ thể, chúng tôi xác định rõ mục tiêu: “Tiết kiệm 10.000 đô/tháng, chủ yếu từ tối ưu EC2, S3, và Data Transfer.”

## 2\. Tối ưu hóa EC2
**EC2** thường chiếm tỷ trọng lớn nhất trong hóa đơn AWS. Công ty tôi có khá nhiều máy chủ **cỡ lớn (m5.2xlarge, m5.4xlarge)** và **ít khi “full tải”**. Lưu ý rằng nếu một instance hoạt động ở mức 5-10% CPU thì chúng ta đang “phung phí” rất nhiều tài nguyên.

### Bước 1: Right-sizing (Chọn kích thước phù hợp)

Tôi dùng **AWS Compute Optimizer** – công cụ AWS miễn phí, tự động gợi ý giảm hoặc tăng kích thước instance dựa trên dữ liệu hiệu năng thực tế. Kết quả là nhiều instance m5.2xlarge được đề xuất giảm xuống m5.xlarge hoặc thậm chí m5.large. Một số trường hợp, thay vì dùng cỡ “xlarge” (4 vCPU, 16 GB RAM), chúng tôi hạ xuống “large” (2 vCPU, 8 GB RAM) mà vẫn đủ chạy tốt.

### Bước 2: Tắt hoặc gom các instance không cần thiết

Chúng tôi phát hiện có nhiều instance dành cho môi trường dev/test bật 24/7, nhưng thực tế chỉ dùng khoảng 8-10 tiếng mỗi ngày, thậm chí không dùng cuối tuần. Sau khi họp với nhóm phát triển, chúng tôi thiết lập **Lịch tắt/mở tự động** bằng AWS Instance Scheduler hoặc tự viết script dùng AWS Lambda. Cách này giúp cắt giảm 60-70% chi phí cho môi trường thử nghiệm.

### Bước 3: Chuyển qua Container (ECS, EKS) hoặc Serverless nếu phù hợp

Một số ứng dụng API dịch vụ tần suất thấp (chỉ chạy batch ban đêm hoặc không cần chạy liên tục) chúng tôi chuyển sang **Fargate** (serverless container) hoặc **AWS Lambda**. Tuy nhiên, nên cẩn trọng vì Lambda nếu chạy với tần suất cực cao hoặc thời gian thực thi dài cũng không hẳn rẻ hơn EC2. Bằng cách này, công ty giảm thêm một phần chi phí đáng kể – vừa giảm công quản trị máy chủ, vừa tận dụng mô hình “pay-per-use”.

Việc right-sizing, tắt máy chủ nhàn rỗi, và chuyển đổi sang container/serverless là ba “đòn bẩy” chính, giúp chi phí EC2 của chúng tôi “xuống dốc” trông thấy, từ 18.000 đô/tháng còn cỡ 12.000 đô/tháng.

## 3\. Tận dụng Spot Instances, Reserved Instances, và Savings Plan
### Spot Instances

**Spot Instances** là cách thuê “phiên bản EC2 giá rẻ” khi AWS có công suất dư. Giá Spot có thể rẻ hơn 70-90% so với On-Demand. Công ty tôi quyết định chuyển **batch job** và **các workload không đòi hỏi tính ổn định cao** (có thể bị dừng đột ngột) sang Spot. Dù không tiết kiệm quá nhiều (vì batch job không chiếm phần lớn), nhưng cũng giúp cắt giảm khoảng 500-700 đô/tháng.

### Reserved Instances (RI)

Đối với các dịch vụ chạy 24/7, ổn định dài hạn như **production database, server web**, chúng tôi mua **Reserved Instances** 1 năm hoặc 3 năm. Mua RI có thể tiết kiệm 30-40% so với On-Demand. Lưu ý chỉ mua khi bạn **chắc chắn** workload đó sẽ tồn tại lâu dài, vì bạn phải trả trước chi phí hoặc cam kết sử dụng.

### Savings Plan

Sau này, AWS cho ra **Savings Plan**, giúp linh hoạt hơn so với RI. Thay vì gắn liền với một loại instance cố định, Savings Plan cho phép cam kết “số đô la/giờ” sử dụng EC2 hoặc Fargate. Chúng tôi chuyển sang Savings Plan cho toàn bộ workload chạy ổn định. Chỉ cần tính mức trung bình sử dụng hàng tháng là bao nhiêu, rồi chọn gói Savings Plan tương ứng.

Nhờ kết hợp Spot Instances cho batch job với Savings Plan cho workload dài hạn, công ty cắt giảm thêm được **~15%** trên tổng chi phí EC2.

## 4\. Tối ưu chi phí lưu trữ S3 và EBS
### Lựa chọn lớp lưu trữ S3 phù hợp

S3 cung cấp nhiều lớp lưu trữ (storage class) như **Standard, Standard-IA (Infrequent Access), Glacier, Deep Archive**. Công ty tôi có hàng chục TB log cũ “đắp chiếu”, hiếm khi truy cập, nhưng vẫn “nằm chềnh ềnh” ở lớp Standard. Đây là sai lầm điển hình.

Chúng tôi tạo **Lifecycle Policy**:

*   **90 ngày** sau khi upload, tự động chuyển từ Standard sang Standard-IA.
    
*   **180 ngày** chuyển sang Glacier.
    
*   Sau **1 năm**, chuyển sang Deep Archive (hoặc xóa hẳn nếu không còn giá trị).
    

Nhờ chính sách này, hóa đơn S3 giảm **30-40%**.

### Kiểm soát phiên bản (S3 Versioning)

S3 cho phép bật versioning để lưu nhiều phiên bản của đối tượng. Tính năng này hữu ích cho backup, nhưng vô tình làm tăng dung lượng lưu trữ. Chúng tôi bật versioning có kiểm soát (chỉ ở bucket quan trọng), đồng thời thiết lập **quy tắc xóa phiên bản cũ** theo thời gian.

### Giảm chi phí EBS (Elastic Block Store)

Nhiều instance EC2 sử dụng EBS cỡ **500GB, 1TB** nhưng dùng thực tế chỉ 20-30%. Thậm chí, có các volume cũ không được detach khi server bị xóa. Bước rà soát EBS gồm:

*   Tìm volume không gắn (unattached volume) và xóa nếu không dùng.
    
*   Giảm kích thước volume nếu có thể, hoặc chuyển sang loại GP3 (General Purpose SSD) mới, rẻ hơn GP2 cho cùng hiệu năng.
    
*   Xóa snapshot cũ, dùng **Data Lifecycle Manager** để tự động xóa snapshot sau N ngày.
    

Mảng lưu trữ này giúp công ty tiết kiệm thêm **1.000 - 2.000 đô/tháng**, tùy từng chu kỳ.

## 5\. Kiểm soát băng thông và Data Transfer
Data Transfer là một hạng mục **thường bị bỏ qua**, nhưng lại có thể “ăn” tiền khủng nếu ứng dụng của bạn kết nối nhiều region, nhiều dịch vụ, hoặc “phun” data ra Internet.

### Giảm di chuyển dữ liệu giữa vùng (Cross-Region Transfer)

Tôi phát hiện nhiều dữ liệu log ở US-West nhưng team phân tích ở US-East vẫn đọc từ bucket gốc, gây băng thông cross-region. Chúng tôi tạo **bucket replica** ở cùng region hoặc áp dụng AWS DataSync để chuyển dữ liệu tối ưu hơn.

### Sử dụng CDN (CloudFront) cho nội dung tĩnh

Thay vì cho client truy cập trực tiếp vào S3 hoặc EC2, chúng tôi cấu hình **CloudFront** để phân phối nội dung tĩnh (hình ảnh, video). CloudFront thường rẻ hơn so với data transfer trực tiếp từ S3, đồng thời tăng tốc độ phân phối nhờ edge location.

### Tối ưu kết nối VPN hay Direct Connect

Với các đường truyền hybrid (từ on-premises lên cloud), chúng tôi kiểm tra xem cài đặt **Direct Connect** hay VPN site-to-site có tối ưu không. Điều chỉnh một số cấu hình để tránh truyền dữ liệu không cần thiết qua lại.

Sau bước này, data transfer giảm, có tháng tiết kiệm **vài trăm đến cả nghìn đô** tùy lưu lượng.

## 6\. Tắt tài nguyên không dùng, giảm quy mô tài nguyên môi trường dev/test
Đây là nguyên tắc **bất di bất dịch** trong mọi nỗ lực giảm chi phí cloud. Nhiều khi, chúng ta “vẽ” ra rất nhiều tài nguyên để phát triển tính năng, nhưng khi xong việc lại quên **dọn**.

*   **EC2 và RDS môi trường dev**: Dùng **Instance Scheduler** để tắt ban đêm và cuối tuần.
    
*   **S3**: Xóa bucket đã hết hạn sử dụng.
    
*   **ELB (Load Balancer)**: Tắt/bỏ load balancer khi không còn host đằng sau.
    
*   **Security Group** và **Elastic IP**: Xóa những tài nguyên “mồ côi” không còn gắn server.
    

Mẹo nhỏ: Thiết lập **Tag** (dán nhãn) “owner” và “purpose” cho mọi tài nguyên AWS, giúp bạn dễ lọc và xóa những thứ không cần thiết.

## 7\. Theo dõi, giám sát chi phí liên tục
Tối ưu một lần chưa đủ. Để chi phí không “phình” trở lại, bạn cần liên tục **theo dõi** và **nhắc nhở** đội ngũ. Tôi thường thiết lập:

*   **AWS Budgets**: Báo email khi chi phí có nguy cơ vượt quá ngưỡng đề ra.
    
*   **Cost Explorer**: Xem biểu đồ xu hướng hàng tuần, hàng tháng.
    
*   **Cảnh báo CloudWatch**: Nếu băng thông đột nhiên tăng gấp 2-3 lần, sẽ có cảnh báo đến Slack channel của đội DevOps.
    

Ngoài ra, thường xuyên trao đổi với **nhóm phát triển** về best practices cũng rất quan trọng. Chẳng hạn, khi họ tạo môi trường mới, phải tuân thủ tag, chọn region hợp lý, và có chiến lược lưu trữ rõ ràng.

## Hành trình cắt giảm 10.000 đô/tháng
Quay lại với mục tiêu ban đầu của công ty: cắt 10.000 đô/tháng. Kết quả mà chúng tôi đạt được đến từ tổng hòa nhiều giải pháp:

1.  **Tối ưu kích thước EC2 (right-sizing) + tắt máy nhàn rỗi**: Giảm từ 18.000 đô còn ~12.000 đô.
    
2.  **Kết hợp Spot, RI/Savings Plan**: Tiết kiệm thêm khoảng 15% chi phí EC2.
    
3.  **Giảm lưu trữ S3 và EBS** (lifecycle, phiên bản, xóa snapshot cũ): Cắt ~1.000 - 2.000 đô.
    
4.  **Kiểm soát data transfer**: Giảm vài trăm đến hơn 1.000 đô/tháng, tùy kỳ.
    
5.  **Triệt để tắt tài nguyên, cleanup**: Chặn “rò rỉ” tiềm ẩn, mỗi tháng tiết kiệm thêm 500 - 1.000 đô.
    

Cộng gộp các nỗ lực, chúng tôi **vượt chỉ tiêu 10.000 đô** và còn tiếp tục tối ưu thêm nữa.

## Những “mẹo” khác đáng cân nhắc
*   **Sử dụng RI/Savings Plan cho RDS**: Nếu bạn có **database** RDS chạy 24/7 (MySQL, PostgreSQL, v.v.), mua RI hoặc Savings Plan cũng giúp tiết kiệm kha khá.
    
*   **Auto Scaling**: Đảm bảo cấu hình **Auto Scaling Group** chính xác, không bị dư thừa instance khi tải xuống thấp.
    
*   **Monitoring memory và disk** (qua CloudWatch custom metrics) để biết chính xác cần bao nhiêu RAM, bao nhiêu dung lượng đĩa.
    
*   **Áp dụng “tag-based” cost allocation**: Xem cost per team, per project, giúp các nhóm chịu trách nhiệm về phần chi phí của họ.
    
*   **Sử dụng AWS Trusted Advisor**: Công cụ này gợi ý bảo mật, hiệu năng, và cả tối ưu chi phí.
    

## Lời kết
Cắt giảm chi phí AWS có thể ví như một cuộc “truy lùng lãng phí” liên tục. Không có giải pháp duy nhất nào phù hợp cho tất cả – thay vào đó, **bạn cần kết hợp nhiều bước nhỏ** để dần dần tối ưu đến mức mong muốn. Từ việc right-sizing instance, tắt máy dev ngoài giờ, tối ưu S3 và EBS, đến lựa chọn Spot Instances hay Savings Plan, mỗi hành động đóng góp một phần.

Trong thực tế, quá trình này không chỉ tiết kiệm tiền, mà còn **nâng cao nhận thức** của cả đội ngũ về việc **quản lý tài nguyên**. Thay vì “xài xả láng” rồi cuối tháng “giật mình” nhìn hóa đơn, mọi người bắt đầu suy nghĩ có trách nhiệm hơn: Có cần instance to thế không? Có cần lưu hết log 2 năm trước ở S3 Standard không? Hay **chỉ mở** môi trường staging khi cần test?

Những kinh nghiệm trên không phải “bí kíp gia truyền”, nhưng chúng đã giúp công ty tôi cắt giảm **hơn 10.000 đô/tháng** – một con số đáng kể. Hãy xem đây là hướng dẫn “mở”, tùy vào quy mô và đặc thù, bạn có thể áp dụng linh hoạt. Quan trọng là **đừng dừng lại sau khi đã tối ưu một lần**. Hãy **liên tục** giám sát, điều chỉnh, và chia sẻ kiến thức với mọi thành viên để xây dựng **văn hóa tiết kiệm chi phí**.

Nếu bạn vẫn đang loay hoay với hóa đơn AWS khổng lồ, hy vọng bài viết này sẽ là nguồn cảm hứng để bắt tay hành động ngay hôm nay! Chúc bạn thành công trên hành trình **cắt giảm chi phí đám mây** và tạo ra giá trị tối đa cho dự án của mình.