The code below contains a design. This design should be used to create a new app or be added to an existing one.

Look at the current open project to determine if a project exists. If no project is open, create a new Vite project then create this view in React after componentizing it.

If a project does exist, determine the framework being used and implement the design within that framework. Identify whether reusable components already exist that can be used to implement the design faithfully and if so use them, otherwise create new components. If other views already exist in the project, make sure to place the view in a sensible route and connect it to the other views.

Ensure the visual characteristics, layout, and interactions in the design are preserved with perfect fidelity.

Run the dev command so the user can see the app once finished.

```
<html lang="en" vid="0"><head vid="1">
    <meta charset="UTF-8" vid="2">
    <meta name="viewport" content="width=device-width, initial-scale=1.0" vid="3">
    <title vid="4">cheQR - Welcome</title>
    <style vid="5">
        :root {
            
            --bg-body: #09090b;
            --bg-surface: #ffffff;
            --bg-surface-hover: #f4f4f5;
            --bg-secondary: #27272a;
            
            
            --text-main: #ffffff;
            --text-inverse: #09090b;
            --text-muted: #a1a1aa;
            --text-muted-inverse: #71717a;
            
            
            --accent-primary: #ffffff;
            --accent-hover: #e4e4e7;
            --accent-highlight: #fde047; 
            
            
            --border-light: #27272a;
            --border-surface: #e4e4e7;
            
            
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
            --space-6: 64px;
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
            line-height: 1.6;
            -webkit-font-smoothing: antialiased;
            display: flex;
            flex-direction: column;
            min-height: 100vh;
            overflow-x: hidden;
        }

        header {
            height: 80px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 var(--space-5);
            background: var(--bg-body);
            position: sticky;
            top: 0;
            z-index: 100;
        }

        .logo {
            display: flex;
            align-items: center;
            gap: var(--space-2);
            font-size: 20px;
            font-weight: 700;
            letter-spacing: -0.02em;
            color: var(--text-main);
        }
        
        .logo svg { color: var(--text-main); }

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
            transition: all 0.2s;
            border: 1px solid transparent;
            white-space: nowrap;
            gap: var(--space-2);
            text-decoration: none;
        }

        .btn-primary { 
            background: var(--accent-primary); 
            color: var(--text-inverse); 
        }
        .btn-primary:hover { 
            background: var(--accent-hover); 
            transform: translateY(-1px);
        }
        
        .btn-secondary { 
            background: var(--bg-secondary); 
            border: 1px solid var(--border-light); 
            color: var(--text-main); 
        }
        .btn-secondary:hover { 
            background: #3f3f46; 
        }

        .hero-section {
            max-width: 1200px;
            margin: 0 auto;
            padding: var(--space-6) var(--space-5);
            text-align: center;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: var(--space-4);
        }

        .hero-title {
            font-size: 56px;
            font-weight: 800;
            letter-spacing: -0.04em;
            line-height: 1.1;
            max-width: 800px;
            color: var(--text-main);
        }

        .hero-subtitle {
            font-size: 18px;
            color: var(--text-muted);
            max-width: 580px;
            margin-bottom: var(--space-3);
            font-weight: 400;
        }

        .onboarding-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: var(--space-4);
            max-width: 1200px;
            margin: var(--space-4) auto var(--space-6);
            padding: 0 var(--space-5);
        }

        
        .onboarding-card {
            background: var(--bg-surface);
            border: none;
            border-radius: var(--radius-lg);
            padding: 40px var(--space-5);
            text-align: left;
            transition: transform 0.2s ease;
            cursor: pointer;
            color: var(--text-inverse);
            position: relative;
            overflow: hidden;
        }

        .onboarding-card:hover {
            transform: translateY(-4px);
        }

        .onboarding-card h3 {
            font-weight: 700;
            font-size: 20px;
            margin-bottom: 8px;
            letter-spacing: -0.02em;
        }

        .card-icon {
            width: 56px;
            height: 56px;
            background: var(--bg-surface-hover);
            border-radius: var(--radius-md);
            display: flex;
            align-items: center;
            justify-content: center;
            color: var(--text-inverse);
            margin-bottom: var(--space-4);
        }

        .template-section {
            background: transparent;
            padding: var(--space-6) 0;
            border-top: 1px solid var(--border-light);
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 0 var(--space-5);
        }

        .section-header {
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
            margin-bottom: var(--space-5);
        }

        .template-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
            gap: var(--space-4);
        }

        
        .template-item {
            background: var(--bg-surface);
            border: none;
            border-radius: 24px;
            padding: var(--space-3);
            display: flex;
            flex-direction: column;
            gap: var(--space-3);
            transition: all 0.2s;
            color: var(--text-inverse);
        }

        .template-item:hover {
            transform: scale(1.02);
        }

        .template-preview {
            aspect-ratio: 1;
            background: var(--bg-surface-hover);
            border-radius: var(--radius-md);
            display: flex;
            align-items: center;
            justify-content: center;
            padding: var(--space-4);
        }

        .template-preview img {
            width: 100%;
            height: 100%;
            object-fit: contain;
            opacity: 1;
            
            filter: grayscale(100%) contrast(1.2);
        }

        .template-info h4 {
            font-size: 16px;
            font-weight: 700;
            margin-bottom: 4px;
            color: var(--text-inverse);
        }

        .template-info p {
            font-size: 13px;
            color: var(--text-muted-inverse);
        }

        .badge {
            display: inline-flex;
            align-items: center;
            padding: 4px 12px;
            background: var(--accent-highlight);
            color: var(--text-inverse);
            font-size: 12px;
            border-radius: var(--radius-pill);
            text-transform: uppercase;
            letter-spacing: 0.05em;
            font-weight: 800;
        }

        footer {
            height: 80px;
            border-top: 1px solid var(--border-light);
            display: flex;
            align-items: center;
            justify-content: center;
            gap: var(--space-4);
            font-size: 13px;
            color: var(--text-muted);
            margin-top: auto;
        }

        footer a { color: var(--text-muted); text-decoration: none; transition: color 0.2s;}
        footer a:hover { color: var(--text-main); }
        
        
        p[style*="color: var(--text-muted)"] {
            color: inherit !important;
            opacity: 0.6;
        }

        @media (max-width: 900px) {
            .onboarding-grid {
                grid-template-columns: 1fr;
            }
            .hero-title {
                font-size: 40px;
            }
        }
    </style>
</head>
<body vid="6">

    <header vid="7">
        <div class="logo" vid="8">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" vid="9">
                <rect x="2" y="2" width="9" height="9" rx="2" stroke="currentColor" stroke-width="2.5" vid="10"></rect>
                <rect x="13" y="2" width="9" height="9" rx="2" stroke="currentColor" stroke-width="2.5" vid="11"></rect>
                <rect x="2" y="13" width="9" height="9" rx="2" stroke="currentColor" stroke-width="2.5" vid="12"></rect>
                <path d="M16 16H20V20" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" vid="13"></path>
            </svg>
            cheQR
        </div>
        
        <div class="btn-group" vid="14">
            <button class="btn btn-primary" onclick="location.reload()" vid="15">Create New</button>
        </div>
    </header>

    <main vid="16">
        <section class="hero-section" vid="17">
            <div class="badge" vid="18">Beta v1.0</div>
            <h1 class="hero-title" vid="19">Design beautiful QR codes in seconds.</h1>
            <p class="hero-subtitle" vid="20">Welcome to cheQR. Create, scan, and manage custom branded QR codes with advanced styling options and batch processing.</p>
            <div style="display: flex; gap: var(--space-3); flex-wrap: wrap; justify-content: center;" vid="21">
                <button class="btn btn-primary" style="height: 56px; padding: 0 40px; font-size: 16px;" vid="22">Start Creating Now</button>
                <button class="btn btn-secondary" style="height: 56px; padding: 0 40px; font-size: 16px;" vid="23">View Documentation</button>
            </div>
        </section>

        <div class="onboarding-grid" vid="24">
            <div class="onboarding-card" vid="25">
                <div class="card-icon" vid="26">
                    <svg width="28" height="28" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="27"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 21a4 4 0 01-4-4V5a2 2 0 012-2h4a2 2 0 012 2v12a4 4 0 01-4 4zm0 0h12a2 2 0 002-2v-4a2 2 0 00-2-2h-2.343M11 7.343l1.172-1.172a4 4 0 115.656 5.656L10 17.657" vid="28"></path></svg>
                </div>
                <h3 vid="29">Custom Styling</h3>
                <p style="color: var(--text-muted-inverse); font-size: 14px; margin-top: 8px; line-height: 1.5;" vid="30">Personalize colors, dot shapes, and corner radiuses to match your brand identity perfectly.</p>
            </div>
            <div class="onboarding-card" vid="31">
                <div class="card-icon" vid="32">
                    <svg width="28" height="28" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="33"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10" vid="34"></path></svg>
                </div>
                <h3 vid="35">Batch Export</h3>
                <p style="color: var(--text-muted-inverse); font-size: 14px; margin-top: 8px; line-height: 1.5;" vid="36">Generate hundreds of QR codes at once using CSV data. Ideal for inventory or event tracking.</p>
            </div>
            <div class="onboarding-card" vid="37">
                <div class="card-icon" vid="38">
                    <svg width="28" height="28" fill="none" stroke="currentColor" viewBox="0 0 24 24" vid="39"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" vid="40"></path></svg>
                </div>
                <h3 vid="41">Frame Layouts</h3>
                <p style="color: var(--text-muted-inverse); font-size: 14px; margin-top: 8px; line-height: 1.5;" vid="42">Add CTA text and decorative frames to increase scan rates and improve professional appearance.</p>
            </div>
        </div>

        <section class="template-section" vid="43">
            <div class="container" vid="44">
                <div class="section-header" vid="45">
                    <div vid="46">
                        <h2 style="font-size: 24px; font-weight: 600; color: var(--text-main);" vid="47">Quick Start Templates</h2>
                        <p style="color: var(--text-muted); margin-top: 4px;" vid="48">Choose a preset to jumpstart your design</p>
                    </div>
                    <button class="btn btn-secondary" vid="49">View All Templates</button>
                </div>

                <div class="template-grid" vid="50">
                    <div class="template-item" vid="51">
                        <div class="template-preview" vid="52">
                            <img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiB2aWV3Qm94PSIwIDAgMTAwIDEwMCI+PHBhdGggZmlsbD0iIzE1ODA2MiIgZD0iTTAgMGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwem0zMC0xMGgzMHYzMEg0MHptNCA0aDIydjIySDQ0em02IDZoMTB2MTBINTB6bS0xMCAyMGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMjAgMGgxMHYxMEg3MHpNMCA0MGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwem00MCAzMGgxMHYxMEg4MHoiLz48L3N2Zz4=" alt="Minimal" vid="53">
                        </div>
                        <div class="template-info" vid="54">
                            <h4 vid="55">Modern Minimal</h4>
                            <p vid="56">Clean, high-contrast dots</p>
                        </div>
                    </div>
                    <div class="template-item" vid="57">
                        <div class="template-preview" vid="58">
                            <img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiB2aWV3Qm94PSIwIDAgMTAwIDEwMCI+PHBhdGggZmlsbD0iIzA0MzMyNSIgZD0iTTAgMGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwem0zMC0xMGgzMHYzMEg0MHptNCA0aDIydjIySDQ0em02IDZoMTB2MTBINTB6bS0xMCAyMGgxMHYxMEg0MHptMTAgMGgxMHYxMEg1MHptMjAgMGgxMHYxMEg3MHpNMCA0MGgzMHYzMEgwem00IDRoMjJ2MjJINFptNiA2aDEwdjEwSDEwem00MCAzMGgxMHYxMEg4MHoiLz48L3N2Zz4=" alt="Classic" style="filter: brightness(2);" vid="59">
                        </div>
                        <div class="template-info" vid="60">
                            <h4 vid="61">Classic Business</h4>
                            <p vid="62">Square modules, sharp corners</p>
                        </div>
                    </div>
                    <div class="template-item" vid="63">
                        <div class="template-preview" vid="64">
                            <img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiB2aWV3Qm94PSIwIDAgMTAwIDEwMCI+PGNpcmNsZSBjeD0iNSIgY3k9IjUiIHI9IjMiIGZpbGw9IiMxNTgwNjIiLz48Y2lyY2xlIGN4PSIxNSIgY3k9IjUiIHI9IjMiIGZpbGw9IiMxNTgwNjIiLz48Y2lyY2xlIGN4PSI1IiBjeT0iMTUiIHI9IjMiIGZpbGw9IiMxNTgwNjIiLz48L3N2Zz4=" alt="Soft" vid="65">
                        </div>
                        <div class="template-info" vid="66">
                            <h4 vid="67">Soft Dots</h4>
                            <p vid="68">Friendly, rounded circular units</p>
                        </div>
                    </div>
                    <div class="template-item" vid="69">
                        <div class="template-preview" vid="70" style="background: #e4e4e7;">
                            <img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiB2aWV3Qm94PSIwIDAgMTAwIDEwMCI+PHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0wIDBoMzB2MzBIMHptNCA0aDIydjIySDRabTYgNmgxMHYxMEgxMHptMzAtMTBoMzB2MzBINTB6bTAtMjBoMTB2MTBINDB6bTQwIDYwaDEwdjEwSDgweiIvPjwvc32zPg==" alt="Inverted" vid="71" style="filter: invert(1);">
                        </div>
                        <div class="template-info" vid="72">
                            <h4 vid="73">High Impact</h4>
                            <p vid="74">Vibrant inverted background style</p>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </main>

    <footer vid="75">
        <span vid="76">cheQR v1.0</span>
        <span vid="77">•</span>
        <a href="#" vid="78">Free &amp; Open Source</a>
        <span vid="79">•</span>
        <a href="#" vid="80">Privacy Policy</a>
    </footer>

</body></html>
```
