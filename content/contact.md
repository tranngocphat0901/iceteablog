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
        <label for="organization">Tổ chức (Tùy chọn):</label>
        <input type="text" id="organization" name="organization">
    </div>
    <div class="form-group">
        <label for="subject">Chủ đề:</label>
        <input type="text" id="subject" name="subject" required>
    </div>
    <div class="form-group">
        <label for="message">Nội dung:</label>
        <textarea id="message" name="message" required></textarea>
    </div>
    <button type="submit">Gửi</button>
    <p id="contact-response"></p>
</form>

<style>
#contact-form {
    max-width: 600px;
    margin: auto;
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 8px;
    background: #f9f9f9;
    font-family: Arial, sans-serif;
}

.form-group {
    display: flex;
    flex-direction: column;
    margin-bottom: 15px;
}

label {
    font-weight: bold;
    margin-bottom: 5px;
}

input, textarea {
    width: 100%;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

textarea {
    height: 100px;
}

button {
    width: 100%;
    padding: 10px;
    border: none;
    background: #007bff;
    color: white;
    font-size: 16px;
    border-radius: 4px;
    cursor: pointer;
}

button:hover {
    background: #0056b3;
}

#contact-response {
    margin-top: 10px;
    font-weight: bold;
}
</style>

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