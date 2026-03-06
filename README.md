# nobody
AS circle map
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>砷代谢通路交互图谱 (Arsenic Metabolism)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@300;400;500;700&family=JetBrains+Mono:wght@400;700&display=swap');

        body {
            font-family: 'Noto Sans SC', sans-serif;
            background-color: #0f172a; /* Slate 900 */
            color: #e2e8f0;
            overflow: hidden; /* Prevent scroll on full app */
        }

        .mono {
            font-family: 'JetBrains Mono', monospace;
        }

        /* Custom Scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #1e293b; 
        }
        ::-webkit-scrollbar-thumb {
            background: #475569; 
            border-radius: 3px;
        }

        /* Canvas Container */
        #canvas-container {
            position: relative;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle at center, #1e293b 0%, #0f172a 100%);
            cursor: grab;
        }
        #canvas-container:active {
            cursor: grabbing;
        }

        /* UI Overlay Elements */
        .glass-panel {
            background: rgba(15, 23, 42, 0.85);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            box-shadow: 0 4px 30px rgba(0, 0, 0, 0.5);
        }

        .node-label {
            text-shadow: 0 2px 4px rgba(0,0,0,0.8);
            pointer-events: none;
        }

        .highlight-pulse {
            animation: pulse-glow 2s infinite;
        }

        @keyframes pulse-glow {
            0% { box-shadow: 0 0 0 0 rgba(56, 189, 248, 0.7); }
            70% { box-shadow: 0 0 0 10px rgba(56, 189, 248, 0); }
            100% { box-shadow: 0 0 0 0 rgba(56, 189, 248, 0); }
        }
    </style>
</head>
<body class="h-screen w-screen flex flex-col md:flex-row">

    <!-- Sidebar: Info Panel -->
    <aside class="w-full md:w-1/3 lg:w-1/4 h-1/3 md:h-full glass-panel z-20 flex flex-col border-r border-slate-700 transition-all duration-300 absolute md:relative bottom-0 md:bottom-auto">
        <div class="p-6 border-b border-slate-700 bg-slate-900/50">
            <h1 class="text-xl font-bold text-sky-400 flex items-center gap-2">
                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19.428 15.428a2 2 0 00-1.022-.547l-2.387-.477a6 6 0 00-3.86.517l-.318.158a6 6 0 01-3.86.517L6.05 15.21a2 2 0 00-1.806.547M8 4h8l-1 1v5.172a2 2 0 00.586 1.414l5 5c1.26 1.26.367 3.414-1.415 3.414H4.828c-1.782 0-2.674-2.154-1.414-3.414l5-5A2 2 0 009 10.172V5L8 4z"></path></svg>
                砷代谢通路
            </h1>
            <p class="text-xs text-slate-400 mt-1">点击节点查看详细反应机制</p>
        </div>
        
        <div id="info-content" class="flex-1 overflow-y-auto p-6 space-y-4">
            <!-- Default State -->
            <div class="text-center text-slate-500 mt-10">
                <svg class="w-16 h-16 mx-auto mb-4 opacity-50" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M7 21a4 4 0 01-4-4V5a2 2 0 012-2h4a2 2 0 012 2v12a4 4 0 01-4 4zm0 0h12a2 2 0 002-2v-4a2 2 0 00-2-2h-2.343M11 7.343l1.657-1.657a2 2 0 012.828 0l2.829 2.829a2 2 0 010 2.828l-8.486 8.485M7 17h.01"></path></svg>
                <p>请在右侧图谱中点击任意反应节点<br>查看化学方程式与基因功能</p>
            </div>
        </div>

        <!-- Legend -->
        <div class="p-4 bg-slate-900/80 text-xs border-t border-slate-700 grid grid-cols-2 gap-2">
            <div class="flex items-center gap-2"><span class="w-3 h-3 rounded-full bg-yellow-400 shadow-[0_0_8px_rgba(250,204,21,0.8)]"></span> As(V) 五价砷</div>
            <div class="flex items-center gap-2"><span class="w-3 h-3 rounded-full bg-rose-500 shadow-[0_0_8px_rgba(244,63,94,0.8)]"></span> As(III) 三价砷</div>
            <div class="flex items-center gap-2"><span class="w-3 h-3 rounded-full bg-emerald-400 shadow-[0_0_8px_rgba(52,211,153,0.8)]"></span> 甲基 (CH₃)</div>
            <div class="flex items-center gap-2"><span class="w-3 h-3 rounded-full bg-sky-400 shadow-[0_0_8px_rgba(56,189,248,0.8)]"></span> 电子 (e⁻)</div>
        </div>
    </aside>

    <!-- Main: Canvas Area -->
    <main class="flex-1 relative h-2/3 md:h-full bg-slate-900 overflow-hidden">
        <div id="canvas-container">
            <canvas id="pathwayCanvas"></canvas>
        </div>
        
        <!-- Floating Controls -->
        <div class="absolute top-4 right-4 flex gap-2">
            <button onclick="resetView()" class="glass-panel p-2 rounded-lg hover:bg-slate-700 transition text-slate-300" title="重置视图">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path></svg>
            </button>
        </div>
    </main>

    <script>
        // --- Data Structure ---
        const reactions = [
            {
                id: 'arsC',
                label: 'As(V) 还原',
                gene: 'arsC1 / arsC2',
                x: 0.3, y: 0.5, // Relative position (0-1)
                type: 'reduction',
                equation: 'H₃AsO₄ + 2GSH → H₃AsO₃ + GSSG + H₂O',
                details: '将高毒性的五价砷(As(V))还原为三价砷(As(III))。这是细胞内解毒的第一步，也是后续甲基化或外排的前提。',
                inputs: ['As5_out'],
                outputs: ['As3_in']
            },
            {
                id: 'aioA',
                label: 'As(III) 氧化',
                gene: 'aioA',
                x: 0.5, y: 0.8,
                type: 'oxidation',
                equation: 'H₃AsO₃ + H₂O → H₃AsO₄ + 2H⁺ + 2e⁻',
                details: '在某些细菌中，将三价砷氧化为五价砷以获取能量。电子进入呼吸链。',
                inputs: ['As3_in'],
                outputs: ['As5_out']
            },
            {
                id: 'arsM',
                label: '砷 甲基化',
                gene: 'arsM',
                x: 0.5, y: 0.35,
                type: 'methylation',
                equation: 'H₃AsO₃ + SAM → CH₃AsO(OH)₂ + SAH',
                details: '核心解毒通路。利用S-腺苷甲硫氨酸(SAM)作为甲基供体，将无机砷逐步转化为单甲基(MMAs)、二甲基(DMAs)，最终生成气态三甲基砷(TMAs)挥发。',
                inputs: ['As3_in'],
                outputs: ['MMAs_in', 'DMAs_in', 'TMAs_gas']
            },
            {
                id: 'acr3',
                label: 'As(III) 外排',
                gene: 'acr3 / arsB',
                x: 0.7, y: 0.5,
                type: 'transport',
                equation: 'H₃AsO₃(内) + ATP → H₃AsO₃(外) + ADP + Pi',
                details: '将还原后的三价砷泵出细胞，降低胞内毒性。Acr3是主要的外排泵。',
                inputs: ['As3_in'],
                outputs: ['As3_out']
            },
            {
                id: 'arsH',
                label: '有机砷氧化',
                gene: 'arsH',
                x: 0.7, y: 0.2,
                type: 'oxidation',
                equation: 'CH₃AsO(OH)₂ + ½O₂ → CH₃AsO(OH)₂O',
                details: '将还原态的甲基砷(MMAs(III))氧化为氧化态(MMAs(V))，后者毒性相对较低。',
                inputs: ['MMAs_in'],
                outputs: ['MMAs5_in']
            },
            {
                id: 'arsP',
                label: '甲基砷外排',
                gene: 'arsP',
                x: 0.85, y: 0.35,
                type: 'transport',
                equation: 'CH₃AsO(OH)₂(内) → CH₃AsO(OH)₂(外)',
                details: '特异性转运蛋白，负责将甲基化砷排出细胞。',
                inputs: ['MMAs_in'],
                outputs: ['MMAs_out']
            }
        ];

        // --- Canvas Setup ---
        const canvas = document.getElementById('pathwayCanvas');
        const ctx = canvas.getContext('2d');
        let width, height;
        let particles = [];
        let activeNode = null;
        let animationId;

        // Resize Handler
        function resize() {
            const container = document.getElementById('canvas-container');
            width = container.clientWidth;
            height = container.clientHeight;
            canvas.width = width;
            canvas.height = height;
        }
        window.addEventListener('resize', resize);
        resize();

        // --- Classes ---

        class Particle {
            constructor(type, startNode, endNode) {
                this.type = type; // 'As5', 'As3', 'Methyl', 'Electron'
                this.startNode = startNode;
                this.endNode = endNode;
                this.progress = 0;
                this.speed = 0.005 + Math.random() * 0.005;
                
                // Visual properties based on type
                switch(type) {
                    case 'As5': this.color = '#facc15'; this.size = 4; break; // Yellow
                    case 'As3': this.color = '#f43f5e'; this.size = 4; break; // Rose
                    case 'Methyl': this.color = '#34d399'; this.size = 3; break; // Emerald
                    case 'Electron': this.color = '#38bdf8'; this.size = 2; break; // Sky
                }
            }

            update() {
                this.progress += this.speed;
                return this.progress >= 1;
            }

            draw(ctx) {
                const startX = this.startNode.x * width;
                const startY = this.startNode.y * height;
                const endX = this.endNode.x * width;
                const endY = this.endNode.y * height;

                // Bezier curve calculation for smooth flow
                const cp1x = startX + (endX - startX) * 0.5;
                const cp1y = startY; 
                const cp2x = startX + (endX - startX) * 0.5;
                const cp2y = endY;

                const t = this.progress;
                const x = (1-t)*(1-t)*(1-t)*startX + 3*(1-t)*(1-t)*t*cp1x + 3*(1-t)*t*t*cp2x + t*t*t*endX;
                const y = (1-t)*(1-t)*(1-t)*startY + 3*(1-t)*(1-t)*t*cp1y + 3*(1-t)*t*t*cp2y + t*t*t*endY;

                ctx.beginPath();
                ctx.arc(x, y, this.size, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.shadowBlur = 10;
                ctx.shadowColor = this.color;
                ctx.fill();
                ctx.shadowBlur = 0;
            }
        }

        // --- Drawing Functions ---

        function drawCompartmentLabels() {
            ctx.font = "bold 14px 'Noto Sans SC'";
            ctx.fillStyle = "rgba(148, 163, 184, 0.3)";
            ctx.textAlign = "center";
            
            // Extracellular
            ctx.fillText("胞外 (Extracellular)", width * 0.15, height * 0.1);
            
            // Intracellular
            ctx.fillText("胞内 (Intracellular)", width * 0.55, height * 0.1);
            
            // Membrane line (Visual guide)
            ctx.beginPath();
            ctx.moveTo(width * 0.35, 0);
            ctx.lineTo(width * 0.35, height);
            ctx.strokeStyle = "rgba(148, 163, 184, 0.1)";
            ctx.setLineDash([5, 5]);
            ctx.stroke();
            ctx.setLineDash([]);
        }

        function drawNode(node) {
            const x = node.x * width;
            const y = node.y * height;
            const isActive = activeNode === node;
            const radius = isActive ? 35 : 25;

            // Glow effect
            if (isActive) {
                ctx.beginPath();
                ctx.arc(x, y, radius + 10, 0, Math.PI * 2);
                ctx.fillStyle = "rgba(56, 189, 248, 0.2)";
                ctx.fill();
            }

            // Node Circle
            ctx.beginPath();
            ctx.arc(x, y, radius, 0, Math.PI * 2);
            ctx.fillStyle = isActive ? "#0ea5e9" : "#1e293b"; // Sky-500 or Slate-800
            ctx.strokeStyle = isActive ? "#38bdf8" : "#475569";
            ctx.lineWidth = 2;
            ctx.fill();
            ctx.stroke();

            // Icon/Text inside node
            ctx.fillStyle = "#fff";
            ctx.font = "bold 12px 'Noto Sans SC'";
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            
            // Simple abbreviation
            let text = node.id.toUpperCase();
            if(node.id === 'arsC') text = "As(V)→III";
            if(node.id === 'aioA') text = "As(III)→V";
            
            ctx.fillText(text, x, y);

            // Label below
            ctx.fillStyle = isActive ? "#38bdf8" : "#94a3b8";
            ctx.font = "12px 'Noto Sans SC'";
            ctx.fillText(node.label, x, y + radius + 15);
        }

        function drawConnections() {
            reactions.forEach(node => {
                // Draw lines to inputs/outputs logic is complex, simplified here:
                // We draw a "Zone" representation
                
                // Draw Input Pool (Left of node)
                /*
                ctx.beginPath();
                ctx.arc((node.x - 0.1) * width, node.y * height, 10, 0, Math.PI*2);
                ctx.fillStyle = "rgba(255,255,255,0.05)";
                ctx.fill();
                */
            });
        }

        function animate() {
            ctx.clearRect(0, 0, width, height);
            
            drawCompartmentLabels();

            // Spawn particles randomly
            if (Math.random() < 0.05) {
                // Randomly pick a reaction to animate
                const r = reactions[Math.floor(Math.random() * reactions.length)];
                // Determine particle type based on reaction
                let type = 'As3';
                if (r.id === 'arsC') type = 'As5';
                if (r.id === 'arsM') type = 'Methyl';
                if (r.id === 'aioA') type = 'Electron';
                
                // Mock source/target for visual flow
                // For visual simplicity, particles flow "through" the node
                const source = { x: r.x - 0.15, y: r.y };
                const target = { x: r.x + 0.15, y: r.y };
                
                // If it's a pump going out
                if (r.id === 'acr3' || r.id === 'arsP') {
                     particles.push(new Particle(type, {x: r.x, y: r.y}, {x: r.x + 0.2, y: r.y}));
                } else if (r.id === 'arsC') {
                     // As5 comes from left, As3 goes to right
                     particles.push(new Particle('As5', {x: r.x - 0.2, y: r.y}, {x: r.x, y: r.y}));
                     particles.push(new Particle('As3', {x: r.x, y: r.y}, {x: r.x + 0.1, y: r.y}));
                } else {
                     particles.push(new Particle(type, source, target));
                }
            }

            // Update and draw particles
            particles = particles.filter(p => {
                const finished = p.update();
                if (!finished) p.draw(ctx);
                return !finished;
            });

            // Draw Nodes
            reactions.forEach(drawNode);

            animationId = requestAnimationFrame(animate);
        }

        // --- Interaction ---

        function getMousePos(evt) {
            const rect = canvas.getBoundingClientRect();
            return {
                x: evt.clientX - rect.left,
                y: evt.clientY - rect.top
            };
        }

        function isInsideNode(pos, node) {
            const x = node.x * width;
            const y = node.y * height;
            const dx = pos.x - x;
            const dy = pos.y - y;
            return (dx*dx + dy*dy) < 30*30; // Radius check
        }

        canvas.addEventListener('click', (e) => {
            const pos = getMousePos(e);
            let clicked = null;

            for (let node of reactions) {
                if (isInsideNode(pos, node)) {
                    clicked = node;
                    break;
                }
            }

            if (clicked) {
                activeNode = clicked;
                updateInfoPanel(clicked);
            } else {
                activeNode = null;
                resetInfoPanel();
            }
        });

        // --- UI Logic ---

        function updateInfoPanel(node) {
            const panel = document.getElementById('info-content');
            
            // Determine badge color
            let badgeColor = "bg-slate-600";
            if(node.type === 'reduction') badgeColor = "bg-rose-500";
            if(node.type === 'oxidation') badgeColor = "bg-yellow-500";
            if(node.type === 'methylation') badgeColor = "bg-emerald-500";
            if(node.type === 'transport') badgeColor = "bg-sky-500";

            panel.innerHTML = `
                <div class="animate-fade-in-up">
                    <div class="flex items-center justify-between mb-4">
                        <h2 class="text-2xl font-bold text-white">${node.label}</h2>
                        <span class="${badgeColor} text-xs font-bold px-2 py-1 rounded uppercase tracking-wider">${node.type}</span>
                    </div>
                    
                    <div class="mb-6">
                        <p class="text-sm text-slate-400 mb-1">关键基因 (Gene)</p>
                        <p class="text-lg font-mono text-sky-300">${node.gene}</p>
                    </div>

                    <div class="bg-slate-800/50 rounded-lg p-4 border border-slate-700 mb-6">
                        <p class="text-xs text-slate-400 mb-2 uppercase tracking-widest">化学反应式</p>
                        <div class="font-mono text-sm text-emerald-300 break-words leading-relaxed">
                            ${node.equation}
                        </div>
                    </div>

                    <div>
                        <p class="text-sm text-slate-400 mb-2">生物学功能</p>
                        <p class="text-slate-200 leading-relaxed text-sm">
                            ${node.details}
                        </p>
                    </div>

                    <div class="mt-6 pt-6 border-t border-slate-700">
                        <div class="flex items-center gap-3 text-xs text-slate-500">
                            <div class="flex items-center gap-1"><div class="w-2 h-2 rounded-full bg-yellow-400"></div> 底物</div>
                            <div class="flex items-center gap-1"><div class="w-2 h-2 rounded-full bg-rose-500"></div> 产物</div>
                            <div class="flex items-center gap-1"><div class="w-2 h-2 rounded-full bg-emerald-400"></div> 甲基化</div>
                        </div>
                    </div>
                </div>
            `;
        }

        function resetInfoPanel() {
            const panel = document.getElementById('info-content');
            panel.innerHTML = `
                <div class="text-center text-slate-500 mt-10 animate-pulse">
                    <svg class="w-16 h-16 mx-auto mb-4 opacity-50" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M7 21a4 4 0 01-4-4V5a2 2 0 012-2h4a2 2 0 012 2v12a4 4 0 01-4 4zm0 0h12a2 2 0 002-2v-4a2 2 0 00-2-2h-2.343M11 7.343l1.657-1.657a2 2 0 012.828 0l2.829 2.829a2 2 0 010 2.828l-8.486 8.485M7 17h.01"></path></svg>
                    <p>请在右侧图谱中点击任意反应节点<br>查看化学方程式与基因功能</p>
                </div>
            `;
        }

        function resetView() {
            activeNode = null;
            resetInfoPanel();
        }

        // Start
        animate();

    </script>
</body>
</html>
