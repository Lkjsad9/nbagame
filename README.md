<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>NBA自建球员选秀模拟</title>
<style>
*{box-sizing:border-box;font-family:system-ui}
body{max-width:900px;margin:12px auto;padding:12px;background:#111;color:#eee;line-height:1.6}
button{padding:8px 14px;margin:4px;background:#2563eb;color:#fff;border:none;border-radius:6px;cursor:pointer;font-size:14px}
button:disabled{background:#444;cursor:not-allowed}
#log{background:#1a1a1a;padding:12px;border-radius:8px;min-height:220px;white-space:pre-wrap;margin:10px 0;border:1px solid #333}
.card{background:#222;padding:12px;border-radius:8px;margin:8px 0}
</style>
</head>
<body>
<h1>NBA生涯模拟｜自定义选秀</h1>
<div class="card">
<h3>你的自定义新秀（已加入2003选秀池）</h3>
<p>姓名：Alex Li | 位置：SF 小前锋 | 身高：201cm | 体重：94kg | 年龄：19 | 潜力：91</p>
<p>属性：内74｜外77｜三分76｜罚球81｜控球80｜传球73｜速度82｜力量73｜弹跳80｜防守75｜篮板71｜体力84｜心智87</p>
</div>
<div>
<button id="btnNewGame">🎮 开启2003选秀新存档</button>
<button id="btnSimStep">⏭ 模拟下一阶段</button>
<button id="btnClear">🗑 清除存档</button>
</div>
<div id="log"></div>

<script>
const STORAGE_KEY = "nba_custom_save";

// ========== 自定义你的球员 你可以在这里直接改参数 ==========
const myCustomRookie = {
    firstName:"Alex",
    lastName:"Li",
    age:19,
    position:"SF",
    height:201,
    weight:94,
    attributes:{
        inside:74,
        outside:77,
        three:76,
        freeThrow:81,
        ballHandle:80,
        pass:73,
        speed:82,
        strength:73,
        vertical:80,
        defense:75,
        rebounding:71,
        stamina:84,
        mental:87
    },
    potential:91,
    isReal:false,
    draftYear:2003
};

// 2003原版部分新秀 + 插入我们的自定义球员
function get2003Prospects(){
    const baseProspects = [
        {firstName:"LeBron",lastName:"James",age:18,position:"SF",height:203,weight:113,potential:98,isReal:true,draftYear:2003},
        {firstName:"Darko",lastName:"Milicic",age:18,position:"C",height:216,weight:113,potential:88,isReal:true,draftYear:2003},
        {firstName:"Carmelo",lastName:"Anthony",age:19,position:"SF",height:203,weight:104,potential:94,isReal:true,draftYear:2003},
        {firstName:"Chris",lastName:"Bosh",age:19,position:"PF",height:211,weight:107,potential:93,isReal:true,draftYear:2003},
        {firstName:"Dwyane",lastName:"Wade",age:21,position:"SG",height:193,weight:100,potential:95,isReal:true,draftYear:2003}
    ];
    baseProspects.push(myCustomRookie); // 把自己扔进选秀名单
    return shuffleArray(baseProspects);
}

function shuffleArray(arr){
    const a=[...arr];
    for(let i=a.length-1;i>0;i--){
        const j=Math.floor(Math.random()*(i+1));
        [a[i],a[j]]=[a[j],a[i]];
    }
    return a;
}

// 游戏状态
let state = null;
const logEl = document.getElementById("log");

function log(text){
    logEl.innerText += text+"\n";
    logEl.scrollTop = logEl.scrollHeight;
}

function clearLog(){logEl.innerText=""}

function save(){
    localStorage.setItem(STORAGE_KEY,JSON.stringify(state));
}
function load(){
    const s = localStorage.getItem(STORAGE_KEY);
    if(s) state = JSON.parse(s);
}
function wipeSave(){
    localStorage.removeItem(STORAGE_KEY);
    state=null;
    clearLog();
    log("存档已清除，请开启新游戏");
}

// 新建游戏：选秀
function newGame(){
    clearLog();
    const prospects = get2003Prospects();
    log("===== 2003 NBA 选秀大会开始 =====");
    let pick=1;
    let myPickNum=null;
    let myDraftedTeam=null;
    const teams = ["骑士","活塞","掘金","猛龙","热火","快船","雄鹿","尼克斯","奇才","灰熊"];

    for(const p of prospects){
        const team = teams[(pick-1)%teams.length];
        log(`第${pick}顺位：${team}选中 ${p.firstName} ${p.lastName}（${p.position}）`);
        if(p.firstName===myCustomRookie.firstName && p.lastName===myCustomRookie.lastName){
            myPickNum = pick;
            myDraftedTeam = team;
        }
        pick++;
        if(pick>10) break;
    }

    log(`\n✅你（${myCustomRookie.firstName} ${myCustomRookie.lastName}）第${myPickNum}顺位被【${myDraftedTeam}】选中！`);
    log("--- 生涯开始 ---");

    state = {
        phase:"regular",
        year:2003,
        player:{...myCustomRookie,ovr:78},
        team:myDraftedTeam,
        pick:myPickNum,
        seasonStats:{ppg:0,rpg:0,apg:0},
        retired:false
    };
    save();
}

// 模拟一步赛季
function simulateStep(){
    if(!state){log("请先开启新游戏");return}
    if(state.retired){log("球员已经退役，开新档游玩");return}

    if(state.phase==="regular"){
        log(`\n[${state.year‑04}赛季｜常规赛进行中]`);
        // 简单随机数据
        const ppg = (12 + Math.random()*14).toFixed(1);
        const rpg = (4 + Math.random()*7).toFixed(1);
        const apg = (3 + Math.random()*6).toFixed(1);
        state.seasonStats={ppg,rpg,apg};
        log(`你的数据：${ppg}分 ${rpg}篮板 ${apg}助攻`);
        log("进入季后赛……");
        state.phase="playoff";
    }else if(state.phase==="playoff"){
        log("\n[季后赛]");
        const winChance = Math.random();
        if(winChance>0.45){
            log("✅打进总决赛，夺得总冠军！");
        }else{
            log("❌季后赛止步，无缘总冠军");
        }
        log("赛季结束，休赛期到来");
        state.phase="offseason";
    }else if(state.phase==="offseason"){
        state.year +=1;
        // 年龄增长，随机成长/下滑
        state.player.age +=1;
        const age = state.player.age;
        if(age<28){
            log(`\n休赛期｜年龄${age}岁，球员获得成长！`);
        }else if(age>34){
            log(`\n休赛期｜年龄${age}岁，身体开始下滑`);
            if(Math.random()>0.7){
                log("🏁你的球员选择退役！生涯结束");
                state.retired=true;
            }
        }else{
            log(`\n休赛期｜年龄${age}岁，保持巅峰状态`);
        }
        state.phase="regular";
    }
    save();
}

// 绑定按钮
document.getElementById("btnNewGame").onclick = newGame;
document.getElementById("btnSimStep").onclick = simulateStep;
document.getElementById("btnClear").onclick = wipeSave;

// 页面加载尝试读取旧存档
load();
if(state){
    log("检测到已有存档，点击【模拟下一阶段】继续你的生涯");
}
</script>
</body>
</html># nbagame
nba
