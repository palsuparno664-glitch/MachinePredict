<!DOCTYPE html>
<html>
<head>
<title>Smart Factory System</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body{
    margin:0;
    font-family:Segoe UI, sans-serif;
    background:#0b1220;
    color:white;
    display:flex;
}

/* Sidebar */
.sidebar{
    width:220px;
    background:#111827;
    padding:20px;
    height:100vh;
}

.sidebar h2{ color:#00e0ff; }

.sidebar ul{ list-style:none; padding:0; }

.sidebar li{
    padding:12px;
    margin:10px 0;
    background:#1f2937;
    border-radius:8px;
    cursor:pointer;
}

.sidebar li:hover{
    background:#00e0ff;
    color:black;
}

/* Main */
.main{
    flex:1;
    padding:20px;
}

.page{ display:none; }
.active{ display:block; }

/* Fleet Cards */
.cards{
    display:grid;
    grid-template-columns:repeat(auto-fit,minmax(250px,1fr));
    gap:20px;
}

.card{
    background:#1f2937;
    border-radius:12px;
    overflow:hidden;
    cursor:pointer;
    transition:0.3s;
}

.card:hover{ transform:scale(1.05); }

.card img{
    width:100%;
    height:160px;
    object-fit:cover;
}

.card-content{
    padding:15px;
}

/* Analytics */
.analytics-top{
    display:flex;
    gap:20px;
    align-items:center;
}

.analytics-top img{
    width:250px;
    height:180px;
    object-fit:cover;
    border-radius:12px;
}

.controlPanel{
    background:#1f2937;
    padding:15px;
    border-radius:12px;
    flex:1;
}

input[type=range]{ width:100%; }

.metric span{
    color:#00e0ff;
    font-weight:bold;
}

.damage{
    color:#ef4444;
    font-weight:bold;
}

canvas{
    background:#111827;
    padding:15px;
    border-radius:12px;
    margin-top:20px;
}
</style>
</head>

<body>

<div class="sidebar">
    <h2>MachinePredict</h2>
    <ul>
        <li onclick="showPage('dashboard')">Dashboard</li>
        <li onclick="showPage('fleet')">Fleet Management</li>
        <li onclick="showPage('analytics')">Analytics</li>
        <li onclick="showPage('alerts')">Active Alerts</li>
        <li onclick="showPage('settings')">Settings</li>
    </ul>
</div>

<div class="main">

<!-- Dashboard -->
<div id="dashboard" class="page active">
    <h1>System Dashboard</h1>
    <p>Status: <span style="color:#16a34a;">Operational</span></p>
</div>

<!-- Fleet -->
<div id="fleet" class="page">
    <h1>Machine Fleet Overview</h1>
    <div class="cards" id="fleetContainer"></div>
</div>

<!-- Analytics -->
<div id="analytics" class="page">
    <h1 id="machineTitle">Machine Analytics</h1>

    <div class="analytics-top">
        <img id="machineImage" src="">
        <div class="controlPanel">

            Load: <span id="loadVal">50</span>% 
            <input type="range" id="load" min="0" max="100" value="50">

            Speed: <span id="speedVal">2000</span> RPM
            <input type="range" id="speed" min="500" max="5000" value="2000">

            Operating Hours: <span id="hoursVal">1000</span>
            <input type="range" id="hours" min="0" max="10000" value="1000">

            <div class="metric">
                ⚡ Power: <span id="powerOut">0</span> kW<br>
                🌡 Temperature: <span id="tempOut">0</span> °C<br>
                📈 Vibration: <span id="vibeOut">0</span> mm/s²<br>
                ⚙ Efficiency: <span id="effOut">0</span>%<br>
                🔧 Health: <span id="healthOut">100</span>%<br>
                ⚠ Damaged Part: <span id="damageOut" class="damage">None</span>
            </div>

        </div>
    </div>

    <canvas id="powerChart"></canvas>
    <canvas id="tempChart"></canvas>
    <canvas id="vibeChart"></canvas>
</div>

<!-- Alerts -->
<div id="alerts" class="page">
    <h1>Active Alerts</h1>
    <p>No active alerts.</p>
</div>

<!-- Settings -->
<div id="settings" class="page">
    <h1>Settings</h1>
    <p>Theme: Dark Industrial</p>
</div>

</div>

<script>

/* Navigation */
function showPage(id){
    document.querySelectorAll('.page').forEach(p=>p.classList.remove('active'));
    document.getElementById(id).classList.add('active');
}

/* Machines */
let machines=[
{
    name:"Primary Compressor A",
    img:"https://images.unsplash.com/photo-1581092918367-1c0e0c1d3d02"
},
{
    name:"Hydraulic Press #4",
    img:"https://images.unsplash.com/photo-1581090700227-4c4f50b79c54"
},
{
    name:"Cooling Fan Unit 7",
    img:"https://images.unsplash.com/photo-1581092160607-ee22621dd758"
},
{
    name:"Conveyor Motor B12",
    img:"https://images.unsplash.com/photo-1581092919530-3c52b67a3b32"
}
];

/* Generate Fleet */
function generateFleet(){
    let html="";
    machines.forEach((m,i)=>{
        html+=`
        <div class="card" onclick="openMachine(${i})">
            <img src="${m.img}">
            <div class="card-content">
                <h3>${m.name}</h3>
                <p>Click to analyze</p>
            </div>
        </div>
        `;
    });
    document.getElementById("fleetContainer").innerHTML=html;
}
generateFleet();

/* Open Machine → Switch to Analytics Tab */
function openMachine(index){
    showPage("analytics");

    document.getElementById("machineTitle").innerText=machines[index].name;
    document.getElementById("machineImage").src=machines[index].img;
}

/* Charts */
let labels=[],powerData=[],tempData=[],vibeData=[];
let powerChart=new Chart(document.getElementById("powerChart"),{
    type:"line",
    data:{labels:labels,datasets:[{label:"Power (kW)",data:powerData,borderColor:"#00e0ff"}]}
});
let tempChart=new Chart(document.getElementById("tempChart"),{
    type:"line",
    data:{labels:labels,datasets:[{label:"Temperature (°C)",data:tempData,borderColor:"#f97316"}]}
});
let vibeChart=new Chart(document.getElementById("vibeChart"),{
    type:"line",
    data:{labels:labels,datasets:[{label:"Vibration (mm/s²)",data:vibeData,borderColor:"#ef4444"}]}
});

/* Real-Time Simulation */
function updateMachine(){

let load=parseFloat(document.getElementById("load").value);
let speed=parseFloat(document.getElementById("speed").value);
let hours=parseFloat(document.getElementById("hours").value);

document.getElementById("loadVal").innerText=load;
document.getElementById("speedVal").innerText=speed;
document.getElementById("hoursVal").innerText=hours;

let power=(load*speed)/10000;
let temp=30+(load/2)+(hours/500);
let vibe=(load/10)+(speed/2000);

let efficiency=((50-power)/50)*100;
if(efficiency<0) efficiency=0;

let wear=(hours/10000)*100 + vibe*2;
let health=100-wear;
if(health<0) health=0;

let damaged="None";
if(vibe>10) damaged="Bearing";
if(temp>100) damaged="Cooling System";
if(hours>8000) damaged="Rotor";
if(load>90) damaged="Overload Shaft";

document.getElementById("powerOut").innerText=power.toFixed(2);
document.getElementById("tempOut").innerText=temp.toFixed(2);
document.getElementById("vibeOut").innerText=vibe.toFixed(2);
document.getElementById("effOut").innerText=efficiency.toFixed(2);
document.getElementById("healthOut").innerText=health.toFixed(2);
document.getElementById("damageOut").innerText=damaged;

if(labels.length>20){
    labels.shift(); powerData.shift(); tempData.shift(); vibeData.shift();
}

labels.push("");
powerData.push(power);
tempData.push(temp);
vibeData.push(vibe);

powerChart.update();
tempChart.update();
vibeChart.update();
}

setInterval(updateMachine,1000);

</script>

</body>
</html>
