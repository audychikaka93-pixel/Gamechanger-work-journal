<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cleaning Company Journal</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 0; background-color: #f4f4f4; }
        nav { background-color: #333; padding: 10px; }
        nav button { margin: 5px; padding: 10px; background: #555; color: white; border: none; cursor: pointer; }
        nav button.active { background: #007bff; }
        section { display: none; padding: 20px; margin: 10px; background: white; border-radius: 5px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
        section.active { display: block; }
        form { margin-bottom: 20px; }
        input, select, textarea, button { margin: 5px; padding: 8px; }
        table { width: 100%; border-collapse: collapse; margin-bottom: 20px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
        .hidden { display: none; }
        canvas { max-width: 100%; height: 300px; }
        .notification { background: #fff3cd; padding: 10px; margin: 10px 0; border: 1px solid #ffeaa7; border-radius: 5px; }
    </style>
</head>
<body>
    <nav>
        <button onclick="showSection(event, 'customers')">Customers</button>
        <button onclick="showSection(event, 'workers')">Workers</button>
        <button onclick="showSection(event, 'statistics')">Statistics</button>
        <button onclick="showSection(event, 'problems')">Problems</button>
        <button onclick="showSection(event, 'inventory')">Inventory</button>
        <button onclick="showSection(event, 'calendar')">Calendar</button>
    </nav>

    <section id="customers-section" class="active">
        <h2>Customers Management</h2>
        <button onclick="showForm('addCustomer')">Add New Customer</button>
        <button onclick="showForm('searchCustomer')">Search Customers</button>
        <div id="customerForm" class="hidden">
            <form id="customerFormEl">
                <input type="hidden" id="customerId">
                <label>Name: <input type="text" id="custName" required></label>
                <label>Location: <input type="text" id="custLocation" required></label>
                <label>Frequency (days): <input type="number" id="frequency" required min="1"></label>
                <label>Number of Kitchens: <input type="number" id="numKitchens" min="1" onchange="generateKitchenInputs()"></label>
                <div id="kitchensContainer"></div>
                <button type="submit">Save Customer</button>
                <button type="button" onclick="hideCustomerForm()">Cancel</button>
            </form>
        </div>
        <div id="customerList"></div>
        <div id="customerDetails" class="hidden"></div>
        <div id="reminders" class="notification"></div>
    </section>

    <section id="workers-section">
        <h2>Workers Management</h2>
        <button onclick="showForm('addWorker')">Add New Worker</button>
        <button onclick="showForm('searchWorker')">Search Workers</button>
        <div id="workerForm" class="hidden">
            <form id="workerFormEl">
                <input type="hidden" id="workerId">
                <label>Name: <input type="text" id="workerName" required></label>
                <label>Surname: <input type="text" id="workerSurname" required></label>
                <label>Contact (WhatsApp): <input type="tel" id="workerContact" required></label>
                <label>Legal: <select id="workerLegal"><option value="true">Yes</option><option value="false">No</option></select></label>
                <label>DOB: <input type="date" id="workerDob"></label>
                <button type="submit">Save Worker</button>
                <button type="button" onclick="hideWorkerForm()">Cancel</button>
            </form>
        </div>
        <h3>Daily Attendance</h3>
        <form id="attendanceForm">
            <label>Date: <input type="date" id="attDate" required></label>
            <label>Worker: <select id="attWorker" required onchange="loadCustomerForAttendance()"></select></label>
            <label>Customer: <select id="attCustomer" required onchange="loadKitchensForAttendance()"></select></label>
            <label>Kitchen: <select id="attKitchen" required onchange="loadKanalsForAttendance()"></select></label>
            <label>Kanal: <select id="attKanal" required></select></label>
            <label>Meters Cleaned: <input type="number" id="attMeters" min="0" required></label>
            <label>Method: <select id="attMethod"><option value="spatula">Spatula</option><option value="tel bez">Tel Bez</option></select></label>
            <button type="submit">Record Attendance</button>
        </form>
        <div id="workerList"></div>
        <div id="workerDetails" class="hidden"></div>
    </section>

    <section id="statistics-section">
        <h2>Statistics</h2>
        <button onclick="showStats('customer')">Customer Stats</button>
        <button onclick="showStats('worker')">Worker Stats</button>
        <select id="statsPeriod"><option value="day">Day</option><option value="week">Week</option><option value="month">Month</option><option value="year">Year</option><option value="all">All</option></select>
        <label>From: <input type="date" id="statsFrom"></label>
        <label>To: <input type="date" id="statsTo"></label>
        <select id="statsCustomer"></select>
        <select id="statsWorker"></select>
        <div id="statsCharts"></div>
    </section>

    <section id="problems-section">
        <h2>Problems</h2>
        <form id="problemForm">
            <label>Customer: <select id="probCustomer" required onchange="loadKitchensForProblem()"></select></label>
            <label>Kitchen: <select id="probKitchen" required></select></label>
            <label>Description: <textarea id="probDesc" required></textarea></label>
            <label>Person Noticed: <input type="text" id="probPerson" required></label>
            <button type="submit">Record Problem</button>
        </form>
        <table id="problemsTable">
            <thead><tr><th>Customer</th><th>Kitchen</th><th>Desc</th><th>Date</th><th>Person</th><th>Solved?</th><th>Solved By</th><th>Time Taken</th><th>Actions</th></tr></thead>
            <tbody></tbody>
        </table>
    </section>

    <section id="inventory-section">
        <h2>Inventory</h2>
        <form id="itemForm">
            <label>Item Name: <input type="text" id="itemName" required></label>
            <label>Quantity: <input type="number" id="itemQty" min="0" required></label>
            <button type="submit">Add/Update Item</button>
        </form>
        <h3>Take Out Item</h3>
        <form id="takeOutForm">
            <label>Item: <select id="takeOutItem" required></select></label>
            <label>Quantity: <input type="number" id="takeOutQty" min="1" required></label>
            <label>To Worker: <select id="takeOutWorker" required></select></label>
            <button type="submit">Take Out</button>
        </form>
        <table id="inventoryTable">
            <thead><tr><th>Item</th><th>Qty</th></tr></thead>
            <tbody></tbody>
        </table>
        <h3>Worker Inventory</h3>
        <select id="invWorker" onchange="showWorkerInventory()"></select>
        <table id="workerInvTable" class="hidden">
            <thead><tr><th>Date</th><th>Item</th><th>Qty</th></tr></thead>
            <tbody></tbody>
        </table>
    </section>

    <section id="calendar-section">
        <h2>Calendar</h2>
        <input type="date" id="calDate" onchange="showCalendarDay()">
        <div id="calendarDay"></div>
    </section>

    <script>
        // Data Storage using localStorage
        function loadData(key) {
            return JSON.parse(localStorage.getItem(key)) || [];
        }
        function saveData(key, data) {
            localStorage.setItem(key, JSON.stringify(data));
        }

        // Customers
        let customers = loadData('customers');
        let cleaningHistory = loadData('cleaningHistory');
        let workers = loadData('workers');
        let problems = loadData('problems');
        let attendance = loadData('attendance');

        function showSection(event, section) {
            document.querySelectorAll('section').forEach(s => s.classList.remove('active'));
            document.getElementById(section + '-section').classList.add('active');
            document.querySelectorAll('nav button').forEach(b => b.classList.remove('active'));
            event.target.classList.add('active');
            if (section === 'customers') renderCustomers();
            if (section === 'workers') renderWorkers();
            if (section === 'problems') renderProblems();
            if (section === 'inventory') renderInventory();
            if (section === 'calendar') showCalendarDay();
            if (section === 'statistics') showStats('customer');
        }

        function showForm(type) {
            if (type === 'addCustomer') {
                document.getElementById('customerFormEl').reset();
                document.getElementById('customerId').value = '';
                generateKitchenInputs();
                document.getElementById('customerForm').classList.remove('hidden');
            } else if (type === 'searchCustomer') {
                renderCustomers();
            } else if (type === 'addWorker') {
                document.getElementById('workerFormEl').reset();
                document.getElementById('workerId').value = '';
                document.getElementById('workerForm').classList.remove('hidden');
            } else if (type === 'searchWorker') {
                renderWorkers();
            }
        }

        function hideCustomerForm() {
            document.getElementById('customerForm').classList.add('hidden');
        }

        function hideWorkerForm() {
            document.getElementById('workerForm').classList.add('hidden');
        }

        function generateKitchenInputs() {
            const num = parseInt(document.getElementById('numKitchens').value) || 0;
            const container = document.getElementById('kitchensContainer');
            container.innerHTML = '';
            for (let i = 0; i < num; i++) {
                const div = document.createElement('div');
                div.innerHTML = `
                    <h4>Kitchen ${i+1}</h4>
                    <label>Name: <input type="text" name="kitchenName${i}" required></label>
                    <label>Davulumbas: <input type="number" name="davulumbas${i}" min="0"></label>
                    <label>Kanals (num): <input type="number" name="kanalsNum${i}" min="0" onchange="generateKanalInputs(${i})"></label>
                    <div class="kanalContainer${i}"></div>
                    <label>Motors (num): <input type="number" name="motorsNum${i}" min="0" onchange="generateMotorInputs(${i})"></label>
                    <div class="motorContainer${i}"></div>
                `;
                container.appendChild(div);
            }
        }

        function generateKanalInputs(kitchenIndex) {
            const num = parseInt(document.querySelector(`input[name="kanalsNum${kitchenIndex}"]`).value) || 0;
            const container = document.querySelector(`.kanalContainer${kitchenIndex}`);
            container.innerHTML = '';
            for (let j = 0; j < num; j++) {
                const div = document.createElement('div');
                div.innerHTML = `
                    <label>Kanal ${j+1} Name: <input type="text" name="kanalName${kitchenIndex}_${j}"></label>
                    <label>Length (m): <input type="number" name="kanalLength${kitchenIndex}_${j}" min="0" required></label>
                `;
                container.appendChild(div);
            }
        }

        function generateMotorInputs(kitchenIndex) {
            const num = parseInt(document.querySelector(`input[name="motorsNum${kitchenIndex}"]`).value) || 0;
            const container = document.querySelector(`.motorContainer${kitchenIndex}`);
            container.innerHTML = '';
            for (let j = 0; j < num; j++) {
                const div = document.createElement('div');
                div.innerHTML = `
                    <label>Motor ${j+1} Type: <select name="motorType${kitchenIndex}_${j}" onchange="toggleFilters(${kitchenIndex}, ${j})">
                        <option value="cube">Cube</option>
                        <option value="solyangoz">Solyangoz</option>
                        <option value="kabinli">Kabinli</option>
                    </select></label>
                    <div class="filters${kitchenIndex}_${j} hidden">
                        <label>Electrostatic: <input type="number" name="electro${kitchenIndex}_${j}" min="0"></label>
                        <label>Carbon: <input type="number" name="carbon${kitchenIndex}_${j}" min="0"></label>
                        <label>Kaset: <input type="number" name="kaset${kitchenIndex}_${j}" min="0"></label>
                    </div>
                `;
                container.appendChild(div);
            }
        }

        function toggleFilters(kitchenIndex, j) {
            const type = document.querySelector(`select[name="motorType${kitchenIndex}_${j}"]`).value;
            const filters = document.querySelector(`.filters${kitchenIndex}_${j}`);
            filters.classList.toggle('hidden', type !== 'kabinli');
        }

        document.getElementById('customerFormEl').onsubmit = function(e) {
            e.preventDefault();
            const id = document.getElementById('customerId').value || Date.now().toString();
            const name = document.getElementById('custName').value;
            const location = document.getElementById('custLocation').value;
            const frequency = parseInt(document.getElementById('frequency').value);
            const numKitchens = parseInt(document.getElementById('numKitchens').value);
            const kitchens = [];
            for (let i = 0; i < numKitchens; i++) {
                const kitchenName = document.querySelector(`input[name="kitchenName${i}"]`).value;
                const dav = parseInt(document.querySelector(`input[name="davulumbas${i}"]`).value) || 0;
                const kanalsNum = parseInt(document.querySelector(`input[name="kanalsNum${i}"]`).value) || 0;
                const kanals = [];
                for (let j = 0; j < kanalsNum; j++) {
                    kanals.push({
                        name: document.querySelector(`input[name="kanalName${i}_${j}"]`).value,
                        length: parseFloat(document.querySelector(`input[name="kanalLength${i}_${j}"]`).value) || 0
                    });
                }
                const motorsNum = parseInt(document.querySelector(`input[name="motorsNum${i}"]`).value) || 0;
                const motors = [];
                for (let j = 0; j < motorsNum; j++) {
                    const type = document.querySelector(`select[name="motorType${i}_${j}"]`).value;
                    const filters = type === 'kabinli' ? {
                        electrostatic: parseInt(document.querySelector(`input[name="electro${i}_${j}"]`).value) || 0,
                        carbon: parseInt(document.querySelector(`input[name="carbon${i}_${j}"]`).value) || 0,
                        kaset: parseInt(document.querySelector(`input[name="kaset${i}_${j}"]`).value) || 0
                    } : null;
                    motors.push({ type, filters });
                }
                kitchens.push({
                    name: kitchenName,
                    davulumbas: dav,
                    kanals,
                    motors
                });
            }
            const customer = {
                id,
                name,
                location,
                frequency,
                kitchens,
                nextClean: new Date(Date.now() + (frequency * 86400000)).toISOString().split('T')[0]
            };
            const idx = customers.findIndex(c => c.id === id);
            if (idx !== -1) {
                customers[idx] = customer;
            } else {
                customers.push(customer);
            }
            saveData('customers', customers);
            hideCustomerForm();
            renderCustomers();
        };

        function renderCustomers(sortBy = 'name') {
            const list = document.getElementById('customerList');
            list.innerHTML = '<h3>Customers</h3><input type="text" id="custSearch" placeholder="Search..." oninput="filterCustomers()"><select id="custSort" onchange="renderCustomers(this.value)"><option value="name">Alphabet</option><option value="due">Due Soon</option><option value="kitchens">Most Kitchens</option></select><table><thead><tr><th>Name</th><th>Location</th><th>Next Clean</th><th>Kitchens</th><th>Actions</th></tr></thead><tbody>' +
                customers.sort((a, b) => {
                    if (sortBy === 'name') return a.name.localeCompare(b.name);
                    if (sortBy === 'due') return new Date(a.nextClean) - new Date(b.nextClean);
                    if (sortBy === 'kitchens') return b.kitchens.length - a.kitchens.length;
                    return 0;
                }).map(c => `<tr><td onclick="showCustomerDetails('${c.id}')">${c.name}</td><td>${c.location}</td><td>${c.nextClean}</td><td>${c.kitchens.length}</td><td><button onclick="editCustomer('${c.id}')">Edit</button> <button onclick="deleteCustomer('${c.id}')">Delete</button></td></tr>`).join('') +
                '</tbody></table>';
            checkReminders();
        }

        function filterCustomers() {
            const term = document.getElementById('custSearch').value.toLowerCase();
            const rows = document.querySelectorAll('#customerList tbody tr');
            rows.forEach(row => {
                row.style.display = row.textContent.toLowerCase().includes(term) ? '' : 'none';
            });
        }

        function showCustomerDetails(id) {
            const cust = customers.find(c => c.id === id);
            if (!cust) return;
            const details = document.getElementById('customerDetails');
            details.innerHTML = `
                <h3>${cust.name}</h3>
                <p>Location: ${cust.location}</p>
                <p>Frequency: ${cust.frequency} days</p>
                <p>Kitchens: ${cust.kitchens.map(k => `<div><strong>${k.name}</strong>: ${k.davulumbas} Davulumbas, ${k.kanals.length} Kanals (${k.kanals.reduce((sum, ka) => sum + ka.length, 0)}m), ${k.motors.length} Motors</div>`).join('')}</p>
                <h4>Cleaning History</h4>
                <div>${cleaningHistory.filter(h => h.customerId === id).map(h => `<div>Date: ${h.date} - Kitchens cleaned: ${h.cleanedKitchens.map(kc => cust.kitchens[kc.kitchenIdx].name).join(', ')} - Workers: ${h.cleanedKitchens.flatMap(kc => kc.cleanedKanals.map(ck => ck.worker)).join(', ')} - Problems: ${h.problems ? h.problems.length : 0}</div>`).join('')}</div>
                <p>Next Clean: ${cust.nextClean}</p>
                <button onclick="recordCleaning('${id}')">Record Cleaning</button>
                <button onclick="hideCustomerDetails()">Back</button>
            `;
            details.classList.remove('hidden');
            document.getElementById('customerList').classList.add('hidden');
        }

        function hideCustomerDetails() {
            document.getElementById('customerDetails').classList.add('hidden');
            document.getElementById('customerList').classList.remove('hidden');
        }

        function editCustomer(id) {
            const cust = customers.find(c => c.id === id);
            if (!cust) return;
            document.getElementById('customerId').value = id;
            document.getElementById('custName').value = cust.name;
            document.getElementById('custLocation').value = cust.location;
            document.getElementById('frequency').value = cust.frequency;
            document.getElementById('numKitchens').value = cust.kitchens.length;
            generateKitchenInputs();
            cust.kitchens.forEach((k, i) => {
                document.querySelector(`input[name="kitchenName${i}"]`).value = k.name;
                document.querySelector(`input[name="davulumbas${i}"]`).value = k.davulumbas;
                document.querySelector(`input[name="kanalsNum${i}"]`).value = k.kanals.length;
                generateKanalInputs(i);
                k.kanals.forEach((kan, j) => {
                    document.querySelector(`input[name="kanalName${i}_${j}"]`).value = kan.name;
                    document.querySelector(`input[name="kanalLength${i}_${j}"]`).value = kan.length;
                });
                document.querySelector(`input[name="motorsNum${i}"]`).value = k.motors.length;
                generateMotorInputs(i);
                k.motors.forEach((m, j) => {
                    document.querySelector(`select[name="motorType${i}_${j}"]`).value = m.type;
                    if (m.type === 'kabinli' && m.filters) {
                        toggleFilters(i, j);
                        document.querySelector(`input[name="electro${i}_${j}"]`).value = m.filters.electrostatic;
                        document.querySelector(`input[name="carbon${i}_${j}"]`).value = m.filters.carbon;
                        document.querySelector(`input[name="kaset${i}_${j}"]`).value = m.filters.kaset;
                    }
                });
            });
            document.getElementById('customerForm').classList.remove('hidden');
        }

        function deleteCustomer(id) {
            if (confirm('Delete customer?')) {
                customers = customers.filter(c => c.id !== id);
                cleaningHistory = cleaningHistory.filter(h => h.customerId !== id);
                problems = problems.filter(p => p.customerId !== id);
                attendance = attendance.filter(a => a.customerId !== id);
                saveData('customers', customers);
                saveData('cleaningHistory', cleaningHistory);
                saveData('problems', problems);
                saveData('attendance', attendance);
                renderCustomers();
            }
        }

        function recordCleaning(customerId) {
            const date = new Date().toISOString().split('T')[0];
            const cust = customers.find(c => c.id === customerId);
            const form = document.createElement('form');
            form.id = 'cleaningForm';
            form.innerHTML = `
                <h4>Record Cleaning for ${cust.name} - ${date}</h4>
                ${cust.kitchens.map((k, i) => `<label><input type="checkbox" name="kitchen${i}"> ${k.name}</label><br>`).join('')}
                <div id="cleanDetails"></div>
                <label>Problems: <textarea id="newProblems"></textarea></label>
                <button type="submit">Save</button>
            `;
            document.getElementById('customerDetails').appendChild(form);
            cust.kitchens.forEach((k, i) => {
                const cb = form.querySelector(`input[name="kitchen${i}"]`);
                cb.onchange = () => {
                    const details = document.getElementById('cleanDetails');
                    if (cb.checked) {
                        const subDiv = document.createElement('div');
                        subDiv.id = `sub${i}`;
                        subDiv.innerHTML = `<h5>${k.name}</h5>`;
                        k.kanals.forEach((kan, j) => {
                            subDiv.innerHTML += `
                                <label>Cleaned Kanal ${kan.name || (j+1)} (${kan.length}m): <input type="checkbox" name="kanal${i}_${j}" onchange="toggleKanalDetails(${i}, ${j}, this.checked)"></label>
                                <div class="kanalDetails${i}_${j} hidden">
                                    <label>Worker: <select name="worker${i}_${j}">${workers.map(w => `<option value="${w.id}">${w.name} ${w.surname}</option>`).join('')}</select></label>
                                    <label>Meters: <input type="number" name="meters${i}_${j}" value="${kan.length}" min="0"></label>
                                    <label>Method: <select name="method${i}_${j}"><option value="spatula">Spatula</option><option value="tel bez">Tel Bez</option></select></label>
                                </div>
                            `;
                        });
                        details.appendChild(subDiv);
                    } else {
                        document.getElementById(`sub${i}`)?.remove();
                    }
                };
            });
            form.onsubmit = function(e) {
                e.preventDefault();
                const cleanedKitchens = [];
                cust.kitchens.forEach((k, i) => {
                    const cb = form.querySelector(`input[name="kitchen${i}"]:checked`);
                    if (cb) {
                        const cleanedKanals = [];
                        k.kanals.forEach((kan, j) => {
                            if (form.querySelector(`input[name="kanal${i}_${j}"]:checked`)) {
                                const workerId = form.querySelector(`select[name="worker${i}_${j}"]`).value;
                                const worker = workers.find(w => w.id === workerId);
                                const meters = parseFloat(form.querySelector(`input[name="meters${i}_${j}"]`).value) || 0;
                                const method = form.querySelector(`select[name="method${i}_${j}"]`).value;
                                cleanedKanals.push({kanalIdx: j, worker: worker.name + ' ' + worker.surname, meters, method});
                                // Record attendance
                                const att = {
                                    id: Date.now().toString(),
                                    date,
                                    workerId,
                                    customerId,
                                    kitchenIdx: i,
                                    kanalIdx: j,
                                    meters,
                                    method
                                };
                                attendance.push(att);
                                saveData('attendance', attendance);
                            }
                        });
                        cleanedKitchens.push({kitchenIdx: i, cleanedKanals});
                    }
                });
                const problemsText = document.getElementById('newProblems').value.trim();
                const newProblems = problemsText ? [{desc: problemsText, person: 'User', date}] : [];
                if (problemsText) {
                    problems.push({id: Date.now().toString(), customerId, kitchenIdx: -1, desc: problemsText, date, person: 'User', solved: false, solvedBy: '', solvedDate: '', timeTaken: 0});
                    saveData('problems', problems);
                }
                const historyEntry = { customerId, date, cleanedKitchens, problems: newProblems };
                cleaningHistory.push(historyEntry);
                saveData('cleaningHistory', cleaningHistory);
                // Update nextClean
                const cleanDate = new Date(date);
                cust.nextClean = new Date(cleanDate.getTime() + (cust.frequency * 86400000)).toISOString().split('T')[0];
                saveData('customers', customers);
                form.remove();
                showCustomerDetails(customerId);
            };
        }

        function toggleKanalDetails(i, j, checked) {
            const details = document.querySelector(`.kanalDetails${i}_${j}`);
            details.classList.toggle('hidden', !checked);
        }

        function checkReminders() {
            const now = new Date();
            const reminders = customers.filter(c => {
                const due = new Date(c.nextClean);
                return (due - now) / 86400000 <= 14;
            }).map(c => `${c.name} due on ${c.nextClean}`);
            document.getElementById('reminders').innerHTML = reminders.length ? `<strong>Reminders:</strong> ${reminders.join(', ')}` : '';
        }

        // Workers
        document.getElementById('workerFormEl').onsubmit = function(e) {
            e.preventDefault();
            const id = document.getElementById('workerId').value || Date.now().toString();
            const worker = {
                id,
                name: document.getElementById('workerName').value,
                surname: document.getElementById('workerSurname').value,
                contact: document.getElementById('workerContact').value,
                legal: document.getElementById('workerLegal').value === 'true',
                dob: document.getElementById('workerDob').value,
                attendance: attendance.filter(a => a.workerId === id)
            };
            const idx = workers.findIndex(w => w.id === id);
            if (idx !== -1) {
                workers[idx] = worker;
            } else {
                workers.push(worker);
            }
            saveData('workers', workers);
            hideWorkerForm();
            renderWorkers();
        };

        function renderWorkers(sortBy = 'name') {
            const list = document.getElementById('workerList');
            list.innerHTML = '<h3>Workers</h3><input type="text" id="workerSearch" placeholder="Search..." oninput="filterWorkers()"><select id="workerSort" onchange="renderWorkers(this.value)"><option value="name">Alphabet</option><option value="meters">Most Meters</option><option value="kitchens">Most Kitchens</option><option value="legal">Legal</option><option value="attendance">Most Days</option></select><table><thead><tr><th>Name</th><th>Contact</th><th>Legal</th><th>Actions</th></tr></thead><tbody>' +
                workers.sort((a, b) => {
                    if (sortBy === 'name') return (a.name + ' ' + a.surname).localeCompare(b.name + ' ' + b.surname);
                    if (sortBy === 'meters') return b.attendance.reduce((sum, att) => sum + (att.meters || 0), 0) - a.attendance.reduce((sum, att) => sum + (att.meters || 0), 0);
                    if (sortBy === 'kitchens') return b.attendance.length - a.attendance.length; // Approximate
                    if (sortBy === 'legal') return b.legal - a.legal;
                    if (sortBy === 'attendance') return b.attendance.length - a.attendance.length;
                    return 0;
                }).map(w => `<tr><td onclick="showWorkerDetails('${w.id}')">${w.name} ${w.surname}</td><td><a href="https://wa.me/${w.contact.replace(/\D/g,'')}">Call on WhatsApp</a></td><td>${w.legal ? 'Yes' : 'No'}</td><td><button onclick="editWorker('${w.id}')">Edit</button> <button onclick="deleteWorker('${w.id}')">Delete</button></td></tr>`).join('') +
                '</tbody></table>';
            populateSelect('attWorker', workers.map(w => ({value: w.id, text: w.name + ' ' + w.surname})));
            populateSelect('takeOutWorker', workers.map(w => ({value: w.id, text: w.name + ' ' + w.surname})));
            populateSelect('invWorker', workers.map(w => ({value: w.id, text: w.name + ' ' + w.surname})));
        }

        function populateSelect(id, options) {
            const sel = document.getElementById(id);
            if (sel) sel.innerHTML = options.map(o => `<option value="${o.value}">${o.text}</option>`).join('');
        }

        function filterWorkers() {
            const term = document.getElementById('workerSearch').value.toLowerCase();
            const rows = document.querySelectorAll('#workerList tbody tr');
            rows.forEach(row => {
                row.style.display = row.textContent.toLowerCase().includes(term) ? '' : 'none';
            });
        }

        function showWorkerDetails(id) {
            const worker = workers.find(w => w.id === id);
            if (!worker) return;
            const att = worker.attendance;
            document.getElementById('workerDetails').innerHTML = `
                <h3>${worker.name} ${worker.surname}</h3>
                <p>Contact: <a href="https://wa.me/${worker.contact.replace(/\D/g,'')}">Call on WhatsApp</a></p>
                <p>Legal: ${worker.legal ? 'Yes' : 'No'}</p>
                <p>DOB: ${worker.dob}</p>
                <p>Attendance: ${att.length} days</p>
                <div>${att.map(a => `<div>${a.date}: Customer ID ${a.customerId} - Kitchen ${a.kitchenIdx} - Kanal ${a.kanalIdx} - ${a.meters}m (${a.method})</div>`).join('')}</div>
                <button onclick="hideWorkerDetails()">Back</button>
            `;
            document.getElementById('workerDetails').classList.remove('hidden');
            document.getElementById('workerList').classList.add('hidden');
        }

        function hideWorkerDetails() {
            document.getElementById('workerDetails').classList.add('hidden');
            document.getElementById('workerList').classList.remove('hidden');
        }

        function editWorker(id) {
            const w = workers.find(w => w.id === id);
            if (!w) return;
            document.getElementById('workerId').value = id;
            document.getElementById('workerName').value = w.name;
            document.getElementById('workerSurname').value = w.surname;
            document.getElementById('workerContact').value = w.contact;
            document.getElementById('workerLegal').value = w.legal ? 'true' : 'false';
            document.getElementById('workerDob').value = w.dob;
            document.getElementById('workerForm').classList.remove('hidden');
        }

        function deleteWorker(id) {
            if (confirm('Delete worker?')) {
                workers = workers.filter(w => w.id !== id);
                attendance = attendance.filter(a => a.workerId !== id);
                saveData('workers', workers);
                saveData('attendance', attendance);
                renderWorkers();
            }
        }

        // Attendance
        function loadCustomerForAttendance() {
            populateSelect('attCustomer', customers.map(c => ({value: c.id, text: c.name})));
        }

        function loadKitchensForAttendance() {
            const custId = document.getElementById('attCustomer').value;
            const cust = customers.find(c => c.id === custId);
            populateSelect('attKitchen', cust ? cust.kitchens.map((k, i) => ({value: i, text: k.name})) : []);
        }

        function loadKanalsForAttendance() {
            const custId = document.getElementById('attCustomer').value;
            const kitchenIdx = document.getElementById('attKitchen').value;
            const cust = customers.find(c => c.id === custId);
            const kitchen = cust ? cust.kitchens[kitchenIdx] : null;
            populateSelect('attKanal', kitchen ? kitchen.kanals.map((kan, j) => ({value: j, text: (kan.name || (j+1)) + ' (' + kan.length + 'm)'})) : []);
        }

        document.getElementById('attendanceForm').onsubmit = function(e) {
            e.preventDefault();
            const att = {
                id: Date.now().toString(),
                date: document.getElementById('attDate').value,
                workerId: document.getElementById('attWorker').value,
                customerId: document.getElementById('attCustomer').value,
                kitchenIdx: parseInt(document.getElementById('attKitchen').value),
                kanalIdx: parseInt(document.getElementById('attKanal').value),
                meters: parseFloat(document.getElementById('attMeters').value),
                method: document.getElementById('attMethod').value
            };
            attendance.push(att);
            saveData('attendance', attendance);
            // Sync to history
            let histEntry = cleaningHistory.find(h => h.customerId === att.customerId && h.date === att.date);
            if (!histEntry) {
                histEntry = { customerId: att.customerId, date: att.date, cleanedKitchens: [], problems: [] };
                cleaningHistory.push(histEntry);
                saveData('cleaningHistory', cleaningHistory);
            }
            let kitchen = histEntry.cleanedKitchens.find(kc => kc.kitchenIdx === att.kitchenIdx);
            if (!kitchen) {
                kitchen = { kitchenIdx: att.kitchenIdx, cleanedKanals: [] };
                histEntry.cleanedKitchens.push(kitchen);
                saveData('cleaningHistory', cleaningHistory);
            }
            const worker = workers.find(w => w.id === att.workerId);
            kitchen.cleanedKanals.push({ kanalIdx: att.kanalIdx, worker: worker.name + ' ' + worker.surname, meters: att.meters, method: att.method });
            saveData('cleaningHistory', cleaningHistory);
            // Update worker attendance
            worker.attendance.push(att);
            saveData('workers', workers);
            this.reset();
            renderWorkers();
        };

        // Statistics
        function showStats(type) {
            let from = document.getElementById('statsFrom').value;
            let to = document.getElementById('statsTo').value;
            const period = document.getElementById('statsPeriod').value;
            const now = new Date();
            if (!to) to = now.toISOString().split('T')[0];
            if (!from) {
                if (period === 'day') from = to;
                else if (period === 'week') from = new Date(now.setDate(now.getDate() - 7)).toISOString().split('T')[0];
                else if (period === 'month') from = new Date(now.setMonth(now.getMonth() - 1)).toISOString().split('T')[0];
                else if (period === 'year') from = new Date(now.setFullYear(now.getFullYear() - 1)).toISOString().split('T')[0];
                else from = '2000-01-01';
            }
            const chartsDiv = document.getElementById('statsCharts');
            chartsDiv.innerHTML = '';
            const fromDate = new Date(from);
            const toDate = new Date(to);
            if (type === 'customer') {
                const custId = document.getElementById('statsCustomer').value;
                const filteredCustomers = custId ? customers.filter(c => c.id === custId) : customers;
                const totalMeters = filteredCustomers.reduce((sum, c) => sum + c.kitchens.reduce((s, k) => s + k.kanals.reduce((ss, ka) => ss + ka.length, 0), 0), 0);
                const filteredHistory = cleaningHistory.filter(h => (!custId || h.customerId === custId) && new Date(h.date) >= fromDate && new Date(h.date) <= toDate);
                const cleanedMeters = filteredHistory.reduce((sum, h) => sum + h.cleanedKitchens.reduce((s, kc) => s + kc.cleanedKanals.reduce((ss, ck) => ss + ck.meters, 0), 0), 0);
                const spatulaMeters = filteredHistory.reduce((sum, h) => sum + h.cleanedKitchens.reduce((s, kc) => s + kc.cleanedKanals.reduce((ss, ck) => ck.method === 'spatula' ? ss + ck.meters : ss, 0), 0), 0);
                const telBezMeters = cleanedMeters - spatulaMeters;
                const canvasMeters = document.createElement('canvas');
                chartsDiv.appendChild(canvasMeters);
                new Chart(canvasMeters, {
                    type: 'bar',
                    data: { labels: ['Total Meters', 'Cleaned Meters'], datasets: [{ label: 'Meters', data: [totalMeters, cleanedMeters] }] },
                    options: { scales: { y: { beginAtZero: true } } }
                });
                const canvasMethods = document.createElement('canvas');
                chartsDiv.appendChild(canvasMethods);
                new Chart(canvasMethods, {
                    type: 'pie',
                    data: { labels: ['Spatula', 'Tel Bez'], datasets: [{ data: [spatulaMeters, telBezMeters] }] }
                });
            } else if (type === 'worker') {
                const workerId = document.getElementById('statsWorker').value;
                const filteredWorkers = workerId ? workers.filter(w => w.id === workerId) : workers;
                const metersData = filteredWorkers.map(w => {
                    const wAtt = w.attendance.filter(a => new Date(a.date) >= fromDate && new Date(a.date) <= toDate);
                    return { name: w.name + ' ' + w.surname, meters: wAtt.reduce((sum, a) => sum + a.meters, 0), days: wAtt.length };
                });
                const canvasMeters = document.createElement('canvas');
                chartsDiv.appendChild(canvasMeters);
                new Chart(canvasMeters, {
                    type: 'bar',
                    data: { labels: metersData.map(m => m.name), datasets: [{ label: 'Meters Cleaned', data: metersData.map(m => m.meters) }] },
                    options: { scales: { y: { beginAtZero: true } } }
                });
                const canvasDays = document.createElement('canvas');
                chartsDiv.appendChild(canvasDays);
                new Chart(canvasDays, {
                    type: 'bar',
                    data: { labels: metersData.map(m => m.name), datasets: [{ label: 'Days Worked', data: metersData.map(m => m.days) }] },
                    options: { scales: { y: { beginAtZero: true } } }
                });
            }
        }

        // Problems
        document.getElementById('problemForm').onsubmit = function(e) {
            e.preventDefault();
            const prob = {
                id: Date.now().toString(),
                customerId: document.getElementById('probCustomer').value,
                kitchenIdx: parseInt(document.getElementById('probKitchen').value),
                desc: document.getElementById('probDesc').value,
                date: new Date().toISOString().split('T')[0],
                person: document.getElementById('probPerson').value,
                solved: false,
                solvedBy: '',
                solvedDate: '',
                timeTaken: 0
            };
            problems.push(prob);
            saveData('problems', problems);
            this.reset();
            renderProblems();
        };

        function loadKitchensForProblem() {
            const custId = document.getElementById('probCustomer').value;
            const cust = customers.find(c => c.id === custId);
            populateSelect('probKitchen', cust ? cust.kitchens.map((k, i) => ({value: i, text: k.name})) : []);
        }

        function renderProblems() {
            const tbody = document.querySelector('#problemsTable tbody');
            tbody.innerHTML = problems.map(p => {
                const cust = customers.find(c => c.id === p.customerId);
                const kitchen = cust && p.kitchenIdx !== -1 ? cust.kitchens[p.kitchenIdx] : {name: 'General'};
                return `<tr>
                    <td>${cust ? cust.name : 'Unknown'}</td>
                    <td>${kitchen.name}</td>
                    <td>${p.desc}</td>
                    <td>${p.date}</td>
                    <td>${p.person}</td>
                    <td>${p.solved ? 'Yes' : 'No'}</td>
                    <td>${p.solvedBy}</td>
                    <td>${p.timeTaken} days</td>
                    <td><button onclick="markSolved('${p.id}')">Solve</button></td>
                </tr>`;
            }).join('');
            const unsolvedDue = problems.filter(p => !p.solved && customers.find(c => c.id === p.customerId && (new Date(c.nextClean) - new Date()) / 86400000 <= 14));
            const notif = document.querySelector('#problems-section .notification');
            if (notif) notif.remove();
            if (unsolvedDue.length) {
                const div = document.createElement('div');
                div.className = 'notification';
                div.innerHTML = `Unsolved Problems Due Soon: ${unsolvedDue.length}`;
                document.getElementById('problems-section').insertAdjacentElement('afterbegin', div);
            }
        }

        function markSolved(id) {
            const p = problems.find(pr => pr.id === id);
            if (p && !p.solved) {
                p.solved = true;
                p.solvedDate = new Date().toISOString().split('T')[0];
                p.solvedBy = prompt('Solved by whom?') || 'Unknown';
                p.timeTaken = Math.round((new Date(p.solvedDate) - new Date(p.date)) / 86400000);
                saveData('problems', problems);
                renderProblems();
            }
        }

        // Inventory
        let inventory = loadData('inventory');
        let transactions = loadData('transactions');

        document.getElementById('itemForm').onsubmit = function(e) {
            e.preventDefault();
            const name = document.getElementById('itemName').value;
            const qty = parseInt(document.getElementById('itemQty').value);
            let item = inventory.find(i => i.name === name);
            if (item) {
                item.qty += qty;
            } else {
                inventory.push({name, qty});
            }
            saveData('inventory', inventory);
            this.reset();
            renderInventory();
        };

        document.getElementById('takeOutForm').onsubmit = function(e) {
            e.preventDefault();
            const name = document.getElementById('takeOutItem').value;
            const qty = parseInt(document.getElementById('takeOutQty').value);
            const workerId = document.getElementById('takeOutWorker').value;
            const item = inventory.find(i => i.name === name);
            if (item && item.qty >= qty) {
                item.qty -= qty;
                transactions.push({date: new Date().toISOString().split('T')[0], item: name, qty: -qty, workerId, type: 'out'});
                saveData('inventory', inventory);
                saveData('transactions', transactions);
                this.reset();
                renderInventory();
            } else {
                alert('Insufficient stock!');
            }
        };

        function renderInventory() {
            const tbody = document.querySelector('#inventoryTable tbody');
            tbody.innerHTML = inventory.map(i => `<tr><td>${i.name}</td><td>${i.qty}</td></tr>`).join('');
            populateSelect('takeOutItem', inventory.map(i => ({value: i.name, text: `${i.name} (${i.qty})`})));
        }

        function showWorkerInventory() {
            const workerId = document.getElementById('invWorker').value;
            const workerTrans = transactions.filter(t => t.workerId === workerId && t.type === 'out');
            const tbody = document.querySelector('#workerInvTable tbody');
            tbody.innerHTML = workerTrans.map(t => `<tr><td>${t.date}</td><td>${t.item}</td><td>${t.qty}</td></tr>`).join('');
            document.getElementById('workerInvTable').classList.remove('hidden');
        }

        // Calendar
        function showCalendarDay() {
            const date = document.getElementById('calDate').value || new Date().toISOString().split('T')[0];
            const dayHistory = cleaningHistory.filter(h => h.date === date);
            const content = dayHistory.map(h => {
                const cust = customers.find(c => c.id === h.customerId);
                if (!cust) return '';
                return `${cust.name}: Kitchens - ${h.cleanedKitchens.map(kc => cust.kitchens[kc.kitchenIdx].name).join(', ')}; Workers - ${h.cleanedKitchens.flatMap(kc => kc.cleanedKanals.map(ck => ck.worker)).join(', ')}`;
            }).filter(Boolean);
            document.getElementById('calendarDay').innerHTML = `<h3>${date}</h3><ul>${content.map(c => `<li>${c}</li>`).join('')}</ul>`;
        }

        // Initialization
        populateSelect('probCustomer', customers.map(c => ({value: c.id, text: c.name})));
        populateSelect('statsCustomer', [{value: '', text: 'All'}].concat(customers.map(c => ({value: c.id, text: c.name}))));
        populateSelect('statsWorker', [{value: '', text: 'All'}].concat(workers.map(w => ({value: w.id, text: w.name + ' ' + w.surname}))));
        document.getElementById('calDate').value = new Date().toISOString().split('T')[0];
        showCalendarDay();
        renderInventory();
        renderWorkers();
        renderProblems();
        renderCustomers();
    </script>
</body>
</html>
