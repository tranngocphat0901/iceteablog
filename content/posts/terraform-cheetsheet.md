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

## 1️⃣ Dùng `terraform fmt` để định dạng code
Mỗi khi viết Terraform, định dạng có thể bị lộn xộn. Dùng lệnh này để format code chuẩn:
```sh
terraform fmt

