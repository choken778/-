<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的冰箱 - 食材管理助手</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .gradient-bg { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
        .item-card { transition: transform 0.1s; }
        .item-card:active { transform: scale(0.98); }
    </style>
</head>
<body class="bg-gray-50 min-h-screen pb-20">

    <!-- Header -->
    <header class="gradient-bg text-white p-6 shadow-lg rounded-b-3xl">
        <h1 class="text-2xl font-bold text-center">🧊 我的冰箱</h1>
        <p class="text-sm opacity-80 text-center mt-1">管理食材，即刻获取食谱</p>
    </header>

    <main class="p-4 max-w-md mx-auto">
        <!-- Input Area -->
        <div class="bg-white p-4 rounded-2xl shadow-sm mb-6">
            <div class="flex gap-2">
                <input id="foodInput" type="text" placeholder="输入食材名称 (如：西红柿)" 
                       class="flex-1 border-gray-200 border rounded-xl px-4 py-2 focus:ring-2 focus:ring-indigo-400 outline-none">
                <button onclick="addItem()" class="bg-indigo-500 text-white px-4 py-2 rounded-xl hover:bg-indigo-600 font-medium">
                    添加
                </button>
            </div>
        </div>

        <!-- Inventory List -->
        <div class="flex justify-between items-center mb-4 px-2">
            <h2 class="text-gray-700 font-bold">库存清单</h2>
            <button onclick="clearList()" class="text-xs text-red-400 hover:underline">清空全部</button>
        </div>
        
        <div id="inventoryList" class="space-y-3">
            <!-- 动态生成食材卡片 -->
        </div>
    </main>

    <!-- Bottom Action Bar -->
    <div class="fixed bottom-0 left-0 right-0 p-4 bg-white/80 backdrop-blur-md border-t border-gray-100">
        <div class="max-w-md mx-auto flex gap-3">
            <button onclick="copyToAI()" class="flex-1 gradient-bg text-white py-3 rounded-2xl font-bold shadow-lg hover:opacity-90 active:scale-95 transition-all">
                📋 复制给 AI 获取食谱
            </button>
        </div>
    </div>

    <!-- Toast Notification -->
    <div id="toast" class="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-black/70 text-white px-6 py-3 rounded-full text-sm hidden">
        已复制到剪贴板
    </div>

    <script>
        let inventory = JSON.parse(localStorage.getItem('myFridge')) || [];

        function render() {
            const listElement = document.getElementById('inventoryList');
            listElement.innerHTML = '';
            
            if (inventory.length === 0) {
                listElement.innerHTML = '<p class="text-center text-gray-400 py-10">冰箱空空如也，快去买点吃的吧！</p>';
                return;
            }

            inventory.forEach((item, index) => {
                const card = document.createElement('div');
                card.className = 'item-card bg-white p-4 rounded-2xl shadow-sm flex justify-between items-center border border-gray-50';
                card.innerHTML = `
                    <span class="font-medium text-gray-800">${item}</span>
                    <button onclick="removeItem(${index})" class="text-gray-300 hover:text-red-500">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                            <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
                        </svg>
                    </button>
                `;
                listElement.appendChild(card);
            });
            localStorage.setItem('myFridge', JSON.stringify(inventory));
        }

        function addItem() {
            const input = document.getElementById('foodInput');
            const val = input.value.trim();
            if (val) {
                inventory.unshift(val);
                input.value = '';
                render();
            }
        }

        function removeItem(index) {
            inventory.splice(index, 1);
            render();
        }

        function clearList() {
            if(confirm('确定要清空冰箱吗？')) {
                inventory = [];
                render();
            }
        }

        function copyToAI() {
            if (inventory.length === 0) return alert('冰箱里还没东西呢');
            
            const prompt = `你好！我的冰箱里目前有以下食材：\n\n【${inventory.join('、')}】\n\n请根据这些食材为我推荐 2-3 道简单易做的食谱，并列出大致的步骤和所需的调料。谢谢！`;
            
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
