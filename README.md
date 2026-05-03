<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>終端數據庫 // TERMINAL_ACCESS</title>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Noto+Sans+TC:wght@300;400;700&display=swap" rel="stylesheet">
    
    <style>
        /* 基礎變數與科幻調色盤 */
        :root {
            --bg-color: #050505;
            --neon-green: #00ff66;
            --neon-pink: #ff00ff;
            --neon-blue: #00e5ff;
            --neon-orange: #ff3300;
            --text-color: #e0e0e0;
        }

        /* 全局重置與字體設定 */
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            font-family: 'Noto Sans TC', 'Orbitron', sans-serif;
            overflow-x: hidden;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            background-image: 
                linear-gradient(rgba(0, 229, 255, 0.03) 1px, transparent 1px),
                linear-gradient(90deg, rgba(0, 229, 255, 0.03) 1px, transparent 1px);
            background-size: 20px 20px;
        }

        /* 掃描線特效 (CRT 螢幕感) */
        body::after {
            content: "";
            position: fixed;
            top: 0; left: 0; width: 100vw; height: 100vh;
            background: linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.25) 50%), linear-gradient(90deg, rgba(255, 0, 0, 0.06), rgba(0, 255, 0, 0.02), rgba(0, 0, 255, 0.06));
            background-size: 100% 2px, 3px 100%;
            pointer-events: none;
            z-index: 999;
        }

        /* 標題區塊 */
        header {
            margin-top: 50px;
            margin-bottom: 30px;
            text-align: center;
            border-bottom: 2px solid var(--neon-blue);
            padding-bottom: 10px;
            width: 80%;
            max-width: 1000px;
            box-shadow: 0 5px 15px rgba(0, 229, 255, 0.2);
        }

        h1 {
            font-family: 'Orbitron', sans-serif;
            color: var(--neon-blue);
            text-shadow: 0 0 10px var(--neon-blue);
            font-size: 2rem;
            letter-spacing: 2px;
        }

        /* 導覽列與按鈕 */
        .nav-container {
            display: flex;
            gap: 15px;
            margin-bottom: 40px;
            flex-wrap: wrap;
            justify-content: center;
            width: 80%;
            max-width: 1000px;
        }

        .tab-btn {
            background: transparent;
            color: var(--text-color);
            border: 1px solid var(--neon-green);
            padding: 12px 24px;
            font-size: 1.1rem;
            font-family: 'Noto Sans TC', sans-serif;
            cursor: pointer;
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
            text-transform: uppercase;
            box-shadow: inset 0 0 0 rgba(0, 255, 102, 0), 0 0 5px rgba(0, 255, 102, 0.3);
        }

        /* 按鈕霓虹特效 */
        .tab-btn:hover {
            background: rgba(0, 255, 102, 0.1);
            color: var(--neon-green);
            box-shadow: inset 0 0 10px rgba(0, 255, 102, 0.5), 0 0 15px rgba(0, 255, 102, 0.5);
            text-shadow: 0 0 5px var(--neon-green);
        }

        .tab-btn.active {
            background: var(--neon-pink);
            color: #fff;
            border-color: var(--neon-pink);
            box-shadow: 0 0 15px var(--neon-pink), inset 0 0 10px rgba(255, 255, 255, 0.5);
            text-shadow: 0 0 5px #fff;
        }

        /* 第三個按鈕套用橘紅色特效 */
        .tab-btn:nth-child(3) {
            border-color: var(--neon-orange);
            box-shadow: inset 0 0 0 rgba(255, 51, 0, 0), 0 0 5px rgba(255, 51, 0, 0.3);
        }
        .tab-btn:nth-child(3):hover {
            background: rgba(255, 51, 0, 0.1);
            color: var(--neon-orange);
            box-shadow: inset 0 0 10px rgba(255, 51, 0, 0.5), 0 0 15px rgba(255, 51, 0, 0.5);
            text-shadow: 0 0 5px var(--neon-orange);
        }
        .tab-btn:nth-child(3).active {
            background: var(--neon-orange);
            color: #fff;
            border-color: var(--neon-orange);
            box-shadow: 0 0 15px var(--neon-orange);
        }

        /* 內容顯示區塊 */
        .content-container {
            width: 80%;
            max-width: 1000px;
            background: rgba(10, 10, 10, 0.8);
            border: 1px solid #333;
            border-left: 4px solid var(--neon-blue);
            min-height: 400px;
            padding: 30px;
            position: relative;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.8);
        }

        /* 裝飾性角標 */
        .content-container::before {
            content: '';
            position: absolute;
            top: -1px; left: -1px;
            width: 20px; height: 20px;
            border-top: 2px solid var(--neon-pink);
            border-left: 2px solid var(--neon-pink);
        }
        .content-container::after {
            content: '';
            position: absolute;
            bottom: -1px; right: -1px;
            width: 20px; height: 20px;
            border-bottom: 2px solid var(--neon-orange);
            border-right: 2px solid var(--neon-orange);
        }

        /* 分頁內容切換邏輯 */
        .tab-content {
            display: none;
            animation: fadeIn 0.4s ease-in-out;
        }

        .tab-content.active {
            display: block;
        }

        /* 內容文字樣式 */
        .tab-content h2 {
            color: var(--neon-blue);
            margin-bottom: 20px;
            font-size: 1.5rem;
            display: flex;
            align-items: center;
        }

        .tab-content h2::before {
            content: '>>';
            color: var(--neon-green);
            margin-right: 10px;
            font-family: 'Orbitron', sans-serif;
            text-shadow: 0 0 5px var(--neon-green);
        }

        .placeholder-text {
            color: var(--neon-pink);
            font-size: 1.2rem;
            animation: pulse 2s infinite;
        }

        /* 動畫設定 */
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        @keyframes pulse {
            0% { opacity: 0.6; text-shadow: 0 0 5px rgba(255,0,255,0.5); }
            50% { opacity: 1; text-shadow: 0 0 15px rgba(255,0,255,0.8); }
            100% { opacity: 0.6; text-shadow: 0 0 5px rgba(255,0,255,0.5); }
        }
        
        /* 角色檔案專用進階樣式 */
.profile-grid {
    display: grid;
    grid-template-columns: 1fr 2fr;
    gap: 20px;
    margin-top: 20px;
}

.profile-header {
    grid-column: 1 / -1;
    border-left: 5px solid var(--neon-pink);
    padding-left: 15px;
    margin-bottom: 20px;
}

.quote-box {
    font-style: italic;
    color: var(--neon-blue);
    margin-top: 5px;
    font-size: 0.9rem;
    letter-spacing: 1px;
}

.info-panel {
    background: rgba(0, 255, 102, 0.05);
    border: 1px solid rgba(0, 255, 102, 0.3);
    padding: 15px;
}

.info-panel h3 {
    color: var(--neon-green);
    font-size: 1rem;
    border-bottom: 1px solid var(--neon-green);
    margin-bottom: 10px;
    padding-bottom: 5px;
}

.data-row {
    display: flex;
    justify-content: space-between;
    margin-bottom: 8px;
    font-size: 0.9rem;
}

.data-label { color: #888; }
.data-value { color: #fff; text-shadow: 0 0 5px rgba(255, 255, 255, 0.5); }

.tag-cloud {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
}

.tag {
    padding: 2px 8px;
    border: 1px solid;
    font-size: 0.8rem;
    border-radius: 3px;
}

.tag-pink { border-color: var(--neon-pink); color: var(--neon-pink); }
.tag-blue { border-color: var(--neon-blue); color: var(--neon-blue); }
.tag-orange { border-color: var(--neon-orange); color: var(--neon-orange); }

.evolution-list {
    list-style: none;
    position: relative;
    padding-left: 20px;
}

.evolution-list::before {
    content: '';
    position: absolute;
    left: 0; top: 0; bottom: 0;
    width: 2px;
    background: var(--neon-orange);
}

.evolution-item {
    margin-bottom: 15px;
    position: relative;
}

.evolution-item::after {
    content: '●';
    position: absolute;
    left: -24px; top: 0;
    color: var(--neon-orange);
    font-size: 0.8rem;
}

.evo-age { color: var(--neon-orange); font-weight: bold; margin-right: 10px; }

/* 反應式調整 */
@media (max-width: 768px) {
    .profile-grid { grid-template-columns: 1fr; }
}

/* 故事大綱專用：時間軸日誌樣式 */
.timeline-container {
    position: relative;
    padding-left: 30px;
    margin-top: 30px;
}

.timeline-container::before {
    content: '';
    position: absolute;
    left: 9px;
    top: 0;
    bottom: 0;
    width: 2px;
    background: var(--neon-blue);
    box-shadow: 0 0 10px var(--neon-blue);
}

.log-entry {
    position: relative;
    margin-bottom: 30px;
    background: rgba(10, 10, 10, 0.6);
    border: 1px solid #333;
    padding: 20px;
    transition: all 0.3s ease;
}

.log-entry:hover {
    transform: translateX(5px);
}

/* 時間軸節點 */
.log-entry::before {
    content: '';
    position: absolute;
    left: -26px;
    top: 20px;
    width: 10px;
    height: 10px;
    background: var(--bg-color);
    border: 2px solid var(--neon-blue);
    border-radius: 50%;
    box-shadow: 0 0 10px var(--neon-blue);
    z-index: 2;
}

/* 不同章節的色彩變體 */
.log-blue { border-left: 3px solid var(--neon-blue); }
.log-blue:hover { border-color: var(--neon-blue); box-shadow: 0 0 15px rgba(0, 229, 255, 0.2); }
.log-blue h4 { color: var(--neon-blue); }

.log-pink { border-left: 3px solid var(--neon-pink); }
.log-pink::before { border-color: var(--neon-pink); box-shadow: 0 0 10px var(--neon-pink); }
.log-pink:hover { border-color: var(--neon-pink); box-shadow: 0 0 15px rgba(255, 0, 255, 0.2); }
.log-pink h4 { color: var(--neon-pink); }

.log-green { border-left: 3px solid var(--neon-green); }
.log-green::before { border-color: var(--neon-green); box-shadow: 0 0 10px var(--neon-green); }
.log-green:hover { border-color: var(--neon-green); box-shadow: 0 0 15px rgba(0, 255, 102, 0.2); }
.log-green h4 { color: var(--neon-green); }

.log-orange { border-left: 3px solid var(--neon-orange); }
.log-orange::before { border-color: var(--neon-orange); box-shadow: 0 0 10px var(--neon-orange); }
.log-orange:hover { border-color: var(--neon-orange); box-shadow: 0 0 15px rgba(255, 51, 0, 0.2); }
.log-orange h4 { color: var(--neon-orange); }

.log-entry h4 {
    font-size: 1.2rem;
    margin-bottom: 10px;
    letter-spacing: 1px;
    font-family: 'Orbitron', 'Noto Sans TC', sans-serif;
}

.log-entry p {
    font-size: 0.95rem;
    line-height: 1.7;
    color: #ccc;
    text-shadow: 0 0 2px rgba(255, 255, 255, 0.1);
}

/* 結論警告框 */
.alert-box {
    margin-top: 40px;
    padding: 20px;
    border: 2px dashed var(--neon-orange);
    background: repeating-linear-gradient(
        45deg,
        rgba(255, 51, 0, 0.05),
        rgba(255, 51, 0, 0.05) 10px,
        rgba(0, 0, 0, 0) 10px,
        rgba(0, 0, 0, 0) 20px
    );
    text-align: center;
    box-shadow: inset 0 0 20px rgba(255, 51, 0, 0.2);
    position: relative;
}

.alert-box::before {
    content: '/// WARNING: SYSTEM OVERRIDE ///';
    position: absolute;
    top: -10px;
    left: 50%;
    transform: translateX(-50%);
    background: var(--bg-color);
    padding: 0 10px;
    color: var(--neon-orange);
    font-size: 0.8rem;
    font-family: 'Orbitron', sans-serif;
}

.alert-box strong {
    color: var(--neon-orange);
    font-size: 1.1rem;
    text-shadow: 0 0 8px var(--neon-orange);
}

/* --- 替換原本的 intercept-log 相關樣式 --- */
.intercept-container {
    background: #0b101a; /* 深邃的藍黑色背景 */
    border-left: 3px solid #339966;
    padding: 20px;
    margin-top: 15px;
    font-family: 'Noto Sans TC', sans-serif;
}

.intercept-header {
    color: #798eb3;
    font-size: 1.1rem;
    font-weight: bold;
    letter-spacing: 1px;
    margin-bottom: 20px;
    border-bottom: 1px dashed rgba(255, 255, 255, 0.15);
    padding-bottom: 15px;
    display: flex;
    align-items: center;
}

.intercept-header span {
    color: #5c7cb8;
    font-family: 'Orbitron', monospace;
    letter-spacing: 3px;
    margin-left: 15px;
    font-weight: normal;
    font-size: 0.9rem;
}

.chat-box {
    display: flex;
    flex-direction: column;
    gap: 15px;
}

.msg {
    max-width: 85%;
    padding: 12px 18px;
    border-radius: 8px;
    font-size: 0.95rem;
    line-height: 1.6;
}

/* 左側通訊氣泡：綠色系 (攔截對象) */
.msg-left {
    align-self: flex-start;
    background: #2a593e; 
    color: #e0e0e0;
    border-bottom-left-radius: 2px;
}

/* 右側通訊氣泡：深灰藍色 (本機/艾露) */
.msg-right {
    align-self: flex-end;
    background: #262b36; 
    color: #a0a8b5;
    border-bottom-right-radius: 2px;
}

.msg strong {
    opacity: 0.7;
    margin-right: 8px;
    font-size: 0.9rem;
}


    </style>
</head>
<body>

    <header>
        <h1>SYS.ARCHIVE // VER.3.1.4</h1>
    </header>

    <nav class="nav-container">
        <button class="tab-btn active" onclick="openTab(event, 'tab-characters')">[人物檔案]</button>
        <button class="tab-btn" onclick="openTab(event, 'tab-story')">[故事大綱]</button>
        <button class="tab-btn" onclick="openTab(event, 'tab-relations')">[角色關係圖譜]</button>
    </nav>

    <main class="content-container">
       <div id="tab-characters" class="tab-content active">
    <div class="profile-header">
        <h2>人物檔案：艾露 (Aynu / アイヌ)</h2>
        <p class="quote-box">「我守護你的傷口，不是因為憐憫，而是因為你是神威。」</p>
    </div>

    <div class="profile-grid">
        <div class="info-panel">
            <h3>基本資訊 // BASE_DATA</h3>
            <div class="data-row">
                <span class="data-label">種族</span>
                <span class="data-value">夜兔 (Yato)</span>
            </div>
            <div class="data-row">
                <span class="data-label">年齡</span>
                <span class="data-value">與神威同歲</span>
            </div>
            <div class="data-row">
                <span class="data-label">瞳孔顏色</span>
                <span class="data-value" style="color: var(--neon-green);">湖水綠</span>
            </div>
            <div class="data-row">
                <span class="data-label">標誌裝備</span>
                <span class="data-value">墨綠色傘 / 花朵髮飾</span>
            </div>
            <div class="data-row">
                <span class="data-label">髮型</span>
                <span class="data-value">粉桃色小圈圈 / 呆毛</span>
            </div>
            
            <h3 style="margin-top:20px;">生物意象 // BIOLOGY</h3>
            <div class="data-row">
                <span class="data-label">4-18 歲</span>
                <span class="data-value">垂耳兔 (依附/受創)</span>
            </div>
            <div class="data-row">
                <span class="data-label">18 歲後</span>
                <span class="data-value">半垂耳 (覺醒/警惕)</span>
            </div>
        </div>

        <div class="info-panel" style="border-color: var(--neon-blue); background: rgba(0, 229, 255, 0.05);">
            <h3>人格與能力標籤 // TAGS</h3>
            <div class="tag-cloud">
                <span class="tag tag-pink">三無少女</span>
                <span class="tag tag-pink">年下控</span>
                <span class="tag tag-blue">春雨の火</span>
                <span class="tag tag-blue">駭客</span>
                <span class="tag tag-blue">金絲雀</span>
                <span class="tag tag-orange">阿爾塔納實驗品</span>
                <span class="tag tag-orange">考古學家</span>
                <span class="tag tag-orange">殺人工具</span>
            </div>

            <h3 style="margin-top:20px; color: var(--neon-orange); border-color: var(--neon-orange);">職業/能力演化史 // EVOLUTION</h3>
            <ul class="evolution-list">
                <li class="evolution-item">
                    <span class="evo-age">4-16 歲</span> 不死不滅的殺人工具（失敗實驗品）
                </li>
                <li class="evolution-item">
                    <span class="evo-age">16-18 歲</span> 宇宙駭客
                </li>
                <li class="evolution-item">
                    <span class="evo-age">18 歲</span> 第七師團團長之『金絲雀』（機密調查員）
                </li>
                <li class="evolution-item">
                    <span class="evo-age">20 歲後</span> 宇宙考古學家（冒險導航者）
                </li>
            </ul>

            <h3 style="margin-top:20px; color: var(--neon-pink); border-color: var(--neon-pink);">生理狀態 // STATUS</h3>
            <p style="font-size: 0.9rem; line-height: 1.6;">
                曾患有味覺失靈與視障（現已恢復但有後遺症）。大腦運算過載時會陷入昏睡，體質上帶有燃燒生命、容易流鼻血的負載特性。
            </p>
        </div>
    </div>
</div>


       <div id="tab-story" class="tab-content">
    <h2>故事大綱：惡黨的浪漫與殘缺的救贖_SYNC</h2>
    
    <div class="timeline-container">
        <div class="log-entry log-blue">
            <h4>第一章：落入深淵的垂耳兔 (童年期)</h4>
            <p>艾露自幼由江華撫養，與神威是互相綁頭髮的青梅竹馬。4 歲時因買藥被「春雨第 0 團」拐帶做為阿爾塔納實驗體，留下一個髮圈後失蹤。在非人道的實驗中，她失去表情與味覺，眼睛受損，被鳳仙收作徒弟並秘密安插在春雨第七師團成為「團寵」。</p>
        </div>

        <div class="log-entry log-pink">
            <h4>第二章：重逢與無聲的依偎 (春雨時期)</h4>
            <p>神威 8 歲加入春雨，兩人於 9 歲重逢並被迫同居。從一開始的排斥到最後在同一張床上抱著入睡，兩人在拒絕感情的同時互相陪伴成長。神威對實驗不知情也因否認感情選擇不調查，只有在觀察艾露的行為，在觀察中慢慢靠近這個沉默的女孩。</p>
        </div>

        <div class="log-entry log-green">
            <h4>第三章：失去與流浪的時光（16-18歲）</h4>
            <p>16 歲時艾露因實驗暴走團滅實驗組織後，為了不連累第七師團而選擇逃亡，在黑市與陸奧結緣，忍受著失明的黑暗與恢復的痛苦，獨自調查春雨與神威的過去。神威對外維持著「不在意」，實則默默收集她的舊物，他在任務空檔獨自徘徊於各個星球，像個弄丟了寶物的瘋子。</p>
        </div>

        <div class="log-entry log-blue">
            <h4>第四章：吉原重遇與「金絲雀」的回歸</h4>
            <p>18 歲的艾露恢復視力後暗中調查神威，兩人在吉原以「遊女與嫖客」的身份重逢。戰鬥中因幼時的約定（手鍊與滿天星）相認並交換初吻。隨後艾露以『金絲雀』的偽裝回歸第七師團，擔任神威背後的機密調查員。在暗處為神威清掃春雨內部的障礙，兩人維持著危險又親密，非正式戀人的關係。（調查時間為：惡黨篇去到將軍暗殺篇）</p>
        </div>

        <div class="log-entry log-pink">
            <h4>第五章：烙陽決戰與正式告白</h4>
            <p>回到故土烙陽，艾露在江華墓前許下終身。神威在家庭的紛爭與和解中，接納了自己的脆弱與愛，並在母親墓前帥氣宣告：「艾露現在是我的女人。」 兩人正式確立關係。並在兩年期間神威偷教導神樂氣功的空檔，簡單交換戒指成婚（神樂因氣功後遺症忘記了此事）。</p>
        </div>

        <div class="log-entry log-green">
            <h4>第六章：宇宙最強與她的守護者</h4>
            <p>結局中，神威成為了海賊王在冒險，艾露則成為考古學家，為海賊團強化實力並尋找夜兔的生存星球。儘管艾露身體依然脆弱且時常昏睡，神威始終為她撐傘守護。這對「瘋子」在殘酷的宇宙中拼湊出了完整的家園，過著吵鬧卻堅定的惡黨生活。</p>
        </div>

        <div class="log-entry log-orange">
            <h4>最終章：宇宙最強家族的「和平」協議</h4>
            <p>惡黨也是要過日子的！神威船長與考古學家艾露的婚後生活，除了尋找星球，更多的是防備那個禿頭老爸的「突擊檢查」。多虧了最強外援陸奧與艾露的情報網，才讓這場家庭聚會不至於演變成星球毀滅大戰。這是一場關於「笨拙哥哥、暴力妹妹、禿頭老爹、流鼻血妻子」以及兩隻「小兔崽子」的混亂生活紀實。<br><br>這不僅是神威與父親、妹妹之間的一場漫長休戰，更是生命意志的延續雙胞胎的誕生，讓這份帶著血腥味的愛有了更溫柔的出口。</p>
        </div>
    </div>

    <div class="alert-box">
        <strong>結論：這家人的字典裡，愛就是用筷子互插對方的眼睛！</strong>
    </div>
</div>


<div id="tab-relations" class="tab-content">
    <h2>角色關係圖譜_NETWORK</h2>
    <div class="system-status">>> DECRYPTING RELATIONSHIP MATRIX... 100% SUCCESS.</div>

    <div class="comm-network">
        
        <div class="comm-node" style="border-top: 3px solid var(--neon-orange);">
            <div class="node-header">
                <div class="node-title"><span class="highlight">艾露</span> × <span class="target" style="color: var(--neon-orange);">神威</span></div>
                <div class="node-theme">【唯一的同類與命定之人】</div>
            </div>
            <div class="node-data">
                <div class="data-field">
                    <span class="field-label">RELATION_TYPE // 關係</span>
                    <span class="field-value">青梅竹馬、年上戀人、彼此的「靈魂錨點」。</span>
                </div>
                <div class="data-field">
                    <span class="field-label">CONNECTION // 羈絆</span>
                    <span class="field-value">兩人都是被家庭或實驗致殘的靈魂。對神威而言，艾露是他「人性」的最後保險絲；對艾露而言，神威是她即便燃燒生命也要守護的唯一。</span>
                </div>
                <div class="data-field">
                    <span class="field-label">PATTERN // 相處模式</span>
                    <span class="field-value">表面清冷，實則極具佔有慾。神威會一邊嫌棄艾露虛弱，一邊為她撐傘。</span>
                </div>
            </div>
            
            <div class="intercept-container">
                <div class="intercept-header">通訊頻道截取 <span>SIGNAL INTERCEPT</span></div>
                <div class="chat-box">
                    <div class="msg msg-left"><strong>神威</strong>「妳知道嗎？如果妳在那些實驗裡死掉，我就真的找不到理由不把這宇宙毀掉了呢～」</div>
                    <div class="msg msg-right"><strong>艾露</strong>「所以我活下來了。。。因為你是神威，是我即便燃燒生命，也要在廢墟中拽住的混蛋丈夫。。。」</div>
                </div>
            </div>
        </div>

        <div class="comm-node" style="border-top: 3px solid #ff6666;">
            <div class="node-header">
                <div class="node-title"><span class="highlight">艾露</span> × <span class="target" style="color: #ff6666;">神樂</span></div>
                <div class="node-theme">【溫柔的長姐與未來的小姑】</div>
            </div>
            <div class="node-data">
                <div class="data-field">
                    <span class="field-label">RELATION_TYPE // 關係</span>
                    <span class="field-value">曾共同生活的「姐姐」、救命恩人的遺志繼承者。</span>
                </div>
                <div class="data-field">
                    <span class="field-label">CONNECTION // 羈絆</span>
                    <span class="field-value">艾露在童年時期曾協助江華和神威照顧神樂。兩年期間神樂並沒能找到把定春救回來的方法，卻習得控制身體大小的技能，神威和艾露曾暗中給予指引。神樂對艾露有種天然的親近感，雖然會吐槽哥哥「找了一個比他還怪的女人」，但心底非常依賴艾露。</span>
                </div>
            </div>
            <div class="intercept-container">
                <div class="intercept-header">通訊頻道截取 <span>SIGNAL INTERCEPT</span></div>
                <div class="chat-box">
                    <div class="msg msg-left"><strong>神樂</strong>「艾露姐，妳為什麼要跟著那個笨蛋哥哥？他連飯都不會分給妳吃阿魯！」</div>
                    <div class="msg msg-right"><strong>艾露</strong>「沒關係，因為妳哥哥他所有的米飯都是我親手盛的。看著你們兩個搶食的樣子，會讓我覺得。。。江華媽媽留下的那個家，還在運轉。」</div>
                </div>
            </div>
        </div>

        <div class="comm-node" style="border-top: 3px solid var(--neon-green);">
            <div class="node-header">
                <div class="node-title"><span class="highlight">艾露</span> × <span class="target" style="color: var(--neon-green);">江華</span></div>
                <div class="node-theme" style="border-color: var(--neon-pink); color: var(--neon-pink);">【救贖的起點與精神母親】</div>
            </div>
            <div class="node-data">
                <div class="data-field">
                    <span class="field-label">RELATION_TYPE // 關係</span>
                    <span class="field-value">養母女、信仰對象。</span>
                </div>
                <div class="data-field">
                    <span class="field-label">CONNECTION // 羈絆</span>
                    <span class="field-value">江華是給予艾露第二次生命的人。艾露所有的溫柔（如編辮子）都源於對江華的教育。她在江華墓前許下婚約，是向這位「母親」證明，她和神威終究沒有走向徹底的毀滅。</span>
                </div>
            </div>
            <div class="intercept-container">
                <div class="intercept-header">殘存記憶回放 <span>MEMORY ARCHIVE.LOG</span></div>
                <div class="chat-box">
                    <div class="msg msg-right"><strong>艾露</strong>「媽，對不起，妳教的溫柔都用在那個最不聽話的孩子身上了，但我還記得妳編辮子時，手心的溫度。」</div>
                    <div class="msg msg-left"><strong>江華</strong>「艾露，如果這世界太過冰冷，妳就學著生火。無須燒毀別人，只需在寒冷的時候，能讓那個孩子找到回家的煙火。」</div>
                </div>
            </div>
        </div>

        <div class="comm-node" style="border-top: 3px solid #999;">
            <div class="node-header">
                <div class="node-title"><span class="highlight">艾露</span> × <span class="target" style="color: #999;">星海坊主</span></div>
                <div class="node-theme" style="border-color: #999; color: #999;">【笨拙的和解與重建】</div>
            </div>
            <div class="node-data">
                <div class="data-field">
                    <span class="field-label">RELATION_TYPE // 關係</span>
                    <span class="field-value">「另一個女兒」般的認可。</span>
                </div>
                <div class="data-field">
                    <span class="field-label">CONNECTION // 羈絆</span>
                    <span class="field-value">雙胞胎的出生是關係的催化劑，達成默契。艾露不擅長社交，在家庭聚會中有時會因為太過冷淡而讓氣氛尷尬，或在照顧孩子時顯得笨拙。她負責幫老頭子在神威面前爭取「爺爺」的探視權，分配星海坊主與孫子女相處的時間雖然嘴上說著「這是為了情報交換」，實則是在幫這對父子修補那條斷裂了十幾年的親情線。</span>
                </div>
            </div>
            <div class="intercept-container">
                <div class="intercept-header">通訊頻道截取 <span>SIGNAL INTERCEPT</span></div>
                <div class="chat-box">
                    <div class="msg msg-left"><strong>星海坊主</strong>「。。。神威現在只聽妳的話。雖然我不想承認，但看在妳流著鼻血也要攔在他面前的份上，我這次就不拔他的辮子了。」</div>
                    <div class="msg msg-right"><strong>艾露</strong>「老頭子。。。筷子拿穩一點，不要在雙胞胎面前展現你那笨拙的性格。想要當個合格的祖父，就先學會安靜地坐在角落撐傘。」</div>
                </div>
            </div>
        </div>

        <div class="comm-node" style="border-top: 3px solid var(--neon-blue);">
            <div class="node-header">
                <div class="node-title"><span class="highlight">艾露</span> × <span class="target" style="color: var(--neon-blue);">陸奧</span></div>
                <div class="node-theme" style="border-color: var(--neon-blue); color: var(--neon-blue);">【事業合夥人與靈魂摯友】</div>
            </div>
            <div class="node-data">
                <div class="data-field">
                    <span class="field-label">RELATION_TYPE // 關係</span>
                    <span class="field-value">救命恩人、互利互惠的情報盟友。</span>
                </div>
                <div class="data-field">
                    <span class="field-label">CONNECTION // 羈絆</span>
                    <span class="field-value">在艾露最黑暗的流浪期（16-18歲），是陸奧在黑市救起她。兩人同為輔佐「宇宙最強『蠢』男人」的女性，艾露社交笨拙，但陸奧會體諒她，有著極高的默契。陸奧負責傳遞星海坊主的動向，艾露負責回饋各方面的情報。</span>
                </div>
            </div>
            <div class="intercept-container">
                <div class="intercept-header">通訊頻道截取 <span>SIGNAL INTERCEPT</span></div>
                <div class="chat-box">
                    <div class="msg msg-left"><strong>陸奧</strong>「妳那個男人又在給我的生意找麻煩了。艾露，我救妳回來，可無意讓妳去給『笨蛋』當保姆的。」</div>
                    <div class="msg msg-right"><strong>艾露</strong>「我知道。但我是在做考古工作啊」</div>
                    <div class="msg msg-right"><strong>艾露</strong>「還有呢，陸奧。。。妳也是在做『笨蛋』的保姆呢。」</div>
                </div>
            </div>
        </div>

        <div class="comm-node" style="border-top: 3px solid #cc9966;">
            <div class="node-header">
                <div class="node-title"><span class="highlight">艾露</span> × <span class="target" style="color: #cc9966;">阿伏兔</span></div>
                <div class="node-theme" style="border-color: #cc9966; color: #cc9966;">【可靠的戰友與「勞碌命」同盟】</div>
            </div>
            <div class="node-data">
                <div class="data-field">
                    <span class="field-label">RELATION_TYPE // 關係</span>
                    <span class="field-value">副官與團長夫人（機密員）。</span>
                </div>
                <div class="data-field">
                    <span class="field-label">CONNECTION // 羈絆</span>
                    <span class="field-value">阿伏兔是少數知道艾露身體狀況的人。他常感嘆：「這對夫妻真是要把老頭子我最後的頭髮也磨光啊。」在神威暴走時，阿伏兔會默契地退後，讓艾露去「降溫」。</span>
                </div>
            </div>
            <div class="intercept-container">
                <div class="intercept-header">通訊頻道截取 <span>SIGNAL INTERCEPT</span></div>
                <div class="chat-box">
                    <div class="msg msg-left"><strong>阿伏兔</strong>「喂喂，夫人，妳看著點那瘋小子啊！我這把老骨頭可禁不起他天天發情。。。喔不，發瘋。」</div>
                    <div class="msg msg-right"><strong>艾露</strong>「阿伏兔。。。既然你覺得辛苦，那下次他『瘋』的時候，就把他的雨傘換成我的造型夾吧。」</div>
                    <div class="msg msg-right"><strong>艾露</strong>「你看著船隊。。。我看著神威。」</div>
                </div>
            </div>
        </div>

    </div>
</div>


    </main>

    <script>
        // 控制分頁切換的 JavaScript 函數
        function openTab(evt, tabId) {
            // 取得所有分頁內容並隱藏
            let tabContents = document.getElementsByClassName("tab-content");
            for (let i = 0; i < tabContents.length; i++) {
                tabContents[i].classList.remove("active");
            }

            // 取得所有分頁按鈕並移除 active 狀態
            let tabBtns = document.getElementsByClassName("tab-btn");
            for (let i = 0; i < tabBtns.length; i++) {
                tabBtns[i].classList.remove("active");
            }

            // 顯示當前點擊的分頁，並將點擊的按鈕設為 active
            document.getElementById(tabId).classList.add("active");
            evt.currentTarget.classList.add("active");
        }
    </script>

</body>
</html>
