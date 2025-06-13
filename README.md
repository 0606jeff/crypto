# crypto
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <title>加密貨幣均價計算器</title>
  <style>
    body { font-family: "Microsoft JhengHei", sans-serif; padding: 20px; max-width: 600px; margin: auto; background: #f9f9f9; }
    h2 { color: #333; }
    input, select, button {
      padding: 8px;
      margin: 5px 0;
      width: 100%;
      box-sizing: border-box;
      font-size: 16px;
    }
    button { cursor: pointer; }
    .output, .history {
      margin-top: 20px;
      padding: 15px;
      background: #fff;
      border: 1px solid #ccc;
      border-radius: 8px;
    }
    .trade-entry {
      display: flex;
      justify-content: space-between;
      margin: 4px 0;
      font-size: 14px;
    }
    .trade-entry button {
      padding: 2px 6px;
      font-size: 12px;
      background-color: #f44336;
      color: white;
      border: none;
      border-radius: 3px;
    }
  </style>
</head>
<body>
  <h2>📈 加密貨幣均價計算器</h2>

  <label>動作：</label>
  <select id="action">
    <option value="buy">買入</option>
    <option value="sell">賣出</option>
  </select>

  <label>數量：</label>
  <input type="number" id="quantity" placeholder="輸入數量" step="0.0001" min="0.0001" required>

  <label>單價：</label>
  <input type="number" id="price" placeholder="輸入單價" step="0.0001" min="0.0001" required>

  <button onclick="addTrade()">➕ 儲存並計算</button>
  <button onclick="resetTrades()">🗑 清除所有交易</button>
  <button onclick="exportCSV()">📁 匯出 CSV</button>

  <div class="output" id="result">💡 請輸入交易資料...</div>
  <div class="history" id="history"></div>

  <script>
    let trades = JSON.parse(localStorage.getItem("cryptoTrades") || "[]");

    function saveTrades() {
      localStorage.setItem("cryptoTrades", JSON.stringify(trades));
    }

    function addTrade() {
      const action = document.getElementById("action").value;
      const quantity = parseFloat(document.getElementById("quantity").value);
      const price = parseFloat(document.getElementById("price").value);

      if (!quantity || !price || quantity <= 0 || price <= 0) {
        alert("請輸入有效的數量與價格");
        return;
      }

      trades.push({ action, quantity, price });
      saveTrades();
      calculateAverage();
      showHistory();

      document.getElementById("quantity").value = "";
      document.getElementById("price").value = "";
    }

    function deleteTrade(index) {
      if (confirm("確定要刪除這筆交易？")) {
        trades.splice(index, 1);
        saveTrades();
        calculateAverage();
        showHistory();
      }
    }

    function calculateAverage() {
      let totalQty = 0;
      let totalCost = 0;

      for (let t of trades) {
        if (t.action === "buy") {
          totalCost += t.quantity * t.price;
          totalQty += t.quantity;
        } else if (t.action === "sell") {
          if (totalQty === 0) continue;
          let avgCost = totalCost / totalQty;
          totalCost -= t.quantity * avgCost;
          totalQty -= t.quantity;
        }
      }

      let result = document.getElementById("result");
      if (totalQty <= 0) {
        result.innerHTML = "🚫 沒有持倉";
      } else {
        let avgPrice = totalCost / totalQty;
        result.innerHTML = `📊 平均成本：$${avgPrice.toFixed(4)}<br>📦 剩餘持倉：${totalQty.toFixed(4)} 單位`;
      }
    }

    function showHistory() {
      const container = document.getElementById("history");
      container.innerHTML = "<strong>📜 交易紀錄：</strong><br>";

      trades.forEach((t, i) => {
        container.innerHTML += `
          <div class="trade-entry">
            ${i + 1}. ${t.action.toUpperCase()} - ${t.quantity.toFixed(4)} @ $${t.price.toFixed(4)}
            <button onclick="deleteTrade(${i})">🗑</button>
          </div>
        `;
      });
    }

    function resetTrades() {
      if (confirm("確定要清除所有交易？")) {
        trades = [];
        saveTrades();
        calculateAverage();
        showHistory();
      }
    }

    function exportCSV() {
      let csv = "動作,數量,價格\n";
      trades.forEach(t => {
        csv += `${t.action},${t.quantity.toFixed(4)},${t.price.toFixed(4)}\n`;
      });

      const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
      const url = URL.createObjectURL(blob);
      const link = document.createElement("a");
      link.setAttribute("href", url);
      link.setAttribute("download", "crypto_trades.csv");
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }

    // 初始顯示
    calculateAverage();
    showHistory();
  </script>
</body>
</html>
