The code below contains a design. This design should be used to create a new app or be added to an existing one.

Look at the current open project to determine if a project exists. If no project is open, create a new Vite project then create this view in React after componentizing it.

If a project does exist, determine the framework being used and implement the design within that framework. Identify whether reusable components already exist that can be used to implement the design faithfully and if so use them, otherwise create new components. If other views already exist in the project, make sure to place the view in a sensible route and connect it to the other views.

Ensure the visual characteristics, layout, and interactions in the design are preserved with perfect fidelity.

Run the dev command so the user can see the app once finished.

```
<html vid="0"><head vid="1">
    <meta charset="UTF-8" vid="2">
    <meta name="viewport" content="width=device-width, initial-scale=1.0" vid="3">
    <title vid="4">cheQR - QR Code Manager</title>
    <style vid="5">
        :root {
            
            --bg-body: #080808;
            --bg-surface: #ffffff;
            --bg-surface-hover: #f5f5f7;
            --bg-input: #f2f2f4;
            
            --accent-primary: #000000;
            --accent-hover: #333333;
            
            --text-main: #000000;
            --text-on-dark: #ffffff;
            --text-muted: #86868b;
            --text-placeholder: #a1a1a6;
            
            --border-light: #e5e5ea;
            --border-active: #000000;
            
            
            --radius-sm: 12px;
            --radius-md: 20px;
            --radius-lg: 32px;
            --radius-pill: 999px;
            
            --font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
            
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
            color: var(--text-main);
            font-family: var(--font-family);
            font-size: 15px;
            line-height: 1.5;
            -webkit-font-smoothing: antialiased;
            display: flex;
            flex-direction: column;
            height: 100vh;
            overflow: hidden;
        }

        h1, h2, h3, h4 {
            font-weight: 700;
            letter-spacing: -0.03em;
            color: var(--text-main);
        }

        h2 {
            font-size: 28px;
            margin-bottom: var(--space-4);
        }

        h3 {
            font-size: 18px;
            font-weight: 600;
        }

        label {
            display: block;
            font-size: 13px;
            font-weight: 500;
            color: var(--text-muted);
            margin-bottom: var(--space-2);
            letter-spacing: -0.01em;
        }

        .badge {
            display: inline-flex;
            align-items: center;
            padding: 4px 10px;
            background: #FFD60A; 
            color: #000;
            font-size: 11px;
            border-radius: var(--radius-pill);
            margin-left: var(--space-2);
            text-transform: uppercase;
            letter-spacing: 0.05em;
            font-weight: 700;
        }

        
        header {
            height: 80px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 var(--space-5);
            background: var(--bg-body);
            border-bottom: none;
            z-index: 100;
            color: var(--text-on-dark);
            flex-shrink: 0;
        }

        .logo {
            display: flex;
            align-items: center;
            gap: var(--space-2);
            font-size: 20px;
            font-weight: 700;
            letter-spacing: -0.04em;
            color: var(--text-on-dark);
        }
        
        .logo svg {
            color: var(--text-on-dark);
        }

        
        main {
            flex: 1;
            display: grid;
            grid-template-columns: 420px 1fr;
            max-width: 1600px;
            margin: 0 auto;
            width: 100%;
            padding: 0 var(--space-4) var(--space-4) var(--space-4);
            gap: var(--space-4);
            height: calc(100vh - 80px); 
            overflow: hidden;
        }

        
        .preview-panel, .settings-panel {
            background: var(--bg-surface);
            border-radius: var(--radius-lg);
            padding: var(--space-5);
            overflow-y: auto;
            border: none;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
            position: relative;
        }
        
        
        .preview-panel {
            border-right: none;
        }

        
        .preview-panel::-webkit-scrollbar,
        .settings-panel::-webkit-scrollbar {
            width: 0px;
            background: transparent;
        }

        
        .segmented-control {
            background: rgba(255, 255, 255, 0.15);
            border: none;
            border-radius: var(--radius-pill);
            padding: 4px;
            display: flex;
            backdrop-filter: blur(10px);
        }

        .segmented-btn {
            background: transparent;
            border: none;
            color: rgba(255, 255, 255, 0.6);
            padding: 8px 28px;
            font-size: 14px;
            border-radius: var(--radius-pill);
            cursor: pointer;
            transition: all 0.3s cubic-bezier(0.25, 1, 0.5, 1);
            font-weight: 600;
        }

        .segmented-btn.active {
            background: #ffffff;
            color: #000000;
            box-shadow: 0 4px 12px rgba(0,0,0,0.2);
        }
        
        
        .settings-panel .segmented-control {
            background: var(--bg-input);
        }
        .settings-panel .segmented-btn {
            color: var(--text-muted);
        }
        .settings-panel .segmented-btn.active {
            background: #fff;
            color: #000;
            box-shadow: 0 2px 8px rgba(0,0,0,0.05);
        }

        
        .btn {
            display: inline-flex;
            align-items: center;
            justify-content: center;
            height: 48px;
            padding: 0 var(--space-5);
            border-radius: var(--radius-pill);
            font-size: 15px;
            font-weight: 600;
            cursor: pointer;
            transition: transform 0.1s, background 0.2s;
            border: none;
            gap: var(--space-2);
            letter-spacing: -0.01em;
        }
        
        .btn:active {
            transform: scale(0.98);
        }

        .btn-primary {
            background: var(--accent-primary);
            color: white;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }
        .btn-primary:hover {
            background: var(--accent-hover);
        }

        .btn-secondary {
            background: var(--bg-input);
            border: none;
            color: var(--text-main);
        }
        .btn-secondary:hover {
            background: #e1e1e3;
        }

        .btn-icon {
            padding: 0;
            width: 40px;
            height: 40px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: rgba(255,255,255,0.7);
            background: rgba(255,255,255,0.1);
            border: none;
            cursor: pointer;
            transition: 0.2s;
        }
        .btn-icon:hover {
            background: rgba(255,255,255,0.2);
            color: #fff;
        }
        
        
        .modal .btn-icon {
            background: var(--bg-input);
            color: var(--text-main);
        }
        .modal .btn-icon:hover {
            background: #e1e1e3;
        }

        .btn-group {
            display: flex;
            gap: var(--space-2);
        }

        
        .input-group {
            margin-bottom: var(--space-4);
        }

        input[type="text"],
        input[type="number"],
        input[type="email"],
        input[type="password"],
        textarea,
        select {
            width: 100%;
            background: var(--bg-input);
            border: 2px solid transparent;
            color: var(--text-main);
            padding: 14px 16px;
            border-radius: var(--radius-md);
            font-family: inherit;
            font-size: 15px;
            font-weight: 500;
            transition: all 0.2s;
            appearance: none;
        }

        input:focus,
        textarea:focus,
        select:focus {
            outline: none;
            background: #fff;
            border-color: #000;
            box-shadow: 0 0 0 4px rgba(0,0,0,0.05);
        }

        textarea {
            resize: vertical;
            min-height: 100px;
        }

        
        .checkbox-wrapper {
            display: flex;
            align-items: center;
            gap: 12px;
            cursor: pointer;
            user-select: none;
            padding: 8px 0;
        }
        
        input[type="checkbox"], input[type="radio"] {
            width: 22px;
            height: 22px;
            accent-color: var(--accent-primary);
            border-radius: 6px;
            background: var(--bg-input);
            border: 1px solid var(--border-light);
            cursor: pointer;
        }

        
        .qr-stage {
            background: var(--bg-input); 
            border-radius: var(--radius-lg);
            aspect-ratio: 1;
            display: flex;
            align-items: center;
            justify-content: center;
            position: relative;
            box-shadow: inset 0 2px 10px rgba(0,0,0,0.03);
            margin-bottom: var(--space-4);
            border: 1px solid rgba(0,0,0,0.03);
        }

        .qr-image {
            width: 75%;
            height: 75%;
            object-fit: contain;
            filter: drop-shadow(0 10px 20px rgba(0,0,0,0.1));
        }

        .export-card {
            border: 1px solid var(--border-light);
            border-radius: var(--radius-md);
            padding: var(--space-4);
            background: #fff;
            box-shadow: 0 10px 30px -10px rgba(0,0,0,0.08);
        }

        
        .accordion-item {
            border: none;
            border-radius: var(--radius-md);
            background: var(--bg-surface);
            overflow: hidden;
            margin-bottom: var(--space-2);
            box-shadow: 0 0 0 1px rgba(0,0,0,0.05);
        }

        .accordion-header {
            padding: var(--space-4);
            display: flex;
            align-items: center;
            justify-content: space-between;
            cursor: pointer;
            user-select: none;
            background: transparent;
            transition: background 0.2s;
        }
        
        .accordion-header:hover {
            background: var(--bg-surface-hover);
        }

        .accordion-content {
            padding: 0 var(--space-4) var(--space-5) var(--space-4);
            display: none;
            border-top: 1px solid var(--border-light);
            margin-top: 0;
            padding-top: var(--space-4);
        }

        .accordion-item.open .accordion-content {
            display: block;
        }

        
        .grid-2 {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: var(--space-3);
        }

        .grid-3 {
            display: grid;
            grid-template-columns: 1fr 1fr 1fr;
            gap: var(--space-3);
        }
        
        .grid-auto {
            display: flex;
            flex-wrap: wrap;
            gap: var(--space-3);
        }

        
        .color-picker-wrapper {
            display: flex;
            align-items: center;
            gap: 10px;
            background: var(--bg-input);
            border: 1px solid transparent;
            padding: 8px 12px;
            border-radius: var(--radius-md);
            cursor: pointer;
            transition: 0.2s;
        }
        .color-picker-wrapper:hover {
            background: #e1e1e3;
        }
        
        .color-swatch {
            width: 28px;
            height: 28px;
            border-radius: 50%;
            border: 2px solid #fff;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        input[type="color"] {
            visibility: hidden;
            width: 0;
            height: 0;
            padding: 0;
            border: none;
        }

        
        .tick-track {
            display: flex;
            justify-content: space-between;
            height: 12px;
            margin-top: 8px;
            opacity: 1;
        }
        .tick {
            width: 2px;
            height: 4px;
            background: #d1d1d6;
            border-radius: 1px;
            margin-top: 4px;
        }
        .tick:nth-child(5n) {
            height: 8px;
            background: #000;
            margin-top: 0;
        }

        
        footer {
            height: auto;
            min-height: 40px;
            border-top: none;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: var(--space-4);
            font-size: 12px;
            color: rgba(255,255,255,0.4);
            background: var(--bg-body);
            padding-bottom: 20px;
        }

        footer a {
            color: rgba(255,255,255,0.6);
            text-decoration: none;
        }
        footer a:hover {
            color: #fff;
            text-decoration: underline;
        }

        
        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0, 0, 0, 0.6);
            backdrop-filter: blur(8px);
            display: none; 
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }
        
        .modal {
            background: var(--bg-surface);
            border: none;
            border-radius: var(--radius-lg);
            width: 600px;
            max-width: 90vw;
            max-height: 85vh;
            display: flex;
            flex-direction: column;
            box-shadow: 0 40px 80px rgba(0,0,0,0.3);
            overflow: hidden;
        }

        .modal-header {
            padding: var(--space-5);
            border-bottom: 1px solid var(--border-light);
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #fff;
        }

        .modal-body {
            padding: var(--space-5);
            overflow-y: auto;
        }

        
        .scan-mode-container {
            display: none; 
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 100%;
            height: 100%;
            max-width: 600px;
            margin: 0 auto;
            text-align: center;
        }
        
        #view-scan {
            color: white; 
        }
        
        #view-scan h2 { color: white; }
        #view-scan p { color: rgba(255,255,255,0.6); }

        .camera-viewport {
            width: 100%;
            aspect-ratio: 1;
            background: #000;
            border-radius: var(--radius-lg);
            position: relative;
            overflow: hidden;
            border: 1px solid rgba(255,255,255,0.1);
            margin-bottom: var(--space-4);
            box-shadow: 0 20px 60px rgba(0,0,0,0.5);
        }

        .scan-overlay {
            position: absolute;
            top: 20%;
            left: 20%;
            right: 20%;
            bottom: 20%;
            border: 2px solid #fff;
            border-radius: var(--radius-md);
            box-shadow: 0 0 0 1000px rgba(0,0,0,0.7);
        }

        .w-full { width: 100%; }
        .hidden { display: none; }
        .text-center { text-align: center; }
        .font-light { font-weight: 300; }
        
        
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: transparent;
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(0,0,0,0.1);
            border-radius: 3px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: rgba(0,0,0,0.2);
        }

        @media (max-width: 1024px) {
            main {
                display: flex;
                flex-direction: column;
                height: auto;
                overflow: visible;
                padding-bottom: 80px;
            }
            .preview-panel, .settings-panel {
                overflow: visible;
            }
        }
    </style>
</head>
<body vid="6">

    <header vid="7">
        <div class="logo" vid="8">
            <svg width="28" height="28" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" vid="9">
                <rect x="2" y="2" width="9" height="9" rx="3" stroke="currentColor" stroke-width="2.5" vid="10"></rect>
                <rect x="13" y="2" width="9" height="9" rx="3" stroke="currentColor" stroke-width="2.5" vid="11"></rect>
                <rect x="2" y="13" width="9" height="9" rx="3" stroke="currentColor" stroke-width="2.5" vid="12"></rect>
                <path d="M16 16H20V20" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" vid="13"></path>
            </svg>
            cheQR
        </div>
        
        <div class="segmented-control" vid="14">
            <button class="segmented-btn active" id="mode-create" vid="15">Create</button>
            <button class="segmented-btn" id="mode-scan" vid="16">Scan</button>
        </div>

        <div class="btn-group" vid="17">
            <button class="btn-icon" aria-label="Dark mode" vid="18">
                <svg width="20" height="20" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="19"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z" vid="20"></path></svg>
            </button>
            <button class="btn-icon" aria-label="Language" vid="21">
                <svg width="20" height="20" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="22"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 5h12M9 3v2m1.204 12.596C7.262 15.924 5.373 15 3 15m18 0c-1.721 0-3.248.608-4.487 1.632M21 4c0 1.657-1.343 3-3 3s-3-1.343-3-3 1.343-3 3-3 3 1.343 3 3z" vid="23"></path></svg>
            </button>
        </div>
    </header>

    <main id="view-create" vid="24">
        
        
        <div class="preview-panel" vid="25">
            <div vid="26">
                <h2 class="font-light" style="font-weight: 800; letter-spacing: -1px;" vid="27">Preview</h2>
                <div class="qr-stage" vid="28">
                    <img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiB2aWV3Qm94PSIwIDAgMTAwIDEwMCI+PHBhdGggZmlsbD0iIzAwMDAwMCIgZD0iTTAgMGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwem0zMC0xMGgzMHYzMEg0MHptNCA0aDIydjIySDQ0em02IDZoMTB2MTBINTB6bS0xMCAyMGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMjAgMGgxMHYxMEg3MHpNMCA0MGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwek00MCA3MGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMTAgMGgxMHYxMEg2MHptMTAgMGgxMHYxMEg3MHptMTAgMGgxMHYxMEg4MHptLTEwIDEwaDEwdjEwSDgwem0tMjAgMGgxMHYxMEg2MHoiLz48L3N2Zz4=" alt="QR Code" class="qr-image" vid="29">
                </div>
            </div>

            <div class="grid-2" vid="30">
                <button class="btn btn-secondary" vid="31">
                    <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="32"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 5H6a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2v-1M8 5a2 2 0 002 2h2a2 2 0 002-2M8 5a2 2 0 012-2h2a2 2 0 012 2m0 0h2a2 2 0 012 2v3m2 4H10m0 0l3-3m-3 3l3 3" vid="33"></path></svg>
                    Copy
                </button>
                <button class="btn btn-secondary" vid="34">
                    <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="35"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12" vid="36"></path></svg>
                    Save JSON
                </button>
            </div>
            <button class="btn btn-secondary w-full" style="margin-top: 16px;" vid="37">
                <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="38"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" vid="39"></path></svg>
                Load Configuration
            </button>

            <div class="export-card" style="margin-top: 24px; border:none; background: var(--bg-input);" vid="40">
                <h3 vid="41">Export QR Code</h3>
                <div style="margin: 16px 0;" vid="42">
                    <label vid="43">File name</label>
                    <input type="text" placeholder="my-qr-code" value="my-qr-code" style="background: white;" vid="44">
                </div>
                <div class="grid-3" vid="45">
                    <button class="btn btn-primary" style="height: 40px;" vid="46">PNG</button>
                    <button class="btn btn-secondary" style="height: 40px; background: white;" vid="47">JPG</button>
                    <button class="btn btn-secondary" style="height: 40px; background: white;" vid="48">SVG</button>
                </div>
            </div>
        </div>

        
        <div class="settings-panel" vid="49">
            
            
            <div class="accordion-item" vid="50">
                <div class="accordion-header" onclick="this.parentElement.classList.toggle('open')" vid="51">
                    <h3 vid="52">Frame Settings <span class="badge" vid="53">New!</span></h3>
                    <svg width="24" height="24" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="54"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" vid="55"></path></svg>
                </div>
                <div class="accordion-content" vid="56">
                    <label class="checkbox-wrapper" style="margin-bottom: var(--space-3)" vid="57">
                        <input type="checkbox" vid="58"> 
                        <span style="color: var(--text-main); font-size: 15px; font-weight: 500;" vid="59">Add decorative frame</span>
                    </label>

                    <div class="input-group" vid="60">
                        <label vid="61">Frame Text</label>
                        <textarea rows="2" vid="62">Scan me</textarea>
                    </div>

                    <div class="grid-2" vid="63">
                        <div class="color-picker-wrapper" vid="64">
                            <div class="color-swatch" style="background: #ffffff; border: 1px solid #ddd;" vid="65"></div>
                            <span style="font-size: 13px; font-weight: 600;" vid="66">Text Color</span>
                        </div>
                        <div class="color-picker-wrapper" vid="67">
                            <div class="color-swatch" style="background: #000000" vid="68"></div>
                            <span style="font-size: 13px; font-weight: 600;" vid="69">Background</span>
                        </div>
                    </div>
                </div>
            </div>

            
            <div class="accordion-item open" vid="70">
                <div class="accordion-header" onclick="this.parentElement.classList.toggle('open')" vid="71">
                    <h3 vid="72">QR Code Settings</h3>
                    <svg width="24" height="24" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="73"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" vid="74"></path></svg>
                </div>
                <div class="accordion-content" vid="75">
                    
                    <div class="input-group" style="display: flex; gap: var(--space-2);" vid="76">
                        <div style="flex:1;" vid="77">
                            <label vid="78">Style Preset</label>
                            <select vid="79">
                                <option vid="80">Modern Minimal</option>
                                <option vid="81">Classic Black</option>
                                <option vid="82">Rounded Dot</option>
                            </select>
                        </div>
                        <div style="margin-top: 28px;" vid="83">
                            <button class="btn btn-secondary" style="width: 48px; padding: 0;" vid="84">
                                <svg width="22" height="22" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="85"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" vid="86"></path></svg>
                            </button>
                        </div>
                    </div>

                    <div class="input-group" vid="87">
                        <label vid="88">Data Content</label>
                        <div class="segmented-control" style="width: fit-content; margin-bottom: 16px;" vid="89">
                            <button class="segmented-btn active" vid="90">Single</button>
                            <button class="segmented-btn" vid="91">Batch</button>
                        </div>
                        <textarea placeholder="https://example.com" rows="3" vid="92"></textarea>
                        <button class="btn btn-secondary" style="margin-top: 12px; font-size: 13px; height: 36px; background: white; border: 1px solid var(--border-light);" onclick="document.querySelector('.modal-overlay').style.display='flex'" vid="93">
                            Open Data Templates
                        </button>
                    </div>

                    <div class="input-group" vid="94">
                        <label vid="95">Colors</label>
                        <div class="grid-auto" vid="96">
                            <div class="color-picker-wrapper" vid="97">
                                <div class="color-swatch" style="background: #000000" vid="98"></div>
                                <span style="font-size: 13px; font-weight: 600;" vid="99">Dots</span>
                            </div>
                            <div class="color-picker-wrapper" vid="100">
                                <div class="color-swatch" style="background: #ffffff; border: 1px solid #ddd;" vid="101"></div>
                                <span style="font-size: 13px; font-weight: 600;" vid="102">Bg</span>
                            </div>
                            <div class="color-picker-wrapper" vid="103">
                                <div class="color-swatch" style="background: #000000" vid="104"></div>
                                <span style="font-size: 13px; font-weight: 600;" vid="105">Corners</span>
                            </div>
                        </div>
                    </div>

                    <div class="grid-3" vid="106">
                        <div class="input-group" vid="107">
                            <label vid="108">Width</label>
                            <input type="number" value="1080" vid="109">
                            
                            <div class="tick-track" vid="110">
                                <div class="tick" vid="111"></div><div class="tick" vid="112"></div><div class="tick" vid="113"></div><div class="tick" vid="114"></div><div class="tick" vid="115"></div>
                                <div class="tick" vid="116"></div><div class="tick" vid="117"></div><div class="tick" vid="118"></div><div class="tick" vid="119"></div><div class="tick" vid="120"></div>
                            </div>
                        </div>
                        <div class="input-group" vid="121">
                            <label vid="122">Height</label>
                            <input type="number" value="1080" vid="123">
                            <div class="tick-track" vid="124">
                                <div class="tick" vid="125"></div><div class="tick" vid="126"></div><div class="tick" vid="127"></div><div class="tick" vid="128"></div><div class="tick" vid="129"></div>
                                <div class="tick" vid="130"></div><div class="tick" vid="131"></div><div class="tick" vid="132"></div><div class="tick" vid="133"></div><div class="tick" vid="134"></div>
                            </div>
                        </div>
                        <div class="input-group" vid="135">
                            <label vid="136">Radius</label>
                            <input type="number" value="16" vid="137">
                            <div class="tick-track" vid="138">
                                <div class="tick" vid="139"></div><div class="tick" vid="140"></div><div class="tick" vid="141"></div><div class="tick" vid="142"></div><div class="tick" vid="143"></div>
                                <div class="tick" vid="144"></div><div class="tick" vid="145"></div><div class="tick" vid="146"></div><div class="tick" vid="147"></div><div class="tick" vid="148"></div>
                            </div>
                        </div>
                    </div>

                    <div class="input-group" vid="149">
                        <label vid="150">Error Correction</label>
                        <div style="display: flex; gap: 16px; margin-top: 12px;" vid="151">
                            <label class="checkbox-wrapper" vid="152"><input type="radio" name="ecc" value="L" vid="153"> <span style="color:var(--text-main); font-weight:600;" vid="154">L</span></label>
                            <label class="checkbox-wrapper" vid="155"><input type="radio" name="ecc" value="M" vid="156"> <span style="color:var(--text-main); font-weight:600;" vid="157">M</span></label>
                            <label class="checkbox-wrapper" vid="158"><input type="radio" name="ecc" value="Q" checked="" vid="159"> <span style="color:var(--text-main); font-weight:600;" vid="160">Q <span class="badge" style="margin-left:4px; font-size: 9px; padding: 2px 6px;" vid="161">Rec</span></span></label>
                            <label class="checkbox-wrapper" vid="162"><input type="radio" name="ecc" value="H" vid="163"> <span style="color:var(--text-main); font-weight:600;" vid="164">H</span></label>
                        </div>
                    </div>

                </div>
            </div>

        </div>
    </main>

    
    <main id="view-scan" class="hidden" vid="165">
        <div class="scan-mode-container" vid="166">
            <h2 class="font-light" style="font-size: 36px; margin-bottom: 16px;" vid="167">Scan QR Code</h2>
            <p style="margin-bottom: 48px; font-size: 16px;" vid="168">Point your camera or upload an image file</p>
            
            <div class="camera-viewport" vid="169">
                <div class="scan-overlay" vid="170"></div>
                
                <div style="position: absolute; inset:0; z-index:-1; background: #111; display: flex; align-items: center; justify-content: center; color: #444; font-weight: 600;" vid="171">
                    [ Camera Feed ]
                </div>
            </div>

            <div class="btn-group" vid="172">
                <button class="btn btn-primary" style="background: white; color: black;" vid="173">Start Camera</button>
                <button class="btn btn-secondary" style="background: rgba(255,255,255,0.1); color: white;" vid="174">Upload Image</button>
            </div>
        </div>
    </main>

    
    <div class="modal-overlay" vid="175">
        <div class="modal" vid="176">
            <div class="modal-header" vid="177">
                <h3 vid="178">Data Templates</h3>
                <button class="btn-icon" onclick="document.querySelector('.modal-overlay').style.display='none'" vid="179">
                    <svg width="24" height="24" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="180"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" vid="181"></path></svg>
                </button>
            </div>
            <div class="modal-body" vid="182">
                <div class="segmented-control" style="margin-bottom: 32px; overflow-x: auto; background: var(--bg-input);" vid="183">
                    <button class="segmented-btn active" vid="184">URL</button>
                    <button class="segmented-btn" vid="185">Text</button>
                    <button class="segmented-btn" vid="186">WiFi</button>
                    <button class="segmented-btn" vid="187">vCard</button>
                    <button class="segmented-btn" vid="188">Email</button>
                </div>

                <div class="input-group" vid="189">
                    <label vid="190">Website URL</label>
                    <input type="text" placeholder="https://" vid="191">
                </div>
                
                <div class="input-group" vid="192">
                    <label vid="193">UTM Source (Optional)</label>
                    <input type="text" placeholder="e.g. newsletter" vid="194">
                </div>

                <div style="margin-top: 32px; display: flex; justify-content: flex-end; gap: var(--space-2);" vid="195">
                    <button class="btn btn-secondary" onclick="document.querySelector('.modal-overlay').style.display='none'" vid="196">Cancel</button>
                    <button class="btn btn-primary" vid="197">Apply Data</button>
                </div>
            </div>
        </div>
    </div>

    <footer vid="198">
        <span vid="199">cheQR v1.0</span>
        <span vid="200">•</span>
        <a href="#" vid="201">Free &amp; Open Source</a>
        <span vid="202">•</span>
        <a href="#" vid="203">Credits</a>
    </footer>

    <script vid="204">
        
        const createBtn = document.getElementById('mode-create');
        const scanBtn = document.getElementById('mode-scan');
        const viewCreate = document.getElementById('view-create');
        const viewScan = document.getElementById('view-scan');

        function setMode(mode) {
            if (mode === 'create') {
                createBtn.classList.add('active');
                scanBtn.classList.remove('active');
                viewCreate.classList.remove('hidden');
                viewCreate.style.display = 'grid'; 
                viewScan.classList.add('hidden');
                viewScan.style.display = 'none';
            } else {
                createBtn.classList.remove('active');
                scanBtn.classList.add('active');
                viewCreate.classList.add('hidden');
                viewCreate.style.display = 'none';
                viewScan.classList.remove('hidden');
                viewScan.style.display = 'flex'; 
            }
        }

        createBtn.addEventListener('click', () => setMode('create'));
        scanBtn.addEventListener('click', () => setMode('scan'));
        
        
        function handleResize() {
            if(window.innerWidth <= 1024 && !viewCreate.classList.contains('hidden')) {
                 viewCreate.style.display = 'flex';
            } else if (!viewCreate.classList.contains('hidden')) {
                viewCreate.style.display = 'grid';
            }
        }
        window.addEventListener('resize', handleResize);
        handleResize();
    </script>

</body></html>
```
