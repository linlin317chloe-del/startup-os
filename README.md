<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Startup OS - 員工行動助手</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700&display=swap');
        body { font-family: 'Noto Sans TC', sans-serif; background-color: #f8fafc; -webkit-tap-highlight-color: transparent; }
        .glass { background: rgba(255, 255, 255, 0.9); backdrop-filter: blur(12px); border: 1px solid rgba(255,255,255,0.3); }
        .app-gradient { background: linear-gradient(135deg, #6366f1 0%, #a855f7 100%); }
        .nav-item.active { color: #6366f1; transform: translateY(-3px); transition: all 0.3s ease; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
    </style>
</head>
<body class="text-slate-800">

<div id="app" class="max-w-md mx-auto min-h-screen pb-24 relative shadow-2xl bg-slate-50 overflow-x-hidden">
    
    <header class="app-gradient p-8 pb-16 rounded-b-[3rem] text-white relative">
        <div class="flex justify-between items-center">
            <div>
                <h1 class="text-2xl font-bold tracking-tight">您好，{{ profile.name || '夥伴' }}</h1>
                <p class="text-indigo-100 text-xs mt-1">{{ todayDate }}</p>
            </div>
            <div class="bg-white/20 p-3 rounded-2xl backdrop-blur-md" @click="activeTab = 'profile'">
                <i class="fa-solid fa-circle-user text-xl"></i>
            </div>
        </div>
    </header>

    <main class="px-5 -mt-8 space-y-6 relative z-10">
        
        <div v-if="activeTab === 'home'" class="space-y-6">
            <section class="glass p-6 rounded-[2rem] shadow-xl">
                <div class="flex justify-between items-center mb-6">
                    <h3 class="font-bold text-slate-700 flex items-center">
                        <i class="fa-solid fa-clock-rotate-left mr-2 text-indigo-500"></i>今日考勤
                    </h3>
                    <span class="text-[10px] bg-indigo-100 text-indigo-600 px-2 py-1 rounded-full font-bold">符合勞基法</span>
                </div>
                <div class="grid grid-cols-2 gap-4">
                    <button @click="clockIn" :disabled="attendance.in" :class="attendance.in ? 'bg-slate-100 text-slate-400' : 'bg-indigo-50 text-indigo-600'" class="py-4 rounded-3xl font-bold transition-all active:scale-95 shadow-sm">
                        <i class="fa-solid fa-sun mb-1 block"></i>
                        {{ attendance.in || '上班打卡' }}
                    </button>
                    <button @click="clockOut" :disabled="!attendance.in || attendance.out" :class="attendance.out ? 'bg-slate-100 text-slate-400' : 'bg-orange-50 text-orange-600'" class="py-4 rounded-3xl font-bold transition-all active:scale-95 shadow-sm">
                        <i class="fa-solid fa-moon mb-1 block"></i>
                        {{ attendance.out || '下班打卡' }}
                    </button>
                </div>
            </section>

            <div class="grid grid-cols-2 gap-4">
                <button @click="activeTab = 'docs'" class="p-5 glass rounded-[2rem] text-left hover:bg-white transition-all shadow-sm">
                    <div class="w-10 h-10 bg-blue-100 text-blue-600 rounded-xl flex items-center justify-center mb-3">
                        <i class="fa-solid fa-book"></i>
                    </div>
                    <p class="text-sm font-bold">工作手冊</p>
                    <p class="text-[10px] text-slate-400">規範與福利</p>
                </button>
                <button @click="activeTab = 'docs'" class="p-5 glass rounded-[2rem] text-left hover:bg-white transition-all shadow-sm">
                    <div class="w-10 h-10 bg-emerald-100 text-emerald-600 rounded-xl flex items-center justify-center mb-3">
                        <i class="fa-solid fa-list-check"></i>
                    </div>
                    <p class="text-sm font-bold">入職清單</p>
                    <p class="text-[10px] text-slate-400">新手必看</p>
                </button>
            </div>
        </div>

        <div v-if="activeTab === 'finance'" class="space-y-6">
            <section class="glass p-6 rounded-[2rem] shadow-xl">
                <div class="flex justify-between items-end mb-4">
                    <h3 class="font-bold text-slate-700">出差報支費</h3>
                    <div class="text-right">
                        <p class="text-[10px] text-slate-400">累積總額</p>
                        <p class="text-2xl font-black text-indigo-600 font-mono">${{ totalExp }}</p>
                    </div>
                </div>
                <div class="space-y-3 bg-slate-50 p-4 rounded-2xl">
                    <div class="flex gap-2">
                        <select v-model="newExp.type" class="flex-1 p-3 rounded-xl text-sm border-none shadow-sm outline-none">
                            <option value="交通費">🚗 交通費</option>
                            <option value="餐飲費">🍱 餐飲費</option>
                            <option value="住宿費">🏨 住宿費</option>
                            <option value="其他費用">📦 其他(零用金)</option>
                        </select>
                        <input v-model.number="newExp.amount" type="number" placeholder="金額" class="w-24 p-3 rounded-xl text-sm shadow-sm outline-none">
                    </div>
                    <input v-model="newExp.note" type="text" placeholder="備註內容" class="w-full p-3 rounded-xl text-sm shadow-sm outline-none">
                    <button @click="addExpense" class="w-full py-3 bg-indigo-600 text-white rounded-xl font-bold shadow-lg shadow-indigo-200">新增紀錄</button>
                </div>

                <div class="mt-6 space-y-2 max-h-60 overflow-y-auto no-scrollbar">
                    <div v-for="(item, idx) in expenses" :key="idx" class="flex justify-between items-center p-3 bg-white/50 rounded-xl text-xs border border-slate-100">
                        <div>
                            <span class="font-bold text-slate-600 block">{{ item.type }}</span>
                            <span class="text-[10px] text-slate-400">{{ item.date }} · {{ item.note }}</span>
                        </div>
                        <div class="flex items-center gap-3">
                            <span class="font-mono font-bold">${{ item.amount }}</span>
                            <i @click="removeExp(idx)" class="fa-solid fa-trash-can text-red-300"></i>
                        </div>
                    </div>
                </div>
                <button @click="exportData" class="w-full mt-4 py-3 bg-slate-800 text-white rounded-xl text-sm font-bold">
                    <i class="fa-solid fa-file-csv mr-2"></i>匯出報表 (CSV)
                </button>
            </section>
        </div>

        <div v-if="activeTab === 'docs'" class="space-y-4">
            <section class="glass p-6 rounded-[2rem] shadow-xl">
                <h3 class="font-bold text-slate-700 mb-4 tracking-wider">📜 行政指南</h3>
                
                <div class="space-y-4">
                    <details class="group bg-indigo-50 rounded-2xl p-4">
                        <summary class="list-none font-bold text-indigo-700 flex justify-between items-center cursor-pointer">
                            入職/離職標準流程 <i class="fa-solid fa-chevron-down text-xs group-open:rotate-180 transition-transform"></i>
                        </summary>
                        <ul class="mt-3 text-xs text-indigo-600 space-y-2 leading-loose">
                            <li>✅ 簽署勞動契約與誠實保密協議 (NDA)</li>
                            <li>✅ 當日完成勞健保與勞退加保</li>
                            <li>✅ 領取辦公設備並確認資產清單</li>
                            <li>✅ 加入 Slack / Line 工作群組</li>
                        </ul>
                    </details>

                    <details class="group bg-white border rounded-2xl p-4">
                        <summary class="list-none font-bold text-slate-700 flex justify-between items-center cursor-pointer">
                            公司工作手冊 (核心摘要) <i class="fa-solid fa-chevron-down text-xs group-open:rotate-180 transition-transform"></i>
                        </summary>
                        <div class="mt-3 text-xs text-slate-500 space-y-3 leading-relaxed">
                            <p><strong>🕒 工時：</strong> 彈性工時 09:00 - 10:00 入鏡，每天滿 8 小時。</p>
                            <p><strong>💰 報帳：</strong> 費用請於每月 25 號前使用本系統匯出給會計。</p>
                            <p><strong>🏖️ 請假：</strong> 兩天內請假於系統標記，三天以上請提早一週告知。</p>
                        </div>
                    </details>
                </div>
            </section>
        </div>

        <div v-if="activeTab === 'profile'" class="space-y-6">
            <section class="glass p-6 rounded-[2rem] shadow-xl">
                <h3 class="font-bold text-slate-700 mb-4">👤 個人基本資料建檔</h3>
                <div class="space-y-4">
                    <div>
                        <label class="text-[10px] font-bold text-slate-400 ml-1">真實姓名</label>
                        <input v-model="profile.name" type="text" placeholder="請輸入姓名" class="w-full p-3 bg-slate-50 rounded-xl text-sm border-none outline-none">
                    </div>
                    <div>
                        <label class="text-[10px] font-bold text-slate-400 ml-1">員工編號/職稱</label>
                        <input v-model="profile.id" type="text" placeholder="例如：S001 / 行銷主管" class="w-full p-3 bg-slate-50 rounded-xl text-sm border-none outline-none">
                    </div>
                    <div>
                        <label class="text-[10px] font-bold text-slate-400 ml-1">緊急聯絡人</label>
                        <input v-model="profile.emergency" type="text" placeholder="姓名 - 電話" class="w-full p-3 bg-slate-50 rounded-xl text-sm border-none outline-none">
                    </div>
                    <button @click="saveProfile" class="w-full py-4 bg-indigo-600 text-white rounded-2xl font-bold shadow-lg shadow-indigo-100">儲存變更</button>
                    <p class="text-[10px] text-center text-slate-400">資料僅保存在您的瀏覽器中</p>
                </div>
            </section>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 max-w-md mx-auto bg-white/80 backdrop-blur-lg border-t border-slate-100 flex justify-around p-4 rounded-t-[2.5rem] z-50">
        <button @click="activeTab = 'home'" :class="{ 'nav-item active': activeTab === 'home' }" class="flex flex-col items-center text-slate-300">
            <i class="fa-solid fa-house-user text-xl"></i>
            <span class="text-[10px] mt-1 font-bold">首頁</span>
        </button>
        <button @click="activeTab = 'finance'" :class="{ 'nav-item active': activeTab === 'finance' }" class="flex flex-col items-center text-slate-300">
            <i class="fa-solid fa-receipt text-xl"></i>
            <span class="text-[10px] mt-1 font-bold">報支</span>
        </button>
        <button @click="activeTab = 'docs'" :class="{ 'nav-item active': activeTab === 'docs' }" class="flex flex-col items-center text-slate-300">
            <i class="fa-solid fa-file-shield text-xl"></i>
            <span class="text-[10px] mt-1 font-bold">手冊</span>
        </button>
        <button @click="activeTab = 'profile'" :class="{ 'nav-item active': activeTab === 'profile' }" class="flex flex-col items-center text-slate-300">
            <i class="fa-solid fa-id-card text-xl"></i>
            <span class="text-[10px] mt-1 font-bold">資料</span>
        </button>
    </nav>
</div>

<script>
    const { createApp, ref, computed, onMounted, watch } = Vue;

    createApp({
        setup() {
            const activeTab = ref('home');
            const todayDate = new Date().toLocaleDateString('zh-TW', { year: 'numeric', month: 'long', day: 'numeric', weekday: 'long' });
            
            // 資料模型
            const profile = ref({ name: '', id: '', emergency: '' });
            const attendance = ref({ in: null, out: null });
            const expenses = ref([]);
            const newExp = ref({ type: '交通費', amount: null, note: '' });

            // 初始化載入
            onMounted(() => {
                const p = localStorage.getItem('emp_profile');
                if (p) profile.ref = JSON.parse(p);
                const a = localStorage.getItem('emp_clock');
                if (a) attendance.value = JSON.parse(a);
                const e = localStorage.getItem('emp_exp');
                if (e) expenses.value = JSON.parse(e);
            });

            // 監聽存檔
            watch(profile, (v) => localStorage.setItem('emp_profile', JSON.stringify(v)), { deep: true });
            watch(expenses, (v) => localStorage.setItem('emp_exp', JSON.stringify(v)), { deep: true });

            const totalExp = computed(() => expenses.value.reduce((s, i) => s + i.amount, 0));

            const clockIn = () => {
                attendance.value.in = new Date().toLocaleTimeString('zh-TW', { hour12: false, hour: '2-digit', minute: '2-digit' });
                localStorage.setItem('emp_clock', JSON.stringify(attendance.value));
            };

            const clockOut = () => {
                attendance.value.out = new Date().toLocaleTimeString('zh-TW', { hour12: false, hour: '2-digit', minute: '2-digit' });
                localStorage.setItem('emp_clock', JSON.stringify(attendance.value));
            };

            const addExpense = () => {
                if (newExp.value.amount > 0) {
                    expenses.value.unshift({ ...newExp.value, date: new Date().toLocaleDateString() });
                    newExp.value = { type: '交通費', amount: null, note: '' };
                }
            };

            const removeExp = (idx) => { if(confirm('確定刪除？')) expenses.value.splice(idx, 1); };

            const saveProfile = () => { 
                localStorage.setItem('emp_profile', JSON.stringify(profile.value));
                alert('基本資料已更新！'); 
            };

            const exportData = () => {
                let csv = "\ufeff日期,類別,金額,備註\n";
                expenses.value.forEach(i => csv += `${i.date},${i.type},${i.amount},${i.note}\n`);
                csv += `\n員工：,${profile.value.name},工號：,${profile.value.id}\n`;
                csv += `上班打卡：,${attendance.value.in || '--'},下班打卡：,${attendance.value.out || '--'}\n`;
                csv += `總計：,${totalExp.value}\n`;

                const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement("a");
                a.href = url;
                a.download = `報支單_${profile.value.name}_${new Date().toISOString().slice(0,10)}.csv`;
                a.click();
            };

            return { activeTab, todayDate, profile, attendance, expenses, newExp, totalExp, clockIn, clockOut, addExpense, removeExp, saveProfile, exportData };
        }
    }).mount('#app');
</script>
</body>
</html>
