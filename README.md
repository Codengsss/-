# -
هذا البرنامج يستخرج النص من الصور 
from flask import Flask, render_template, request, send_file
import pytesseract
from PIL import Image
from werkzeug.utils import secure_filename
import os
from docx import Document

# إعداد Flask
app = Flask(__name__)
UPLOAD_FOLDER = 'uploads'
RESULT_FOLDER = 'results'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
app.config['RESULT_FOLDER'] = RESULT_FOLDER

# تأكد من وجود المجلدات
os.makedirs(UPLOAD_FOLDER, exist_ok=True)
os.makedirs(RESULT_FOLDER, exist_ok=True)

# الصفحة الرئيسية
@app.route('/')
def home():
    return '''
    <!DOCTYPE html>
    <html>
    <head>
        <title>استخراج النصوص من الصور</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                text-align: center;
                background-color: #f4f4f9;
                color: #333;
            }
            .container {
                margin: 50px auto;
                padding: 20px;
                border: 1px solid #ddd;
                border-radius: 10px;
                background-color: #fff;
                width: 50%;
            }
            input[type="file"] {
                margin: 10px;
            }
            button {
                padding: 10px 20px;
                background-color: #007bff;
                color: #fff;
                border: none;
                border-radius: 5px;
                cursor: pointer;
            }
            button:hover {
                background-color: #0056b3;
            }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>استخراج النصوص من الصور</h1>
            <p>قم برفع الصورة لاستخراج النصوص منها وحفظها في ملف Word.</p>
            <form action="/upload" method="post" enctype="multipart/form-data">
                <input type="file" name="image" accept="image/*" required>
                <br>
                <button type="submit">رفع الصورة</button>
            </form>
        </div>
    </body>
    </html>
    '''

# معالجة رفع الصورة
@app.route('/upload', methods=['POST'])
def upload_file():
    if 'image' not in request.files:
        return "لم يتم رفع أي صورة"
    
    file = request.files['image']
    if file.filename == '':
        return "لم يتم اختيار أي ملف"
    
    filename = secure_filename(file.filename)
    file_path = os.path.join(app.config['UPLOAD_FOLDER'], filename)
    file.save(file_path)

    # استخراج النصوص
    extracted_text = extract_text_from_image(file_path)

    # إنشاء ملف Word
    word_file_path = create_word_file(extracted_text, filename)

    return send_file(word_file_path, as_attachment=True)

# استخراج النصوص باستخدام Tesseract
def extract_text_from_image(image_path):
    try:
        image = Image.open(image_path)
        text = pytesseract.image_to_string(image, lang='ara+eng')
        return text
    except Exception as e:
        return f"حدث خطأ أثناء استخراج النصوص: {e}"

# إنشاء ملف Word
def create_word_file(text, original_filename):
    try:
        document = Document()
        document.add_heading('الن
