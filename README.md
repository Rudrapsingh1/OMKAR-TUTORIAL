<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Omkar Tutorial Classes</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background: linear-gradient(90deg, #f8b400, #f85f73);
            color: #333;
        }
        header {
            background-color: #4caf50;
            color: white;
            padding: 15px;
            text-align: center;
            font-size: 24px;
            font-weight: bold;
        }
        .container {
            max-width: 800px;
            margin: 20px auto;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .form-group {
            margin-bottom: 15px;
        }
        .form-group label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        .form-group input {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        button {
            background-color: #4caf50;
            color: white;
            border: none;
            padding: 10px 15px;
            font-size: 16px;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover {
            background-color: #45a049;
        }
        .class-list {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-top: 20px;
        }
        .class-item {
            background-color: #f8b400;
            color: white;
            padding: 10px;
            border-radius: 5px;
            cursor: pointer;
            text-align: center;
            min-width: 100px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .class-item:hover {
            background-color: #f85f73;
        }
        .class-item button {
            margin-left: 10px;
            background-color: red;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            padding: 5px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        table th, table td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        table th {
            background-color: #f4f4f4;
        }
        .attendance-buttons button {
            margin-right: 5px;
            padding: 5px 10px;
            background-color: white;
            color: black;
            border: 1px solid #ddd;
            cursor: pointer;
        }
        .attendance-buttons button.present {
            background-color: green;
            color: white;
        }
        .attendance-buttons button.absent {
            background-color: red;
            color: white;
        }

        @media screen and (max-width: 768px) {
            .container {
                padding: 10px;
                margin: 10px;
            }
            .class-list {
                flex-direction: column;
                align-items: flex-start;
            }
            table {
                font-size: 12px;
            }
        }
    </style>
</head>
<body>
    <header>Omkar Tutorial Classes</header>
    <div class="container" id="homePage">
        <h2>Classes</h2>
        <div class="form-group">
            <label for="className">Add Class:</label>
            <input type="text" id="className" placeholder="Enter class name">
        </div>
        <button onclick="addClass()">Add Class</button>
        <div class="class-list" id="classList"></div>
    </div>

    <div class="container" id="classPage" style="display: none;">
        <button onclick="goBack()">Back</button>
        <h2 id="currentClass">Class Name</h2>
        <div class="form-group">
            <label for="studentName">Add Student:</label>
            <input type="text" id="studentName" placeholder="Enter student name">
        </div>
        <button onclick="addStudent()">Add Student</button>

        <div class="form-group">
            <label for="dateTime">Select Date and Time:</label>
            <input type="datetime-local" id="dateTime">
        </div>

        <table>
            <thead>
                <tr>
                    <th>Student Name</th>
                    <th>Attendance</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody id="studentTable"></tbody>
        </table>
        <button onclick="downloadAttendance()">Download Attendance PDF</button>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.4.0/jspdf.umd.min.js"></script>
    <script>
        const classes = JSON.parse(localStorage.getItem('classes')) || {};
        let currentClass = '';
        let selectedDateTime = '';

        function saveClasses() {
            localStorage.setItem('classes', JSON.stringify(classes));
        }

        function addClass() {
            const className = document.getElementById('className').value;
            if (!className) {
                alert('Please enter a class name');
                return;
            }
            if (!classes[className]) {
                classes[className] = [];
                saveClasses();
                renderClasses();
            }
            document.getElementById('className').value = '';
        }

        function removeClass(className) {
            delete classes[className];
            saveClasses();
            renderClasses();
        }

        function renderClasses() {
            const classList = document.getElementById('classList');
            classList.innerHTML = '';
            Object.keys(classes).forEach(className => {
                const classItem = document.createElement('div');
                classItem.className = 'class-item';
                classItem.innerHTML = `
                    ${className}
                    <button onclick="removeClass('${className}')">Remove</button>
                `;
                classItem.onclick = () => openClass(className);
                classList.appendChild(classItem);
            });
        }

        function openClass(className) {
            currentClass = className;
            document.getElementById('homePage').style.display = 'none';
            document.getElementById('classPage').style.display = 'block';
            document.getElementById('currentClass').textContent = `Class: ${className}`;
            renderStudents();
        }

        function goBack() {
            document.getElementById('homePage').style.display = 'block';
            document.getElementById('classPage').style.display = 'none';
        }

        function addStudent() {
            const studentName = document.getElementById('studentName').value;
            if (!studentName) {
                alert('Please enter a student name');
                return;
            }
            if (!classes[currentClass].some(student => student.name === studentName)) {
                classes[currentClass].push({ name: studentName, present: null });
                saveClasses();
                renderStudents();
            }
            document.getElementById('studentName').value = '';
        }

        function removeStudent(index) {
            classes[currentClass].splice(index, 1);
            saveClasses();
            renderStudents();
        }

        function renderStudents() {
            const studentTable = document.getElementById('studentTable');
            studentTable.innerHTML = '';
            classes[currentClass].forEach((student, index) => {
                const row = document.createElement('tr');

                const nameCell = document.createElement('td');
                nameCell.textContent = student.name;
                row.appendChild(nameCell);

                const attendanceCell = document.createElement('td');
                attendanceCell.className = 'attendance-buttons';

                const presentButton = document.createElement('button');
                presentButton.textContent = 'Present';
                presentButton.className = student.present === true ? 'present' : '';
                presentButton.onclick = (e) => {
                    e.stopPropagation();
                    student.present = true;
                    saveClasses();
                    renderStudents();
                };

                const absentButton = document.createElement('button');
                absentButton.textContent = 'Absent';
                absentButton.className = student.present === false ? 'absent' : '';
                absentButton.onclick = (e) => {
                    e.stopPropagation();
                    student.present = false;
                    saveClasses();
                    renderStudents();
                };

                attendanceCell.appendChild(presentButton);
                attendanceCell.appendChild(absentButton);
                row.appendChild(attendanceCell);

                const actionCell = document.createElement('td');
                const removeButton = document.createElement('button');
                removeButton.textContent = 'Remove';
                removeButton.onclick = () => removeStudent(index);
                actionCell.appendChild(removeButton);
                row.appendChild(actionCell);

                studentTable.appendChild(row);
            });
        }

        async function downloadAttendance() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();
            const currentDateTime = new Date().toLocaleString();
            const dateTime = selectedDateTime || currentDateTime;

            // Add Tuition Name at the top of the PDF
            doc.setFontSize(18);
            doc.text('OMKAR TUTORIAL CLASSES, NAGPUR', 10, 10);

            // Add Class Name and Date & Time
            doc.setFontSize(16);
            doc.text(`Attendance Sheet for ${currentClass}`, 10, 20);
            doc.text(`Date & Time: ${dateTime}`, 10, 30);

            let y = 40;
            doc.setFontSize(12);
            classes[currentClass].forEach((student, index) => {
                const status = student.present === true ? 'Present' : student.present === false ? 'Absent' : 'Not Marked';
                doc.text(`${index + 1}. ${student.name} - ${status}`, 10, y);
                y += 10;
            });

            doc.save(`${currentClass}_Attendance.pdf`);
        }

        renderClasses();
    </script>
</body>
</html>
