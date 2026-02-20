The code below contains a design. This design should be used to create a new app or be added to an existing one.

Look at the current open project to determine if a project exists. If no project is open, create a new Vite project then create this view in React after componentizing it.

If a project does exist, determine the framework being used and implement the design within that framework. Identify whether reusable components already exist that can be used to implement the design faithfully and if so use them, otherwise create new components. If other views already exist in the project, make sure to place the view in a sensible route and connect it to the other views.

Ensure the visual characteristics, layout, and interactions in the design are preserved with perfect fidelity.

Run the dev command so the user can see the app once finished.

```
<html lang="en" vid="0"><head vid="1">
    <meta charset="UTF-8" vid="2">
    <meta name="viewport" content="width=device-width, initial-scale=1.0" vid="3">
    <title vid="4">cheQR - Batch Generation</title>
    <style vid="5">
        :root {
            
            --bg-body: #050505;
            --bg-surface: #FFFFFF;
            --bg-surface-secondary: #F3F4F6;
            
            
            --text-main: #111111;
            --text-inverse: #FFFFFF;
            --text-muted: #6B7280;
            --text-placeholder: #9CA3AF;
            
            
            --input-bg: #F3F4F6;
            --border-light: #E5E7EB;
            --border-active: #000000;
            
            
            --accent-primary: #000000;
            --accent-hover: #333333;
            --btn-secondary-bg: #F3F4F6;
            --btn-secondary-hover: #E5E7EB;

            
            --radius-sm: 8px;
            --radius-md: 16px;
            --radius-lg: 32px;
            --radius-pill: 999px;
            
            --font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            
            --space-1: 4px;
            --space-2: 8px;
            --space-3: 16px;
            --space-4: 24px;
            --space-5: 32px;
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
            font-size: 14px;
            line-height: 1.5;
            -webkit-font-smoothing: antialiased;
            display: flex;
            flex-direction: column;
            height: 100vh;
            overflow: hidden;
        }

        
        header {
            height: 80px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 var(--space-5);
            background: var(--bg-body);
            color: var(--text-inverse);
            flex-shrink: 0;
            border-bottom: none;
        }

        .logo {
            display: flex;
            align-items: center;
            gap: var(--space-2);
            font-size: 20px;
            font-weight: 600;
            color: var(--text-inverse);
        }
        
        .logo svg { 
            color: var(--text-inverse); 
        }

        
        header .segmented-control {
            background: #1F1F1F;
            border: none;
            padding: 4px;
        }

        header .segmented-btn {
            color: #888;
        }

        header .segmented-btn.active {
            background: #FFFFFF;
            color: #000000;
        }

        
        main {
            flex: 1;
            display: grid;
            grid-template-columns: 380px 1fr;
            max-width: 1600px;
            margin: 0 auto;
            width: 100%;
            overflow: hidden;
            padding: 0 var(--space-4) var(--space-4) var(--space-4);
            gap: var(--space-4);
        }

        
        .sidebar {
            background: var(--bg-surface);
            border-radius: var(--radius-lg);
            padding: var(--space-5);
            display: flex;
            flex-direction: column;
            gap: var(--space-4);
            overflow-y: auto;
            border: none;
            color: var(--text-main);
        }

        
        .sidebar textarea, 
        .sidebar input[type="text"] {
            background: var(--input-bg);
            border: 1px solid transparent;
            color: var(--text-main);
            border-radius: var(--radius-md);
            padding: 16px;
            font-size: 14px;
            transition: all 0.2s;
        }

        .sidebar textarea:focus, 
        .sidebar input[type="text"]:focus {
            outline: none;
            background: #FFFFFF;
            border-color: var(--border-light);
            box-shadow: 0 0 0 2px rgba(0,0,0,0.05);
        }

        
        .sidebar .segmented-control {
            background: var(--bg-surface-secondary);
            border: none;
            padding: 4px;
        }

        .sidebar .segmented-btn {
            color: var(--text-muted);
            font-weight: 500;
        }

        .sidebar .segmented-btn.active {
            background: #FFFFFF;
            color: #000000;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
        }

        
        .content-panel {
            background: var(--bg-surface);
            border-radius: var(--radius-lg);
            padding: var(--space-5);
            overflow-y: auto;
            color: var(--text-main);
        }

        
        
        .segmented-control {
            border-radius: var(--radius-pill);
            display: flex;
        }

        .segmented-btn {
            background: transparent;
            border: none;
            padding: 8px 20px;
            font-size: 14px;
            border-radius: var(--radius-pill);
            cursor: pointer;
            transition: all 0.2s ease;
            font-weight: 600;
            flex: 1;
        }

        .btn {
            display: inline-flex;
            align-items: center;
            justify-content: center;
            height: 44px;
            padding: 0 var(--space-4);
            border-radius: var(--radius-pill);
            font-size: 14px;
            font-weight: 600;
            cursor: pointer;
            border: none;
            gap: var(--space-2);
            transition: transform 0.1s, opacity 0.2s;
        }
        
        .btn:active {
            transform: scale(0.98);
        }

        .btn-primary { 
            background: var(--accent-primary); 
            color: white; 
        }
        .btn-primary:hover {
            background: var(--accent-hover);
        }

        .btn-secondary { 
            background: var(--btn-secondary-bg); 
            color: var(--text-main); 
            border: 1px solid transparent;
        }
        .btn-secondary:hover {
            background: var(--btn-secondary-hover);
        }

        .input-group { margin-bottom: var(--space-4); }
        
        label { 
            display: block; 
            font-size: 13px; 
            font-weight: 600;
            color: var(--text-main); 
            margin-bottom: var(--space-2); 
        }
        
        
        .batch-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
            gap: var(--space-4);
        }

        .qr-card {
            background: #FFFFFF;
            border: 1px solid var(--border-light);
            border-radius: var(--radius-md);
            padding: var(--space-3);
            transition: all 0.2s;
            position: relative;
        }

        .qr-card:hover {
            border-color: #000;
            box-shadow: 0 4px 12px rgba(0,0,0,0.05);
            transform: translateY(-2px);
        }

        .qr-card-preview {
            background: var(--bg-surface-secondary);
            aspect-ratio: 1;
            border-radius: var(--radius-sm);
            margin-bottom: var(--space-3);
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        
        
        .qr-card-preview img {
            filter: brightness(0);
            opacity: 0.9;
        }

        .qr-card-info {
            font-size: 13px;
        }

        .qr-card-title {
            font-weight: 600;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            margin-bottom: 4px;
            color: var(--text-main);
        }

        .qr-card-meta {
            color: var(--text-muted);
            display: flex;
            justify-content: space-between;
            font-size: 12px;
            font-weight: 500;
        }

        
        .selection-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: var(--space-5);
            padding-bottom: var(--space-4);
            border-bottom: 1px solid var(--border-light);
        }
        
        .selection-header span {
            font-weight: 600;
        }

        
        .checkbox-custom {
            width: 20px;
            height: 20px;
            border-radius: 6px;
            border: 2px solid var(--border-light);
            background: white;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: all 0.2s;
        }
        
        
        .qr-card .checkbox-custom {
            position: absolute;
            top: 12px;
            right: 12px;
            border-color: transparent;
            background: white;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        .checkbox-custom.checked {
            background: var(--accent-primary);
            border-color: var(--accent-primary);
        }
        
        .checkbox-custom svg {
            stroke: white;
            stroke-width: 3px;
        }

        
        footer {
            height: 48px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: var(--space-4);
            font-size: 12px;
            color: rgba(255,255,255,0.4);
            background: var(--bg-body);
            flex-shrink: 0;
            border-top: none;
        }
        
        footer a {
            color: rgba(255,255,255,0.4) !important;
            transition: color 0.2s;
        }
        
        footer a:hover {
            color: rgba(255,255,255,0.8) !important;
        }

        
        .w-full { width: 100%; }
        
        
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
        ::-webkit-scrollbar-thumb:hover {
            background: rgba(0,0,0,0.2);
        }
        
        
    </style>
</head>
<body vid="6">

    <header vid="7">
        <div class="logo" vid="8">
            <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" vid="9">
                <rect x="2" y="2" width="9" height="9" rx="2" vid="10"></rect>
                <rect x="13" y="2" width="9" height="9" rx="2" vid="11"></rect>
                <rect x="2" y="13" width="9" height="9" rx="2" vid="12"></rect>
                <path d="M16 16H20V20" stroke-linecap="round" vid="13"></path>
            </svg>
            cheQR
        </div>
        <div class="segmented-control" style="width: 220px" vid="14">
            <button class="segmented-btn active" vid="15">Create</button>
            <button class="segmented-btn" vid="16">Scan</button>
        </div>
        <div style="width: 100px; display: flex; justify-content: flex-end; gap: 12px;" vid="17">
             
            <div style="width: 32px; height: 32px; background: #1F1F1F; border-radius: 50%; display: flex; align-items: center; justify-content: center;" vid="18">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2" vid="19"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z" vid="20"></path></svg>
            </div>
        </div>
    </header>

    <main vid="21">
        <aside class="sidebar" vid="22">
            <div vid="23">
                <h2 style="font-size: 24px; margin-bottom: 8px; font-weight: 700;" vid="24">Batch Mode</h2>
                <p style="color: var(--text-muted); font-size: 14px; margin-bottom: 32px;" vid="25">Generate multiple codes at once</p>
                
                <div class="input-group" vid="26">
                    <label vid="27">Data Source</label>
                    <div class="segmented-control" style="margin-bottom: 16px;" vid="28">
                        <button class="segmented-btn active" vid="29">CSV/Text List</button>
                        <button class="segmented-btn" vid="30">Sequential</button>
                    </div>
                    <textarea rows="8" placeholder="Enter one URL or text per line..." vid="31">https://google.com
https://github.com
https://dribbble.com
https://figma.com
https://twitter.com
https://linkedin.com</textarea>
                </div>

                <div class="input-group" vid="32">
                    <label vid="33">File Naming Pattern</label>
                    <input type="text" value="qr_{index}_{data}" vid="34">
                </div>

                <div style="margin-top: 32px; display: flex; flex-direction: column; gap: 12px;" vid="35">
                    <button class="btn btn-primary w-full" vid="36">
                        <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="37"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" vid="38"></path></svg>
                        Refresh All Codes
                    </button>
                    <button class="btn btn-secondary w-full" vid="39">
                        <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="40"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6V4m0 2a2 2 0 100 4m0-4a2 2 0 110 4m-6 8a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4m6 6v10m6-2a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4" vid="41"></path></svg>
                        Global Style Settings
                    </button>
                </div>
            </div>
        </aside>

        <section class="content-panel" vid="42">
            <div class="selection-header" vid="43">
                <div style="display: flex; align-items: center; gap: 16px;" vid="44">
                    <label class="checkbox-custom checked" vid="45">
                        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="3" vid="46"><path d="M20 6L9 17l-5-5" vid="47"></path></svg>
                    </label>
                    <span vid="48">Selected 6 of 6 items</span>
                </div>
                <div style="display: flex; gap: 12px;" vid="49">
                    <button class="btn btn-secondary" vid="50">Delete Selected</button>
                    <button class="btn btn-primary" vid="51">
                        <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="52"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" vid="53"></path></svg>
                        Bulk Export (.zip)
                    </button>
                </div>
            </div>

            <div class="batch-grid" vid="54">
                
                <div class="qr-card" vid="55">
                    <div class="checkbox-custom checked" vid="56">
                        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="3" vid="57"><path d="M20 6L9 17l-5-5" vid="58"></path></svg>
                    </div>
                    <div class="qr-card-preview" vid="59">
                        <img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiB2aWV3Qm94PSIwIDAgMTAwIDEwMCI+PHBhdGggZmlsbD0iIzE1ODA2MiIgZD0iTTAgMGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwem0zMC0xMGgzMHYzMEg0MHptNCA0aDIydjIySDQ0em02IDZoMTB2MTBINTB6bS0xMCAyMGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMjAgMGgxMHYxMEg3MHpNMCA0MGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwek00MCA3MGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMTAgMGgxMHYxMEg2MHptMTAgMGgxMHYxMEg3MHptMTAgMGgxMHYxMEg4MHptLTEwIDEwaDEwdjEwSDgwem0tMjAgMGgxMHYxMEg2MHoiLz48L3N2Zz4=" width="120" vid="60">
                    </div>
                    <div class="qr-card-info" vid="61">
                        <div class="qr-card-title" vid="62">google.com</div>
                        <div class="qr-card-meta" vid="63">
                            <span vid="64">1080px</span>
                            <span vid="65">PNG</span>
                        </div>
                    </div>
                </div>

                
                <div class="qr-card" vid="66">
                    <div class="checkbox-custom checked" vid="67">
                        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="3" vid="68"><path d="M20 6L9 17l-5-5" vid="69"></path></svg>
                    </div>
                    <div class="qr-card-preview" vid="70">
                        <img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiB2aWV3Qm94PSIwIDAgMTAwIDEwMCI+PHBhdGggZmlsbD0iIzE1ODA2MiIgZD0iTTAgMGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwem0zMC0xMGgzMHYzMEg0MHptNCA0aDIydjIySDQ0em02IDZoMTB2MTBINTB6bS0xMCAyMGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMjAgMGgxMHYxMEg3MHpNMCA0MGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwek00MCA3MGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMTAgMGgxMHYxMEg2MHptMTAgMGgxMHYxMEg3MHptMTAgMGgxMHYxMEg4MHptLTEwIDEwaDEwdjEwSDgwem0tMjAgMGgxMHYxMEg2MHoiLz48L3N2Zz4=" width="120" vid="71">
                    </div>
                    <div class="qr-card-info" vid="72">
                        <div class="qr-card-title" vid="73">github.com</div>
                        <div class="qr-card-meta" vid="74">
                            <span vid="75">1080px</span>
                            <span vid="76">PNG</span>
                        </div>
                    </div>
                </div>

                
                <div class="qr-card" vid="77">
                    <div class="checkbox-custom checked" vid="78">
                        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="3" vid="79"><path d="M20 6L9 17l-5-5" vid="80"></path></svg>
                    </div>
                    <div class="qr-card-preview" vid="81">
                        <img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiB2aWV3Qm94PSIwIDAgMTAwIDEwMCI+PHBhdGggZmlsbD0iIzE1ODA2MiIgZD0iTTAgMGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwem0zMC0xMGgzMHYzMEg0MHptNCA0aDIydjIySDQ0em02IDZoMTB2MTBINTB6bS0xMCAyMGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMjAgMGgxMHYxMEg3MHpNMCA0MGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwek00MCA3MGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMTAgMGgxMHYxMEg2MHptMTAgMGgxMHYxMEg3MHptMTAgMGgxMHYxMEg4MHptLTEwIDEwaDEwdjEwSDgwem0tMjAgMGgxMHYxMEg2MHoiLz48L3N2Zz4=" width="120" vid="82">
                    </div>
                    <div class="qr-card-info" vid="83">
                        <div class="qr-card-title" vid="84">dribbble.com</div>
                        <div class="qr-card-meta" vid="85">
                            <span vid="86">1080px</span>
                            <span vid="87">PNG</span>
                        </div>
                    </div>
                </div>

                
                <div class="qr-card" vid="88">
                    <div class="checkbox-custom checked" vid="89">
                        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="3" vid="90"><path d="M20 6L9 17l-5-5" vid="91"></path></svg>
                    </div>
                    <div class="qr-card-preview" vid="92">
                        <img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiB2aWV3Qm94PSIwIDAgMTAwIDEwMCI+PHBhdGggZmlsbD0iIzE1ODA2MiIgZD0iTTAgMGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwem0zMC0xMGgzMHYzMEg0MHptNCA0aDIydjIySDQ0em02IDZoMTB2MTBINTB6bS0xMCAyMGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMjAgMGgxMHYxMEg3MHpNMCA0MGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwek00MCA3MGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMTAgMGgxMHYxMEg2MHptMTAgMGgxMHYxMEg3MHptMTAgMGgxMHYxMEg8MHptLTEwIDEwaDEwdjEwSDgwem0tMjAgMGgxMHYxMEg2MHoiLz48L3N2Zz4=" width="120" vid="93">
                    </div>
                    <div class="qr-card-info" vid="94">
                        <div class="qr-card-title" vid="95">figma.com</div>
                        <div class="qr-card-meta" vid="96">
                            <span vid="97">1080px</span>
                            <span vid="98">PNG</span>
                        </div>
                    </div>
                </div>

                
                <div class="qr-card" vid="99">
                    <div class="checkbox-custom checked" vid="100">
                        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="3" vid="101"><path d="M20 6L9 17l-5-5" vid="102"></path></svg>
                    </div>
                    <div class="qr-card-preview" vid="103">
                        <img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiB2aWV3Qm94PSIwIDAgMTAwIDEwMCI+PHBhdGggZmlsbD0iIzE1ODA2MiIgZD0iTTAgMGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwem0zMC0xMGgzMHYzMEg0MHptNCA0aDIydjIySDQ0em02IDZoMTB2MTBINTB6bS0xMCAyMGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMjAgMGgxMHYxMEg3MHpNMCA0MGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwek00MCA3MGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMTAgMGgxMHYxMEg2MHptMTAgMGgxMHYxMEg3MHptMTAgMGgxMHYxMEg8MHptLTEwIDEwaDEwdjEwSDgwem0tMjAgMGgxMHYxMEg2MHoiLz48L3N2Zz4=" width="120" vid="104">
                    </div>
                    <div class="qr-card-info" vid="105">
                        <div class="qr-card-title" vid="106">twitter.com</div>
                        <div class="qr-card-meta" vid="107">
                            <span vid="108">1080px</span>
                            <span vid="109">PNG</span>
                        </div>
                    </div>
                </div>

                
                <div class="qr-card" vid="110">
                    <div class="checkbox-custom checked" vid="111">
                        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="3" vid="112"><path d="M20 6L9 17l-5-5" vid="113"></path></svg>
                    </div>
                    <div class="qr-card-preview" vid="114">
                        <img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiB2aWV3Qm94PSIwIDAgMTAwIDEwMCI+PHBhdGggZmlsbD0iIzE1ODA2MiIgZD0iTTAgMGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwem0zMC0xMGgzMHYzMEg0MHptNCA0aDIydjIySDQ0em02IDZoMTB2MTBINTB6bS0xMCAyMGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMjAgMGgxMHYxMEg3MHpNMCA0MGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwek00MCA3MGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMTAgMGgxMHYxMEg2MHptMTAgMGgxMHYxMEg3MHptMTAgMGgxMHYxMEg8MHptLTEwIDEwaDEwdjEwSDgwem0tMjAgMGgxMHYxMEg2MHoiLz48L3N2Zz4=" width="120" vid="115">
                    </div>
                    <div class="qr-card-info" vid="116">
                        <div class="qr-card-title" vid="117">linkedin.com</div>
                        <div class="qr-card-meta" vid="118">
                            <span vid="119">1080px</span>
                            <span vid="120">PNG</span>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </main>

    <footer vid="121">
        <span vid="122">cheQR v1.0</span>
        <span vid="123">•</span>
        <a href="#" style="color: var(--text-muted); text-decoration: none;" vid="124">Free &amp; Open Source</a>
        <span vid="125">•</span>
        <a href="#" style="color: var(--text-muted); text-decoration: none;" vid="126">Credits</a>
    </footer>


</body></html>
```
