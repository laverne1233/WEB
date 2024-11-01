
import { DBPaths } from '/Bus Operator/js/DB.js';
import { convertTo12Hour, convertToMilitaryTime, convertToPascal, getCurrentDateTimeInMillis } from '/Bus Operator/utils/Utils.js';
import firebaseConfig from '/CONFIG.js';

firebase.initializeApp(firebaseConfig);
const database = firebase.database();
const myData = JSON.parse(sessionStorage.getItem('currentUser'));

const loader = document.querySelector('.loader-container');

const searchEmpInput = document.getElementById('searchEmpInput');
// ADD EMPLOYEE FORM ELEMENTS
const addEmpFormModal = document.getElementById('employeeModal');
const opCloseBtn = document.querySelector('.opClose');
const employeeTable = document.querySelector(".employee-content > #bus-table");
const addBusopBtn = document.getElementById('addBusopBtn');
const addEmpForm = document.getElementById('addEmpForm');
const empPhotoId = document.getElementById('empPhotoId');
const addEmpPicBtn = document.getElementById('addEmpPicBtn');
const empFullNameInput = document.getElementById('empFullName');
const empEmailInput = document.getElementById('empEmail');
const empContactNumInput = document.getElementById('empContactNum');
const empPasswordInput = document.getElementById('empPassword');
const empTypeInput = document.getElementById('empType');
const saveBusBtn = document.getElementById('saveBusBtn');

let empId;
let fileNameEmpPhoto;
let fileEmpPhoto;
let action;
let empArray;

document.addEventListener('DOMContentLoaded', init);
addEmpForm.addEventListener('submit', saveEmpData);
opCloseBtn.addEventListener('click', hideAddBusForm)
addBusopBtn.addEventListener('click', addEmp)
searchEmpInput.addEventListener('input', handleSearchEmp);


//TAB SECTION
function init() {
    // Add event listener to radio buttons to switch content on change
    const radioButtons = document.querySelectorAll('input[name="data"]');
    radioButtons.forEach(radio => {
        radio.addEventListener('change', function () {
            switchContent(this.id);
        });
    });

    generateEmployees();
    generateReports();
    generatePayStatements();
    generateUsers();
};

function switchContent(selectedTab) {
    // Get all content divs
    const contentDivs = document.querySelectorAll('.employee-content, .status-content, .account-content');

    // Hide all content divs
    contentDivs.forEach(div => {
        div.style.display = 'none';
    });

    // Show the selected content div based on the selected tab
    if (selectedTab === 'employee') {
        document.querySelector('.employee-content').style.display = 'block';
    } else if (selectedTab === 'status') {
        document.querySelector('.status-content').style.display = 'block';
    } else if (selectedTab === 'account') {
        document.querySelector('.account-content').style.display = 'block';
    }
}



// EMPLOYEE SECTION
function generateEmployees() {

    createTableHeader();

    const busDriverRef = database.ref(`${DBPaths.EMPLOYEES}`);
    empArray = [];

    busDriverRef.once('value',
        (snapshot) => {
            snapshot.forEach((busDriver) => {

                const empKey = busDriver.key;
                const empData = busDriver.val();
                empData["key"] = empKey;

                if (empData.companyId === myData.companyId) {
                    empArray.push(empData);
                    createEmpTables(empData);
                }

            });
        }
    )
}

function handleSearchEmp() {
    createTableHeader();

    const searchTerm = searchEmpInput.value.toLowerCase().trim();

    // Filter data based on search term
    const results = empArray.filter(item => item.fullName.toLowerCase().includes(searchTerm));
    // Render search results
    renderResults(results);
}

function renderResults(results) {

    results.forEach(result => {

        createEmpTables(result);
    });
}

function createTableHeader() {

    employeeTable.innerHTML = "";
    const tr = document.createElement("tr");

    // Array of column headers
    const headers = [
        "ID no.",
        "Fullname",
        "Email",
        "Contact No.",
        "Picture",
        "Password",
        "Type",
        "Actions"
    ];

    // Create <th> elements for each column header and append them to the <tr> element
    headers.forEach(headerText => {
        const th = document.createElement("th");
        th.textContent = headerText;
        tr.appendChild(th);
    });

    employeeTable.appendChild(tr);

}

function createEmpTables(empData) {


    const row = document.createElement("tr");

    const idTd = document.createElement("td");
    idTd.textContent = empData.key;

    const fullNameTd = document.createElement("td");
    fullNameTd.textContent = empData.fullName;

    const emailTd = document.createElement("td");
    emailTd.textContent = empData.email;

    const phoneNumTd = document.createElement("td");
    phoneNumTd.textContent = empData.phoneNum;

    const userImgTd = document.createElement("td");
    const userImg = document.createElement("img");
    userImgTd.appendChild(userImg);
    userImg.src = empData.imageUrl;

    const passwordTd = document.createElement("td");
    passwordTd.textContent = empData.password;

    const typeTd = document.createElement("td");
    typeTd.textContent = empData.type;

    const actionsTd = document.createElement("td");
    const editLink = document.createElement("a");
    editLink.href = "#";
    editLink.setAttribute("data-target", "edit-operator");
    const editIcon = document.createElement("i");
    editIcon.classList.add("fa-solid", "fa-user-pen", "edit");
    const editSpan = document.createElement("span");
    editLink.appendChild(editIcon);
    editLink.appendChild(editSpan);

    const deleteLink = document.createElement("a");
    deleteLink.href = "#";
    deleteLink.setAttribute("data-target", "edit-operator");
    const deleteIcon = document.createElement("i");
    deleteIcon.classList.add("fa-solid", "fa-eraser", "delete");
    const deleteSpan = document.createElement("span");
    deleteLink.appendChild(deleteIcon);
    deleteLink.appendChild(deleteSpan);



    actionsTd.appendChild(editLink);
    actionsTd.appendChild(deleteLink);

    row.appendChild(idTd);
    row.appendChild(fullNameTd);
    row.appendChild(emailTd);
    row.appendChild(phoneNumTd);
    row.appendChild(userImgTd);
    row.appendChild(passwordTd);
    row.appendChild(typeTd);
    row.appendChild(actionsTd);

    employeeTable.appendChild(row);

    editIcon.addEventListener("click", function () {
        editEmp(empData)
    });
    deleteIcon.addEventListener("click", function () {
        deleteEmp(empData)
    });
}

function editEmp(empData) {
    action = 'Edit';
    empId = empData.key;
    empFullNameInput.value = empData.fullName;
    empEmailInput.value = empData.email;
    empContactNumInput.value = empData.phoneNum;
    empPhotoId.src = empData.imageUrl;
    empPasswordInput.value = empData.password;
    empTypeInput.value = empData.type;
    showAddBusForm();
}

function deleteEmp(empData) {
    const isConfirmed = window.confirm("Are you sure you want to remove this account?");

    if (isConfirmed) {
        const dbRef = firebase.database().ref(`${DBPaths.EMPLOYEES}/${empData.key}`);

        dbRef.remove()
            .then(() => {
                alert('Employee deleted successfully.');
                generateEmployees();
            })
            .catch((error) => {
                alert('Employee deletion failed.');
            });
    }
}

function addEmp() {
    action = 'Add';
    empFullNameInput.value = '';
    empEmailInput.value = '';
    empContactNumInput.value = '';
    empPhotoId.src = '/Bus Operator/images/profile.png';
    empPasswordInput.value = '';
    empTypeInput.value = '';
    showAddBusForm();
}

async function saveEmpData(event) {
    event.preventDefault();

    const isConfirmed = window.confirm("Are you sure all information are correct?");

    if (isConfirmed && empDetailsAreValid()) {
        showLoader();

        const emailExists = await checkIfEmailOrPhoneExists(empEmailInput.value, 'email');
        const phoneExists = await checkIfEmailOrPhoneExists(empContactNumInput.value, 'phoneNum');

        if (emailExists) {
            alert('Email already exists.');
            hideLoader();
            return;
        }

        if (phoneExists) {
            alert('Contact number already exists.');
            hideLoader();
            return;
        }

        if (action === 'Add') {
            uploadEmpImage();
        }
        if (action === 'Edit') {
            validateImage();
        }
    }
}

function checkIfEmailOrPhoneExists(value, type) {
    return new Promise((resolve, reject) => {
        const busDriverRef = database.ref(`${DBPaths.EMPLOYEES}`);
        busDriverRef.once('value', (snapshot) => {
            let exists = false;
            snapshot.forEach((busDriver) => {
                const empData = busDriver.val();
                if ((type === 'email' && empData.email === value) || (type === 'phoneNum' && empData.phoneNum === value)) {
                    exists = true;
                }
            });
            resolve(exists);
        }, (error) => {
            reject(error);
        });
    });
}

function uploadEmpImage() {
    const ref = firebase.storage().ref(`${DBPaths.EMPLOYEES}`);

    const metadata = {
        contentType: fileEmpPhoto.type
    };

    const task = ref.child(fileNameEmpPhoto).put(fileEmpPhoto, metadata);

    // Monitor the upload progress
    task.on('state_changed',
        function (snapshot) {
            // Handle progress
            const progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
            console.log('Upload bus coop photo is ' + progress + '% done');
        },
        function (error) {
            // Handle errors
            console.error('Error uploading file: ', error);
        },
        function () {
            // Handle successful upload
            task.snapshot.ref.getDownloadURL().then(function (downloadURL) {

                if (action === 'Add') {
                    createAccount(downloadURL);
                }
                if (action === 'Edit') {
                    updateAccount(downloadURL);
                }

            });
        }
    );
}

function createAccount(empImgUrl) {

    const empData = {
        fullName: empFullNameInput.value,
        email: empEmailInput.value,
        password: empPasswordInput.value,
        phoneNum: empContactNumInput.value,
        imageUrl: empImgUrl,
        type: convertToPascal(empTypeInput.value),
        busOperatorId: myData.key,
        companyName: myData.companyName,
        companyId: myData.companyId,
        datetimeAdded: new Date().toISOString()
    }; 

    if (empData.type.toLowerCase().includes('driver')) {
        console.log('driver');
        firebase.auth().createUserWithEmailAndPassword(empData.email, empData.password)
        .then((userCredential) => {
            // User created successfully

            const user = userCredential.user;         
            console.log('userCredential', userCredential);   
            console.log('user', user);   
            const empRef = database.ref(`${DBPaths.EMPLOYEES}/${userCredential.uid}`);

            empRef.set(empData)
                .then(() => {

                    hideAddBusForm();
                    generateEmployees();
                })
                .catch(error => {
                    // An error occurred while setting data
                    console.error('Error setting data:', error);
                });
        })
        .catch((error) => {
            // An error occurred while creating the user
            console.error('Error creating user:', error);
        });
    }
    else {

        const id = getCurrentDateTimeInMillis();       
        const empRef = database.ref(`${DBPaths.EMPLOYEES}/${id}`);

        empRef.set(empData)
            .then(() => {

                hideAddBusForm();
                generateEmployees();
            })
            .catch(error => {
                // An error occurred while setting data
                console.error('Error setting data:', error);
            });    }
    



    hideLoader();
}

async function createFirebaseAccount(empData) {
    const { email, password } = empData;
    const userCredential = await firebase.auth().createUserWithEmailAndPassword(email, password);
}


function validateImage() {

    if (addEmpPicBtn && (addEmpPicBtn.files.length === 0 || addEmpPicBtn.value === '')) {
        const imgeSrc = empPhotoId.src;
        updateAccount(imgeSrc);
        console.log('With out Photo');

    }
    else {
        uploadEmpImage();
        console.log('With Photo');
    }

    hideLoader();
}

function updateAccount(empImgUrl) {

    const empData = {
        fullName: empFullNameInput.value,
        email: empEmailInput.value,
        password: empPasswordInput.value,
        phoneNum: empContactNumInput.value,
        imageUrl: empImgUrl,
        type: convertToPascal(empTypeInput.value),
    };

    const empRef = firebase.database().ref(`${DBPaths.EMPLOYEES}/${empId}`);
    empRef.update(empData)
        .then(() => {
            hideAddBusForm();
            generateEmployees();
        })
        .catch(error => {
            console.error('Error updating employee:', error);
        });

}


function empDetailsAreValid() {

    // Check if user photo is different from the placeholder image
    const empPhotoIsNotValid = empPhotoId.src.includes('/images/profile.png');

    // Validate Email
    const emailValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(empEmailInput.value.trim());

    // Validate Password
    const passwordValid = empPasswordInput.value.trim().length >= 8;

    // Validate Phone Number
    const phoneNumValid = /^\d{11}$/.test(empContactNumInput.value.trim());

    if (empPhotoIsNotValid) {
        alert('Please select a user photo');
        return false;
    }

    if (!emailValid) {
        alert('Please enter a valid email address');
        return false;
    }

    if (!passwordValid) {
        alert('Password must be at least 8 characters long');
        return false;
    }

    if (!phoneNumValid) {
        alert('Please enter a valid 11-digit phone number');
        return false;
    }

    return true;
}

window.addEventListener('load', function () {

    addEmpPicBtn.addEventListener('change', function (event) {
        if (this.files && this.files[0]) {
            empPhotoId.onload = () => {
                URL.revokeObjectURL(empPhotoId.src);
            }
            empPhotoId.src = URL.createObjectURL(this.files[0]);
            fileNameEmpPhoto = this.files[0].name;
            fileEmpPhoto = event.target.files[0];
        }
    });
});

// When the Emp clicks anywhere outside of the modal, close it
window.onclick = function (event) {
    if (event.target == addEmpFormModal) {
        hideAddBusForm();
    }
}


//EMPLOYEE STATUS SECTION ELEMENTS
const addReportModal = document.getElementById('reportModal');
const reportCloseFormBtn = document.querySelector(".reportCloseFormBtn");
const addReportForm = document.getElementById('addReportForm');

const reportsTable = document.querySelector(".reports-table");
const addReportBtn = document.getElementById("addReportBtn");

// Input elements
const employeeInput = document.getElementById("employeeInput");
const absentRadioInput = document.getElementById("absentRadioInput");
const dowInput = document.getElementById("dowInput");
const timeInInput = document.getElementById("timeInInput");
const timeOutInput = document.getElementById("timeOutInput");
const breakTimeInput = document.getElementById("breakTimeInput");
const overTimeInput = document.getElementById("overTimeInput");
const dayOffInput = document.getElementById("dayOffInput");

let empReportArray;
let reportId

addReportBtn.addEventListener('click', addReport)
addReportForm.addEventListener('submit', saveReportData);
reportCloseFormBtn.addEventListener('click', hideAddReportForm)

// EMPLOYEE REPORTS
function generateReports() {
    createReportsTableHeader();

    const empReportsRef = database.ref(`${DBPaths.EMPLOYEES_REPORTS}`);
    empReportArray = [];

    empReportsRef.once('value',
        (snapshot) => {
            snapshot.forEach((report) => {

                const empReportKey = report.key;
                const empReportData = report.val();
                empReportData["key"] = empReportKey;

                if (empReportData.companyId === myData.companyId) {
                    empReportArray.push(empReportData);
                    createEmpReportTables(empReportData);
                }
            });
        }
    )
}

function createEmpReportTables(empReportData) {

    const row = document.createElement("tr");

    const employeeTd = document.createElement("td");
    employeeTd.textContent = empReportData.employee;

    const dowTd = document.createElement("td");
    dowTd.textContent = empReportData.date_of_work;

    const timeInTd = document.createElement("td");
    timeInTd.textContent = empReportData.time_in;

    const timeOutTd = document.createElement("td");
    timeOutTd.textContent = empReportData.time_out;

    const breakTimeTd = document.createElement("td");
    breakTimeTd.textContent = empReportData.break_time;

    const overTimeTd = document.createElement("td");
    overTimeTd.textContent = empReportData.over_time;

    const dayOfTd = document.createElement("td");
    dayOfTd.textContent = empReportData.day_off;

    const actionsTd = document.createElement("td");
    const editLink = document.createElement("a");
    editLink.href = "#";
    editLink.setAttribute("data-target", "edit-operator");
    const editIcon = document.createElement("i");
    editIcon.classList.add("fa-solid", "fa-user-pen", "edit");
    const editSpan = document.createElement("span");
    editLink.appendChild(editIcon);
    editLink.appendChild(editSpan);

    const deleteLink = document.createElement("a");
    deleteLink.href = "#";
    deleteLink.setAttribute("data-target", "edit-operator");
    const deleteIcon = document.createElement("i");
    deleteIcon.classList.add("fa-solid", "fa-eraser", "delete");
    const deleteSpan = document.createElement("span");
    deleteLink.appendChild(deleteIcon);
    deleteLink.appendChild(deleteSpan);



    actionsTd.appendChild(editLink);
    actionsTd.appendChild(deleteLink);

    row.appendChild(employeeTd);
    row.appendChild(dowTd);
    row.appendChild(timeInTd);
    row.appendChild(timeOutTd);
    row.appendChild(breakTimeTd);
    row.appendChild(overTimeTd);
    row.appendChild(dayOfTd);
    row.appendChild(actionsTd);

    reportsTable.appendChild(row);

    editIcon.addEventListener("click", function () {
        editReport(empReportData)
    });
    deleteIcon.addEventListener("click", function () {
        deleteReport(empReportData)
    });

    hideLoader();
}

function createReportsTableHeader() {

    reportsTable.innerHTML = "";
    const tr = document.createElement("tr");

    // Array of column headers
    const headers = [
        "Employee",
        "DOW",
        "Time in",
        "Time out",
        "Break Time",
        "Overtime",
        "Day Off",
        "Actions"
    ];

    headers.forEach(headerText => {
        const th = document.createElement("th");
        th.textContent = headerText;
        tr.appendChild(th);
    });

    reportsTable.appendChild(tr);

}

function addReport() {
    action = 'Add';
    employeeInput.value = "";
    absentRadioInput.checked = false;
    dowInput.value = "";
    timeInInput.value = "08:00";
    timeOutInput.value = "17:00";
    breakTimeInput.value = "00:00";
    overTimeInput.value = "18:00";
    dayOffInput.value = "";
    showAddReportForm();
}

function editReport(empReportData) {

    action = 'Edit';
    reportId = empReportData.key;
    employeeInput.value = empReportData.employee;
    absentRadioInput.checked = empReportData.date_of_work === 'Absent' ? true : false;
    dowInput.value = absentRadioInput.checked ? '' : empReportData.date_of_work;
    timeInInput.value = absentRadioInput.checked ? '' : convertToMilitaryTime(empReportData.time_in);
    timeOutInput.value = absentRadioInput.checked ? '' : convertToMilitaryTime(empReportData.time_out);
    breakTimeInput.value = convertToMilitaryTime(empReportData.break_time);
    overTimeInput.value = convertToMilitaryTime(empReportData.over_time);
    dayOffInput.value = empReportData.day_off;
    showAddReportForm();
}

function deleteReport(empReportData) {
    const isConfirmed = window.confirm("Confirm delete?");

    if (isConfirmed) {
        const dbRef = firebase.database().ref(`${DBPaths.EMPLOYEES_REPORTS}/${empReportData.key}`);

        dbRef.remove()
            .then(() => {
                alert('Employee Report deleted successfully.');
                generateReports();
            })
            .catch((error) => {
                alert('Employee Report deletion failed.');
            });
    }
}

function saveReportData(event) {
    event.preventDefault();

    const isConfirmed = window.confirm("Are you sure all information are correct?");

    if (isConfirmed) {
        showLoader();

        if (action === 'Add') {
            createReport();
        }
        if (action === 'Edit') {
            updateReport()
        }
    }
}

function createReport() {

    const reportData = {
        employee: employeeInput.value,
        date_of_work: absentRadioInput.checked ? 'Absent' : dowInput.value,
        time_in: absentRadioInput.checked ? 'Absent' : convertTo12Hour(timeInInput.value),
        time_out: absentRadioInput.checked ? 'Absent' : convertTo12Hour(timeOutInput.value),
        break_time: convertTo12Hour(breakTimeInput.value),
        over_time: convertTo12Hour(overTimeInput.value),
        day_off: dayOffInput.value,
        busOperatorId: myData.key,
        companyName: myData.companyName,
        companyId: myData.companyId,
        datetimeAdded: new Date().toISOString()
    };

    const id = getCurrentDateTimeInMillis();

    const empRef = database.ref(`${DBPaths.EMPLOYEES_REPORTS}/${id}`);

    empRef.set(reportData)
        .then(() => {
            hideAddReportForm();
            generateReports();
        })
        .catch(error => {
            // An error occurred while setting data
            console.error('Error setting report:', error);
        });

    hideLoader();
}

function updateReport() {

    const reportData = {
        employee: employeeInput.value,
        absent: absentRadioInput.checked,
        date_of_work: dowInput.value,
        time_in: convertTo12Hour(timeInInput.value),
        time_out: convertTo12Hour(timeOutInput.value),
        break_time: convertTo12Hour(breakTimeInput.value),
        over_time: convertTo12Hour(overTimeInput.value),
    };

    const empRef = firebase.database().ref(`${DBPaths.EMPLOYEES_REPORTS}/${reportId}`);
    empRef.update(reportData)
        .then(() => {
            hideAddReportForm();
            generateReports();
        })
        .catch(error => {
            console.error('Error updating report:', error);
        });
}

function showAddReportForm() {
    addReportModal.style.display = 'block';
}

function hideAddReportForm() {
    addReportModal.style.display = "none";
}

window.onclick = function (event) {
    if (event.target == addReportModal) {
        hideAddReportForm();
    }
}


// PAY STATEMENTS
const payStatementModal = document.getElementById('payStatementModal');

const statementCloseFormBtn = document.querySelector(".statementCloseFormBtn");
const payStatementTable = document.querySelector(".pay-statements-table");
const addStatementBtn = document.getElementById("addStatementBtn");

const addStatementForm = document.getElementById("addStatementForm");

const employeeStatementInput = document.getElementById("employeeStatementInput");
const ticketCountInput = document.getElementById("ticketCountInput");
const earnPerDayInput = document.getElementById("earnPerDayInput");
const quotaPerDayInput = document.getElementById("quotaPerDayInput");
const weeklyWageInput = document.getElementById("weeklyWageInput");
const overtimePayInput = document.getElementById("overtimePayInput");
const doneWageRemit = document.getElementById("doneWageRemit");
const pendingWageRemit = document.getElementById("pendingWageRemit");
const dateRemitInput = document.getElementById("dateRemitInput");

const totalAbsentInput = document.getElementById("totalAbsentInput");
const totalInInput = document.getElementById("totalInInput");
const totalOutInput = document.getElementById("totalOutInput");
const sssInput = document.getElementById("sssInput");
const pagIbigInput = document.getElementById("pagIbigInput");
const savingsInput = document.getElementById("savingsInput");
const loanInput = document.getElementById("loanInput");
const statusInputActive = document.getElementById("statusInputActive");
const statusInputInactive = document.getElementById("statusInputInactive");

let empStatementArray;
let statementId
addStatementBtn.addEventListener('click', addStatement)
addStatementForm.addEventListener('submit', saveStatementData);
statementCloseFormBtn.addEventListener('click', hideAddStatementForm)

doneWageRemit.addEventListener("change", function () {
    if (doneWageRemit.checked) {
        pendingWageRemit.checked = false;
    }
    else {
        doneWageRemit.checked = true;
    }
});
pendingWageRemit.addEventListener("change", function () {
    if (pendingWageRemit.checked) {
        doneWageRemit.checked = false;
    }
    else {
        pendingWageRemit.checked = true;
    }
});
statusInputActive.addEventListener("change", function () {
    if (statusInputActive.checked) {
        statusInputInactive.checked = false;
    }
    else {
        statusInputActive.checked = true;
    }
});
statusInputInactive.addEventListener("change", function () {
    if (statusInputInactive.checked) {
        statusInputActive.checked = false;
    }
    else {
        statusInputInactive.checked = true;
    }
});

function generatePayStatements() {
    createPayStatementTableHeader();

    const empStatementsRef = database.ref(`${DBPaths.EMPLOYEES_PAY_STATEMENTS}`);
    empStatementArray = [];

    empStatementsRef.once('value',
        (snapshot) => {
            snapshot.forEach((Statement) => {

                const empStatementKey = Statement.key;
                const empStatementData = Statement.val();
                empStatementData["key"] = empStatementKey;

                if (empStatementData.companyId === myData.companyId) {
                    empStatementArray.push(empStatementData);
                    createEmpStatementTables(empStatementData);
                }

            });
        }
    )
}

function createPayStatementTableHeader() {

    payStatementTable.innerHTML = "";
    const tr = document.createElement("tr");

    // Array of column headers
    const headers = [
        "Employee",
        "Tickets",
        "Total Earn / Day",
        "Quota / Day",
        "Weekly Wage",
        "OT Pay",
        "Wage Remit",
        "Date Remit",
        "Total Absent",
        "Total In",
        "Total Out",
        "SSS",
        "Pag-ibig",
        "Savings",
        "Loan",
        "Status",
        "Actions"
    ];

    headers.forEach(headerText => {
        const th = document.createElement("th");
        th.textContent = headerText;
        tr.appendChild(th);
    });

    payStatementTable.appendChild(tr);

}

function createEmpStatementTables(empStatementData) {

    const row = document.createElement("tr");

    const employeeTd = document.createElement("td");
    employeeTd.textContent = empStatementData.employeeStatement;

    const dowTd = document.createElement("td");
    dowTd.textContent = empStatementData.ticketCount;

    const timeInTd = document.createElement("td");
    timeInTd.textContent = empStatementData.earnPerDay;

    const timeOutTd = document.createElement("td");
    timeOutTd.textContent = empStatementData.quotaPerDay;

    const breakTimeTd = document.createElement("td");
    breakTimeTd.textContent = empStatementData.weeklyWage;

    const overTimeTd = document.createElement("td");
    overTimeTd.textContent = empStatementData.overtimePay;

    const dayOfTd = document.createElement("td");
    dayOfTd.textContent = empStatementData.doneWageRemit === true ? 'Done' : 'Pending';


    const dateRemit = document.createElement("td");
    dateRemit.textContent = empStatementData.dateRemit;

    const totalAbsent = document.createElement("td");
    totalAbsent.textContent = empStatementData.totalAbsent;

    const totalIn = document.createElement("td");
    totalIn.textContent = empStatementData.totalIn;

    const totalOut = document.createElement("td");
    totalOut.textContent = empStatementData.totalOut;

    const sss = document.createElement("td");
    sss.textContent = empStatementData.sss;

    const pagIbig = document.createElement("td");
    pagIbig.textContent = empStatementData.pagIbig;

    const savings = document.createElement("td");
    savings.textContent = empStatementData.savings;

    const loan = document.createElement("td");
    loan.textContent = empStatementData.loan;

    const statusActive = document.createElement("td");
    statusActive.textContent = empStatementData.statusActive === true ? 'Active' : 'Inactive';

    const actionsTd = document.createElement("td");
    const editLink = document.createElement("a");
    editLink.href = "#";
    editLink.setAttribute("data-target", "edit-operator");
    const editIcon = document.createElement("i");
    editIcon.classList.add("fa-solid", "fa-user-pen", "edit");
    const editSpan = document.createElement("span");
    editLink.appendChild(editIcon);
    editLink.appendChild(editSpan);

    const deleteLink = document.createElement("a");
    deleteLink.href = "#";
    deleteLink.setAttribute("data-target", "edit-operator");
    const deleteIcon = document.createElement("i");
    deleteIcon.classList.add("fa-solid", "fa-eraser", "delete");
    const deleteSpan = document.createElement("span");
    deleteLink.appendChild(deleteIcon);
    deleteLink.appendChild(deleteSpan);


    actionsTd.appendChild(editLink);
    actionsTd.appendChild(deleteLink);

    row.appendChild(employeeTd);
    row.appendChild(dowTd);
    row.appendChild(timeInTd);
    row.appendChild(timeOutTd);
    row.appendChild(breakTimeTd);
    row.appendChild(overTimeTd);
    row.appendChild(dayOfTd);
    row.appendChild(dateRemit);

    row.appendChild(totalAbsent);
    row.appendChild(totalIn);
    row.appendChild(totalOut);
    row.appendChild(sss);
    row.appendChild(pagIbig);
    row.appendChild(savings);
    row.appendChild(loan);
    row.appendChild(statusActive);

    row.appendChild(actionsTd);


    payStatementTable.appendChild(row);

    editIcon.addEventListener("click", function () {
        editStatement(empStatementData)
    });
    deleteIcon.addEventListener("click", function () {
        deleteStatement(empStatementData)
    });

    hideLoader();
}

function addStatement() {
    action = 'Add';

    employeeStatementInput.value = "";
    ticketCountInput.value = "";
    earnPerDayInput.value = "";
    quotaPerDayInput.value = "";
    weeklyWageInput.value = "";
    overtimePayInput.value = "";

    doneWageRemit.checked = true;
    pendingWageRemit.checked = false;

    dateRemitInput.value = new Date().toISOString().split("T")[0];

    totalAbsentInput.value = "";
    totalInInput.value = "";
    totalOutInput.value = "";
    sssInput.value = "";
    pagIbigInput.value = "";
    savingsInput.value = "";
    loanInput.value = "";

    statusInputActive.checked = true;
    statusInputInactive.checked = false;
    showAddStatementForm();

}

function editStatement(empStatementData) {
    action = 'Edit';

    statementId = empStatementData.key
    employeeStatementInput.value = empStatementData.employeeStatement;
    ticketCountInput.value = empStatementData.ticketCount;
    earnPerDayInput.value = empStatementData.earnPerDay;
    quotaPerDayInput.value = empStatementData.quotaPerDay;
    weeklyWageInput.value = empStatementData.weeklyWage;
    overtimePayInput.value = empStatementData.overtimePay;

    doneWageRemit.checked = empStatementData.doneWageRemit;
    pendingWageRemit.checked = !empStatementData.doneWageRemit;

    dateRemitInput.value = empStatementData.dateRemit;

    totalAbsentInput.value = empStatementData.totalAbsent;
    totalInInput.value = empStatementData.totalIn;
    totalOutInput.value = empStatementData.totalOut;
    sssInput.value = empStatementData.sss;
    pagIbigInput.value = empStatementData.pagIbig;
    savingsInput.value = empStatementData.savings;
    loanInput.value = empStatementData.loan;

    statusInputActive.checked = empStatementData.statusActive;
    statusInputInactive.checked = !empStatementData.statusActive;
    showAddStatementForm();
}

function deleteStatement(empStatementData) {
    const isConfirmed = window.confirm("Confirm delete?");

    if (isConfirmed) {
        const dbRef = firebase.database().ref(`${DBPaths.EMPLOYEES_PAY_STATEMENTS}/${empStatementData.key}`);

        dbRef.remove()
            .then(() => {
                alert('Employee Statement deleted successfully.');
                generatePayStatements();
            })
            .catch((error) => {
                alert('Employee Statement deletion failed.');
            });
    }
}

function saveStatementData(event) {
    event.preventDefault();

    const isConfirmed = window.confirm("Are you sure all information are correct?");

    if (isConfirmed) {
        showLoader();

        if (action === 'Add') {
            createStatement();
        }
        if (action === 'Edit') {
            updateStatement()
        }
    }
}

function createStatement() {

    const statementData = {
        employeeStatement: employeeStatementInput.value,
        ticketCount: ticketCountInput.value,
        earnPerDay: earnPerDayInput.value,
        quotaPerDay: quotaPerDayInput.value,
        weeklyWage: weeklyWageInput.value,
        overtimePay: overtimePayInput.value,
        doneWageRemit: doneWageRemit.checked,
        dateRemit: dateRemitInput.value,

        totalAbsent: totalAbsentInput.value,
        totalIn: totalInInput.value,
        totalOut: totalOutInput.value,
        sss: sssInput.value,
        pagIbig: pagIbigInput.value,
        savings: savingsInput.value,
        loan: loanInput.value,
        statusActive: statusInputActive.checked,

        busOperatorId: myData.key,
        companyName: myData.companyName,
        companyId: myData.companyId,

        datetimeAdded: new Date().toISOString()
    };

    console.log(statementData);

    const id = getCurrentDateTimeInMillis();

    const empRef = database.ref(`${DBPaths.EMPLOYEES_PAY_STATEMENTS}/${id}`);

    empRef.set(statementData)
        .then(() => {
            hideAddStatementForm();
            generatePayStatements();
        })
        .catch(error => {
            // An error occurred while setting data
            console.error('Error setting report:', error);
        });

    hideLoader();
}

function updateStatement() {

    const statementData = {
        employeeStatement: employeeStatementInput.value,
        ticketCount: ticketCountInput.value,
        earnPerDay: earnPerDayInput.value,
        quotaPerDay: quotaPerDayInput.value,
        weeklyWage: weeklyWageInput.value,
        overtimePay: overtimePayInput.value,
        doneWageRemit: doneWageRemit.checked,
        dateRemit: dateRemitInput.value,

        totalAbsent: totalAbsentInput.value,
        totalIn: totalInInput.value,
        totalOut: totalOutInput.value,
        sss: sssInput.value,
        pagIbig: pagIbigInput.value,
        savings: savingsInput.value,
        loan: loanInput.value,
        statusActive: statusInputActive.checked
    };

    const empRef = firebase.database().ref(`${DBPaths.EMPLOYEES_PAY_STATEMENTS}/${statementId}`);
    empRef.update(statementData)
        .then(() => {
            hideAddStatementForm();
            generatePayStatements();
        })
        .catch(error => {
            console.error('Error updating report:', error);
        });
}

function showAddStatementForm() {
    payStatementModal.style.display = 'block';
}

function hideAddStatementForm() {
    payStatementModal.style.display = "none";
}

window.onclick = function (event) {
    if (event.target == payStatementModal) {
        hideAddStatementForm();
    }
}

// USER ACCOUNT
const usersTable = document.querySelector(".users-table");
const userFromCloseBtn = document.querySelector(".userFromCloseBtn");

const userModal = document.getElementById('userModal');

const userForm = document.getElementById('userForm');
const searchUserInput = document.getElementById('searchUserInput');

const userPhotoId = document.getElementById('userPhotoId');
const userFullNameInput = document.getElementById('userFullName');
const userEmailInput = document.getElementById('userEmail');
const userContactNumInput = document.getElementById('userContactNum');

let passengerArray;

userFromCloseBtn.addEventListener('click', hideUserModal);
searchUserInput.addEventListener('input', handleSearchUser);

function generateUsers() {
    createUserTableHeaders()

    const passengerRef = database.ref(`${DBPaths.PASSENGERS}`);
    passengerArray = [];

    passengerRef.once('value',
        (snapshot) => {
            snapshot.forEach((passenger) => {

                const passengerKey = passenger.key;
                const passengerData = passenger.val();
                passengerData["key"] = passengerKey;
                passengerArray.push(passengerData);

                createPassengerTables(passengerData);
            });
        }
    )
}

function handleSearchUser() {
    createUserTableHeaders();

    const searchTerm = searchUserInput.value.toLowerCase().trim();

    // Filter data based on search term
    const results = passengerArray.filter(item => item.fullName.toLowerCase().includes(searchTerm));
    // Render search results
    results.forEach(result => {

        createPassengerTables(result);
    });
}

function createUserTableHeaders() {
    usersTable.innerHTML = "";
    const tr = document.createElement("tr");

    // Array of column headers
    const headers = [
        "ID No.",
        "Fullname",
        "Email",
        "Contact No.",
        "Picture",
        "User type",
        "Actions",
    ];

    headers.forEach(headerText => {
        const th = document.createElement("th");
        th.textContent = headerText;
        tr.appendChild(th);
    });

    usersTable.appendChild(tr);
}

function createPassengerTables(passengerData) {
    const row = document.createElement("tr");

    const idTd = document.createElement("td");
    idTd.textContent = passengerData.key;

    const fullNameTd = document.createElement("td");
    fullNameTd.textContent = passengerData.fullName;

    const emailTd = document.createElement("td");
    emailTd.textContent = passengerData.email;

    const phoneNumTd = document.createElement("td");
    phoneNumTd.textContent = passengerData.phoneNum;

    const userImgTd = document.createElement("td");
    const userImg = document.createElement("img");
    userImgTd.appendChild(userImg);
    userImg.src = passengerData.imageUrl;

    const typeTd = document.createElement("td");
    typeTd.textContent = 'Passenger';

    const actionsTd = document.createElement("td");
    const editLink = document.createElement("a");
    editLink.href = "#";
    editLink.setAttribute("data-target", "edit-operator");
    const editIcon = document.createElement("i");
    editIcon.classList.add("fa-solid", "fa-user-pen", "edit");
    const editSpan = document.createElement("span");
    editLink.appendChild(editIcon);
    editLink.appendChild(editSpan);

    const deleteLink = document.createElement("a");
    deleteLink.href = "#";
    deleteLink.setAttribute("data-target", "edit-operator");
    const deleteIcon = document.createElement("i");
    deleteIcon.classList.add("fa-solid", "fa-eraser", "delete");
    const deleteSpan = document.createElement("span");
    deleteLink.appendChild(deleteIcon);
    deleteLink.appendChild(deleteSpan);


    actionsTd.appendChild(editLink);
    actionsTd.appendChild(deleteLink);

    row.appendChild(idTd);
    row.appendChild(fullNameTd);
    row.appendChild(emailTd);
    row.appendChild(phoneNumTd);
    row.appendChild(userImgTd);
    row.appendChild(typeTd);
    row.appendChild(actionsTd);

    usersTable.appendChild(row);

    editIcon.addEventListener("click", function () {
        editUser(passengerData)
    });
    deleteIcon.addEventListener("click", function () {
        deleteUser(passengerData)
    });

    hideLoader();
}

function editUser(passengerData) {
    action = 'Edit';
    userPhotoId.src = passengerData.imageUrl;
    userFullNameInput.value = passengerData.fullName;
    userEmailInput.value = passengerData.email;
    userContactNumInput.value = passengerData.phoneNum;
    showUserModal();
}

function deleteUser(passengerData) {
    const isConfirmed = window.confirm("Confirm delete?");

    if (isConfirmed) {
        const dbRef = firebase.database().ref(`${DBPaths.PASSENGERS}/${passengerData.key}`);

        dbRef.remove()
            .then(() => {
                alert('User deleted successfully.');
                generatePayStatements();
            })
            .catch((error) => {
                alert('User deletion failed.');
            });
    }
}

function showUserModal() {
    userModal.style.display = 'block';
}

function hideUserModal() {
    userModal.style.display = "none";
}

window.onclick = function (event) {
    if (event.target == userModal) {
        hideUserModal();
    }
}

//MISC FUNCTIONS
function showAddBusForm() {
    addEmpFormModal.style.display = 'block';
}

function hideAddBusForm() {
    addEmpFormModal.style.display = "none";
}

function showLoader() {
    loader.style.display = 'flex'
}

function hideLoader() {
    setTimeout(function () {
        loader.style.display = "none";
    }, 2000); // 3000 milliseconds = 3 seconds
}

