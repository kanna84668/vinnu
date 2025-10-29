
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Loan Lending App v3</title>
<style>
  body { font-family: 'Segoe UI', sans-serif; background: #f3f6fb; color: #333; margin:0; padding:20px; }
  h1,h2,h3{color:#2a4d9b;}
  .container { max-width: 650px; margin:auto; background:#fff; padding:20px; border-radius:10px; box-shadow:0 2px 10px rgba(0,0,0,0.1);}
  label { display:block; margin-top:10px; font-weight:600; }
  input, select { width:100%; padding:10px; margin-top:5px; border:1px solid #ccc; border-radius:5px; font-size:1rem;}
  button { background:#2a4d9b; color:#fff; border:none; padding:12px 20px; border-radius:5px; font-size:1rem; margin-top:10px; cursor:pointer;}
  button:hover { background:#1e3873;}
  .logout { background:#c33; margin-top:10px; }
  .loan-card { border:1px solid #d5e1ff; background:#f0f4ff; border-radius:8px; padding:15px; margin-top:10px;}
  .hidden { display:none; }
  ul { list-style:none; padding:0; }
  li { margin-bottom:8px; }
  .payment-btn { background:#28a745; margin-top:5px; }
  .paid { text-decoration: line-through; color: gray; }
</style>
</head>
<body>

<h1>Loan Lending App v3</h1>

<!-- Login / Signup -->
<div class="container" id="authContainer">
  <h3>Login / Signup</h3>
  <label for="username">Username:</label>
  <input type="text" id="username" required>
  <label for="role">Role:</label>
  <select id="role">
    <option value="user">User</option>
    <option value="admin">Admin</option>
  </select>
  <button id="loginBtn">Login / Signup</button>
</div>

<!-- User Dashboard -->
<div class="container hidden" id="loanContainer">
  <h3>Welcome, <span id="userLabel"></span>!</h3>
  <button class="logout" id="logoutBtn">Logout</button>

  <form id="loanForm">
    <h3>Apply for Loan</h3>
    <label for="amount">Loan Amount ($):</label>
    <input type="number" id="amount" required min="100">

    <label for="term">Term (months):</label>
    <input type="number" id="term" required min="1">

    <label for="rate">Interest Rate (% per year):</label>
    <input type="number" id="rate" required step="0.1">

    <button type="submit">Apply</button>
  </form>

  <div id="result" class="loan-card hidden"></div>

  <div id="loanHistory" class="hidden">
    <h3>Your Loan History</h3>
    <div id="historyList"></div>
  </div>
</div>

<!-- Admin Dashboard -->
<div class="container hidden" id="adminContainer">
  <h3>Admin Dashboard</h3>
  <button class="logout" id="adminLogoutBtn">Logout</button>
  <div id="pendingLoans">
    <h3>Pending Loans</h3>
  </div>
</div>

<script>
let currentUser = localStorage.getItem('currentUser') || null;
let currentRole = localStorage.getItem('currentRole') || null;

// --- Elements ---
const authContainer = document.getElementById('authContainer');
const loanContainer = document.getElementById('loanContainer');
const adminContainer = document.getElementById('adminContainer');
const loginBtn = document.getElementById('loginBtn');
const logoutBtn = document.getElementById('logoutBtn');
const adminLogoutBtn = document.getElementById('adminLogoutBtn');
const usernameInput = document.getElementById('username');
const roleSelect = document.getElementById('role');
const userLabel = document.getElementById('userLabel');

const loanForm = document.getElementById('loanForm');
const result = document.getElementById('result');
const loanHistoryDiv = document.getElementById('loanHistory');
const historyList = document.getElementById('historyList');
const pendingLoansDiv = document.getElementById('pendingLoans');

// --- Functions ---
function showUserDashboard() {
  authContainer.classList.add('hidden');
  loanContainer.classList.remove('hidden');
  adminContainer.classList.add('hidden');
  userLabel.textContent = currentUser;
  renderLoanHistory();
}

function showAdminDashboard() {
  authContainer.classList.add('hidden');
  loanContainer.classList.add('hidden');
  adminContainer.classList.remove('hidden');
  renderPendingLoans();
}

function showAuth() {
  authContainer.classList.remove('hidden');
  loanContainer.classList.add('hidden');
  adminContainer.classList.add('hidden');
}

function saveLoan(user, loan) {
  let allLoans = JSON.parse(localStorage.getItem('allLoans') || '[]');
  // initialize payments array
  loan.payments = Array(loan.term).fill(false);
  allLoans.push({...loan, username: user, status:'Pending', id:Date.now()});
  localStorage.setItem('allLoans', JSON.stringify(allLoans));
}

// --- Login / Signup ---
loginBtn.addEventListener('click', () => {
  const username = usernameInput.value.trim();
  const role = roleSelect.value;
  if(!username) return alert('Enter a username');

  currentUser = username;
  currentRole = role;
  localStorage.setItem('currentUser', currentUser);
  localStorage.setItem('currentRole', currentRole);

  if(role==='admin') showAdminDashboard();
  else showUserDashboard();
});

logoutBtn.addEventListener('click', () => { localStorage.removeItem('currentUser'); localStorage.removeItem('currentRole'); currentUser=null; showAuth(); });
adminLogoutBtn.addEventListener('click', () => { localStorage.removeItem('currentUser'); localStorage.removeItem('currentRole'); currentUser=null; showAuth(); });

// --- Loan Application ---
loanForm.addEventListener('submit', e => {
  e.preventDefault();
  const amount = parseFloat(document.getElementById('amount').value);
  const term = parseInt(document.getElementById('term').value);
  const rate = parseFloat(document.getElementById('rate').value);

  const monthlyRate = rate / 100 / 12;
  const monthlyPayment = (amount * monthlyRate) / (1 - Math.pow(1 + monthlyRate, -term));
  const totalPayment = monthlyPayment * term;

  const loan = { amount, term, rate, monthlyPayment: monthlyPayment.toFixed(2), totalPayment: totalPayment.toFixed(2), date: new Date().toLocaleString() };
  saveLoan(currentUser, loan);

  result.classList.remove('hidden');
  result.innerHTML = `<h3>Loan Submitted ✅</h3><p>Status: Pending</p>`;
  renderLoanHistory();
});

// --- Loan History ---
function renderLoanHistory() {
  const allLoans = JSON.parse(localStorage.getItem('allLoans') || '[]');
  const userLoans = allLoans.filter(l => l.username===currentUser);
  if(userLoans.length===0) { loanHistoryDiv.classList.add('hidden'); return; }
  loanHistoryDiv.classList.remove('hidden');
  historyList.innerHTML = userLoans.map(l=>{
    let paymentsHTML = l.payments.map((p,i)=>`
      <li class="${p ? 'paid' : ''}">
        Month ${i+1}: $${l.monthlyPayment} ${!p && l.status==='Approved' ? `<button class="payment-btn" onclick="pay(${l.id},${i})">Pay</button>` : '' }
      </li>
    `).join('');
    return `
      <div class="loan-card">
        ${l.date}: $${l.amount.toFixed(2)} for ${l.term} months @ ${l.rate}%.<br>
        Status: ${l.status} | Monthly: $${l.monthlyPayment} | Total: $${l.totalPayment}<br>
        <strong>Payments:</strong>
        <ul>${paymentsHTML}</ul>
      </div>
    `;
  }).join('');
}

// --- Make Payment ---
window.pay = function(loanId, monthIndex) {
  let allLoans = JSON.parse(localStorage.getItem('allLoans') || '[]');
  allLoans = allLoans.map(l => {
    if(l.id === loanId) {
      l.payments[monthIndex] = true;
    }
    return l;
  });
  localStorage.setItem('allLoans', JSON.stringify(allLoans));
  renderLoanHistory();
}

// --- Admin View ---
function renderPendingLoans() {
  const allLoans = JSON.parse(localStorage.getItem('allLoans') || '[]');
  const pendingLoans = allLoans.filter(l => l.status==='Pending');
  if(pendingLoans.length===0) { pendingLoansDiv.innerHTML = '<p>No pending loans</p>'; return; }

  pendingLoansDiv.innerHTML = pendingLoans.map(l=>`
    <div class="loan-card" id="loan_${l.id}">
      <strong>User:</strong> ${l.username}<br>
      Amount: $${l.amount.toFixed(2)} | Term: ${l.term} months | Rate: ${l.rate}%<br>
      <button onclick="updateLoanStatus(${l.id},'Approved')">Approve ✅</button>
      <button onclick="updateLoanStatus(${l.id},'Rejected')">Reject ❌</button>
    </div>
  `).join('');
}

window.updateLoanStatus = function(id,status) {
  let allLoans = JSON.parse(localStorage.getItem('allLoans') || '[]');
  allLoans = allLoans.map(l=>l.id===id ? {...l, status} : l);
  localStorage.setItem('allLoans', JSON.stringify(allLoans));
  renderPendingLoans();
  renderLoanHistory();
}

// --- Initial Display ---
if(currentUser) {
  if(currentRole==='admin') showAdminDashboard();
  else showUserDashboard();
}
</script>
</body>
</html>
