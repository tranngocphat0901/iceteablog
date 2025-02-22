---
title: "Terraform Cheat Sheet: 10 Mẹo Hay Không Ai Chỉ Bạn"
date: 2025-02-22T12:00:00Z
draft: false
tags: ["Terraform", "DevOps", "Cloud"]
categories: ["Infrastructure as Code", "DevOps"]
author: "Patrick Tran"
summary: "Những mẹo Terraform giúp bạn làm việc hiệu quả hơn và tránh các lỗi phổ biến."
---

## Giới Thiệu

Terraform là một công cụ mạnh mẽ cho Infrastructure as Code (IaC), cho phép bạn mô tả hạ tầng của mình thông qua các tệp cấu hình. Thay vì phải tạo tài nguyên (server, database, load balancer, v.v.) bằng tay trên giao diện web hay dòng lệnh, Terraform giúp bạn tự động hóa toàn bộ quá trình này. Khi hệ thống ngày càng mở rộng, việc sử dụng Terraform một cách hiệu quả sẽ giúp bạn tiết kiệm nhiều thời gian, giảm thiểu rủi ro và hạn chế tối đa sai sót.

Tuy nhiên, Terraform cũng có những góc khuất mà nếu không nắm rõ, bạn sẽ dễ gặp lỗi hoặc phải tốn thời gian sửa chữa. Dưới đây là **10 mẹo Terraform** giúp bạn làm việc nhanh hơn, hiệu quả hơn và hạn chế “tai nạn” khi triển khai hạ tầng.

---

## 1. Định Dạng Code với `terraform fmt`

### Tại sao cần định dạng code?

Khi làm việc nhóm, mỗi người có một thói quen viết code khác nhau. Không đồng bộ về cách định dạng sẽ khiến code khó đọc, khó bảo trì, và dễ tạo ra những xung đột (conflict) khi merge pull request. Terraform cung cấp công cụ định dạng code tự động, giúp tất cả tệp `.tf` tuân theo một style thống nhất.

### Cách sử dụng

```sh
terraform fmt
```

Lệnh này sẽ tự động căn chỉnh dấu ngoặc, khoảng trắng, thụt lề, v.v. theo quy tắc chung của Terraform.
Mẹo thêm
Tích hợp `terraform fmt` vào **pre-commit hook**: Mỗi khi bạn commit, code sẽ được định dạng lại tự động, đảm bảo luôn sạch đẹp trước khi đẩy lên repo.
Cấu hình CI/CD: Bạn có thể thêm bước “terraform fmt check” để pipeline fail nếu code chưa được format đúng chuẩn.
Lợi ích: Code thống nhất, dễ đọc, tránh lỗi “nhỏ nhưng khó chịu” khi nhiều người cùng phát triển.

2️⃣ Sử Dụng Variables để Tăng Tính Linh Hoạt
Tại sao nên dùng variables?
Việc hard-code giá trị (ví dụ: kích thước máy chủ, tên subnet) trực tiếp trong file .tf sẽ gây khó khăn khi cần thay đổi hoặc mở rộng. Biến (variables) cho phép bạn điều chỉnh cấu hình mà không phải sửa nhiều nơi.

Ví dụ
hcl
Sao chép
Chỉnh sửa
variable "instance_type" {
  type    = string
  default = "t3.micro"
}

resource "aws_instance" "example" {
  instance_type = var.instance_type
}
variable "instance_type" cho phép định nghĩa kiểu dữ liệu và giá trị mặc định.
Khi triển khai, bạn có thể truyền thêm file terraform.tfvars hoặc dùng flag -var để override giá trị.
Mẹo thực tế
File terraform.tfvars: Chứa các biến dành riêng cho môi trường (dev, staging, production).
Biến nhạy cảm: Nếu có password hay API key, hãy đánh dấu sensitive = true trong variable hoặc sử dụng các giải pháp lưu trữ an toàn (Vault, AWS Parameter Store).
Lợi ích: Code gọn gàng, dễ tái sử dụng. Thay đổi cấu hình nhanh chóng mà không phải chỉnh nhiều chỗ.

3️⃣ Luôn Validate Code Trước Khi Apply
Mục đích
Lệnh terraform validate cho bạn biết code Terraform có hợp lệ hay không. Điều này giúp ngăn chặn các lỗi cú pháp hay lỗi logic trước khi đi vào giai đoạn triển khai thực tế.

Cách sử dụng
sh
Sao chép
Chỉnh sửa
terraform validate
Nếu code có vấn đề, Terraform sẽ báo lỗi và dừng ngay.
Lợi ích
Giảm thiểu nguy cơ “deploy sai”.
Cần thiết trong pipeline CI/CD để không đẩy code lỗi lên môi trường production.
Ví dụ:

Trước mỗi lần commit hoặc merge pull request, bạn chạy terraform validate để chắc chắn code không có syntax error.
4️⃣ Luôn Xem Trước Thay Đổi với terraform plan
Tại sao quan trọng?
terraform plan hiển thị những thay đổi Terraform sẽ thực hiện khi apply. Nó cho bạn thấy tài nguyên nào sẽ được tạo, cập nhật hay xóa. Điều này giúp tránh những “tai nạn” vô tình xóa hạ tầng đang hoạt động.

Cách sử dụng
sh
Sao chép
Chỉnh sửa
terraform plan
Terraform sẽ so sánh trạng thái hiện tại với cấu hình mới, rồi liệt kê chi tiết thay đổi.
Mẹo
terraform plan -out=tfplan: Lưu kết quả plan vào file tfplan. Sau đó, bạn có thể dùng terraform apply tfplan để chắc chắn triển khai đúng như bản plan đã xem.
Lợi ích: Biết trước những gì sẽ xảy ra, giảm thiểu sai sót và bất ngờ.

5️⃣ Quản Lý Terraform State Hiệu Quả
Terraform State là gì?
Terraform lưu trạng thái (state) của toàn bộ hạ tầng trong file terraform.tfstate. File này đóng vai trò “nguồn sự thật” để Terraform biết hiện đang có tài nguyên nào, cấu hình ra sao.

Vấn đề
Nếu chỉ lưu terraform.tfstate ở local, bạn dễ bị mất file hoặc gây xung đột khi làm việc nhóm.
Sao lưu state thủ công có thể phức tạp.
Giải pháp
Remote Backend: Sử dụng S3 (AWS), GCS (Google Cloud), Azure Blob Storage… để lưu terraform.tfstate một cách tập trung và an toàn.
Kích hoạt versioning trên S3 để rollback nếu state bị lỗi.
Lệnh hữu ích
sh
Sao chép
Chỉnh sửa
terraform state list
Liệt kê toàn bộ tài nguyên Terraform đang quản lý.
sh
Sao chép
Chỉnh sửa
terraform state show <resource>
Xem chi tiết về một tài nguyên (ví dụ: aws_instance.example).
Lợi ích:

Quản lý trạng thái tập trung, giảm nguy cơ mất file.
Dễ dàng làm việc nhóm, mọi người đều dùng chung một state.
6️⃣ Dùng Modules để Tối Ưu Hóa Code
Modules là gì?
Modules là các “khối” code Terraform có thể tái sử dụng. Thay vì copy-paste cùng một cấu hình (VPC, ECS, RDS, v.v.) ở nhiều nơi, bạn tách chúng thành module và chỉ việc gọi lại.

Ví dụ
hcl
Sao chép
Chỉnh sửa
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  name   = "my-vpc"
}
source = "terraform-aws-modules/vpc/aws": Lấy module từ registry Terraform chính thức.
Lợi ích
Dễ bảo trì, chỉ cần sửa code module ở một chỗ.
Dễ chia sẻ trong nội bộ công ty (VD: module chuẩn cho VPC, module chuẩn cho database).
Mẹo
Tạo module riêng cho những pattern lặp lại như network, nhóm security, IAM policy.
Đặt module vào repo riêng để chia sẻ cho các team trong tổ chức.
7️⃣ Dùng terraform taint để Bắt Buộc Recreate Resource
Hoàn cảnh
Đôi khi, resource (ví dụ máy chủ) gặp vấn đề cần khởi tạo lại, nhưng bạn không muốn sửa code hoặc xóa thủ công trong AWS/GCP. Lệnh terraform taint sẽ đánh dấu resource đó là “bị hỏng” và buộc Terraform tái tạo trong lần apply tiếp theo.

Cách dùng
sh
Sao chép
Chỉnh sửa
terraform taint aws_instance.example
terraform apply
Terraform hiểu rằng resource aws_instance.example hỏng và sẽ xóa, sau đó tạo mới.
Lợi ích
Nhanh chóng tạo lại resource mà không cần can thiệp quá nhiều.
Hữu ích khi resource “đang chạy nhưng hoạt động lỗi” và bạn muốn reset nó.
8️⃣ Dùng terraform graph để Hiểu Rõ Dependencies
Tại sao cần?
Khi hạ tầng phức tạp, việc xác định resource nào phụ thuộc vào resource nào trở nên khó khăn. terraform graph giúp bạn tạo ra sơ đồ quan hệ, từ đó phân tích, debug hoặc làm tài liệu.

Cách sử dụng
sh
Sao chép
Chỉnh sửa
terraform graph | dot -Tpng > graph.png
Graphviz cần thiết để chuyển nội dung graph sang file ảnh .png.
Lợi ích
Minh họa trực quan cách hạ tầng liên kết.
Hữu ích cho quá trình onboarding thành viên mới, cho họ thấy toàn cảnh hạ tầng.
9️⃣ Dùng terraform import để Quản Lý Resource Đã Có
Tình huống
Bạn có sẵn một EC2 instance được tạo thủ công trong AWS Console. Giờ muốn Terraform quản lý nó, nhưng không muốn xóa và tạo lại. terraform import cho phép nhập resource đã tồn tại vào state Terraform.

Cách dùng
sh
Sao chép
Chỉnh sửa
terraform import aws_instance.example i-1234567890abcdef0
Sau đó, Terraform sẽ ghi nhận instance i-1234567890abcdef0 chính là aws_instance.example.
Lợi ích
Không gián đoạn dịch vụ vì không cần xóa resource cũ.
Dễ “Terraform hóa” hạ tầng có sẵn.
🔟 Cẩn Thận Khi Dùng terraform destroy
Chức năng
terraform destroy xóa toàn bộ hạ tầng do Terraform quản lý. Đây là lệnh “nguy hiểm” vì chỉ cần 1 sai sót, bạn sẽ mất toàn bộ tài nguyên đang chạy.

Mẹo
Dùng -target để xóa cụ thể resource thay vì toàn bộ:
sh
Sao chép
Chỉnh sửa
terraform destroy -target aws_instance.example
Luôn xem lại plan trước khi destroy để tránh xóa nhầm.
Lợi ích
Khi cần dọn dẹp hoặc kết thúc dự án, terraform destroy rất hữu ích.
Tuy nhiên, hãy backup state và double-check vì bạn sẽ không khôi phục được sau khi xóa.

