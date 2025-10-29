const STORAGE_KEY = "unibudget_ulima_pro_v1";
let txs = []; // 🔹 Ya no hay transacciones por defecto

document.addEventListener("DOMContentLoaded", () => {
  load();
  render();
  renderChart();

  document.getElementById("txForm").addEventListener("submit", addTx);
  document.getElementById("btnClear").addEventListener("click", clearForm);
  document.getElementById("themeToggle").addEventListener("click", toggleTheme);
});

function addTx(e) {
  e.preventDefault();
  const data = {
    id: Date.now(),
    date: document.getElementById("date").value,
    category: document.getElementById("category").value,
    type: document.getElementById("type").value,
    amount: parseFloat(document.getElementById("amount").value),
    note: document.getElementById("note").value
  };
  txs.unshift(data);
  save();
  render();
  renderChart();
  clearForm();
}

function render() {
  const list = document.getElementById("txList");
  list.innerHTML = "";
  txs.forEach(t => {
    const li = document.createElement("li");
    li.textContent = `${t.date} - ${t.category}: ${t.type === "gasto" ? "-" : "+"}S/ ${t.amount.toFixed(2)}`;
    list.appendChild(li);
  });
  document.getElementById("txCount").textContent = `${txs.length} registros`;
  updateTotals();
}

function updateTotals() {
  const inc = txs.filter(t => t.type === "ingreso").reduce((a, b) => a + b.amount, 0);
  const exp = txs.filter(t => t.type === "gasto").reduce((a, b) => a + b.amount, 0);
  document.getElementById("totalIncome").textContent = `S/ ${inc.toFixed(2)}`;
  document.getElementById("totalExpense").textContent = `S/ ${exp.toFixed(2)}`;
  document.getElementById("balance").textContent = `S/ ${(inc - exp).toFixed(2)}`;
}

function renderChart() {
  const ctx = document.getElementById("pieChart").getContext("2d");
  const gastos = txs.filter(t => t.type === "gasto");
  const categorias = [...new Set(gastos.map(g => g.category))];
  const montos = categorias.map(c => gastos.filter(g => g.category === c).reduce((a, b) => a + b.amount, 0));
  new Chart(ctx, {
    type: "pie",
    data: { labels: categorias, datasets: [{ data: montos }] },
    options: { plugins: { legend: { labels: { color: getComputedStyle(document.body).getPropertyValue('--text') } } } }
  });
}

function clearForm() {
  document.getElementById("txForm").reset();
  document.getElementById("date").value = new Date().toISOString().split("T")[0];
}

function save() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(txs));
}

function load() {
  const data = localStorage.getItem(STORAGE_KEY);
  txs = data ? JSON.parse(data) : [];
}

function toggleTheme() {
  const current = document.body.dataset.theme;
  document.body.dataset.theme = current === "dark" ? "light" : "dark";
}
