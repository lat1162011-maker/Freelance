my freelance work

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Company Portal</title>
    <style>
        body {
            font-family: 'Segoe UI', sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f5f5f5;
            color: #333;
        }
        .container {
            max-width: 800px;
            margin: 20px auto;
            padding: 20px;
            background: white;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #2c3e50;
            text-align: center;
        }
        .btn {
            display: block;
            width: 100%;
            padding: 12px;
            margin: 10px 0;
            background: #3498db;
            color: white;
            border: none;
            border-radius: 4px;
            font-size: 16px;
            cursor: pointer;
            transition: background 0.3s;
        }
        .btn:hover {
            background: #2980b9;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box;
        }
        .contract-item {
            border: 1px solid #eee;
            padding: 15px;
            margin-bottom: 10px;
            border-radius: 4px;
        }
        .progress-bar {
            height: 20px;
            background: #ecf0f1;
            border-radius: 4px;
            margin-top: 5px;
        }
        .progress-fill {
            height: 100%;
            border-radius: 4px;
            background: #2ecc71;
        }
        .hidden {
            display: none;
        }
        .logout-btn {
            background: #e74c3c;
        }
        .logout-btn:hover {
            background: #c0392b;
        }
    </style>
</head>
<body>
    <div class="container">
        <div id="login-page">
            <h1>Company Portal Login</h1>
            <div class="form-group">
                <label for="employee-id">Employee ID</label>
                <input type="text" id="employee-id" placeholder="Enter your employee ID">
            </div>
            <div class="form-group">
                <label for="password">Password</label>
                <input type="password" id="password" placeholder="Enter your password">
            </div>
            <button class="btn" onclick="login()">Login</button>
        </div>

        <div id="home-page" class="hidden">
            <h1>Company Portal</h1>
            <button class="btn" onclick="showPage('contracts-page')">Manage Contracts</button>
            <button class="btn" onclick="showPage('clockin-page')">Clock In/Out</button>
            <button class="btn logout-btn" onclick="logout()">Logout</button>
        </div>

        <div id="contracts-page" class="hidden">
            <h1>Manage Contracts</h1>
            <button class="btn" onclick="addContract()">Add New Contract</button>
            <div id="contracts-list"></div>
            <button class="btn" onclick="showPage('home-page')">Back to Home</button>
        </div>

        <div id="clockin-page" class="hidden">
            <h1>Clock In/Out</h1>
            <div id="clock-status">You are currently clocked out</div>
            <button class="btn" id="clock-btn" onclick="toggleClock()">Clock In</button>
            <div class="form-group">
                <label for="hours-worked">Hours Worked Today</label>
                <input type="text" id="hours-worked" readonly value="0">
            </div>
            <button class="btn" onclick="showPage('home-page')">Back to Home</button>
        </div>

        <div id="add-contract-form" class="hidden">
            <h1>Add New Contract</h1>
            <div class="form-group">
                <label for="contract-name">Contract Name</label>
                <input type="text" id="contract-name">
            </div>
            <div class="form-group">
                <label for="contract-client">Client Company</label>
                <input type="text" id="contract-client">
            </div>
            <div class="form-group">
                <label for="renewal-chance">Renewal Chance (%)</label>
                <input type="number" id="renewal-chance" min="0" max="100">
            </div>
            <button class="btn" onclick="saveContract()">Save Contract</button>
            <button class="btn" onclick="cancelAddContract()">Cancel</button>
        </div>
    </div>

    <script>
        // Sample employee data
        const employees = {
            "Lathan T1": {
                password: "LATHANT1",
                canManageContracts: true,
                clockedIn: false,
                hoursWorked: 0
            }
        };

        // Sample contracts data
        let contracts = [
            { id: 1, name: "Service Agreement", client: "ABC Corp", renewalChance: 75 },
            { id: 2, name: "Consulting Contract", client: "XYZ Inc", renewalChance: 60 }
        ];

        let currentUser = null;
        let clockInterval = null;

        function login() {
            const employeeId = document.getElementById('employee-id').value;
            const password = document.getElementById('password').value;

            if (employees[employeeId] && employees[employeeId].password === password) {
                currentUser = employeeId;
                document.getElementById('login-page').classList.add('hidden');
                document.getElementById('home-page').classList.remove('hidden');
                updateContractsList();
            } else {
                alert("Invalid employee ID or password");
            }
        }

        function logout() {
            currentUser = null;
            clearInterval(clockInterval);
            document.getElementById('home-page').classList.add('hidden');
            document.getElementById('contracts-page').classList.add('hidden');
            document.getElementById('clockin-page').classList.add('hidden');
            document.getElementById('login-page').classList.remove('hidden');
            document.getElementById('employee-id').value = "";
            document.getElementById('password').value = "";
        }

        function showPage(pageId) {
            document.querySelectorAll('.container > div').forEach(div => {
                div.classList.add('hidden');
            });
            document.getElementById(pageId).classList.remove('hidden');
        }

        function updateContractsList() {
            const listElement = document.getElementById('contracts-list');
            listElement.innerHTML = "";
            
            contracts.forEach(contract => {
                const contractElement = document.createElement('div');
                contractElement.className = 'contract-item';
                
                contractElement.innerHTML = `
                    <h3>${contract.name}</h3>
                    <p>Client: ${contract.client}</p>
                    <p>Renewal Chance: ${contract.renewalChance}%</p>
                    <div class="progress-bar">
                        <div class="progress-fill" style="width: ${contract.renewalChance}%"></div>
                    </div>
                    ${employees[currentUser].canManageContracts ? 
                        `<button onclick="deleteContract(${contract.id})" style="margin-top: 10px;">Delete</button>` : ''}
                `;
                
                listElement.appendChild(contractElement);
            });
        }

        function toggleClock() {
            const employee = employees[currentUser];
            employee.clockedIn = !employee.clockedIn;
            
            const clockBtn = document.getElementById('clock-btn');
            const clockStatus = document.getElementById('clock-status');
            
            if (employee.clockedIn) {
                clockBtn.textContent = "Clock Out";
                clockStatus.textContent = "You are currently clocked in";
                
                // Start tracking hours
                clockInterval = setInterval(() => {
                    employee.hoursWorked += 0.1;
                    document.getElementById('hours-worked').value = employee.hoursWorked.toFixed(1);
                }, 60000); // Update every minute
            } else {
                clockBtn.textContent = "Clock In";
                clockStatus.textContent = "You are currently clocked out";
                
                // Stop tracking hours
                clearInterval(clockInterval);
            }
        }

        function addContract() {
            document.getElementById('contracts-page').classList.add('hidden');
            document.getElementById('add-contract-form').classList.remove('hidden');
        }

        function cancelAddContract() {
            document.getElementById('add-contract-form').classList.add('hidden');
            document.getElementById('contracts-page').classList.remove('hidden');
        }

        function saveContract() {
            const name = document.getElementById('contract-name').value;
            const client = document.getElementById('contract-client').value;
            const renewalChance = parseInt(document.getElementById('renewal-chance').value);
            
            if (name && client && renewalChance >= 0 && renewalChance <= 100) {
                const newId = contracts.length > 0 ? Math.max(...contracts.map(c => c.id)) + 1 : 1;
                contracts.push({
                    id: newId,
                    name,
                    client,
                    renewalChance
                });
                
                document.getElementById('contract-name').value = "";
                document.getElementById('contract-client').value = "";
                document.getElementById('renewal-chance').value = "";
                
                cancelAddContract();
                updateContractsList();
            } else {
                alert("Please fill all fields correctly");
            }
        }

        function deleteContract(id) {
            contracts = contracts.filter(contract => contract.id !== id);
            updateContractsList();
        }
    </script>
</body>
</html>
