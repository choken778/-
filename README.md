<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的冰箱 V2 - 带数量管理</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .gradient-bg { background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%); }
        .item-card { transition: all 0.2s; }
        .item-card:hover { border-color: #43e97b; }
    </style>
</head>
<body class="bg-slate-50 min-h-screen pb-28">

    <header class="gradient-bg text-gray-800 p-8 shadow-md rounded-b-[40px]">
        <h1 class="text-3xl font-black text-center mb-1">🧊 我的冰箱</h1>
        <p class="text-sm opacity-70 text-center font-medium">精准掌控每一份食材</p>
    </header>

    <main class="p-4 max-w-md mx-auto -mt-6">
        <div class="bg-white p-5 rounded-3xl shadow-xl mb-6 border border-gray-100">
            <div class="space-y-3">
                <input id="foodName" type="text" placeholder="食材名称 (如：牛肉)" 
                       class="w-full border-gray-100 border-2 bg-gray-50 rounded-2xl px-4 py-3 focus:ring-2 focus:ring-emerald-400 outline-none transition-all">
                
                <div class="flex gap-2">
                    <input id="foodQty" type="text" placeholder="数量" 
                           class="w-1/3 border-gray-100 border-2 bg-gray-50 rounded-2xl px-4 py-3 focus:ring-2 focus:ring-emerald-400 outline-none transition-all">
                    
                    <select id="foodUnit" class="w-2/3 border-gray-100 border-2 bg-gray-50 rounded-2xl px-3 py-3 focus:ring-2 focus:ring-emerald-400 outline-none">
                        <option value="个">个/颗</option>
                        <option value="g">克 (g)</option>
                        <option value="kg">千克 (kg)</option>
                        <option value="把">把/束</option>
                        <option value="盒">盒/袋</option>
                        <option value="一半">一半</option>
                        <option value="适量">适量</option>
                    </select>
                </div>

                <button onclick="addItem()" class="w-full bg-slate-800 text-white py-4 rounded-2xl font-bold hover:bg-slate-700 active:scale-95 transition-all shadow-lg">
                    放入冰箱
                </button>
            </div>
        </div>

        <div class="flex justify-between items-center mb-4 px-2">
            <h2 class="text-gray-800 font-extrabold text-lg">当前库存</h2>
            <button onclick="clearList()" class="text-xs text-gray-400 hover:text-red-500 transition-colors">清空冰箱</button>
        </div>
        
        <div id="inventoryList" class="space-y-3">
            </div>
    </main>

    <div class="fixed bottom-0 left-0 right-0 p-6 bg-white/90 backdrop-blur-xl border-t border-gray-100">
        <div class="max-w-md mx-auto">
            <button onclick="copyToAI()" class="w-full gradient-bg text-gray-800 py-4 rounded-2xl font-black shadow-xl hover:shadow-2xl active:scale-95 transition-all flex items-center justify-center gap-2">
                <span>📋 复制给 AI 获取食谱</span>
            </button>
        </div>
    </div>

    <div id="toast" class="fixed top-20 left-1/2 -translate-x-1/2 bg-slate-800 text-white px-8 py-3 rounded-2xl text-sm font-bold shadow-2xl hidden z-50">
        已成功复制食材清单！
    </div>

    <script>
        let inventory = JSON.parse(localStorage.getItem('myFridgeV2')) || [];

        function render() {
            const listElement = document.getElementById('inventoryList');
            listElement.innerHTML = '';
            
            if (inventory.length === 0) {
                listElement.innerHTML = `
                    <div class="text-center py-12">
                        <div class="text-5xl mb-4 text-gray-200">🥗</div>
                        <p class="text-gray-400 font-medium">冰箱是空的，快添加食材吧</p>
                    </div>`;
                return;
            }

            inventory.forEach((item, index) => {
                const card = document.createElement('div');
                card.className = 'item-card bg-white p-5 rounded-3xl shadow-sm flex justify-between items-center border-2 border-transparent';
                card.innerHTML = `
                    <div class="flex flex-col">
                        <span class="font-bold text-gray-800 text-lg">${item.name}</span>
                        <span class="text-sm text-emerald-500 font-bold bg-emerald-50 px-2 py-0.5 rounded-lg w-fit mt-1">
                            ${item.qty} ${item.unit}
                        </span>
                    </div>
                    <button onclick="removeItem(${index})" class="bg-gray-50 p-2 rounded-xl text-gray-300 hover:text-red-500 hover:bg-red-50 transition-all">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                        </svg>
                    </button>
                `;
                listElement.appendChild(card);
            });
            localStorage.setItem('myFridgeV2', JSON.stringify(inventory));
        }

        function addItem() {
            const name = document.getElementById('foodName').value.trim();
            const qty = document.getElementById('foodQty').value.trim();
            const unit = document.getElementById('foodUnit').value;

            if (name) {
                inventory.unshift({
                    name: name,
                    qty: qty || '适量',
                    unit: qty ? unit : ''
                });
                document.getElementById('foodName').value = '';
                document.getElementById('foodQty').value = '';
                render();
            } else {
                alert('请输入食材名称');
            }
        }

        function removeItem(index) {
            inventory.splice(index, 1);
            render();
        }

        function clearList() {
            if(confirm('确定要清空冰箱清单吗？')) {
                inventory = [];
                render();
            }
        }

        function copyToAI() {
            if (inventory.length === 0) return alert('冰箱里还没东西呢');
            
            const listString = inventory.map(item => `- ${item.name} (${item.qty}${item.unit})`).join('\n');
            
            const prompt = `你好！这是我冰箱里剩下的食材清单：\n\n${listString}\n\n请以此为基础，为我推荐 2-3 道菜谱。要求：\n1. 优先消耗快过期的食材。\n2. 详细列出每道菜的步骤。\n3. 如果还需要少量必不可少的配料（如葱姜蒜），请提醒我。`;
            
            navigator.clipboard.writeText(prompt).then(() => {
                const toast = document.getElementById('toast');
                toast.classList.remove('hidden');
                setTimeout(() => toast.classList.add('hidden'), 2000);
            });
        }

        render();
    </script>
</body>
</html>
