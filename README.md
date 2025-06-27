<!DOCTYPE html>
<html lang="sv">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gemensam Ekonomi</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            color: #333;
        }

        .container {
            max-width: 500px;
            margin: 0 auto;
            background: white;
            min-height: 100vh;
            box-shadow: 0 0 20px rgba(0,0,0,0.1);
        }

        .header {
            background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
            color: white;
            padding: 20px;
            text-align: center;
            position: sticky;
            top: 0;
            z-index: 100;
        }

        .nav-tabs {
            display: flex;
            background: #f8f9fa;
            border-bottom: 1px solid #dee2e6;
        }

        .nav-tab {
            flex: 1;
            padding: 15px;
            text-align: center;
            background: none;
            border: none;
            cursor: pointer;
            font-size: 14px;
            font-weight: 500;
            transition: all 0.3s;
        }

        .nav-tab.active {
            background: #007bff;
            color: white;
        }

        .tab-content {
            display: none;
            padding: 20px;
        }

        .tab-content.active {
            display: block;
        }

        .card {
            background: white;
            border-radius: 12px;
            padding: 20px;
            margin-bottom: 20px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
            border: 1px solid #e9ecef;
        }

        .card-title {
            font-size: 18px;
            font-weight: 600;
            margin-bottom: 15px;
            color: #2c3e50;
        }

        .input-group {
            margin-bottom: 15px;
        }

        .input-group label {
            display: block;
            margin-bottom: 5px;
            font-weight: 500;
            color: #555;
        }

        .input-group input, .input-group select {
            width: 100%;
            padding: 12px;
            border: 2px solid #e9ecef;
            border-radius: 8px;
            font-size: 16px;
            transition: border-color 0.3s;
        }

        .input-group input:focus, .input-group select:focus {
            outline: none;
            border-color: #007bff;
        }

        .btn {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            padding: 12px 20px;
            border-radius: 8px;
            cursor: pointer;
            font-size: 16px;
            font-weight: 500;
            width: 100%;
            margin-top: 10px;
            transition: transform 0.2s;
        }

        .btn:hover {
            transform: translateY(-2px);
        }

        .btn-danger {
            background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%);
        }

        .summary-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 20px;
        }

        .summary-item {
            background: linear-gradient(135deg, #a8edea 0%, #fed6e3 100%);
            padding: 15px;
            border-radius: 10px;
            text-align: center;
        }

        .summary-item h3 {
            font-size: 14px;
            margin-bottom: 5px;
            color: #555;
        }

        .summary-item .amount {
            font-size: 20px;
            font-weight: 700;
            color: #2c3e50;
        }

        .expense-item, .saving-item {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 10px;
            border-left: 4px solid #007bff;
        }

        .expense-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }

        .expense-title {
            font-weight: 600;
            color: #2c3e50;
        }

        .expense-amount {
            font-size: 18px;
            font-weight: 700;
            color: #007bff;
        }

        .expense-details {
            font-size: 14px;
            color: #666;
            margin-bottom: 5px;
        }

        .delete-btn {
            background: #dc3545;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 12px;
        }

        .month-selector {
            display: flex;
            align-items: center;
            justify-content: center;
            margin-bottom: 20px;
            gap: 15px;
        }

        .month-btn {
            background: #007bff;
            color: white;
            border: none;
            padding: 8px 12px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 18px;
        }

        .current-month {
            font-size: 18px;
            font-weight: 600;
        }

        .balance-summary {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 20px;
            border-radius: 12px;
            margin-bottom: 20px;
            text-align: center;
        }

        .balance-amount {
            font-size: 24px;
            font-weight: 700;
            margin-bottom: 10px;
        }

        .percentage-input {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .percentage-input input {
            flex: 1;
        }

        .percentage-input span {
            font-weight: 600;
            color: #007bff;
        }

        @media (max-width: 480px) {
            .summary-grid {
                grid-template-columns: 1fr;
            }
            
            .nav-tab {
                font-size: 12px;
                padding: 12px 8px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>💰 Gemensam Ekonomi</h1>
            <p>Hantera er ekonomi tillsammans</p>
        </div>

        <div class="nav-tabs">
            <button class="nav-tab active" onclick="showTab('overview')">Översikt</button>
            <button class="nav-tab" onclick="showTab('monthly')">Månad</button>
            <button class="nav-tab" onclick="showTab('settings')">Inställningar</button>
        </div>

        <!-- Översikt Tab -->
        <div id="overview" class="tab-content active">
            <div class="balance-summary">
                <div class="balance-amount" id="totalBalance">0 kr</div>
                <div>Total balans denna månad</div>
            </div>

            <div class="summary-grid">
                <div class="summary-item">
                    <h3>Total inkomst</h3>
                    <div class="amount" id="totalIncome">0 kr</div>
                </div>
                <div class="summary-item">
                    <h3>Totala utgifter</h3>
                    <div class="amount" id="totalExpenses">0 kr</div>
                </div>
                <div class="summary-item">
                    <h3>Din andel</h3>
                    <div class="amount" id="yourShare">0 kr</div>
                </div>
                <div class="summary-item">
                    <h3>Partners andel</h3>
                    <div class="amount" id="partnerShare">0 kr</div>
                </div>
            </div>

            <div class="card">
                <div class="card-title">Senaste transaktioner</div>
                <div id="recentTransactions">
                    <p style="color: #666; text-align: center;">Inga transaktioner än</p>
                </div>
            </div>
        </div>

        <!-- Månad Tab -->
        <div id="monthly" class="tab-content">
            <div class="month-selector">
                <button class="month-btn" onclick="changeMonth(-1)">‹</button>
                <div class="current-month" id="currentMonth">Januari 2025</div>
                <button class="month-btn" onclick="changeMonth(1)">›</button>
            </div>

            <!-- Räkningar -->
            <div class="card">
                <div class="card-title">Lägg till räkning</div>
                <div class="input-group">
                    <label>Beskrivning</label>
                    <input type="text" id="billDescription" placeholder="t.ex. Hyra, El, Internet">
                </div>
                <div class="input-group">
                    <label>Belopp (kr)</label>
                    <input type="number" id="billAmount" placeholder="0">
                </div>
                <div class="input-group">
                    <label>Vem betalade?</label>
                    <select id="billPayer">
                        <option value="you">Du</option>
                        <option value="partner">Partner</option>
                    </select>
                </div>
                <button class="btn" onclick="addBill()">Lägg till räkning</button>
            </div>

            <!-- Sparande/Rörliga utgifter -->
            <div class="card">
                <div class="card-title">Lägg till sparande/utgift</div>
                <div class="input-group">
                    <label>Beskrivning</label>
                    <input type="text" id="savingDescription" placeholder="t.ex. Mat, Bensin, Sparande">
                </div>
                <div class="input-group">
                    <label>Belopp (kr)</label>
                    <input type="number" id="savingAmount" placeholder="0">
                </div>
                <div class="input-group">
                    <label>Fördelning</label>
                    <select id="savingDistribution" onchange="togglePercentageInput()">
                        <option value="equal">Jämn fördelning (50/50)</option>
                        <option value="percentage">Enligt löneprocent</option>
                    </select>
                </div>
                <div class="input-group">
                    <label>Vem betalade?</label>
                    <select id="savingPayer">
                        <option value="you">Du</option>
                        <option value="partner">Partner</option>
                    </select>
                </div>
                <button class="btn" onclick="addSaving()">Lägg till utgift/sparande</button>
            </div>

            <!-- Lista räkningar -->
            <div class="card">
                <div class="card-title">Räkningar denna månad</div>
                <div id="billsList">
                    <p style="color: #666; text-align: center;">Inga räkningar tillagda</p>
                </div>
            </div>

            <!-- Lista sparande -->
            <div class="card">
                <div class="card-title">Sparande/Utgifter denna månad</div>
                <div id="savingsList">
                    <p style="color: #666; text-align: center;">Inga utgifter tillagda</p>
                </div>
            </div>
        </div>

        <!-- Inställningar Tab -->
        <div id="settings" class="tab-content">
            <div class="card">
                <div class="card-title">Månadslöner</div>
                <div class="input-group">
                    <label>Din nettolön (kr)</label>
                    <input type="number" id="yourSalary" placeholder="0" onblur="saveSalaries()">
                </div>
                <div class="input-group">
                    <label>Partners nettolön (kr)</label>
                    <input type="number" id="partnerSalary" placeholder="0" onblur="saveSalaries()">
                </div>
                <div class="summary-item" style="margin-top: 15px;">
                    <h3>Din andel av totallönen</h3>
                    <div class="amount" id="yourPercentage">0%</div>
                </div>
                <div class="summary-item">
                    <h3>Partners andel av totallönen</h3>
                    <div class="amount" id="partnerPercentage">0%</div>
                </div>
            </div>

            <div class="card">
                <div class="card-title">Namn</div>
                <div class="input-group">
                    <label>Ditt namn</label>
                    <input type="text" id="yourName" placeholder="Ditt namn" onblur="saveNames()">
                </div>
                <div class="input-group">
                    <label>Partners namn</label>
                    <input type="text" id="partnerName" placeholder="Partners namn" onblur="saveNames()">
                </div>
            </div>

            <div class="card">
                <div class="card-title">Datahantering</div>
                <button class="btn btn-danger" onclick="clearAllData()">Rensa all data</button>
                <button class="btn" onclick="exportData()" style="margin-top: 10px;">Exportera data</button>
            </div>
        </div>
    </div>

    <script>
        // Global variables
        let currentMonthIndex = new Date().getMonth();
        let currentYear = new Date().getFullYear();
        const months = ['Januari', 'Februari', 'Mars', 'April', 'Maj', 'Juni', 
                       'Juli', 'Augusti', 'September', 'Oktober', 'November', 'December'];

        // Data structure
        let appData = {
            yourSalary: 0,
            partnerSalary: 0,
            yourName: 'Du',
            partnerName: 'Partner',
            months: {}
        };

        // Initialize app
        function initApp() {
            loadData();
            updateMonth();
            updateOverview();
            updateSalaryPercentages();
        }

        // Tab management
        function showTab(tabName) {
            document.querySelectorAll('.tab-content').forEach(tab => {
                tab.classList.remove('active');
            });
            document.querySelectorAll('.nav-tab').forEach(tab => {
                tab.classList.remove('active');
            });
            
            document.getElementById(tabName).classList.add('active');
            event.target.classList.add('active');
        }

        // Month navigation
        function changeMonth(direction) {
            currentMonthIndex += direction;
            if (currentMonthIndex < 0) {
                currentMonthIndex = 11;
                currentYear--;
            } else if (currentMonthIndex > 11) {
                currentMonthIndex = 0;
                currentYear++;
            }
            updateMonth();
            updateOverview();
        }

        function updateMonth() {
            document.getElementById('currentMonth').textContent = `${months[currentMonthIndex]} ${currentYear}`;
            displayMonthData();
        }

        function getCurrentMonthKey() {
            return `${currentYear}-${currentMonthIndex + 1}`;
        }

        function getCurrentMonthData() {
            const key = getCurrentMonthKey();
            if (!appData.months[key]) {
                appData.months[key] = {
                    bills: [],
                    savings: []
                };
            }
            return appData.months[key];
        }

        // Bills management
        function addBill() {
            const description = document.getElementById('billDescription').value;
            const amount = parseFloat(document.getElementById('billAmount').value);
            const payer = document.getElementById('billPayer').value;

            if (!description || !amount) {
                alert('Fyll i alla fält');
                return;
            }

            const monthData = getCurrentMonthData();
            monthData.bills.push({
                id: Date.now(),
                description,
                amount,
                payer,
                date: new Date().toISOString()
            });

            // Clear form
            document.getElementById('billDescription').value = '';
            document.getElementById('billAmount').value = '';

            saveData();
            displayMonthData();
            updateOverview();
        }

        function deleteBill(id) {
            const monthData = getCurrentMonthData();
            monthData.bills = monthData.bills.filter(bill => bill.id !== id);
            saveData();
            displayMonthData();
            updateOverview();
        }

        // Savings management
        function addSaving() {
            const description = document.getElementById('savingDescription').value;
            const amount = parseFloat(document.getElementById('savingAmount').value);
            const distribution = document.getElementById('savingDistribution').value;
            const payer = document.getElementById('savingPayer').value;

            if (!description || !amount) {
                alert('Fyll i alla fält');
                return;
            }

            const monthData = getCurrentMonthData();
            monthData.savings.push({
                id: Date.now(),
                description,
                amount,
                distribution,
                payer,
                date: new Date().toISOString()
            });

            // Clear form
            document.getElementById('savingDescription').value = '';
            document.getElementById('savingAmount').value = '';

            saveData();
            displayMonthData();
            updateOverview();
        }

        function deleteSaving(id) {
            const monthData = getCurrentMonthData();
            monthData.savings = monthData.savings.filter(saving => saving.id !== id);
            saveData();
            displayMonthData();
            updateOverview();
        }

        // Display functions
        function displayMonthData() {
            const monthData = getCurrentMonthData();
            
            // Display bills
            const billsList = document.getElementById('billsList');
            if (monthData.bills.length === 0) {
                billsList.innerHTML = '<p style="color: #666; text-align: center;">Inga räkningar tillagda</p>';
            } else {
                billsList.innerHTML = monthData.bills.map(bill => {
                    const payerName = bill.payer === 'you' ? appData.yourName : appData.partnerName;
                    return `
                        <div class="expense-item">
                            <div class="expense-header">
                                <div class="expense-title">${bill.description}</div>
                                <div class="expense-amount">${bill.amount.toLocaleString()} kr</div>
                            </div>
                            <div class="expense-details">Betald av: ${payerName}</div>
                            <button class="delete-btn" onclick="deleteBill(${bill.id})">Ta bort</button>
                        </div>
                    `;
                }).join('');
            }

            // Display savings
            const savingsList = document.getElementById('savingsList');
            if (monthData.savings.length === 0) {
                savingsList.innerHTML = '<p style="color: #666; text-align: center;">Inga utgifter tillagda</p>';
            } else {
                savingsList.innerHTML = monthData.savings.map(saving => {
                    const payerName = saving.payer === 'you' ? appData.yourName : appData.partnerName;
                    const distributionText = saving.distribution === 'equal' ? 'Jämn fördelning' : 'Enligt löneprocent';
                    return `
                        <div class="saving-item">
                            <div class="expense-header">
                                <div class="expense-title">${saving.description}</div>
                                <div class="expense-amount">${saving.amount.toLocaleString()} kr</div>
                            </div>
                            <div class="expense-details">Betald av: ${payerName}</div>
                            <div class="expense-details">Fördelning: ${distributionText}</div>
                            <button class="delete-btn" onclick="deleteSaving(${saving.id})">Ta bort</button>
                        </div>
                    `;
                }).join('');
            }
        }

        // Calculate balances
        function calculateBalance() {
            const monthData = getCurrentMonthData();
            const totalSalary = appData.yourSalary + appData.partnerSalary;
            const yourPercentage = totalSalary > 0 ? appData.yourSalary / totalSalary : 0.5;
            const partnerPercentage = 1 - yourPercentage;

            let yourOwed = 0;
            let partnerOwed = 0;
            let yourPaid = 0;
            let partnerPaid = 0;

            // Calculate bills (split by salary percentage)
            monthData.bills.forEach(bill => {
                yourOwed += bill.amount * yourPercentage;
                partnerOwed += bill.amount * partnerPercentage;
                
                if (bill.payer === 'you') {
                    yourPaid += bill.amount;
                } else {
                    partnerPaid += bill.amount;
                }
            });

            // Calculate savings
            monthData.savings.forEach(saving => {
                if (saving.distribution === 'equal') {
                    yourOwed += saving.amount * 0.5;
                    partnerOwed += saving.amount * 0.5;
                } else {
                    yourOwed += saving.amount * yourPercentage;
                    partnerOwed += saving.amount * partnerPercentage;
                }

                if (saving.payer === 'you') {
                    yourPaid += saving.amount;
                } else {
                    partnerPaid += saving.amount;
                }
            });

            return {
                yourBalance: yourPaid - yourOwed,
                partnerBalance: partnerPaid - partnerOwed,
                totalIncome: totalSalary,
                totalExpenses: yourOwed + partnerOwed,
                yourOwed,
                partnerOwed
            };
        }

        function updateOverview() {
            const balance = calculateBalance();
            
            document.getElementById('totalBalance').textContent = 
                (balance.yourBalance + balance.partnerBalance).toLocaleString() + ' kr';
            document.getElementById('totalIncome').textContent = 
                balance.totalIncome.toLocaleString() + ' kr';
            document.getElementById('totalExpenses').textContent = 
                balance.totalExpenses.toLocaleString() + ' kr';
            document.getElementById('yourShare').textContent = 
                balance.yourOwed.toLocaleString() + ' kr';
            document.getElementById('partnerShare').textContent = 
                balance.partnerOwed.toLocaleString() + ' kr';

            // Update recent transactions
            const monthData = getCurrentMonthData();
            const allTransactions = [
                ...monthData.bills.map(b => ({...b, type: 'bill'})),
                ...monthData.savings.map(s => ({...s, type: 'saving'}))
            ].sort((a, b) => new Date(b.date) - new Date(a.date)).slice(0, 5);

            const recentDiv = document.getElementById('recentTransactions');
            if (allTransactions.length === 0) {
                recentDiv.innerHTML = '<p style="color: #666; text-align: center;">Inga transaktioner än</p>';
            } else {
                recentDiv.innerHTML = allTransactions.map(t => {
                    const payerName = t.payer === 'you' ? appData.yourName : appData.partnerName;
                    const typeText = t.type === 'bill' ? 'Räkning' : 'Utgift';
                    return `
                        <div class="expense-details" style="padding: 8px 0; border-bottom: 1px solid #eee;">
                            <strong>${t.description}</strong> - ${t.amount.toLocaleString()} kr<br>
                            <small>${typeText} • Betald av ${payerName}</small>
                        </div>
                    `;
                }).join('');
            }
        }

        // Settings
        function saveSalaries() {
            appData.yourSalary = parseFloat(document.getElementById('yourSalary').value) || 0;
            appData.partnerSalary = parseFloat(document.getElementById('partnerSalary').value) || 0;
            saveData();
            updateSalaryPercentages();
            updateOverview();
        }

        function saveNames() {
            appData.yourName = document.getElementById('yourName').value || 'Du';
            appData.partnerName = document.getElementById('partnerName').value || 'Partner';
            saveData();
            displayMonthData();
            updateOverview();
        }

        function updateSalaryPercentages() {
            const total = appData.yourSalary + appData.partnerSalary;
            if (total > 0) {
                const yourPerc = Math.round((appData.yourSalary / total) * 100);
                const partnerPerc = 100 - yourPerc;
                document.getElementById('yourPercentage').textContent = yourPerc + '%';
                document.getElementById('partnerPercentage').textContent = partnerPerc + '%';
            } else {
                document.getElementById('yourPercentage').textContent = '50%';
                document.getElementById('partnerPercentage').textContent = '50%';
            }
        }

        // Data management
        function saveData() {
            const data = JSON.stringify(appData);
            // Store in memory instead of localStorage
            window.economyData = data;
        }

        function loadData() {
            try {
                // Load from memory if available
                if (window.economyData) {
                    appData = JSON.parse(window.economyData);
                }
            } catch (e) {
                console.log('No previous data found');
            }

            // Update UI with loaded data
            document.getElementById('yourSalary').value = appData.yourSalary || '';
            document.getElementById('partnerSalary').value = appData.partnerSalary || '';
            document.getElementById('yourName').value = appData.yourName || 'Du';
            document.getElementById('partnerName').value = appData.partnerName || 'Partner';
        }

        function clearAllData() {
            if (confirm('Är du säker på att du vill radera all data? Detta kan inte ångras.')) {
                appData = {
                    yourSalary: 0,
                    partnerSalary: 0,
                    yourName: 'Du',
                    partnerName: 'Partner',
                    months: {}
                };
                window.economyData = null;
                initApp();
            }
        }

        function exportData() {
            const dataStr = JSON.stringify(appData, null, 2);
            const dataBlob = new Blob([dataStr], {type: 'application/json'});
            const url = URL.createObjectURL(dataBlob);
            const link = document.createElement('a');
            link.href = url;
            link.download = 'ekonomi-data.json';
            link.click();
            URL.revokeObjectURL(url);
        }

        // Initialize app when page loads
        document.addEventListener('DOMContentLoaded', initApp);
    </script>
</body>
</html>
