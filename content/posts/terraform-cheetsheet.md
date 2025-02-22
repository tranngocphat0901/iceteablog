---
title: "Terraform Cheat Sheet: 10 Mẹo Hay Không Ai Chỉ Bạn"
date: 2025-02-22T12:00:00Z
draft: false
tags: ["Terraform", "DevOps", "Cloud"]
categories: ["Infrastructure as Code", "DevOps"]
author: "Admin"
summary: "Những mẹo Terraform giúp bạn làm việc hiệu quả hơn."
---

# Terraform Cheat Sheet: 10 Mẹo Hay Không Ai Chỉ Bạn

## Giới Thiệu
Terraform là một trong những công cụ Infrastructure as Code (IaC) phổ biến nhất hiện nay, giúp quản lý hạ tầng một cách tự động và có thể repeatable. Tuy nhiên, Terraform không chỉ đơn thuần là "viết file .tf" và apply, mà nó còn nhiều điều hay ho mà không ai chỉ bạn!

Dưới đây là **10 mẹo Terraform** giúp tăng hiệu suất và giảm thiểu sai sót khi deploy hạ tầng!

---

## 1. Dùng `terraform fmt` để định dạng code
Mỗi khi viết Terraform, định dạng có thể bị lộn xộn. Dùng lệnh này để format code chuẩn:
```sh
terraform fmt
```
Mẹo: Cài đặt pre-commit hook để tự động chạy `terraform fmt` trước khi commit.

## 2. Sử dụng Variables để tăng tính linh hoạt
Thay vì hardcode giá trị, hãy sử dụng variables:
```sh
variable "instance_type" {
  default = "t3.micro"
}
resource "aws_instance" "example" {
  instance_type = var.instance_type
}
```
Mẹo: Dùng file `terraform.tfvars` để dễ quản lý values.

## 3. Dùng `terraform validate` trước khi apply
Tránh deploy sai, luôn validate code trước:
```sh
terraform validate
```

## 4. Dùng `terraform plan` để xem trước thay đổi
Trước khi terraform apply, hãy chạy:
```sh
terraform plan
```
Mẹo: Dùng `terraform plan -out=tfplan` để lưu lại kết quả plan.

## 5. Dùng Terraform State hiệu quả
Dùng lệnh này để xem và quản lý state:
```sh
terraform state list
```
Mẹo: Dùng **remote backend (S3, GCS)** thay vì lưu local!

## 6. Dùng Modules để tối ưu hoá code
Tách code thành modules giúp reuse dễ dàng:
```sh
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  name   = "my-vpc"
}
```

## 7. Dùng terraform taint để bắt buộc recreate resource
Nếu muốn recreate resource, dùng:
```sh
terraform taint aws_instance.example
terraform apply
```

## 8. Dùng terraform graph để visualize hạ tầng
Vừa debug dễ hơn, vừa hiểu rõ dependencies:
```sh
terraform graph | dot -Tpng > graph.png
```
Mẹo: Dùng **Graphviz** để render file PNG.

## 9. Dùng terraform import để quản lý resource đã có
Import tài nguyên có sẵn vào Terraform:
```sh
terraform import aws_instance.example i-1234567890abcdef0
```

## 10. Dùng terraform destroy cẩn thận!
```sh
terraform destroy
```
Mẹo: Dùng `-target` để xoá resource cụ thể.