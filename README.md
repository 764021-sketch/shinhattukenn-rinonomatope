<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>深発見アプリ（臨書編）</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <style>
        :root {
            --bg-main: #e3f2fd; 
            --bg-card: #ffffff;
            --bg-intuition: #fce4ec; 
            --bg-brush: #e8f5e9;     
            --bg-shape: #fffde7;     
            --accent-blue: #1a73e8;
            --accent-red: #d93025;
            --text-dark: #202124;
            --border-color: #dadce0;
        }

        body {
            font-family: "Helvetica Neue", Arial, "Hiragino Kaku Gothic ProN", "Hiragino Sans", Meiryo, sans-serif;
            background-color: #f0f2f5;
            color: var(--text-dark);
            margin: 0;
            padding: 20px;
            line-height: 1.4;
        }

        .container { max-width: 1400px; margin: 0 auto; }

        /* マニュアル表示エリア */
        .manual-area {
            background: #ffffff;
            padding: 20px;
            border-radius: 12px;
            margin-bottom: 20px;
            border: 1px solid var(--border-color);
            font-size: 14px;
        }
        .manual-title { font-weight: bold; color: var(--accent-blue); font-size: 16px; margin-bottom: 10px; display: block; }
        .manual-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        .manual-col { background: #f8f9fa; padding: 15px; border-radius: 8px; }

        /* 教師用ツールエリア */
        .utility-area {
            background: #fff9c4;
            padding: 15px;
            border-radius: 12px;
            margin-bottom: 20px;
            border: 2px solid #fbc02d;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        .utility-header { display: flex; justify-content: space-between; align-items: center; width: 100%; }

        /* 一時保存・復元エリア（生徒用） */
        .save-load-area {
            background: #e1f5fe;
            padding: 10px 20px;
            border-radius: 50px;
            margin-bottom: 15px;
            display: flex;
            gap: 10px;
            align-items: center;
            border: 1px solid #03a9f4;
        }

        .btn-sm {
            padding: 5px 15px;
            border-radius: 20px;
            border: none;
            cursor: pointer;
            font-weight: bold;
            font-size: 14px;
        }
        .btn-save { background: #03a9f4; color: white; }
        .btn-load { background: #4caf50; color: white; }
        .btn-export { background: #fbc02d; color: #333; }

        #report-area {
            background-color: var(--bg-main);
            padding: 40px 50px;
            border-radius: 10px;
            border: 1px solid #bbdefb;
            min-width: 1200px;
            box-sizing: border-box;
            position: relative;
        }

        header { text-align: center; margin-bottom: 25px; }
        header h1 { margin: 0 0 15px 0; color: #1565c0; font-size: 3rem; }

        .info-inputs { display: flex; justify-content: center; gap: 30px; }
        
        .capture-text-replacement {
            display: none; background: white; border: 1px solid #ccc;
            border-radius: 8px; padding: 10px 15px; font-size: 24px;
            font-weight: bold; min-height: 1.2em; text-align: left;
            white-space: pre-wrap; word-break: break-all;
        }

        .info-inputs input {
            padding: 10px 15px; border: 1px solid #ccc; border-radius: 8px;
            font-size: 24px; font-weight: bold; background: white; box-sizing: border-box;
        }

        .input-date { width: 250px; }
        .input-unit { width: 350px; }
        .input-name { width: 250px; }

        .main-layout { display: flex; flex-direction: column; gap: 20px; }
        .top-row { display: grid; grid-template-columns: 1.5fr 1.5fr 1.2fr; gap: 20px; }

        .image-card {
            background: var(--bg-card); border-radius: 15px; padding: 20px;
            position: relative; box-shadow: 0 4px 8px rgba(0,0,0,0.1); border: 4px solid transparent;
            cursor: pointer;
        }
        .image-card.active { border-color: var(--accent-blue); }

        .image-preview {
            width: 100%; height: 400px; background: #f1f3f4; border-radius: 8px;
            display: flex; align-items: center; justify-content: center; overflow: hidden; margin-top: 10px;
        }
        .image-preview img { max-width: 100%; max-height: 100%; object-fit: contain; }

        .section-box { padding: 15px; border-radius: 12px; border: 1px solid rgba(0,0,0,0.1); }
        .section-title { font-weight: bold; font-size: 24px; margin-bottom: 8px; color: #1a237e; display: block; }

        .word-card {
            display: inline-block; padding: 6px 14px; margin: 3px; background: white;
            border: 1px solid #ccc; border-radius: 10px; font-size: 19px; cursor: pointer;
            user-select: none;
        }
        .word-card.selected { background: var(--accent-blue); color: white; border-color: var(--accent-blue); }

        .bottom-row { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        textarea {
            width: 100%; padding: 20px; border: 2px solid #ccc; border-radius: 12px;
            resize: none; font-size: 26px; font-weight: bold; height: 250px;
            box-sizing: border-box; line-height: 1.4; font-family: inherit;
        }

        .textarea-replacement {
            display: none; width: 100%; min-height: 250px; padding: 20px;
            border: 2px solid #ccc; border-radius: 12px; background: white;
            font-size: 26px; font-weight: bold; line-height: 1.4; box-sizing: border-box; white-space: pre-wrap;
        }

        .controls { margin-top: 30px; text-align: center; padding-bottom: 50px; }
        .btn { padding: 20px 50px; border-radius: 50px; font-size: 1.5rem; font-weight: bold; cursor: pointer; border: none; }
        .btn-primary { background: #ff9800; color: white; }
        .btn:disabled { background: #ccc; cursor: not-allowed; }

        .btn-delete {
            position: absolute; top: 5px; right: 5px; background: var(--accent-red);
            color: white; border: none; width: 35px; height: 35px; border-radius: 50%;
            cursor: pointer; display: none; font-size: 20px; z-index: 10;
        }
        .has-image .btn-delete { display: block; }
        .hidden { display: none !important; }

        .capturing input, .capturing textarea { display: none !important; }
        .capturing .capture-text-replacement, .capturing .textarea-replacement { display: block !important; }
    </style>
</head>
<body>

<div class="container">
    <div class="manual-area" id="app-manual">
        <span class="manual-title">📘 アプリ活用マニュアル</span>
        <div class="manual-grid">
            <div class="manual-col">
                <strong>👨‍🏫 教師用：お手本埋め込み手順</strong>
                <ol>
                    <li>「古典（お手本）」枠から画像を読み込む。</li>
                    <li>下の黄色いエリアのボタンから「配布用ファイルを保存」する。</li>
                    <li>保存されたファイルは、開いた瞬間にこのツール類が隠れた状態で始まります。</li>
                </ol>
            </div>
            <div class="manual-col">
                <strong>🎓 生徒用：学習の手順</strong>
                <ul>
                    <li><strong>一時保存：</strong>「今の状態を保存」し、次回「続きから再開」できます。</li>
                    <li><strong>提出：</strong>全て入力すると「画像として保存」が押せるようになります。</li>
                </ul>
            </div>
        </div>
        <div style="text-align: right; margin-top: 10px;">
            <button onclick="document.getElementById('app-manual').classList.add('hidden')" style="font-size: 12px; cursor: pointer;">マニュアルを閉じる</button>
        </div>
    </div>

    <div class="utility-area" id="teacher-tools">
        <div class="utility-header">
            <strong>🎓 教科担任用：お手本配布準備</strong>
            <button onclick="toggleTeacherMode()">設定完了（ツールを隠す）</button>
        </div>
        <div style="background: rgba(0,0,0,0.05); padding: 10px; border-radius: 5px; font-size: 13px;">
            <p style="margin:0 0 5px 0;"><strong>配布用ファイルの作成：</strong></p>
            <p style="margin:0 0 10px 0;">お手本をセットして下のボタンを押すと、<strong>「このマニュアルや教師ツールが最初から隠れた状態」</strong>の生徒用ファイルが書き出されます。</p>
            <button class="btn-sm btn-export" onclick="exportAppWithImage()">お手本を固定して配布用ファイルを保存</button>
        </div>
    </div>

    <div class="save-load-area" id="student-save-area">
        <span>💾 <strong>作業の保存：</strong></span>
        <button class="btn-sm btn-save" onclick="saveToLocal()">今の状態を保存する</button>
        <button class="btn-sm btn-load" onclick="loadFromLocal()">前回の続きから再開する</button>
        <small>※同じPC・ブラウザで有効</small>
    </div>

    <div id="report-area">
        <header>
            <h1>深発見アプリ（臨書編）</h1>
            <div class="info-inputs">
                <div class="input-container">
                    <input type="date" id="doc-date" class="input-date">
                    <div id="replace-date" class="capture-text-replacement input-date"></div>
                </div>
                <div class="input-container">
                    <input type="text" id="doc-unit" class="input-unit" placeholder="単元名">
                    <div id="replace-unit" class="capture-text-replacement input-unit"></div>
                </div>
                <div class="input-container">
                    <input type="text" id="doc-name" class="input-name" placeholder="氏名">
                    <div id="replace-name" class="capture-text-replacement input-name"></div>
                </div>
            </div>
        </header>

        <div class="main-layout">
            <div class="top-row">
                <div class="image-card" id="card-koten" onclick="setActiveCard('koten')">
                    <button class="btn-delete" onclick="removeImage(event, 'koten')">×</button>
                    <span class="section-title">古典（お手本）</span>
                    <div class="image-preview" id="preview-koten"><span>画像を選択</span></div>
                    <div class="no-capture-ui" id="ui-koten" style="text-align:center; margin-top:10px;">
                        <input type="file" id="input-koten" accept="image/*" style="display:none">
                        <button class="word-card" style="color:var(--accent-blue);" onclick="event.stopPropagation(); document.getElementById('input-koten').click()">画像を選ぶ</button>
                    </div>
                </div>

                <div class="image-card" id="card-self" onclick="setActiveCard('self')">
                    <button class="btn-delete" onclick="removeImage(event, 'self')">×</button>
                    <span class="section-title">自分の作品</span>
                    <div class="image-preview" id="preview-self"><span>枠を選択して貼り付け</span></div>
                    <div class="no-capture-ui" style="text-align:center; margin-top:10px;">
                        <input type="file" id="input-self" accept="image/*" style="display:none">
                        <button class="word-card" onclick="event.stopPropagation(); document.getElementById('input-self').click()">画像を選ぶ</button>
                    </div>
                </div>

                <div class="analysis-stack" style="display:flex; flex-direction:column; gap:10px;">
                    <div class="section-box" style="background-color: var(--bg-intuition);">
                        <span class="section-title">直感的鑑賞（オノマトペ）</span>
                        <div id="words-intuition">
                            <!-- JSで生成 -->
                        </div>
                    </div>
                    <div class="section-box" style="background-color: var(--bg-brush);">
                        <span class="section-title">分析的鑑賞（筆遣い）</span>
                        <div id="words-brush">
                            <!-- JSで生成 -->
                        </div>
                    </div>
                    <div class="section-box" style="background-color: var(--bg-shape);">
                        <span class="section-title">分析的鑑賞（字形）</span>
                        <div id="words-shape">
                            <!-- JSで生成 -->
                        </div>
                    </div>
                </div>
            </div>

            <div class="bottom-row">
                <div class="section-box" style="background: white;">
                    <span class="section-title">発見したこと</span>
                    <textarea id="text-analysis" placeholder="具体的に書きましょう。"></textarea>
                    <div id="replace-analysis" class="textarea-replacement"></div>
                </div>
                <div class="section-box" style="background: white; border: 3px solid var(--accent-blue);">
                    <span class="section-title">振り返り</span>
                    <textarea id="text-self-eval" placeholder="次はどう書きたいですか。"></textarea>
                    <div id="replace-self-eval" class="textarea-replacement"></div>
                </div>
            </div>
        </div>
    </div>

    <div class="controls">
        <button id="btn-download" class="btn btn-primary" disabled>📸 画像として保存して提出</button>
    </div>
</div>

<script>
    // --- 埋め込みデータ用変数 ---
    let embeddedModelImage = null; 
    let isDistributionMode = false; 

    const onomatopoeias = ["くねくね", "どんどん", "すかすか", "すらっと", "へなへな", "ひらひら", "ぽかぽか", "すらすら", "きりっと", "ゆらゆら", "がんがん", "あっさり", "ふわっと", "びしっと", "まるまる", "かるがる", "とげとげ", "ギザギザ"];
    const brushWords = ["太い", "細い", "直線的", "曲線的", "角ばった", "丸みのある", "露鋒", "蔵鋒"];
    const shapeWords = ["縦長", "横長", "背勢", "向勢"];

    let activeCard = 'self';
    let imageData = { koten: null, self: null };

    window.onload = () => {
        // 重複防止：生成前に一旦コンテナを空にする
        document.getElementById('words-intuition').innerHTML = '';
        document.getElementById('words-brush').innerHTML = '';
        document.getElementById('words-shape').innerHTML = '';

        // ワードカードの生成
        renderWords('words-intuition', onomatopoeias, 'intuition');
        renderWords('words-brush', brushWords, 'brush');
        renderWords('words-shape', shapeWords, 'shape');

        document.getElementById('doc-date').value = new Date().toISOString().split('T')[0];

        // 配布モード（お手本埋め込み済み）の場合の処理
        if (embeddedModelImage) {
            updateImageDisplay('koten', embeddedModelImage);
            document.getElementById('ui-koten').classList.add('hidden');
            document.getElementById('card-koten').querySelector('.btn-delete').classList.add('hidden');
        }

        if (isDistributionMode) {
            document.getElementById('teacher-tools').classList.add('hidden');
            document.getElementById('app-manual').classList.add('hidden');
        }
    };

    function renderWords(containerId, words, category) {
        const container = document.getElementById(containerId);
        words.forEach(w => {
            const div = document.createElement('div');
            div.className = 'word-card';
            div.innerText = w;
            div.dataset.category = category;
            div.onclick = () => {
                div.classList.toggle('selected');
                checkValidation();
            };
            container.appendChild(div);
        });
    }

    function updateImageDisplay(type, src) {
        imageData[type] = src;
        document.getElementById(`preview-${type}`).innerHTML = `<img src="${src}">`;
        document.getElementById(`card-${type}`).classList.add('has-image');
        checkValidation();
    }

    document.getElementById('input-koten').addEventListener('change', e => {
        const file = e.target.files[0];
        if (file) {
            const reader = new FileReader();
            reader.onload = f => updateImageDisplay('koten', f.target.result);
            reader.readAsDataURL(file);
        }
    });

    document.getElementById('input-self').addEventListener('change', e => {
        const file = e.target.files[0];
        if (file) {
            const reader = new FileReader();
            reader.onload = f => updateImageDisplay('self', f.target.result);
            reader.readAsDataURL(file);
        }
    });

    function toggleTeacherMode() {
        if(confirm("教科担任用ツールを隠しますか？")) document.getElementById('teacher-tools').classList.add('hidden');
    }

    function setActiveCard(type) {
        activeCard = type;
        document.getElementById('card-koten').classList.toggle('active', type==='koten');
        document.getElementById('card-self').classList.toggle('active', type==='self');
    }

    function removeImage(e, type) {
        e.stopPropagation();
        imageData[type] = null;
        document.getElementById(`preview-${type}`).innerHTML = `<span>画像を選択</span>`;
        document.getElementById(`card-${type}`).classList.remove('has-image');
        checkValidation();
    }

    const fields = ['doc-unit', 'doc-name', 'text-analysis', 'text-self-eval'];
    fields.forEach(id => document.getElementById(id).addEventListener('input', checkValidation));

    function checkValidation() {
        const filled = fields.every(id => document.getElementById(id).value.trim() !== "");
        const images = imageData.koten && imageData.self;
        const selectedCards = document.querySelectorAll('.word-card.selected').length > 0;
        document.getElementById('btn-download').disabled = !(filled && images && selectedCards);
    }

    function exportAppWithImage() {
        if (!imageData.koten) {
            alert("まず「古典（お手本）」に画像をセットしてください。");
            return;
        }

        // 保存前にJSで生成されたワードカードを一旦削除して、HTMLが汚れるのを防ぐ
        document.getElementById('words-intuition').innerHTML = '';
        document.getElementById('words-brush').innerHTML = '';
        document.getElementById('words-shape').innerHTML = '';
        
        let currentHtml = document.documentElement.outerHTML;
        
        // データの埋め込み
        let newHtml = currentHtml.replace(
            "let embeddedModelImage = null;", 
            `let embeddedModelImage = "${imageData.koten}";`
        );
        newHtml = newHtml.replace(
            "let isDistributionMode = false;", 
            "let isDistributionMode = true;"
        );
        
        const blob = new Blob([newHtml], { type: "text/html" });
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = "shodo_learning_distributed.html";
        link.click();
        
        // 元の画面に戻すために再生成
        renderWords('words-intuition', onomatopoeias, 'intuition');
        renderWords('words-brush', brushWords, 'brush');
        renderWords('words-shape', shapeWords, 'shape');

        alert("配布用ファイルを作成しました。");
    }

    function saveToLocal() {
        const selectedWords = [];
        document.querySelectorAll('.word-card.selected').forEach(card => selectedWords.push(card.innerText));
        const data = {
            date: document.getElementById('doc-date').value,
            unit: document.getElementById('doc-unit').value,
            name: document.getElementById('doc-name').value,
            analysis: document.getElementById('text-analysis').value,
            selfEval: document.getElementById('text-self-eval').value,
            words: selectedWords,
            images: imageData
        };
        localStorage.setItem('shodo_app_save', JSON.stringify(data));
        alert("ブラウザに一時保存しました。");
    }

    function loadFromLocal() {
        const savedData = localStorage.getItem('shodo_app_save');
        if (!savedData) return alert("保存されたデータが見つかりません。");
        if (!confirm("前回の続きを復元しますか？")) return;
        const data = JSON.parse(savedData);
        document.getElementById('doc-date').value = data.date;
        document.getElementById('doc-unit').value = data.unit;
        document.getElementById('doc-name').value = data.name;
        document.getElementById('text-analysis').value = data.analysis;
        document.getElementById('text-self-eval').value = data.selfEval;
        document.querySelectorAll('.word-card').forEach(card => {
            card.classList.toggle('selected', data.words.includes(card.innerText));
        });
        if (data.images.koten) updateImageDisplay('koten', data.images.koten);
        if (data.images.self) updateImageDisplay('self', data.images.self);
        checkValidation();
    }

    document.getElementById('btn-download').addEventListener('click', async () => {
        const area = document.getElementById('report-area');
        
        document.getElementById('replace-date').innerText = document.getElementById('doc-date').value;
        document.getElementById('replace-unit').innerText = document.getElementById('doc-unit').value;
        document.getElementById('replace-name').innerText = document.getElementById('doc-name').value;
        document.getElementById('replace-analysis').innerText = document.getElementById('text-analysis').value;
        document.getElementById('replace-self-eval').innerText = document.getElementById('text-self-eval').value;

        const hideTargets = document.querySelectorAll('.no-capture-ui, .btn-delete, .save-load-area, #app-manual, .utility-area');
        hideTargets.forEach(el => el.style.visibility = 'hidden');

        try {
            const canvas = await html2canvas(area, {
                scale: 2, useCORS: true, backgroundColor: '#e3f2fd',
                onclone: (clonedDoc) => {
                    clonedDoc.getElementById('report-area').classList.add('capturing');
                }
            });
            const link = document.createElement('a');
            link.download = `臨書レポート_${document.getElementById('doc-name').value}.png`;
            link.href = canvas.toDataURL("image/png");
            link.click();
        } catch (e) {
            alert("画像の保存に失敗しました。");
        } finally {
            hideTargets.forEach(el => el.style.visibility = 'visible');
        }
    });

    document.addEventListener('paste', e => {
        const item = Array.from(e.clipboardData.items).find(i => i.type.includes('image'));
        if (item) {
            const reader = new FileReader();
            reader.onload = f => updateImageDisplay(activeCard, f.target.result);
            reader.readAsDataURL(item.getAsFile());
        }
    });
</script>
</body>
</html>
