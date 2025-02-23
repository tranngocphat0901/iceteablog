---
title: "Terraform Cheat Sheet: 10 Mẹo Hay Không Ai Chỉ Bạn"
date: 2025-02-22T12:00:00Z
draft: false
tags: ["Terraform", "DevOps", "Cloud"]
categories: ["Infrastructure as Code", "DevOps"]
author: "Patrick Tran"
summary: "Những mẹo Terraform giúp bạn làm việc hiệu quả hơn và tránh các lỗi phổ biến."
cover:
  image: "/images/terraform-cheatsheet.png"  # Image path
  alt: "Terraform Cheat Sheet Cover Image"  # Alternative text if the image fails to load
  caption: "Latest Terraform Guide"  # Caption below the image (optional)
  relative: true  # If true, the image is located in the static/ directory
  hidden: false  # If true, the image will not be displayed in the post, only on the post list
---

## Giới Thiệu

Hệ sinh thái DevOps hiện đại không thể thiếu các công cụ IaC (Infrastructure as Code) để quản lý hạ tầng. Trong số đó, **Terraform** là một trong những công cụ phổ biến nhất và được ưa chuộng bởi tính linh hoạt, khả năng mở rộng, cùng cộng đồng hỗ trợ đông đảo. Hầu hết tài liệu về Terraform đều tập trung vào cú pháp và các lệnh cơ bản, nhưng lại ít khi đề cập các mẹo hay, những kinh nghiệm “xương máu” mà lập trình viên chỉ học được khi làm thực tế.

Trong bài viết này, chúng ta sẽ cùng khám phá **10 mẹo hay về Terraform** mà “không ai chỉ bạn”, giúp bạn nâng cao hiệu suất làm việc, tránh rủi ro và tối ưu quy trình triển khai hạ tầng. Bài viết dài khoảng 1500 từ này được thiết kế như một “cheat sheet” nâng cao dành cho các bạn đang muốn sử dụng Terraform một cách chuyên nghiệp.

## 1\. Sử dụng **Workspaces** một cách khéo léo
**Workspaces** cho phép bạn quản lý nhiều môi trường (environment) khác nhau trên cùng một cấu hình Terraform. Thay vì phải nhân bản cấu trúc thư mục và file cấu hình cho từng môi trường (development, staging, production), bạn có thể tận dụng `terraform workspace` để:

*   **Tách biệt state**: Mỗi workspace sẽ tương ứng với một tệp state (terraform.tfstate) riêng, giúp bạn không lo xung đột giữa các môi trường.
    
*   **Dễ dàng chuyển đổi**: Tạo workspace bằng lệnh `terraform workspace new <tên_workspace>` và chuyển đổi giữa chúng với `terraform workspace select <tên_workspace>`.
    

Tuy nhiên, việc lạm dụng workspaces có thể khiến bạn **khó kiểm soát thay đổi** khi nhiều môi trường dùng chung một tập cấu hình. Hãy đảm bảo có quy tắc rõ ràng về cách sử dụng workspace cũng như đánh version cho cấu hình. Nếu môi trường staging và production có cấu hình hạ tầng khác nhau đáng kể, đôi khi bạn vẫn cần tách cấu hình để tiện quản lý.

## 2\. Tách biệt **Backend** và **Module** khi chia sẻ

Một thói quen hay gặp trong các dự án Terraform là nhúng luôn cấu hình backend (chứa thông tin về nơi lưu trữ state, ví dụ: S3, Terraform Cloud, Azure Storage, v.v.) và các biến bí mật trong cùng một module. Điều này có thể gây phiền toái khi bạn cần chia sẻ module cho nhóm khác. Mẹo ở đây:

*   **Đưa backend ra ngoài**: Tạo một file riêng (chẳng hạn `backend.tf`) để định nghĩa backend.
    
*   **Module chỉ chứa logic**: Phần module (thường đặt trong thư mục `modules/`) chỉ nên chứa logic hạ tầng, định nghĩa resource, biến đầu vào (variables) và đầu ra (outputs).
    

Cách tách này giúp bạn tái sử dụng module ở nhiều dự án hoặc đội nhóm khác nhau mà không cần thay đổi cách khai báo backend hay biến bí mật. Đồng thời, nó cũng giúp việc cấu hình CI/CD dễ dàng hơn, vì mỗi môi trường CI/CD có thể tự inject thông tin backend phù hợp.

## 3\. Dùng **Terraform Variables** như “cấu hình động”

Thay vì “cứng hóa” mọi giá trị ngay trong file `.tf`, hãy **tận dụng triệt để** sức mạnh của biến (variable) trong Terraform:

*   **Variable file**: Tách biến thành các file `.tfvars` tương ứng từng môi trường. Ví dụ: `dev.tfvars`, `staging.tfvars`, `prod.tfvars`.
    
*   **Phân loại biến**: Sử dụng biến đầu vào (input variables) cho bất kỳ giá trị nào có thể thay đổi: tên tài khoản AWS, loại máy chủ (instance type), vùng (region), số lượng instance, danh sách subnet, v.v.
    
*   **Sử dụng default**: Đối với những giá trị mặc định dùng chung cho hầu hết các môi trường, hãy thiết lập trong phần `default` của `variable`.
    

Nhờ việc khai báo biến, bạn sẽ dễ dàng tùy chỉnh, quản lý thay đổi và tái sử dụng cấu hình. Điều này cũng giúp giảm đáng kể rủi ro sai sót khi deploy sang môi trường khác.

## 4\. **Locking State** để tránh va chạm

Khi nhiều thành viên trong nhóm làm việc trên cùng một môi trường (và cùng một state), xung đột về state là điều không thể tránh khỏi. Terraform có cơ chế **State Locking** giúp “khóa” state khi một lệnh `terraform plan` hoặc `terraform apply` đang chạy.

*   **Chọn backend có hỗ trợ locking**: S3 (kết hợp DynamoDB), Terraform Cloud, Azure Blob Storage, Google Cloud Storage, v.v.
    
*   **Bật tính năng lock**: Chẳng hạn với S3, bạn cần kích hoạt DynamoDB Table để quản lý lock. Với Terraform Cloud, việc lock được tích hợp sẵn.
    

Locking bảo vệ bạn khỏi các tình huống như hai người cùng chạy `terraform apply` và ghi đè lên state của nhau. Đây là một trong những mẹo mà nhiều người mới quên dẫn đến mất thời gian xử lý conflict.

## 5\. Tận dụng **Local Values** để tổ chức logic

Các biến local (local values) trong Terraform hoạt động tương tự biến tạm trong code, cho phép bạn gán một giá trị trung gian để tái sử dụng nhiều lần. Ví dụ:
```bash
locals {
  common_tags = {
    project     = "my-sample-app"
    environment = var.env
  }
}

resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = local.common_tags
}
```

Nhờ `local.common_tags`, bạn không phải lặp lại cùng một danh sách thẻ (tags) ở nhiều nơi. Ngoài ra, local còn giúp bạn “tiền xử lý” dữ liệu phức tạp, ví dụ như ghép chuỗi, lọc danh sách, chuyển đổi kiểu dữ liệu. Đây là mẹo quan trọng để cấu hình Terraform của bạn trở nên **gọn gàng, dễ đọc** và dễ bảo trì hơn.

## 6\. Dùng **Data Sources** để linh hoạt khi tra cứu tài nguyên

**Data Sources** trong Terraform cho phép bạn lấy thông tin của tài nguyên ngoài, vốn đã tồn tại, để dùng trong cấu hình của mình. Ví dụ: Bạn có sẵn VPC hoặc subnet được tạo bởi một nhóm khác, hoặc được tạo thủ công từ trước; bạn có thể sử dụng data source để tham chiếu đến tài nguyên này mà không cần tạo lại.
```bash
data "aws_vpc" "selected" {
  filter {
    name   = "tag:Name"
    values = ["my-shared-vpc"]
  }
}
```

Sau đó, bạn có thể truy cập `data.aws_vpc.selected.id` hoặc các thuộc tính khác. Data sources giúp **giảm thiểu trùng lặp** và cho phép Terraform tương tác với hạ tầng “có sẵn” một cách tự động, giảm thiểu lỗi do cấu hình thủ công.

## 7\. Cấu trúc **Module** hướng “mở rộng”

Khi dự án lớn dần, bạn sẽ không muốn một file `.tf` duy nhất chứa tất cả resource. Thay vào đó, hãy chia nhỏ theo **module**:

*   **Module “nhân bản”**: Ví dụ, bạn có module tạo nhóm EC2 instance có cấu hình giống nhau, chỉ khác về số lượng, loại máy, hoặc vùng. Mỗi lần cần triển khai một nhóm EC2 khác, bạn chỉ cần gọi lại module đó với các biến phù hợp.
    
*   **Module “chuyên biệt”**: Tạo module riêng cho mỗi dịch vụ (VPC, IAM, RDS, EKS, v.v.) để dễ bảo trì.
    
*   **Đặt tên module có ý nghĩa**: Sử dụng tên gợi nhớ, ví dụ `modules/vpc`, `modules/ec2_group`, `modules/eks_cluster`.
    

Đừng quên cập nhật `version` (thường thông qua Git tag hoặc lock version của module trong trường hợp bạn tải module từ registry) để tránh tình trạng “đang chạy ngon bỗng dưng ai đó update module gây xung đột”.

## 8\. **Testing** với `terraform plan` và CI/CD

Một chuỗi CI/CD tốt nên có bước kiểm thử cấu hình Terraform trước khi thực thi:

*   **terraform fmt**: Kiểm tra và format code Terraform để thống nhất coding style.
    
*   **terraform validate**: Kiểm tra xem file .tf có hợp lệ về cú pháp hay không.
    
*   **terraform plan**: Mô phỏng những thay đổi sẽ áp dụng lên hạ tầng.
    

Một số công cụ như **Terratest** (viết bằng Go) hoặc mô hình GitOps cũng cho phép bạn test “end-to-end” bằng cách mô phỏng toàn bộ quá trình deploy và kiểm tra thực tế xem hạ tầng có được tạo như kỳ vọng. Mặc dù việc viết test cho IaC có thể tốn công sức, nhưng nó mang lại sự **an tâm** và bền vững lâu dài khi dự án phát triển.

## 9\. Quản lý **biến bí mật** một cách an toàn

Dù Terraform không lưu trực tiếp password hay token trong code, bạn vẫn cần cẩn thận với cách truyền biến bí mật:

*   **Sử dụng công cụ bí mật**: Vault của HashiCorp, AWS Secrets Manager, Azure Key Vault, hoặc GitHub Actions Secrets, v.v.
    
*   **Không commit file .tfstate lên git**: State file có thể chứa password hoặc token. Hãy đảm bảo thêm `.tfstate` vào `.gitignore` nếu bạn lưu state cục bộ. Trong trường hợp dùng remote backend, bạn vẫn nên thận trọng vì state trên S3 hoặc Terraform Cloud cũng chứa thông tin nhạy cảm.
    
*   **Sử dụng biến môi trường**: Bạn có thể truyền giá trị bí mật bằng biến môi trường, sau đó gọi Terraform với lệnh `terraform apply -var "db_password=$DB_PASSWORD"` (hoặc sử dụng file .tfvars bí mật được mã hóa).
    

Đừng đánh giá thấp rủi ro rò rỉ dữ liệu. Hãy đặt tính bảo mật lên hàng đầu, đặc biệt khi xử lý các môi trường production.

## 10\. Chia sẻ và **đóng gói** module dưới dạng Private Registry

Nếu công ty của bạn có nhiều nhóm cùng sử dụng Terraform, việc **đóng gói** module và chia sẻ nội bộ là rất quan trọng. HashiCorp Terraform hỗ trợ tính năng **Private Registry** - cho phép bạn:

1.  Lưu trữ module nội bộ (giống như module trên Terraform Public Registry nhưng là private).
    
2.  Kiểm soát phiên bản module.
    
3.  Tìm kiếm module dễ dàng trong phạm vi tổ chức, công ty.
    

Nhờ vậy, đội ngũ có thể tuân thủ chuẩn chung, dùng chung code, và hạn chế việc “tự chế” khiến khó bảo trì về sau. Việc thiết lập Private Registry ban đầu mất chút thời gian (yêu cầu Terraform Cloud, Enterprise, hoặc tự host), nhưng sẽ vô cùng có lợi về lâu dài.

## Kết luận

**Terraform** không chỉ đơn giản là viết file .tf rồi terraform apply. Để thực sự làm chủ, bạn cần nắm được những “mẹo hay” như sử dụng **workspaces** linh hoạt, chia nhỏ cấu hình thành **module**, quản lý state và **locking** cẩn thận, bảo mật **biến bí mật**, tận dụng **data sources**, và triển khai **test** cũng như **CI/CD**. Ngoài ra, đầu tư vào **Private Registry** để chia sẻ module nội bộ sẽ giúp tăng cường tính nhất quán và tiết kiệm thời gian phát triển.

Dưới đây là bản tóm tắt 10 mẹo chính:

1.  **Workspaces**: Tối ưu cho nhiều môi trường trên cùng cấu hình.
    
2.  Tách biệt **Backend** và **Module** để dễ chia sẻ.
    
3.  Dùng **Variables** và file .tfvars để cấu hình linh hoạt.
    
4.  **Locking State** nhằm tránh xung đột giữa nhiều người dùng.
    
5.  Dùng **Local Values** để giữ code gọn gàng.
    
6.  Tận dụng **Data Sources** để truy xuất tài nguyên có sẵn.
    
7.  Tổ chức **Module** hướng mở rộng và gọn gàng.
    
8.  **Testing** Terraform bằng terraform plan, CI/CD, Terratest.
    
9.  Bảo mật **biến bí mật**, tránh lộ thông tin nhạy cảm.
    
10.  Đóng gói và chia sẻ module qua **Private Registry** cho tổ chức.
    

Qua những điểm trên, hi vọng bạn đã có thêm góc nhìn rộng hơn và những “thủ thuật” triển khai hạ tầng bằng Terraform. Hãy coi chúng như **“cheat sheet”** để áp dụng vào thực tế, giúp quy trình Infrastructure as Code của bạn trở nên **bền vững, dễ bảo trì** và **an toàn hơn**.

Chúc bạn thành công trong hành trình chinh phục DevOps và Terraform!