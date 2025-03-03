---
title: "Contact Me"
date: 2025-03-03T07:00:00Z
---

<form id="contact-form">
    <label for="name">Họ và Tên:</label>
    <input type="text" id="name" name="name" required>

    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>

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
        body: JSON.stringify(formData), // 🔥 Chuyển dữ liệu sang JSON
        headers: { "Content-Type": "application/json" } // 🔥 Đảm bảo đúng Content-Type
    })
    .then(response => response.json())
    .then(data => {
        document.getElementById("contact-response").innerText = data.message || "Gửi thành công!";
    })
    .catch(error => {
        document.getElementById("contact-response").innerText = "Có lỗi xảy ra, vui lòng thử lại!";
        console.error("Lỗi khi gửi contact form:", error);
    });
});
</script>