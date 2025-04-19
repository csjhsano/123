<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>深致學生資料系統</title>
  <style>
    body {
      font-family: "Microsoft JhengHei", sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 0;
    }
    .header {
      background-color: #2c3e50;
      color: white;
      padding: 20px;
      text-align: center;
    }
    .container {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-around;
      padding: 20px;
    }
    .card {
      background-color: white;
      border-radius: 10px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      padding: 20px;
      margin: 10px;
      flex: 1 1 400px;
    }
    .form-group {
      margin-bottom: 15px;
    }
    label {
      font-weight: bold;
    }
    input[type="text"],
    input[type="email"],
    input[type="file"] {
      width: 100%;
      padding: 8px;
      margin-top: 5px;
    }
    button {
      background-color: #3498db;
      color: white;
      border: none;
      padding: 10px 20px;
      cursor: pointer;
      border-radius: 5px;
    }
    button:hover {
      background-color: #2980b9;
    }
    .result-area {
      margin-top: 15px;
      padding: 10px;
      background-color: #ecf0f1;
      border-left: 5px solid #2980b9;
      display: none;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
    }
    th, td {
      border: 1px solid #aaa;
      padding: 8px;
      text-align: left;
    }
  </style>
</head>
<body>
  <div class="header">
    <h1>深致學生資料系統</h1>
    <p>照片上傳與成績查詢平台</p>
  </div>

  <div class="container">
    <!-- 上傳卡片 -->
    <div class="card">
      <h2>照片上傳</h2>
      <form id="uploadForm">
        <div class="form-group">
          <label for="name">姓名</label>
          <input type="text" id="name" name="name" required />
        </div>
        <div class="form-group">
          <label for="email">電子郵件</label>
          <input type="email" id="email" name="email" required />
        </div>
        <div class="form-group">
          <label for="subject">科目</label>
          <input type="text" id="subject" name="subject" required />
        </div>
        <div class="form-group">
          <label for="file">選擇照片（JPG/PNG）</label>
          <input type="file" id="file" name="file" accept="image/*" required />
        </div>
        <button type="submit">上傳照片</button>
      </form>
      <div id="uploadResult" class="result-area"></div>
    </div>

    <!-- 查詢卡片 -->
    <div class="card">
      <h2>成績查詢</h2>
      <div class="form-group">
        <label for="queryEmail">請輸入電子郵件</label>
        <input type="email" id="queryEmail" required />
      </div>
      <button id="queryBtn">查詢成績</button>
      <div id="queryResult" class="result-area"></div>
    </div>
  </div>

  <script>
    const PHOTO_UPLOAD_URL = "https://script.google.com/macros/s/AKfycbz1Pm6T7KhW6WllAOlBtLxeBXlhZHsLjxvkTRv-i6gHQBD5Zil8uAgc2p50yVITHATI/exec";
    const SCORE_QUERY_URL = "https://script.google.com/macros/s/AKfycbz1Pm6T7KhW6WllAOlBtLxeBXlhZHsLjxvkTRv-i6gHQBD5Zil8uAgc2p50yVITHATI/exec";

    // 照片上傳處理
    document.getElementById('uploadForm').addEventListener('submit', async function(e) {
      e.preventDefault();
      const name = document.getElementById('name').value.trim();
      const email = document.getElementById('email').value.trim();
      const subject = document.getElementById('subject').value.trim();
      const fileInput = document.getElementById('file');
      const resultArea = document.getElementById('uploadResult');

      if (!fileInput.files.length) {
        resultArea.textContent = '請選擇照片檔案';
        resultArea.style.display = 'block';
        return;
      }

      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function() {
        const base64Data = reader.result;

        fetch(PHOTO_UPLOAD_URL, {
          method: 'POST',
          body: new URLSearchParams({
            name: name,
            email: email,
            subject: subject,
            file: base64Data
          })
        })
        .then(res => res.text())
        .then(text => {
          resultArea.textContent = text.includes('上傳成功') ? '✅ 上傳成功！' : '❌ 上傳失敗：' + text;
          resultArea.style.display = 'block';
        })
        .catch(err => {
          resultArea.textContent = '❌ 發生錯誤：' + err.message;
          resultArea.style.display = 'block';
        });
      };
      reader.readAsDataURL(file);
    });

    // 成績查詢處理
    document.getElementById("queryBtn").addEventListener("click", function () {
      const email = document.getElementById("queryEmail").value.trim();
      const resultArea = document.getElementById("queryResult");

      if (!email) {
        resultArea.textContent = "請輸入電子郵件";
        resultArea.style.display = "block";
        return;
      }

      resultArea.innerHTML = "🔍 查詢中...";
      resultArea.style.display = "block";

      fetch(`${SCORE_QUERY_URL}?email=${encodeURIComponent(email)}`)
        .then(res => res.json())
        .then(data => {
          if (data.found) {
            let html = "<h3>查詢結果</h3><table>";
            for (let key in data) {
              if (key !== "found") {
                html += `<tr><th>${key}</th><td>${data[key]}</td></tr>`;
              }
            }
            html += "</table>";
            resultArea.innerHTML = html;
          } else {
            resultArea.textContent = "❌ 查無資料：" + (data.message || "請確認輸入的 Email 是否正確");
          }
        })
        .catch(err => {
          resultArea.textContent = "❌ 查詢失敗：" + err.message;
        });
    });
  </script>
</body>
</html>
