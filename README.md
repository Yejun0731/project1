<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>비트 세포막 투과성 분석 도구 (시간별 사진 분석)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        #canvas-container {
            position: relative;
            cursor: crosshair;
            max-width: 100%;
            overflow: hidden;
            border: 2px dashed #cbd5e1;
            border-radius: 0.5rem;
            background-color: #f8fafc;
            min-height: 300px;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        canvas {
            display: block;
            margin: 0 auto;
        }
        .temp-selection {
            position: absolute;
            border: 2px dashed #3b82f6;
            background-color: rgba(59, 130, 246, 0.1);
            pointer-events: none;
            z-index: 10;
        }
        .fixed-selection {
            position: absolute;
            border: 2px solid;
            background-color: rgba(255, 255, 255, 0.1);
            pointer-events: none;
            z-index: 5;
        }
        .selection-label {
            position: absolute;
            top: -20px;
            left: -2px;
            font-size: 10px;
            font-weight: bold;
            color: white;
            padding: 2px 4px;
            border-radius: 2px 2px 0 0;
            white-space: nowrap;
        }
        .data-cell {
            min-width: 60px;
            text-align: center;
        }
        .data-value {
            display: block;
            width: 100%;
            height: 100%;
            padding: 8px 4px;
            cursor: pointer;
            transition: all 0.2s;
        }
        .data-value:hover {
            background-color: #fee2e2;
            color: #be123c;
            text-decoration: line-through;
        }
        .slot-btn.active {
            background-color: #2563eb;
            color: white;
            border-color: #2563eb;
        }
    </style>
</head>
<body class="bg-slate-50 min-h-screen p-4 md:p-8">
    <div class="max-w-7xl mx-auto">
        <header class="mb-8 text-center">
            <h1 class="text-3xl font-bold text-slate-800">비트 세포막 투과성 분석기</h1>
            <p class="text-slate-600 mt-2">시간대별 사진 탭을 클릭하여 데이터를 분석하세요.</p>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
            <!-- Left: Image Analysis & Input -->
            <div class="lg:col-span-7 space-y-6">
                <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-200">
                    <div class="flex flex-col md:flex-row md:items-center justify-between mb-6 gap-4">
                        <div class="flex flex-wrap gap-2">
                            <button onclick="switchSlot(0)" id="slotBtn0" class="slot-btn active px-4 py-2 border rounded-lg text-sm font-bold transition">10분 사진</button>
                            <button onclick="switchSlot(1)" id="slotBtn1" class="slot-btn px-4 py-2 border rounded-lg text-sm font-bold transition">15분 사진</button>
                            <button onclick="switchSlot(2)" id="slotBtn2" class="slot-btn px-4 py-2 border rounded-lg text-sm font-bold transition">20분 사진</button>
                            <button onclick="switchSlot(3)" id="slotBtn3" class="slot-btn px-4 py-2 border rounded-lg text-sm font-bold transition">24시간 사진</button>
                        </div>
                        <div class="flex gap-2">
                            <input type="file" id="imageInput" accept="image/*" class="hidden">
                            <label for="imageInput" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg cursor-pointer transition text-sm shadow-sm font-bold">
                                사진 불러오기
                            </label>
                            <button id="clearBoxes" class="text-sm text-rose-600 hover:text-rose-800 font-medium px-2">초기화</button>
                        </div>
                    </div>

                    <div id="canvas-container">
                        <div id="placeholder" class="text-slate-400 text-sm italic">해당 시간의 사진을 불러와주세요</div>
                        <canvas id="imageCanvas" class="hidden"></canvas>
                        <div id="tempBox" class="temp-selection hidden"></div>
                        <div id="fixedBoxesContainer"></div>
                    </div>
                </div>

                <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-200">
                    <h2 class="text-xl font-semibold mb-4 text-slate-700">데이터 기록 도구</h2>
                    <div class="grid grid-cols-1 md:grid-cols-5 gap-4 items-end">
                        <div class="md:col-span-1">
                            <label class="block text-sm font-medium text-slate-700">용액 종류</label>
                            <select id="solutionSelect" class="mt-1 block w-full border border-slate-300 rounded-md p-2 focus:ring-2 focus:ring-blue-500 bg-white">
                                <option value="증류수">증류수</option>
                                <option value="주방세제">주방세제</option>
                                <option value="베이킹소다">베이킹소다</option>
                                <option value="과탄산수소">과탄산수소</option>
                                <option value="세탁세제">세탁세제</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-slate-700">기록할 시간</label>
                            <select id="timeSelect" class="mt-1 block w-full border border-slate-300 rounded-md p-2 focus:ring-2 focus:ring-blue-500 font-bold bg-slate-50">
                                <option value="10분">10분</option>
                                <option value="15분">15분</option>
                                <option value="20분">20분</option>
                                <option value="24시간">24시간</option>
                            </select>
                        </div>
                        <div class="md:col-span-2">
                            <label class="block text-sm font-medium text-slate-700">현재 추출 RGB (R값)</label>
                            <div class="flex gap-2 mt-1">
                                <div id="rDisplay" class="flex-1 bg-slate-100 p-2 rounded text-center font-mono text-sm border-b-4 border-rose-500 text-rose-700 font-bold">R: 0</div>
                                <div id="gDisplay" class="flex-1 bg-slate-100 p-2 rounded text-center font-mono text-sm border-b-4 border-emerald-500 text-emerald-700 font-bold">G: 0</div>
                                <div id="bDisplay" class="flex-1 bg-slate-100 p-2 rounded text-center font-mono text-sm border-b-4 border-blue-500 text-blue-700 font-bold">B: 0</div>
                            </div>
                        </div>
                        <button id="addRecord" class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg transition h-12 shadow-md font-bold text-sm">
                            결과 저장
                        </button>
                    </div>
                </div>
            </div>

            <!-- Right: Results Dashboard -->
            <div class="lg:col-span-5 space-y-6">
                <!-- Data Table -->
                <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-200">
                    <div class="flex flex-col sm:flex-row sm:items-center justify-between mb-4 gap-3">
                        <h2 class="text-xl font-semibold text-rose-700">실험 결과 비교</h2>
                        <select id="filterSolution" class="text-[10px] border border-slate-300 rounded p-1 bg-slate-50 outline-none">
                            <option value="all">모든 용액</option>
                            <option value="증류수">증류수</option>
                            <option value="주방세제">주방세제</option>
                            <option value="베이킹소다">베이킹소다</option>
                            <option value="과탄산수소">과탄산수소</option>
                            <option value="세탁세제">세탁세제</option>
                        </select>
                    </div>
                    
                    <div class="overflow-x-auto border rounded-lg">
                        <table class="w-full text-[11px] border-collapse">
                            <thead>
                                <tr class="bg-slate-100 border-b border-slate-200">
                                    <th class="py-3 px-2 text-left border-r border-slate-200">용액 종류</th>
                                    <th class="py-3 px-1 data-cell">10분</th>
                                    <th class="py-3 px-1 data-cell">15분</th>
                                    <th class="py-3 px-1 data-cell">20분</th>
                                    <th class="py-3 px-1 data-cell border-r">24시간</th>
                                    <th class="py-3 px-2 text-center">삭제</th>
                                </tr>
                            </thead>
                            <tbody id="horizontalTableBody"></tbody>
                        </table>
                    </div>
                </div>

                <!-- Trend Chart -->
                <div class="bg-white p-6 rounded-xl shadow-sm border border-slate-200">
                    <h2 class="text-xl font-semibold mb-4 text-slate-700">추출 데이터 시각화 (R값)</h2>
                    <canvas id="resultChart"></canvas>
                </div>
            </div>
        </div>
    </div>

    <script>
        const imageInput = document.getElementById('imageInput');
        const canvas = document.getElementById('imageCanvas');
        const ctx = canvas.getContext('2d', { willReadFrequently: true });
        const tempBox = document.getElementById('tempBox');
        const fixedBoxesContainer = document.getElementById('fixedBoxesContainer');
        const placeholder = document.getElementById('placeholder');
        const rDisplay = document.getElementById('rDisplay');
        const gDisplay = document.getElementById('gDisplay');
        const bDisplay = document.getElementById('bDisplay');
        const timeSelect = document.getElementById('timeSelect');
        const horizontalTableBody = document.getElementById('horizontalTableBody');
        const filterSolution = document.getElementById('filterSolution');

        const timeOrder = ['10분', '15분', '20분', '24시간'];
        const solutionOrder = ['증류수', '주방세제', '베이킹소다', '과탄산수소', '세탁세제'];
        const solutionColors = {
            '증류수': '#64748b', '주방세제': '#0ea5e9', '베이킹소다': '#f59e0b',
            '과탄산수소': '#8b5cf6', '세탁세제': '#be123c'
        };

        // State Management
        let currentSlot = 0;
        const slots = [
            { img: null, boxes: [] },
            { img: null, boxes: [] },
            { img: null, boxes: [] },
            { img: null, boxes: [] }
        ];

        let experimentData = [];
        let currentRGB = { r: 0, g: 0, b: 0 };
        let isDragging = false;
        let startX, startY;

        // Chart Init
        const chartCtx = document.getElementById('resultChart').getContext('2d');
        const resultChart = new Chart(chartCtx, {
            type: 'line',
            data: {
                labels: timeOrder,
                datasets: solutionOrder.map(name => ({
                    label: name,
                    data: [null, null, null, null],
                    borderColor: solutionColors[name],
                    backgroundColor: 'transparent',
                    borderWidth: 2,
                    tension: 0.2,
                    pointRadius: 4
                }))
            },
            options: {
                responsive: true,
                plugins: { legend: { position: 'bottom', labels: { font: { size: 10 } } } },
                scales: {
                    y: { reverse: true, title: { display: true, text: 'R 값 (낮을수록 투과성 높음)' } },
                    x: { title: { display: true, text: '실험 시간' } }
                }
            }
        });

        function switchSlot(index) {
            currentSlot = index;
            document.querySelectorAll('.slot-btn').forEach((btn, idx) => {
                btn.classList.toggle('active', idx === index);
            });
            
            // 탭 이름에 맞춰 하단 시간 선택 박스도 자동 변경
            timeSelect.value = timeOrder[index];
            
            renderCanvas();
        }

        function renderCanvas() {
            const slot = slots[currentSlot];
            fixedBoxesContainer.innerHTML = '';
            
            if (!slot.img) {
                canvas.classList.add('hidden');
                placeholder.classList.remove('hidden');
                rDisplay.innerText = 'R: 0'; gDisplay.innerText = 'G: 0'; bDisplay.innerText = 'B: 0';
                return;
            }

            canvas.classList.remove('hidden');
            placeholder.classList.add('hidden');
            
            const maxWidth = 800;
            const scale = maxWidth / slot.img.width;
            canvas.width = maxWidth;
            canvas.height = slot.img.height * scale;
            ctx.drawImage(slot.img, 0, 0, canvas.width, canvas.height);

            slot.boxes.forEach(box => {
                const boxEl = document.createElement('div');
                boxEl.className = 'fixed-selection';
                boxEl.style.left = box.x + 'px'; boxEl.style.top = box.y + 'px'; 
                boxEl.style.width = box.w + 'px'; boxEl.style.height = box.h + 'px';
                boxEl.style.borderColor = solutionColors[box.solution];
                const label = document.createElement('div');
                label.className = 'selection-label';
                label.style.backgroundColor = solutionColors[box.solution];
                label.innerText = box.solution;
                boxEl.appendChild(label);
                fixedBoxesContainer.appendChild(boxEl);
            });
        }

        imageInput.addEventListener('change', (e) => {
            const file = e.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = (event) => {
                const img = new Image();
                img.onload = () => {
                    slots[currentSlot].img = img;
                    slots[currentSlot].boxes = [];
                    renderCanvas();
                };
                img.src = event.target.result;
            };
            reader.readAsDataURL(file);
        });

        canvas.addEventListener('mousedown', (e) => {
            if (!slots[currentSlot].img) return;
            isDragging = true;
            const rect = canvas.getBoundingClientRect();
            startX = e.clientX - rect.left;
            startY = e.clientY - rect.top;
            tempBox.classList.remove('hidden');
            tempBox.style.left = startX + 'px';
            tempBox.style.top = startY + 'px';
            tempBox.style.width = '0px';
            tempBox.style.height = '0px';
            tempBox.style.borderColor = solutionColors[document.getElementById('solutionSelect').value];
        });

        window.addEventListener('mousemove', (e) => {
            if (!isDragging) return;
            const rect = canvas.getBoundingClientRect();
            const currentX = Math.max(0, Math.min(e.clientX - rect.left, canvas.width));
            const currentY = Math.max(0, Math.min(e.clientY - rect.top, canvas.height));
            const width = currentX - startX;
            const height = currentY - startY;
            const l = (width > 0 ? startX : currentX);
            const t = (height > 0 ? startY : currentY);
            const w = Math.abs(width);
            const h = Math.abs(height);
            tempBox.style.width = w + 'px';
            tempBox.style.height = h + 'px';
            tempBox.style.left = l + 'px';
            tempBox.style.top = t + 'px';
            
            if (w > 2 && h > 2) {
                const imageData = ctx.getImageData(l, t, w, h);
                const data = imageData.data;
                let rS = 0, gS = 0, bS = 0;
                for (let i = 0; i < data.length; i += 4) {
                    rS += data[i]; gS += data[i+1]; bS += data[i+2];
                }
                const cnt = data.length / 4;
                currentRGB = { r: Math.round(rS / cnt), g: Math.round(gS / cnt), b: Math.round(bS / cnt) };
                rDisplay.innerText = `R: ${currentRGB.r}`;
                gDisplay.innerText = `G: ${currentRGB.g}`;
                bDisplay.innerText = `B: ${currentRGB.b}`;
            }
        });

        window.addEventListener('mouseup', () => {
            if (!isDragging) return;
            isDragging = false;
            const w = parseFloat(tempBox.style.width), h = parseFloat(tempBox.style.height);
            if (w > 5 && h > 5) {
                const sol = document.getElementById('solutionSelect').value;
                slots[currentSlot].boxes = slots[currentSlot].boxes.filter(b => b.solution !== sol);
                slots[currentSlot].boxes.push({
                    x: parseFloat(tempBox.style.left),
                    y: parseFloat(tempBox.style.top),
                    w, h, solution: sol
                });
                renderCanvas();
            }
            tempBox.classList.add('hidden');
        });

        document.getElementById('clearBoxes').addEventListener('click', () => {
            slots[currentSlot].boxes = [];
            renderCanvas();
        });

        document.getElementById('addRecord').addEventListener('click', () => {
            if (currentRGB.r === 0) return alert('이미지에서 추출 영역을 선택해주세요.');
            const time = timeSelect.value;
            const sol = document.getElementById('solutionSelect').value;
            const idx = experimentData.findIndex(d => d.solution === sol && d.time === time);
            if (idx > -1) experimentData[idx] = { solution: sol, time, ...currentRGB };
            else experimentData.push({ solution: sol, time, ...currentRGB });
            updateUI();
        });

        function deleteSingleEntry(sol, time) {
            experimentData = experimentData.filter(d => !(d.solution === sol && d.time === time));
            updateUI();
        }

        function deleteRow(sol) {
            experimentData = experimentData.filter(d => d.solution !== sol);
            updateUI();
        }

        filterSolution.addEventListener('change', updateUI);

        function updateUI() {
            horizontalTableBody.innerHTML = '';
            const solFilter = filterSolution.value;
            solutionOrder.forEach(solName => {
                if (solFilter !== 'all' && solFilter !== solName) return;
                const row = document.createElement('tr');
                row.className = 'border-b hover:bg-slate-50 transition';
                let html = `<td class="py-3 px-2 border-r font-bold bg-slate-50/50" style="color:${solutionColors[solName]}">${solName}</td>`;
                timeOrder.forEach(timeLabel => {
                    const data = experimentData.find(d => d.solution === solName && d.time === timeLabel);
                    if (data) {
                        html += `<td class="data-cell p-0"><span class="data-value font-mono text-rose-600 font-bold bg-rose-50/40" onclick="deleteSingleEntry('${solName}', '${timeLabel}')" title="개별 삭제">${data.r}</span></td>`;
                    } else {
                        html += `<td class="data-cell text-slate-300">-</td>`;
                    }
                });
                html += `<td class="py-2 px-2 text-center border-l"><button onclick="deleteRow('${solName}')" class="text-slate-300 hover:text-rose-600 transition">✕</button></td>`;
                row.innerHTML = html;
                horizontalTableBody.appendChild(row);
            });

            resultChart.data.datasets.forEach(dataset => {
                dataset.hidden = solFilter !== 'all' && dataset.label !== solFilter;
                dataset.data = timeOrder.map(label => {
                    const match = experimentData.find(d => d.solution === dataset.label && d.time === label);
                    return match ? match.r : null;
                });
            });
            resultChart.update();
        }

        window.switchSlot = switchSlot;
        window.deleteSingleEntry = deleteSingleEntry;
        window.deleteRow = deleteRow;
    </script>
</body>
</html>
