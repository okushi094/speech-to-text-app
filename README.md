<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>音声文字起こしアプリ</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <style>
        html, body {
            height: 100%; /* HTMLとbodyがビューポートの高さを最大限に占めるように設定 */
            margin: 0;
            padding: 0;
            overflow-x: hidden; /* 横方向のスクロールバーを非表示にする */
        }
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* 背景色 */
            display: flex; /* デスクトップでの中央寄せに利用 */
            justify-content: center; /* デスクトップでの中央寄せに利用 */
            align-items: center; /* デスクトップでの中央寄せに利用 */
            min-height: 100vh; /* bodyがビューポートの高さを少なくとも満たすように */
            color: #1a202c;
        }
        .app-container {
            background-color: #ffffff;
            border-radius: 36px; /* デスクトップでのスマホ風の角丸 */
            box-shadow: 0 25px 60px rgba(0, 0, 0, 0.15); /* デスクトップでの影 */
            width: 100%;
            max-width: 600px; /* デスクトップでの最大幅 */
            height: 100%; /* 親要素(body)の高さに合わせる */
            max-height: 900px; /* デスクトップでの最大高さ */
            display: flex;
            flex-direction: column;
            overflow-y: auto; /* コンテナ内で縦方向のスクロールを許可 */
            box-sizing: border-box; /* パディングとボーダーを幅と高さに含める */
        }

        /* ヘッダー */
        .header {
            padding: 20px 25px 15px;
            display: flex;
            align-items: center;
            justify-content: center; /* タイトルを中央に配置 */
            border-bottom: 1px solid #f0f4f8;
            position: relative;
            flex-shrink: 0; /* ヘッダーが縮まないようにする */
        }
        .header .title {
            font-size: 1.1rem;
            font-weight: 600;
            color: #1a202c;
            text-align: center;
        }

        /* メインコンテンツエリア */
        .main-content {
            flex-grow: 1; /* 利用可能なスペースを占める */
            padding: 30px; /* パディングを増やす */
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 25px; /* 要素間の間隔を増やす */
        }
        .profile-section {
            display: flex;
            align-items: center;
            gap: 10px;
            width: 100%;
            justify-content: flex-start;
            margin-bottom: 10px;
            flex-shrink: 0;
        }
        .profile-icon {
            width: 32px;
            height: 32px;
            border-radius: 50%;
            background-color: #e2e8f0;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #4a5568;
            flex-shrink: 0;
        }
        .profile-icon svg {
            width: 20px;
            height: 20px;
        }
        .voice-recognition-text {
            font-size: 1.25rem;
            font-weight: 600;
            color: #1a202c;
            text-align: left;
            width: 100%;
            flex-shrink: 0;
        }

        /* サウンドウェーブビジュアライザー */
        .sound-wave-visualizer {
            width: 180px; /* サイズを拡大 */
            height: 180px;
            background-color: #f0f4f8;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            margin-top: 20px;
            margin-bottom: 30px;
            box-shadow: inset 0 2px 5px rgba(0,0,0,0.05);
            position: relative;
            flex-shrink: 0;
        }
        .sound-wave-visualizer svg {
            color: #a0aec0; /* 非アクティブ時のグレー */
            width: 100px; /* アイコンサイズを拡大 */
            height: 100px;
            transition: color 0.3s ease;
        }
        .sound-wave-visualizer.active svg {
            color: #6366f1; /* アクティブ時の青 */
            animation: wave-pulse 1.5s infinite ease-out;
        }
        @keyframes wave-pulse {
            0%, 100% { transform: scale(0.9); opacity: 0.8; }
            50% { transform: scale(1.0); opacity: 1; }
        }

        /* 停止ボタン (大型の赤) */
        .stop-button-large {
            background-color: #ef4444;
            color: white;
            padding: 18px 40px;
            border-radius: 12px;
            font-size: 1.2rem;
            font-weight: 600;
            border: none;
            cursor: pointer;
            box-shadow: 0 8px 20px rgba(239, 68, 68, 0.3);
            transition: all 0.3s ease;
            width: 90%; /* 幅を広げる */
            max-width: 300px; /* 最大幅を拡大 */
            margin-bottom: 20px;
            display: none; /* デフォルトで非表示 */
            flex-shrink: 0;
        }
        .stop-button-large:hover {
            background-color: #dc2626;
            transform: translateY(-2px);
            box-shadow: 0 12px 25px rgba(239, 68, 68, 0.4);
        }
        .stop-button-large:disabled {
            background-color: #cbd5e0;
            color: #a0aec0;
            cursor: not-allowed;
            box-shadow: none;
            transform: none;
        }

        /* テキストエリア */
        textarea {
            width: 100%;
            min-height: 150px; /* 最小高さを調整 */
            padding: 15px;
            border: 1px solid #e2e8f0;
            border-radius: 10px;
            font-size: 1rem;
            color: #2d3748;
            background-color: #f7fafc;
            resize: vertical;
            box-shadow: inset 0 1px 3px rgba(0,0,0,0.05);
            transition: border-color 0.3s ease;
            flex-grow: 1; /* メインコンテンツ内で拡大・縮小可能にする */
        }
        textarea:focus {
            outline: none;
            border-color: #6366f1;
        }

        /* アクションボタン (録音, 保存) */
        .action-buttons {
            display: flex;
            justify-content: center; /* ボタンを水平中央に配置 */
            align-items: center;
            width: 100%;
            margin-top: 20px;
            gap: 15px;
            flex-shrink: 0;
        }
        .action-btn {
            background-color: #ffffff;
            border: 1px solid #e2e8f0;
            border-radius: 12px;
            padding: 12px 20px;
            font-size: 1rem;
            font-weight: 500;
            color: #4a5568;
            cursor: pointer;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
            transition: all 0.2s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
            flex: 1; /* スペースを均等に分配 */
            max-width: 150px; /* ボタンの最大幅を制限 */
        }
        .action-btn:hover {
            background-color: #f0f4f8;
            transform: translateY(-1px);
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.08);
        }
        .action-btn:disabled {
            background-color: #f7fafc;
            color: #a0aec0;
            cursor: not-allowed;
            box-shadow: none;
            transform: none;
        }
        .action-btn svg {
            width: 20px;
            height: 20px;
            color: #4a5568;
        }

        /* 下部ナビゲーションバー */
        .bottom-nav {
            background-color: #ffffff;
            border-top: 1px solid #f0f4f8;
            padding: 10px 0;
            display: flex;
            justify-content: center; /* 単一のナビアイテムを中央に配置 */
            align-items: center;
            box-shadow: 0 -5px 15px rgba(0, 0, 0, 0.05);
            border-bottom-left-radius: 36px; /* コンテナの角丸に合わせる */
            border-bottom-right-radius: 36px;
            flex-shrink: 0;
        }
        .nav-item {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 3px;
            cursor: pointer;
            color: #a0aec0; /* 非アクティブ時の色 */
            font-size: 0.75rem;
            font-weight: 500;
            transition: color 0.2s ease;
        }
        .nav-item.active {
            color: #ef4444; /* アクティブ時の色 (赤) */
        }
        .nav-item svg {
            width: 24px;
            height: 24px;
            transition: color 0.2s ease;
        }
        .nav-item.active svg {
            color: #ef4444;
        }

        /* ステータスメッセージ */
        .status-message {
            font-size: 0.85rem;
            padding: 10px 15px;
            border-radius: 8px;
            margin-top: 10px;
            font-weight: 500;
            width: 100%;
            text-align: center;
            flex-shrink: 0;
        }
        .status-message.info {
            background-color: #e0f2fe;
            color: #2563eb;
        }
        .status-message.error {
            background-color: #fee2e2;
            color: #dc2626;
        }
        .status-message.success {
            background-color: #dcfce7;
            color: #16a34a;
        }

        /* スマートフォンなどの小さな画面向けのレスポンシブ調整 */
        @media (max-width: 768px) { /* 一般的なスマートフォンの画面幅に合わせる */
            body {
                display: block; /* Flexboxの中央寄せを解除し、ブロック表示に */
                padding: 0; /* bodyのパディングをなくす */
                margin: 0; /* bodyのマージンをなくす */
                overflow-y: auto; /* 必要に応じてbody全体でスクロールを許可 */
            }
            .app-container {
                border-radius: 0; /* 画面端まで広がるように角丸をなくす */
                max-width: unset; /* 最大幅の制限を解除 */
                min-height: 100vh; /* 画面の高さを満たすように */
                height: auto; /* コンテンツに合わせて高さを自動調整 */
                max-height: unset; /* 最大高さの制限を解除 */
                box-shadow: none; /* 影をなくす */
                width: 100vw; /* ビューポートの幅いっぱいに広げる */
            }
            .header {
                padding: 15px 20px 10px; /* パディングを削減 */
            }
            .header .title {
                font-size: 1rem; /* タイトルフォントを小さく */
            }
            .main-content {
                padding: 20px; /* パディングを削減 */
                gap: 15px; /* 要素間の間隔を削減 */
            }
            .sound-wave-visualizer {
                width: 140px; /* モバイル向けに少し小さく */
                height: 140px;
                margin-top: 15px;
                margin-bottom: 20px;
            }
            .sound-wave-visualizer svg {
                width: 80px; /* モバイル向けに少し小さく */
                height: 80px;
            }
            .stop-button-large {
                padding: 15px 30px;
                font-size: 1.1rem;
                width: 90%; /* 小さな画面で幅を広げる */
                max-width: unset;
            }
            textarea {
                min-height: 100px; /* モバイル向けに最小高さを調整 */
                padding: 12px;
                font-size: 0.95rem;
            }
            .action-buttons {
                margin-top: 15px;
                gap: 10px;
            }
            .action-btn {
                padding: 10px 15px;
                font-size: 0.9rem;
                border-radius: 10px;
            }
            .action-btn svg {
                width: 18px;
                height: 18px;
            }
            .bottom-nav {
                padding: 8px 0; /* パディングを削減 */
                border-bottom-left-radius: 0; /* 角丸をなくす */
                border-bottom-right-radius: 0;
            }
            .nav-item {
                font-size: 0.7rem;
            }
            .nav-item svg {
                width: 20px;
                height: 20px;
            }
            .status-message {
                font-size: 0.8rem;
                padding: 8px 12px;
            }
        }
    </style>
</head>
<body>
    <div class="app-container">
        <!-- Header -->
        <div class="header">
            <span class="title">音声文字起こしアプリ</span>
        </div>

        <!-- Main Content -->
        <div class="main-content">
            <div class="profile-section">
                <div class="profile-icon">
                    <svg fill="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"></path></svg>
                </div>
                <span class="voice-recognition-text">Voice Recognition</span>
            </div>

            <div id="soundWaveVisualizer" class="sound-wave-visualizer">
                <svg fill="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path d="M6 18h2V6H6v12zm4 0h2V6h-2v12zm4 0h2V6h-2v12zm4 0h2V6h-2v12z"/>
                </svg>
            </div>

            <!-- Large Stop Button - Visible only when recording -->
            <button id="stopButtonLarge" class="stop-button-large">Stop</button>

            <textarea id="transcript" placeholder="ここに文字起こしされたテキストが表示されます..." class="shadow-inner"></textarea>

            <div class="action-buttons">
                <button id="recordButton" class="action-btn">
                    <svg fill="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M12 14c1.66 0 2.99-1.34 2.99-3L15 5c0-1.66-1.34-3-3-3S9 3.34 9 5v6c0 1.66 1.34 3 3 3zm5.2-3c0 3.53-2.91 6.4-6.2 6.4S4.8 14.53 4.8 11H3c0 4.07 3.06 7.44 7 7.93V22h4v-3.07c3.94-.49 7-3.86 7-7.93h-1.8z"/></svg>
                    Record
                </button>
                <button id="saveButton" class="action-btn">
                    <svg fill="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M19 21H5a2 2 0 01-2-2V5a2 2 0 012-2h11l4 4v11a2 2 0 01-2 2zM12 17V9"/></svg>
                    Save
                </button>
            </div>
            <p id="status" class="status-message info">準備完了</p>
        </div>

        <div class="bottom-nav">
            <div class="nav-item active">
                <svg fill="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M12 14c1.66 0 2.99-1.34 2.99-3L15 5c0-1.66-1.34-3-3-3S9 3.34 9 5v6c0 1.66 1.34 3 3 3zm5.2-3c0 3.53-2.91 6.4-6.2 6.4S4.8 14.53 4.8 11H3c0 4.07 3.06 7.44 7 7.93V22h4v-3.07c3.94-.49 7-3.86 7-7.93h-1.8z"/></svg>
                <span>Record</span>
            </div>
        </div>
    </div>

    <script>
        // DOM要素の取得
        const recordButton = document.getElementById('recordButton');
        const stopButtonLarge = document.getElementById('stopButtonLarge');
        const saveButton = document.getElementById('saveButton');
        const transcriptTextarea = document.getElementById('transcript');
        const statusMessage = document.getElementById('status');
        const soundWaveVisualizer = document.getElementById('soundWaveVisualizer');

        // Web Speech APIの初期化
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        let recognition;
        let isRecognizing = false; // 音声認識中かどうかを管理するフラグ
        let lastFinalTranscript = ''; // 最終確定したテキストを保持するための変数

        if (SpeechRecognition) {
            recognition = new SpeechRecognition();
            recognition.continuous = true; // 連続的な音声認識を有効にする
            recognition.interimResults = true; // 暫定的な結果も表示する
            recognition.lang = 'ja-JP'; // 日本語を設定

            // 音声認識が開始されたときのイベント
            recognition.onstart = () => {
                isRecognizing = true;
                lastFinalTranscript = transcriptTextarea.value; // 既存のテキストを保持
                recordButton.style.display = 'none'; // Recordボタンを非表示
                stopButtonLarge.style.display = 'block'; // Stopボタンを表示
                soundWaveVisualizer.classList.add('active'); // ビジュアライザーをアクティブに
                statusMessage.textContent = '録音中... 話し始めてください';
                statusMessage.className = 'status-message success';
            };

            // 音声認識の結果が得られたときのイベント
            recognition.onresult = (event) => {
                let interimTranscript = '';
                let tempFinalTranscript = ''; // 今回のイベントで確定した部分

                for (let i = event.resultIndex; i < event.results.length; ++i) {
                    const transcript = event.results[i][0].transcript;
                    if (event.results[i].isFinal) {
                        tempFinalTranscript += transcript;
                    } else {
                        interimTranscript += transcript;
                    }
                }

                // 新しい確定部分があれば、lastFinalTranscriptに追加して改行
                if (tempFinalTranscript) {
                    lastFinalTranscript += tempFinalTranscript + '\n';
                    // 暫定結果は表示しないので、lastFinalTranscriptでテキストエリアを更新
                    transcriptTextarea.value = lastFinalTranscript;
                } else {
                    // 新しい確定部分がない場合、lastFinalTranscriptに現在の暫定結果を結合して表示
                    // これにより、話している途中の文字がリアルタイムで表示される
                    transcriptTextarea.value = lastFinalTranscript + interimTranscript;
                }

                transcriptTextarea.scrollTop = transcriptTextarea.scrollHeight; // スクロールを一番下へ
            };

            // 音声認識が終了したときのイベント
            recognition.onend = () => {
                isRecognizing = false;
                recordButton.style.display = 'flex'; // Recordボタンを表示
                stopButtonLarge.style.display = 'none'; // Stopボタンを非表示
                soundWaveVisualizer.classList.remove('active'); // ビジュアライザーを非アクティブに
                statusMessage.textContent = '録音停止';
                statusMessage.className = 'status-message info';
            };

            // 音声認識中にエラーが発生したときのイベント
            recognition.onerror = (event) => {
                isRecognizing = false;
                recordButton.style.display = 'flex';
                stopButtonLarge.style.display = 'none';
                soundWaveVisualizer.classList.remove('active');
                statusMessage.textContent = `エラーが発生しました: ${event.error}`;
                statusMessage.className = 'status-message error';
                console.error('Speech recognition error:', event.error);

                if (event.error === 'no-speech' || event.error === 'audio-capture') {
                    statusMessage.textContent = 'マイクが検出されないか、音声が小さすぎます。マイクの設定を確認してください。';
                } else if (event.error === 'not-allowed') {
                    statusMessage.textContent = 'マイクの使用が許可されていません。ブラウザの設定を確認してください。';
                }
            };

            // Recordボタンのクリックイベント
            recordButton.addEventListener('click', () => {
                if (!isRecognizing) {
                    try {
                        recognition.start();
                    } catch (e) {
                        console.error("Recognition start error:", e);
                        statusMessage.textContent = '音声認識の開始に失敗しました。';
                        statusMessage.className = 'status-message error';
                    }
                }
            });

            // Stopボタン（大）のクリックイベント
            stopButtonLarge.addEventListener('click', () => {
                if (isRecognizing) {
                    recognition.stop();
                }
            });

            // Saveボタンのクリックイベント (テキストコピー機能)
            saveButton.addEventListener('click', () => {
                transcriptTextarea.select();
                document.execCommand('copy');
                statusMessage.textContent = 'テキストがコピーされました！';
                statusMessage.className = 'status-message success';
                setTimeout(() => {
                    statusMessage.textContent = '準備完了';
                    statusMessage.className = 'status-message info';
                }, 2000);
            });

        } else {
            recordButton.disabled = true;
            stopButtonLarge.disabled = true;
            saveButton.disabled = true;
            statusMessage.textContent = 'お使いのブラウザはWeb Speech APIをサポートしていません。Google Chromeなどの最新ブラウザをご利用ください。';
            statusMessage.className = 'status-message error';
        }
    </script>
</body>
</html>
