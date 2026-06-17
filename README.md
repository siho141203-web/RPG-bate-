<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<title>Simple RPG</title>

<style>
body {
    background:#0f0f0f;
    color:white;
    font-family:Arial;
    text-align:center;
}

button {
    padding:10px;
    margin:5px;
    cursor:pointer;
}

/* 로그 */
#log {
    height:220px;
    overflow:auto;
    border:3px solid white;
    padding:10px;
    margin-top:10px;
    color:#00ff00;
    text-align:left;
}

/* SHOP */
#shop {
    display:none;
    position:fixed;
    top:50%;
    left:50%;
    transform:translate(-50%,-50%);
    background:#1c1c1c;
    border:2px solid #555;
    padding:20px;
    width:320px;
    border-radius:10px;
}

.shop-btn {
    display:block;
    width:100%;
    margin:10px 0;
    padding:10px;
    background:#333;
    border:none;
    color:white;
}
.shop-btn:hover { background:#555; }

#inv {
    margin-top:10px;
    border:1px solid #333;
    padding:10px;
    display:none;
}
</style>
</head>

<body>

<h1>⚔️ SIMPLE RPG</h1>

<p>❤️ HP: <span id="hp">0</span> / <span id="maxhp">0</span></p>
<p>💰 Gold: <span id="gold">0</span></p>
<p>⭐ Level: <span id="level">1</span></p>
<p>🗡️ ATK: <span id="atk">10</span></p>
<p>🗡️ 무기: <span id="weapon">없음</span></p>

<button onclick="startBattle()">⚔️ 사냥 시작</button>
<button onclick="playerAttack()">👉 공격</button>
<button onclick="usePotion()">🧪 물약 사용</button>
<button onclick="returnTown()">🏠 마을 귀환</button>
<button onclick="openShop()">🏪 잡화점</button>
<button onclick="toggleInv()">🎒 인벤토리</button>

<div id="inv">
🧪 물약: <span id="potion">0</span>
</div>

<div id="log"></div>

<div id="shop">
    <h2>🏪 잡화점</h2>

    <button class="shop-btn" onclick="buyPotion()">🧪 물약 구매 (30G)</button>
    <button class="shop-btn" onclick="rollWeapon()">🎲 무기 뽑기 (100G)</button>
    <button class="shop-btn" onclick="closeShop()">❌ 닫기</button>
</div>

<script>

/* =====================
   상태
===================== */
let hp = 100;
let maxHp = 100;

let gold = 0;
let level = 1;
let exp = 0;

let baseAtk = 10;
let weaponAtk = 0;
let weaponName = "없음";

let monsterHp = 0;
let monsterName = "";

let inBattle = false;
let playerTurn = true;

/* 인벤토리 */
let inventory = {
    potion: 0
};

/* =====================
   로그
===================== */
function log(msg){
    const d = document.getElementById("log");

    const line = document.createElement("div");
    line.innerText = msg;

    d.appendChild(line);
    d.scrollTop = d.scrollHeight;
}

/* =====================
   UI
===================== */
function update(){
    document.getElementById("hp").innerText = hp;
    document.getElementById("maxhp").innerText = maxHp;
    document.getElementById("gold").innerText = gold;
    document.getElementById("level").innerText = level;
    document.getElementById("atk").innerText = baseAtk + weaponAtk;
    document.getElementById("weapon").innerText = weaponName;
    document.getElementById("potion").innerText = inventory.potion;
}

/* =====================
   인벤토리
===================== */
function toggleInv(){
    const inv = document.getElementById("inv");
    inv.style.display = (inv.style.display === "block") ? "none" : "block";
}

/* =====================
   전투 시작
===================== */
function startBattle(){

    if(inBattle){
        log("⚠️ 이미 전투 중!");
        return;
    }

    inBattle = true;
    playerTurn = true;

    const monsters = [
        {name:"슬라임", hp:40},
        {name:"고블린", hp:60},
        {name:"오크", hp:100}
    ];

    let m = monsters[Math.floor(Math.random()*monsters.length)];

    monsterName = m.name;
    monsterHp = m.hp + level * 15;

    log("🧟 " + monsterName + " 등장!");
}

/* =====================
   공격
===================== */
function playerAttack(){

    if(!inBattle){
        log("⚠️ 사냥 먼저!");
        return;
    }

    if(!playerTurn){
        log("⏳ 몬스터 턴!");
        return;
    }

    let atk = baseAtk + weaponAtk;

    if(Math.random() < 0.05){
        atk *= 1.5;
        atk = Math.floor(atk);
        log("💥 크리티컬!");
    }

    monsterHp -= atk;
    log("⚔️ -" + atk + " (몬스터 HP: " + monsterHp + ")");

    if(monsterHp <= 0){
        log("🏆 승리!");
        gold += 20;
        exp += 10;
        inBattle = false;
        update();
        return;
    }

    playerTurn = false;

    setTimeout(()=>{
        if(!inBattle) return; // 🔥 핵심 버그 방지

        let dmg = 5 + Math.floor(Math.random()*10);
        hp -= dmg;

        log("🧟 -" + dmg + " (HP: " + hp + ")");

        if(hp <= 0){

            log("💀 사망!");

            inBattle = false;
            playerTurn = true;

            monsterHp = 0;
            monsterName = "";

            hp = maxHp;
            gold = 0;
            level = 1;
            exp = 0;

            update();
            return;
        }

        playerTurn = true;
        update();

    }, 600);
}

/* =====================
   물약
===================== */
function usePotion(){

    if(inventory.potion <= 0){
        log("🧪 물약 없음!");
        return;
    }

    inventory.potion--;

    let heal = Math.floor(maxHp * 0.1) + 15;
    hp += heal;

    if(hp > maxHp) hp = maxHp;

    log("🧪 +" + heal);
    update();
}

/* =====================
   상점
===================== */
function buyPotion(){

    if(gold < 30){
        log("💰 돈 부족!");
        return;
    }

    gold -= 30;
    inventory.potion++;

    log("🧪 물약 +1");
    update();
}

function rollWeapon(){

    if(gold < 100){
        log("💰 돈 부족!");
        return;
    }

    gold -= 100;

    let r = Math.random();

    if(r < 0.5){
        weaponName = "낡은 검";
        weaponAtk = 5;
    } else if(r < 0.8){
        weaponName = "철검";
        weaponAtk = 10;
    } else if(r < 0.95){
        weaponName = "마검";
        weaponAtk = 20;
    } else {
        weaponName = "전설의 검";
        weaponAtk = 35;
    }

    log("🎲 " + weaponName);
    update();
}

/* =====================
   마을
===================== */
function returnTown(){
    inBattle = false;
    monsterHp = 0;
    monsterName = "";
    hp = maxHp;

    log("🏠 마을 귀환");
    update();
}

/* =====================
   UI
===================== */
function openShop(){
    document.getElementById("shop").style.display = "block";
}

function closeShop(){
    document.getElementById("shop").style.display = "none";
}

update();

</script>

</body>
</html>
