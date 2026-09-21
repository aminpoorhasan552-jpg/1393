<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>سامانه مدیریت هنرستان خیامی</title>
    <style>
        body { font-family: Tahoma, sans-serif; background: #f0f2f5; margin: 0; padding: 10px; font-size: 14px; }
        .card { background: white; padding: 15px; border-radius: 12px; margin-bottom: 15px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
        h3, h4 { color: #2c3e50; margin-top: 0; border-bottom: 2px solid #eee; padding-bottom: 5px; }
        input, select { width: 100%; padding: 10px; margin: 8px 0; border: 1px solid #ddd; border-radius: 6px; box-sizing: border-box; }
        button { width: 100%; padding: 12px; border: none; border-radius: 6px; cursor: pointer; font-weight: bold; margin-top: 5px; }
        .btn-primary { background: #3498db; color: white; }
        .btn-success { background: #27ae60; color: white; }
        .btn-danger { background: #e74c3c; color: white; }
        .btn-info { background: #9b59b6; color: white; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: center; }
        th { background: #f8f9fa; }
        .hidden { display: none; }
        .nav { display: flex; gap: 5px; margin-bottom: 15px; }
        .nav button { font-size: 12px; padding: 8px; }
    </style>
</head>
<body>

<div id="loginSection" class="card">
    <h3>ورود به سامانه مدیریت</h3>
    <input type="text" id="user" placeholder="نام کاربری">
    <input type="password" id="pass" placeholder="رمز عبور">
    <button class="btn-primary" onclick="login()">ورود</button>
</div>

<div id="mainApp" class="hidden">
    <div class="nav">
        <button class="btn-info" onclick="show('regStu')">ثبت دانش‌آموز</button>
        <button class="btn-info" onclick="show('listStu')">لیست دانش‌آموزان</button>
        <button class="btn-success" onclick="show('entryGrades')">ثبت نمره</button>
        <button class="btn-primary" onclick="show('reportSection')">کارنامه</button>
    </div>

    <!-- ۱. ثبت دانش‌آموز -->
    <div id="regStu" class="card hidden">
        <h3>ثبت دانش‌آموز جدید</h3>
        <input type="text" id="newStuName" placeholder="نام و نام خانوادگی">
        <input type="text" id="newStuID" placeholder="کد ملی / شماره دانشجویی">
        <select id="newStuMajor">
            <option value="الکترونیک">الکترونیک</option>
            <option value="کامپیوتر">کامپیوتر</option>
            <option value="مکانیک">مکانیک</option>
        </select>
        <button class="btn-success" onclick="addStudent()">ثبت در سیستم</button>
    </div>

    <!-- ۲. لیست دانش‌آموزان -->
    <div id="listStu" class="card hidden">
        <h3>لیست دانش‌آموزان</h3>
        <div id="studentList"></div>
    </div>

    <!-- ۳. ثبت نمرات -->
    <div id="entryGrades" class="card hidden">
        <h3>ثبت نمرات</h3>
        <select id="selectStudent" onchange="loadGradeInputs()"></select>
        
        <div id="gradeInputs" class="hidden">
            <hr>
            <h5>دروس عمومی</h5>
            مستمر۱: <input type="number" id="g-m1" placeholder="۰-۲۰">
            پایانی۱: <input type="number" id="g-t1" placeholder="۰-۲۰">
            مستمر۲: <input type="number" id="g-m2" placeholder="۰-۲۰">
            پایانی۲: <input type="number" id="g-t2" placeholder="۰-۲۰">

            <hr>
            <h5>دروس پودمانی (۵ پودمان)</h5>
            <div id="poudmanFields"></div>
            <button class="btn-success" onclick="saveGrades()">ذخیره نمرات</button>
        </div>
    </div>

    <!-- ۴. کارنامه -->
    <div id="reportSection" class="card hidden">
        <h3>کارنامه تحصیلی</h3>
        <select id="reportStudentSelect" onchange="generateReport()"></select>
        <div id="reportOutput"></div>
    </div>
</div>

<script>
    // دیتابیس فرضی (ذخیره در مرورگر)
    let students = JSON.parse(localStorage.getItem('students')) || [];
    let grades = JSON.parse(localStorage.getItem('grades')) || {};

    // تولید ورودی‌های پودمان
    let phtml = "";
    for(let i=1; i<=5; i++) {
        phtml += `<div style="border-bottom:1px solid #eee; margin-bottom:5px;">
            پودمان ${i}: 
            <input type="number" class="p-m" placeholder="مستمر (۰-۵)" style="width:45%">
            <select class="p-p" style="width:45%">
                <option value="1">شایستگی ۱</option>
                <option value="2">شایستگی ۲</option>
                <option value="3">شایستگی ۳</option>
            </select>
        </div>`;
    }
    document.getElementById('poudmanFields').innerHTML = phtml;

    function login() {
        if(document.getElementById('user').value === "admin") {
            document.getElementById('loginSection').classList.add('hidden');
            document.getElementById('mainApp').classList.remove('hidden');
            updateSelectors();
        } else alert("خطا در ورود");
    }

    function show(id) {
        ['regStu','listStu','entryGrades','reportSection'].forEach(s => document.getElementById(s).classList.add('hidden'));
        document.getElementById(id).classList.remove('hidden');
        if(id === 'listStu') renderStudentList();
        if(id === 'entryGrades') updateSelectors();
        if(id === 'reportSection') updateSelectors();
    }

    // مدیریت دانش‌آموزان
    function addStudent() {
        const name = document.getElementById('newStuName').value;
        const id = document.getElementById('newStuID').value;
        const major = document.getElementById('newStuMajor').value;
        if(!name || !id) return alert("لطفا مشخصات را پر کنید");
        
        students.push({id, name, major});
        localStorage.setItem('students', JSON.stringify(students));
        alert("ثبت شد");
        document.getElementById('newStuName').value = "";
        document.getElementById('newStuID').value = "";
    }

    function renderStudentList() {
        let html = "<table><tr><th>نام</th><th>رشته</th></tr>";
        students.forEach(s => { html += `<tr><td>${s.name}</td><td>${s.major}</td></tr>`; });
        html += "</table>";
        document.getElementById('studentList').innerHTML = html;
    }

    function updateSelectors() {
        let options = '<option value="">انتخاب کنید...</option>';
        students.forEach(s => { options += `<option value="${s.id}">${s.name}</option>`; });
        document.getElementById('selectStudent').innerHTML = options;
        document.getElementById('reportStudentSelect').innerHTML = options;
    }

    function loadGradeInputs() {
        const id = document.getElementById('selectStudent').value;
        document.getElementById('gradeInputs').classList.toggle('hidden', !id);
    }

    // مدیریت نمرات
    function saveGrades() {
        const id = document.getElementById('selectStudent').value;
        if(!id) return;

        const gData = {
            gen: { 
                m1: parseFloat(document.getElementById('g-m1').value)||0, 
                t1: parseFloat(document.getElementById('g-t1').value)||0,
                m2: parseFloat(document.getElementById('g-m2').value)||0, 
                t2: parseFloat(document.getElementById('g-t2').value)||0 
            },
            mods: []
        };

        document.querySelectorAll('#poudmanFields > div').forEach(div => {
            gData.mods.push({
                m: parseFloat(div.querySelector('.p-m').value)||0,
                p: parseInt(div.querySelector('.p-p').value)||0
            });
        });

        grades[id] = gData;
        localStorage.setItem('grades', JSON.stringify(grades));
        alert("نمرات ذخیره شد");
    }

    // تولید کارنامه
    function generateReport() {
        const id = document.getElementById('reportStudentSelect').value;
        const student = students.find(s => s.id === id);
        const g = grades[id];

        if(!student || !g) { alert("نمره یا دانش‌آموز یافت نشد"); return; }

        // محاسبه عمومی
        const genScore = ((g.gen.m1*2) + (g.gen.t1*1) + (g.gen.m2*4) + (g.gen.t2*1)) / 8;
        
        // محاسبه پودمانی
        let totalModScore = 0;
        let allPassed = true;
        let modDetails = "";

        g.mods.forEach((m, i) => {
            let pScore = m.p * 5;
            let currentModFinal = m.m + pScore;
            totalModScore += currentModFinal;
            if(currentModFinal < 12) allPassed = false;
            modDetails += `<li>پودمان ${i+1}: ${currentModFinal.toFixed(1)}</li>`;
        });

        const finalModAvg = totalModScore / 4;

        document.getElementById('reportOutput').innerHTML = `
            <div style="text-align:center">
                <h4>کارنامه: ${student.name}</h4>
                <p>رشته: ${student.major}</p>
            </div>
            <table>
                <tr><th>درس</th><th>نمره</th><th>وضعیت</th></tr>
                <tr><td>عمومی</td><td>${genScore.toFixed(2)}</td><td>${genScore >= 10 ? 'قبول' : 'مردود'}</td></tr>
                <tr><td>پودمانی</td><td>${finalModAvg.toFixed(2)}</td><td>${(allPassed && finalModAvg >= 12) ? 'قبول' : 'ناتمام'}</td></tr>
            </table>
            <hr>
            <p>جزئیات پودمان‌ها:</p>
            <ul>${modDetails}</ul>
        `;
    }
</script>
</body>
</html>

