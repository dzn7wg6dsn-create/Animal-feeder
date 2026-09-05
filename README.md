# Animal-feeder
Feed your hippo and watch your baby grow
Save this as animal-clicker.html and open it in Safari:

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Animal Clicker</title>
<style>
    * {
        box-sizing: border-box;
        -webkit-tap-highlight-color: transparent;
    }
    body {
        margin: 0;
        font-family: Arial, sans-serif;
        background: linear-gradient(135deg, #b7f7c1, #8ddcff);
        min-height: 100vh;
        text-align: center;
        color: #222;
    }
    header {
        background: rgba(255,255,255,.75);
        padding: 18px;
        backdrop-filter: blur(10px);
        position: sticky;
        top: 0;
        z-index: 5;
    }
    h1 {
        margin: 0;
        font-size: 34px;
    }
    .stats {
        margin-top: 8px;
        font-size: 18px;
        font-weight: bold;
    }
    .game {
        max-width: 650px;
        margin: auto;
        padding: 20px;
    }
    #hippo {
        font-size: 180px;
        border: none;
        background: transparent;
        cursor: pointer;
        transition: transform .08s;
        user-select: none;
    }
    #hippo:active {
        transform: scale(.9);
    }
    #weight {
        font-size: 42px;
        font-weight: 900;
        margin: 5px;
    }
    #rate {
        font-size: 18px;
        margin-bottom: 15px;
    }
    .shop {
        background: rgba(255,255,255,.8);
        border-radius: 22px;
        padding: 18px;
        margin-top: 20px;
        box-shadow: 0 8px 25px rgba(0,0,0,.12);
    }
    .shop h2 {
        margin-top: 0;
    }
    .upgrade {
        width: 100%;
        padding: 15px;
        margin: 8px 0;
        border: none;
        border-radius: 15px;
        background: #fff;
        text-align: left;
        font-size: 16px;
        cursor: pointer;
        box-shadow: 0 3px 8px rgba(0,0,0,.1);
    }
    .upgrade:disabled {
        opacity: .45;
        cursor: not-allowed;
    }
    .upgrade strong {
        display: block;
        font-size: 19px;
    }
    .price {
        float: right;
        font-weight: bold;
        color: #16833b;
    }
    .floating {
        position: fixed;
        pointer-events: none;
        font-weight: bold;
        font-size: 22px;
        animation: floatUp 1s forwards;
    }
    @keyframes floatUp {
        from {
            opacity: 1;
            transform: translateY(0);
        }
        to {
            opacity: 0;
            transform: translateY(-90px);
        }
    }
    .reset {
        margin: 20px;
        padding: 10px 18px;
        border: none;
        border-radius: 12px;
        background: #ff7777;
        color: white;
        font-weight: bold;
    }
</style>
</head>
<body>
<header>
    <h1>🦛 Animal Clicker</h1>
    <div class="stats">
        💰 Coins: <span id="coins">0</span>
    </div>
</header>
<div class="game">
    <div id="weight">0.0 lbs</div>
    <div id="rate">+0.2 lbs per click</div>
    <button id="hippo">🦛</button>
    <div class="shop">
        <h2>🛒 Upgrade Shop</h2>
        <button class="upgrade" id="melon">
            <span class="price">50 🪙</span>
            <strong>🍉 Melon</strong>
            +1 lb per click
        </button>
        <button class="upgrade" id="meat">
            <span class="price">150 🪙</span>
            <strong>🥩 Meat Chunk</strong>
            +1.5 lbs per click
        </button>
        <button class="upgrade" id="banana">
            <span class="price">400 🪙</span>
            <strong>🍌 Banana Mountain</strong>
            +3 lbs per click
        </button>
        <button class="upgrade" id="pizza">
            <span class="price">1,000 🪙</span>
            <strong>🍕 Infinite Pizza</strong>
            +8 lbs per click
        </button>
        <button class="upgrade" id="farm">
            <span class="price">750 🪙</span>
            <strong>🌾 Hippo Farm</strong>
            +1 lb every second
        </button>
        <button class="upgrade" id="chef">
            <span class="price">3,000 🪙</span>
            <strong>👨‍🍳 Personal Hippo Chef</strong>
            +5 lbs every second
        </button>
        <button class="upgrade" id="golden">
            <span class="price">10,000 🪙</span>
            <strong>👑 Golden Hippo</strong>
            Doubles all click power
        </button>
    </div>
    <button class="reset" onclick="resetGame()">Reset Game</button>
</div>
<script>
let weight = Number(localStorage.getItem("weight")) || 0;
let coins = Number(localStorage.getItem("coins")) || 0;
let clickPower = Number(localStorage.getItem("clickPower")) || 0.2;
let autoIncome = Number(localStorage.getItem("autoIncome")) || 0;
let bought = JSON.parse(localStorage.getItem("bought")) || {};
const hippo = document.getElementById("hippo");
const upgrades = {
    melon: {
        cost: 50,
        action: () => clickPower += 1
    },
    meat: {
        cost: 150,
        action: () => clickPower += 1.5
    },
    banana: {
        cost: 400,
        action: () => clickPower += 3
    },
    pizza: {
        cost: 1000,
        action: () => clickPower += 8
    },
    farm: {
        cost: 750,
        action: () => autoIncome += 1
    },
    chef: {
        cost: 3000,
        action: () => autoIncome += 5
    },
    golden: {
        cost: 10000,
        action: () => clickPower *= 2
    }
};
function format(num) {
    if (num >= 1000000) return (num / 1000000).toFixed(2) + "M";
    if (num >= 1000) return (num / 1000).toFixed(2) + "K";
    return Math.floor(num * 10) / 10;
}
function update() {
    document.getElementById("weight").textContent =
        format(weight) + " lbs";
    document.getElementById("coins").textContent =
        format(coins);
    document.getElementById("rate").textContent =
        "+" + format(clickPower) + " lbs per click";
    for (const id in upgrades) {
        const button = document.getElementById(id);
        if (bought[id]) {
            button.disabled = true;
            button.innerHTML = `
                <strong>✅ Purchased</strong>
                Upgrade activated!
            `;
        } else {
            button.disabled = coins < upgrades[id].cost;
            button.querySelector(".price").textContent =
                format(upgrades[id].cost) + " 🪙";
        }
    }
    localStorage.setItem("weight", weight);
    localStorage.setItem("coins", coins);
    localStorage.setItem("clickPower", clickPower);
    localStorage.setItem("autoIncome", autoIncome);
    localStorage.setItem("bought", JSON.stringify(bought));
}
hippo.addEventListener("click", function(event) {
    weight += clickPower;
    // Coins are earned from clicking too
    coins += Math.max(1, Math.floor(clickPower));
    const popup = document.createElement("div");
    popup.className = "floating";
    popup.textContent = "+" + format(clickPower) + " lbs";
    popup.style.left = event.clientX + "px";
    popup.style.top = event.clientY + "px";
    document.body.appendChild(popup);
    setTimeout(() => popup.remove(), 1000);
    update();
});
for (const id in upgrades) {
    document.getElementById(id).addEventListener("click", function() {
        if (bought[id]) return;
        if (coins >= upgrades[id].cost) {
            coins -= upgrades[id].cost;
            upgrades[id].action();
            bought[id] = true;
            update();
        }
    });
}
// Automatic income
setInterval(() => {
    if (autoIncome > 0) {
        weight += autoIncome;
        coins += Math.max(1, Math.floor(autoIncome));
        update();
    }
}, 1000);
function resetGame() {
    if (confirm("Reset all Animal Clicker progress?")) {
        localStorage.clear();
        location.reload();
    }
}
update();
</script>
</body>
</html>
