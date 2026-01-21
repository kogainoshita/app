[index.html](https://github.com/user-attachments/files/24755774/index.html)
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>世界国旗マスター：ランダム20問</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1c:wght@400;700;900&display=swap');
        
        body {
            font-family: 'M PLUS Rounded 1c', sans-serif;
            background: #0f172a;
            color: white;
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            margin: 0;
            padding: 20px;
        }

        .main-card {
            background: #1e293b;
            border: 2px solid #334155;
            border-radius: 2.5rem;
            width: 100%;
            max-width: 600px;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
            position: relative;
            overflow: hidden;
        }

        .flag-box {
            background: #ffffff;
            border-radius: 1.5rem;
            height: 220px;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 1rem;
            margin-bottom: 2rem;
            box-shadow: inset 0 2px 4px 0 rgba(0, 0, 0, 0.06);
        }

        .flag-img {
            max-width: 100%;
            max-height: 100%;
            object-fit: contain;
            filter: drop-shadow(0 4px 6px rgba(0,0,0,0.1));
        }

        .option-btn {
            background: #334155;
            border: 2px solid #475569;
            transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
        }

        .option-btn:not(:disabled):hover {
            background: #4f46e5;
            border-color: #818cf8;
            transform: translateY(-2px);
        }

        .option-btn:disabled {
            opacity: 0.8;
            cursor: default;
        }

        .difficulty-badge {
            font-size: 0.75rem;
            padding: 0.25rem 0.75rem;
            border-radius: 9999px;
            text-transform: uppercase;
            font-weight: 900;
        }

        .diff-easy { background: #10b981; color: white; }
        .diff-medium { background: #f59e0b; color: white; }
        .diff-hard { background: #ef4444; color: white; }

        .progress-bar {
            height: 12px;
            background: #334155;
            border-radius: 6px;
            overflow: hidden;
        }

        .progress-fill {
            height: 100%;
            background: linear-gradient(90deg, #6366f1, #a855f7, #ec4899);
            transition: width 0.4s ease;
        }

        .correct { background: #059669 !important; border-color: #10b981 !important; color: white !important; }
        .wrong { background: #dc2626 !important; border-color: #ef4444 !important; color: white !important; }
    </style>
</head>
<body>

<div class="main-card p-6 md:p-10">
    <!-- UI ヘッダー -->
    <div class="mb-6">
        <div class="flex justify-between items-center mb-4">
            <div>
                <span id="difficulty-label" class="difficulty-badge">Loading</span>
                <h2 class="text-xl font-bold mt-1">Level <span id="current-level">1</span>/20</h2>
            </div>
            <div class="text-right">
                <p class="text-sm text-slate-400">Score</p>
                <p id="score-display" class="text-2xl font-black text-indigo-400">0</p>
            </div>
        </div>
        <div class="progress-bar">
            <div id="progress-fill" class="progress-fill" style="width: 5%"></div>
        </div>
    </div>

    <!-- クイズ表示エリア -->
    <div id="quiz-area">
        <div class="flag-box">
            <img id="flag-img" class="flag-img" src="" alt="Flag">
        </div>
        <div id="options-grid" class="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <!-- ボタンがここに生成される -->
        </div>
    </div>

    <!-- 結果表示エリア -->
    <div id="result-area" class="hidden text-center py-8">
        <div class="text-7xl mb-6">🏆</div>
        <h2 class="text-4xl font-black mb-2">チャレンジ終了！</h2>
        <p class="text-slate-400 mb-8 font-medium text-lg">あなたのスコア</p>
        <div class="text-6xl font-black text-white mb-10">
            <span id="final-score" class="text-indigo-500">0</span> <span class="text-3xl text-slate-500">/ 20</span>
        </div>
        <div id="rank-message" class="mb-10 text-xl font-bold text-indigo-300"></div>
        <button onclick="location.reload()" class="w-full bg-indigo-600 hover:bg-indigo-500 text-white font-black py-5 rounded-2xl transition-all shadow-xl shadow-indigo-500/20 active:scale-95">
            新しい問題で再挑戦
        </button>
    </div>
</div>

<script>
    // 問題プール（難易度別に分類）
    const questionPool = {
        easy: [
            { code: "jp", name: "日本", options: ["日本", "韓国", "中国", "台湾"] },
            { code: "us", name: "アメリカ", options: ["アメリカ", "イギリス", "リベリア", "カナダ"] },
            { code: "fr", name: "フランス", options: ["フランス", "ロシア", "オランダ", "ルクセンブルク"] },
            { code: "it", name: "イタリア", options: ["イタリア", "ハンガリー", "アイルランド", "メキシコ"] },
            { code: "gb", name: "イギリス", options: ["イギリス", "オーストラリア", "ニュージーランド", "フィジー"] },
            { code: "ca", name: "カナダ", options: ["カナダ", "アメリカ", "デンマーク", "スイス"] },
            { code: "br", name: "ブラジル", options: ["ブラジル", "アルゼンチン", "ジャマイカ", "コロンビア"] },
            { code: "cn", name: "中国", options: ["中国", "ベトナム", "モンゴル", "トルコ"] },
            { code: "au", name: "オーストラリア", options: ["オーストラリア", "イギリス", "ニュージーランド", "ツバル"] },
            { code: "kr", name: "韓国", options: ["韓国", "日本", "北朝鮮", "パラオ"] }
        ],
        medium: [
            { code: "de", name: "ドイツ", options: ["ドイツ", "ベルギー", "リトアニア", "オランダ"] },
            { code: "in", name: "インド", options: ["インド", "ニジェール", "アイルランド", "コートジボワール"] },
            { code: "ar", name: "アルゼンチン", options: ["アルゼンチン", "ウルグアイ", "ギリシャ", "グアテマラ"] },
            { code: "eg", name: "エジプト", options: ["エジプト", "イラク", "シリア", "ヨルダン"] },
            { code: "se", name: "スウェーデン", options: ["スウェーデン", "フィンランド", "ノルウェー", "デンマーク"] },
            { code: "mx", name: "メキシコ", options: ["メキシコ", "イタリア", "セネガル", "ポルトガル"] },
            { code: "th", name: "タイ", options: ["タイ", "コスタリカ", "北朝鮮", "オランダ"] },
            { code: "gr", name: "ギリシャ", options: ["ギリシャ", "ウルグアイ", "フィンランド", "イスラエル"] },
            { code: "tr", name: "トルコ", options: ["トルコ", "チュニジア", "パキスタン", "アゼルバイジャン"] },
            { code: "es", name: "スペイン", options: ["スペイン", "ポルトガル", "アンドラ", "モルドバ"] }
        ],
        hard: [
            { code: "mc", name: "モナコ", options: ["モナコ", "インドネシア", "ポーランド", "シンガポール"] },
            { code: "ro", name: "ルーマニア", options: ["ルーマニア", "チャド", "モルドバ", "アンドラ"] },
            { code: "bt", name: "ブータン", options: ["ブータン", "スリランカ", "ウェールズ", "アルバニア"] },
            { code: "kz", name: "カザフスタン", options: ["カザフスタン", "ウクライナ", "パラオ", "キルギス"] },
            { code: "sz", name: "エスワティニ", options: ["エスワティニ", "南アフリカ", "レソト", "エリトリア"] },
            { code: "vu", name: "バヌアツ", options: ["バヌアツ", "ソロモン諸島", "パプアニューギニア", "フィジー"] },
            { code: "dj", name: "ジブチ", options: ["ジブチ", "エリトリア", "ソマリア", "エチオピア"] },
            { code: "kg", name: "キルギス", options: ["キルギス", "カザフスタン", "モンゴル", "マケドニア"] },
            { code: "ad", name: "アンドラ", options: ["アンドラ", "モルドバ", "ルーマニア", "チャド"] },
            { code: "ml", name: "マリ", options: ["マリ", "ギニア", "セネガル", "カメルーン"] }
        ]
    };

    let quizSet = [];
    let currentIdx = 0;
    let score = 0;
    let canAnswer = true;

    // 配列をシャッフルするユーティリティ
    function shuffle(array) {
        for (let i = array.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [array[i], array[j]] = [array[j], array[i]];
        }
        return array;
    }

    // クイズセットの作成 (Easy 7問, Medium 7問, Hard 6問 = 計20問)
    function createQuizSet() {
        const easy = shuffle([...questionPool.easy]).slice(0, 7).map(q => ({...q, diff: 'Easy'}));
        const medium = shuffle([...questionPool.medium]).slice(0, 7).map(q => ({...q, diff: 'Medium'}));
        const hard = shuffle([...questionPool.hard]).slice(0, 6).map(q => ({...q, diff: 'Hard'}));
        quizSet = [...easy, ...medium, ...hard];
    }

    const flagImg = document.getElementById('flag-img');
    const optionsGrid = document.getElementById('options-grid');
    const levelText = document.getElementById('current-level');
    const scoreDisplay = document.getElementById('score-display');
    const progressFill = document.getElementById('progress-fill');
    const diffLabel = document.getElementById('difficulty-label');

    function loadQuestion() {
        canAnswer = true;
        const q = quizSet[currentIdx];
        
        levelText.textContent = currentIdx + 1;
        progressFill.style.width = `${((currentIdx + 1) / quizSet.length) * 100}%`;
        diffLabel.textContent = q.diff;
        diffLabel.className = `difficulty-badge diff-${q.diff.toLowerCase()}`;

        // FlagCDN から画像を読み込み
        flagImg.src = `https://flagcdn.com/w640/${q.code}.png`;

        optionsGrid.innerHTML = '';
        const shuffledOptions = shuffle([...q.options]);
        
        shuffledOptions.forEach(option => {
            const btn = document.createElement('button');
            btn.className = 'option-btn p-4 rounded-xl font-bold text-lg text-white';
            btn.textContent = option;
            btn.onclick = () => checkAnswer(option, btn);
            optionsGrid.appendChild(btn);
        });
    }

    function checkAnswer(selected, btn) {
        if (!canAnswer) return;
        canAnswer = false;

        const correct = quizSet[currentIdx].name;
        const allButtons = optionsGrid.querySelectorAll('button');

        if (selected === correct) {
            score++;
            scoreDisplay.textContent = score;
            btn.classList.add('correct');
        } else {
            btn.classList.add('wrong');
            allButtons.forEach(b => {
                if (b.textContent === correct) b.classList.add('correct');
            });
        }

        setTimeout(() => {
            currentIdx++;
            if (currentIdx < quizSet.length) {
                loadQuestion();
            } else {
                showResults();
            }
        }, 1200);
    }

    function showResults() {
        document.getElementById('quiz-area').classList.add('hidden');
        document.getElementById('result-area').classList.remove('hidden');
        document.getElementById('final-score').textContent = score;

        const rankMsg = document.getElementById('rank-message');
        if (score === 20) rankMsg.textContent = "完璧です！世界最高ランクの知識です！";
        else if (score >= 15) rankMsg.textContent = "素晴らしい！国旗博士まであと一歩！";
        else if (score >= 10) rankMsg.textContent = "平均以上の知識です！さらに磨きましょう！";
        else rankMsg.textContent = "再挑戦して、知識を深めていきましょう！";
    }

    // 初期化
    window.onload = () => {
        createQuizSet();
        loadQuestion();
    };
</script>

</body>
</html>
