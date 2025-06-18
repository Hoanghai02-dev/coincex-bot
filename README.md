# coincex-bot
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Coincex Price Bot</title>
  <style>
    body { font-family: Arial; padding: 20px; background: #f4f4f4; }
    h1 { color: #333; }
    .section { margin-bottom: 30px; padding: 20px; background: white; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
    select { padding: 5px; font-size: 16px; }
    p span { font-weight: bold; color: #2b9348; }
  </style>
</head>
<body>

  <div class="section">
    <h1>📡 Cập nhật giá coin từ Coincex</h1>

    <label for="market">Chọn cặp coin:</label>
    <select id="market">
      <option value="BTCUSDT">BTC/USDT</option>
      <option value="ETHUSDT">ETH/USDT</option>
      <option value="LTCUSDT">LTC/USDT</option>
    </select>
  </div>

  <div class="section">
    <h2>🌐 REST API (5s/lần)</h2>
    <p>Giá hiện tại: <span id="price_rest">--</span></p>
    <p>Thời gian: <span id="time_rest">--:--:--</span></p>
  </div>

  <div class="section">
    <h2>⚡ WebSocket Real-time</h2>
    <p>Giá hiện tại: <span id="price_ws">--</span></p>
    <p>Trạng thái kết nối: <span id="ws_status">--</span></p>
  </div>

  <script>
    const marketSelect = document.getElementById("market");
    const priceREST = document.getElementById("price_rest");
    const timeREST = document.getElementById("time_rest");
    const priceWS = document.getElementById("price_ws");
    const wsStatus = document.getElementById("ws_status");

    async function fetchREST() {
      const market = marketSelect.value;
      try {
        const res = await fetch(`https://api.coinex.com/v2/spot/ticker?market=${market}`);
        const data = await res.json();
        if (data.code === 0) {
          const ticker = data.data[0];
          priceREST.innerText = ticker.last;
          timeREST.innerText = new Date().toLocaleTimeString();
        } else {
          priceREST.innerText = 'Lỗi API';
        }
      } catch (e) {
        priceREST.innerText = 'Không kết nối';
      }
    }

    let ws;

    function startWebSocket() {
      const market = marketSelect.value;
      if (ws) ws.close();

      ws = new WebSocket("wss://socket.coinex.com/v2/spot");
      wsStatus.innerText = '🔄 Đang kết nối...';

      ws.onopen = () => {
        wsStatus.innerText = '✅ Đã kết nối';
        ws.send(JSON.stringify({
          method: "state.subscribe",
          params: { market_list: [market] },
          id: 1
        }));
      };

      ws.onmessage = (event) => {
        const msg = JSON.parse(event.data);
        if (msg.method === "state.update") {
          const state = msg.params[0];
          if (state.market === marketSelect.value) {
            priceWS.innerText = state.last;
          }
        }
      };

      ws.onclose = () => {
        wsStatus.innerText = '❌ Mất kết nối';
      };
    }

    marketSelect.addEventListener("change", () => {
      fetchREST();
      startWebSocket();
    });

    // Chạy khi load trang
    fetchREST();
    startWebSocket();
    setInterval(fetchREST, 5000); // REST API mỗi 5 giây
  </script>

</body>
</html>
