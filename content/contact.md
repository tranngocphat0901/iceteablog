---
title: "Contact Me"
date: 2025-03-03T06:00:00Z
---

<h2>Liên hệ với tôi</h2>
<p>Vui lòng điền vào form dưới đây, tôi sẽ phản hồi sớm nhất có thể.</p>

<form id="contact-form">
    <label for="name">Họ và Tên:</label>
    <input type="text" id="name" name="name" required>

    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>

    <label for="organization">Tổ chức (Tùy chọn):</label>
    <input type="text" id="organization" name="organization">

    <label for="subject">Chủ đề:</label>
    <input type="text" id="subject" name="subject" required>

    <label for="message">Nội dung:</label>
    <textarea id="message" name="message" required></textarea>

    <button type="submit">Gửi</button>
    <p id="contact-response"></p>
</form>

<script>
const contactAPIUrl = "https://255125pygl.execute-api.ap-southeast-1.amazonaws.com/production/contact";

document.getElementById("contact-form").addEventListener("submit", function(event) {
    event.preventDefault();
    
    const formData = {
        name: document.getElementById("name").value,
        email: document.getElementById("email").value,
        organization: document.getElementById("organization").value || "",
        subject: document.getElementById("subject").value || "",
        message: document.getElementById("message").value
    };

    fetch(contactAPIUrl, {
        method: "POST",
        body: JSON.stringify(formData), // 🔥 Gửi dữ liệu dưới dạng JSON
        headers: { "Content-Type": "application/json" }
    })
    .then(response => response.json())
    .then(data => {
        console.log("Dữ liệu từ API:", data);
        document.getElementById("contact-response").innerText = data.message || "Gửi thành công!";
    })
    .catch(error => {
        document.getElementById("contact-response").innerText = "Có lỗi xảy ra, vui lòng thử lại!";
        console.error("Lỗi khi gửi contact form:", error);
    });
});
</script>
