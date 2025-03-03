---
title: "Contact Me"
date: 2025-02-22T12:00:00Z
---

<h2>Liên hệ với tôi</h2>
<p>Vui lòng điền vào form dưới đây, tôi sẽ phản hồi sớm nhất có thể.</p>

<form id="contact-form">
    <div class="form-group">
        <label for="name">Họ và Tên:</label>
        <input type="text" id="name" name="name" required>
    </div>
    <div class="form-group">
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required>
    </div>
    <div class="form-group">
        <label for="subject">Chủ đề:</label>
        <input type="text" id="subject" name="subject" required>
    </div>
    <div class="form-group">
        <label for="message">Nội dung:</label>
        <textarea id="message" name="message" required></textarea>
    </div>
    <!-- Thêm Google reCAPTCHA -->
    <div class="g-recaptcha" data-sitekey="6Lc4UOgqAAAAAA15BsINUHWaZoUt8Tbu9rFxF4is"></div>
    <button type="submit">Gửi</button>
    <p id="contact-response"></p>
</form>

<!-- Thêm script reCAPTCHA -->
<script src="https://www.google.com/recaptcha/api.js" async defer></script>

<script>
const contactAPIUrl = "https://255125pygl.execute-api.ap-southeast-1.amazonaws.com/production/contact";

document.getElementById("contact-form").addEventListener("submit", function(event) {
    event.preventDefault();

    const recaptchaResponse = document.querySelector(".g-recaptcha-response").value;
    if (!recaptchaResponse) {
        document.getElementById("contact-response").innerText = "Vui lòng xác minh reCAPTCHA!";
        return;
    }

    const formData = {
        name: document.getElementById("name").value,
        email: document.getElementById("email").value,
        subject: document.getElementById("subject").value,
        message: document.getElementById("message").value,
        recaptcha: recaptchaResponse  // Gửi token reCAPTCHA về server
    };

    fetch(contactAPIUrl, {
        method: "POST",
        body: JSON.stringify(formData),
        headers: { "Content-Type": "application/json" }
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
