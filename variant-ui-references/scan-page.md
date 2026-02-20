The code below contains a design. This design should be used to create a new app or be added to an existing one.

Look at the current open project to determine if a project exists. If no project is open, create a new Vite project then create this view in React after componentizing it.

If a project does exist, determine the framework being used and implement the design within that framework. Identify whether reusable components already exist that can be used to implement the design faithfully and if so use them, otherwise create new components. If other views already exist in the project, make sure to place the view in a sensible route and connect it to the other views.

Ensure the visual characteristics, layout, and interactions in the design are preserved with perfect fidelity.

Run the dev command so the user can see the app once finished.

```
<html lang="en" vid="0"><head vid="1">
    <meta charset="UTF-8" vid="2">
    <meta name="viewport" content="width=device-width, initial-scale=1.0" vid="3">
    <title vid="4">cheQR - Scanning Results</title>
    <style vid="5">
        :root {
            --bg-body: #09090b;
            --bg-surface: #ffffff;
            --bg-surface-alt: #f4f4f5;
            --bg-element-dark: #27272a;
            --text-main: #18181b;
            --text-body-inverse: #ffffff;
            --text-muted: #71717a;
            --text-placeholder: #a1a1aa;
            --border-light: #e4e4e7;
            --accent-primary: #18181b;
            --radius-md: 12px;
            --radius-lg: 32px;
            --radius-pill: 999px;
            --font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            --space-1: 4px;
            --space-2: 8px;
            --space-3: 16px;
            --space-4: 24px;
            --space-5: 32px;
            --space-6: 48px;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            background-color: var(--bg-body);
            color: var(--text-body-inverse);
            font-family: var(--font-family);
            font-size: 14px;
            line-height: 1.5;
            -webkit-font-smoothing: antialiased;
            display: flex;
            flex-direction: column;
            width: 1280px;
            height: 960px;
            border-radius: 20px;
            overflow: hidden;
            margin: auto;
        }

        header {
            height: 80px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 var(--space-5);
            background: var(--bg-body);
            border-bottom: none;
        }

        .logo {
            display: flex;
            align-items: center;
            gap: var(--space-2);
            font-size: 18px;
            font-weight: 600;
            color: #fff;
        }
        
        .logo svg { color: #fff; }

        .segmented-control {
            background: var(--bg-element-dark);
            border: none;
            border-radius: var(--radius-pill);
            padding: 4px;
            display: flex;
        }

        .segmented-btn {
            background: transparent;
            border: none;
            color: var(--text-placeholder);
            padding: 6px 24px;
            font-size: 13px;
            border-radius: var(--radius-pill);
            cursor: pointer;
            font-weight: 500;
            transition: all 0.2s;
        }

        .segmented-btn.active {
            background: #ffffff;
            color: #000000;
            box-shadow: 0 1px 2px rgba(0,0,0,0.1);
        }

        main {
            flex: 1;
            display: grid;
            grid-template-columns: 1fr 400px;
            gap: 20px;
            padding: 0 20px 20px 20px;
            overflow: hidden;
        }

        
        .results-panel {
            padding: 0;
            border: none;
            overflow: hidden;
            display: flex;
            flex-direction: column;
        }

        .result-card {
            background: var(--bg-surface);
            color: var(--text-main);
            border: none;
            border-radius: var(--radius-lg);
            width: 100%;
            height: 100%;
            max-width: none;
            padding: var(--space-6);
            text-align: center;
            display: flex;
            flex-direction: column;
            justify-content: center;
        }

        
        .history-panel {
            padding: var(--space-5);
            background: var(--bg-surface);
            color: var(--text-main);
            border-radius: var(--radius-lg);
            overflow-y: auto;
        }

        .history-panel h3 {
            font-weight: 600;
        }

        .success-icon {
            width: 80px;
            height: 80px;
            background: var(--bg-surface-alt);
            color: #18181b;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            margin: 0 auto var(--space-4);
        }
        
        
        .success-icon svg path {
            stroke-width: 2.5;
        }

        .result-card h2 {
            font-weight: 600;
            color: #000;
            font-size: 28px;
            margin-bottom: 8px;
        }
        
        .result-card p {
            color: var(--text-muted);
            font-size: 15px;
        }

        .data-display {
            background: var(--bg-surface-alt);
            border: none;
            border-radius: var(--radius-md);
            padding: var(--space-4);
            margin: 32px 0;
            text-align: left;
            word-break: break-all;
            font-family: monospace;
            font-size: 14px;
            color: var(--text-main);
        }

        .action-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: var(--space-3);
            margin-top: 0;
            max-width: 480px;
            margin: 0 auto;
            width: 100%;
        }

        .btn {
            display: inline-flex;
            align-items: center;
            justify-content: center;
            height: 52px;
            padding: 0 var(--space-4);
            border-radius: var(--radius-pill);
            font-size: 14px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.2s;
            border: 1px solid transparent;
            gap: var(--space-2);
            text-decoration: none;
        }

        .btn-primary { 
            background: #000000; 
            color: white; 
        }
        .btn-primary:hover { 
            background: #27272a; 
            transform: translateY(-1px);
        }
        
        .btn-secondary { 
            background: var(--bg-surface-alt); 
            border: none; 
            color: var(--text-main); 
        }
        .btn-secondary:hover { 
            background: #e4e4e7; 
        }

        
        header .btn-secondary {
            background: var(--bg-element-dark);
            color: #fff;
        }
        header .btn-secondary:hover {
            background: #3f3f46;
        }

        .history-item {
            display: flex;
            align-items: center;
            gap: var(--space-3);
            padding: 12px;
            border-radius: var(--radius-md);
            border: 1px solid transparent;
            cursor: pointer;
            transition: all 0.2s;
            margin-bottom: 8px;
        }

        .history-item:hover {
            background: var(--bg-surface-alt);
        }

        .history-icon {
            width: 40px;
            height: 40px;
            background: var(--bg-surface-alt);
            border-radius: var(--radius-md);
            display: flex;
            align-items: center;
            justify-content: center;
            color: #000;
        }
        
        .history-item:hover .history-icon {
            background: #fff;
        }

        .history-details { flex: 1; min-width: 0; }
        .history-title { font-weight: 600; font-size: 14px; color: var(--text-main); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
        .history-meta { font-size: 12px; color: var(--text-muted); margin-top: 2px; }

        footer {
            height: 40px;
            border-top: none;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: var(--space-4);
            font-size: 12px;
            color: #52525b;
            background: transparent;
        }
        
        
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: transparent;
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(0,0,0,0.1);
            border-radius: 10px;
        }
    </style>
</head>
<body vid="6">

    <header vid="7">
        <div class="logo" vid="8">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" vid="9">
                <rect x="2" y="2" width="9" height="9" rx="2" vid="10"></rect>
                <rect x="13" y="2" width="9" height="9" rx="2" vid="11"></rect>
                <rect x="2" y="13" width="9" height="9" rx="2" vid="12"></rect>
                <path d="M16 16H20V20" stroke-linecap="round" vid="13"></path>
            </svg>
            cheQR
        </div>
        
        <div class="segmented-control" vid="14">
            <button class="segmented-btn" vid="15">Create</button>
            <button class="segmented-btn active" vid="16">Scan</button>
        </div>

        <div style="display: flex; gap: 8px;" vid="17">
            <button class="btn-secondary" style="border-radius: 50%; width: 40px; height: 40px; padding:0; display:flex; align-items:center; justify-content:center;" vid="18">
                <svg width="20" height="20" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="19"><path d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z" stroke-width="2" vid="20"></path></svg>
            </button>
        </div>
    </header>

    <main vid="21">
        <div class="results-panel" vid="22">
            <div class="result-card" vid="23">
                <div class="success-icon" vid="24">
                    <svg width="32" height="32" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="25"><path d="M5 13l4 4L19 7" stroke-width="3" stroke-linecap="round" stroke-linejoin="round" vid="26"></path></svg>
                </div>
                <h2 vid="27">Successfully Scanned</h2>
                <p vid="28">Extracted content from the QR code</p>

                <div class="data-display" vid="29">
                    https://cheqr.app/design-system/v1/documentation
                </div>

                <div class="action-grid" vid="30">
                    <button class="btn btn-primary" style="grid-column: span 2;" vid="31">
                        <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="32"><path d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" vid="33"></path></svg>
                        Open URL
                    </button>
                    <button class="btn btn-secondary" vid="34">
                        <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="35"><path d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2" stroke-width="2" stroke-linecap="round" vid="36"></path></svg>
                        Copy
                    </button>
                    <button class="btn btn-secondary" vid="37">
                        <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="38"><path d="M8.684 13.342C8.886 12.938 9 12.482 9 12c0-.482-.114-.938-.316-1.342m0 2.684a3 3 0 110-2.684m0 2.684l6.632 3.316m-6.632-6l6.632-3.316m0 0a3 3 0 105.367-2.684 3 3 0 00-5.367 2.684zm0 9.316a3 3 0 105.368 2.684 3 3 0 00-5.368-2.684z" stroke-width="2" vid="39"></path></svg>
                        Share
                    </button>
                </div>

                <div style="margin-top: 32px; padding-top: 24px;" vid="40">
                    <button class="btn btn-secondary" style="width: auto; padding: 0 24px; border: none; background: transparent; color: var(--text-muted);" vid="41">
                        Scan Another Code
                    </button>
                </div>
            </div>
        </div>

        <div class="history-panel" vid="42">
            <div style="display: flex; align-items: center; justify-content: space-between; margin-bottom: 24px;" vid="43">
                <h3 vid="44">Recent Scans</h3>
                <button style="background:none; border:none; color: var(--text-muted); font-size: 12px; cursor: pointer; font-weight: 500;" vid="45">Clear All</button>
            </div>

            <div style="display: flex; flex-direction: column; gap: 4px;" vid="46">
                <div class="history-item" vid="47">
                    <div class="history-icon" vid="48"><svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="49"><path d="M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656 5.656l1.102-1.101" stroke-width="2" vid="50"></path><path d="M10.172 13.828a4 4 0 015.656 0l4 4a4 4 0 11-5.656 5.656l-1.102-1.101" stroke-width="2" vid="51"></path></svg></div>
                    <div class="history-details" vid="52">
                        <div class="history-title" vid="53">https://cheqr.app/v1/docs</div>
                        <div class="history-meta" vid="54">Just now • URL</div>
                    </div>
                </div>

                <div class="history-item" vid="55">
                    <div class="history-icon" vid="56"><svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="57"><path d="M8.111 16.404a5.5 5.5 0 017.778 0M12 20h.01m-7.08-7.071a9.5 9.5 0 0113.436 0m-17.678-7.072a15.5 15.5 0 0121.92 0" stroke-width="2" stroke-linecap="round" vid="58"></path></svg></div>
                    <div class="history-details" vid="59">
                        <div class="history-title" vid="60">Starbucks_Guest_5G</div>
                        <div class="history-meta" vid="61">15 mins ago • WiFi Network</div>
                    </div>
                </div>

                <div class="history-item" vid="62">
                    <div class="history-icon" vid="63"><svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="64"><path d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" stroke-width="2" vid="65"></path></svg></div>
                    <div class="history-details" vid="66">
                        <div class="history-title" vid="67">John Smith (Contact)</div>
                        <div class="history-meta" vid="68">2 hours ago • vCard</div>
                    </div>
                </div>

                <div class="history-item" vid="69">
                    <div class="history-icon" vid="70"><svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="71"><path d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" stroke-width="2" vid="72"></path></svg></div>
                    <div class="history-details" vid="73">
                        <div class="history-title" vid="74">Pickup Note: Gate 45...</div>
                        <div class="history-meta" vid="75">Yesterday • Plain Text</div>
                    </div>
                </div>
            </div>
        </div>
    </main>

    <footer vid="76">
        <span vid="77">cheQR v1.0</span>
        <span vid="78">•</span>
        <a href="#" style="color: inherit; text-decoration: none;" vid="79">Privacy &amp; Security</a>
        <span vid="80">•</span>
        <a href="#" style="color: inherit; text-decoration: none;" vid="81">History Settings</a>
    </footer>

</body></html>
```
