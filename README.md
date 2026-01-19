<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>تتبع الكريدي (ألوان وتعديلات)</title>
    
    <!-- مكتبات Firebase Compat -->
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-firestore-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-auth-compat.js"></script>

    <style>
        :root {
            --bg-body: #f8fafc; --bg-surface: #ffffff;
            --primary: #2563eb; --primary-light: #eff6ff;
            --danger: #ef4444; --danger-light: #fef2f2;
            --success: #22c55e; --success-light: #f0fdf4;
            --text-main: #0f172a; --text-sub: #64748b;
            --border: #e2e8f0; --radius: 16px;
            --purple: #8b5cf6;
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; outline: none; }
        body { font-family: system-ui, -apple-system, sans-serif; background-color: var(--bg-body); margin: 0; padding-bottom: 120px; color: var(--text-main); user-select: none; }
        
        /* Auth Screen */
        #auth-screen { position: fixed; inset: 0; background: var(--bg-body); z-index: 200; display: flex; flex-direction: column; justify-content: center; align-items: center; padding: 20px; overflow-y: auto; }
        .auth-card { background: white; padding: 2rem; border-radius: 20px; width: 100%; max-width: 400px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); text-align: center; }
        .auth-input { width: 100%; padding: 15px; margin-bottom: 15px; border: 2px solid var(--border); border-radius: 12px; font-size: 1rem; font-weight: bold; transition: 0.3s; }
        .auth-input:focus { border-color: var(--primary); }
        .auth-btn { width: 100%; padding: 15px; background: var(--primary); color: white; border: none; border-radius: 12px; font-weight: 900; font-size: 1.1rem; cursor: pointer; margin-bottom: 10px; }
        .auth-link { color: var(--primary); font-weight: bold; cursor: pointer; font-size: 0.9rem; text-decoration: none; display: block; margin-top: 10px; }
        .checkbox-container { display: flex; align-items: center; gap: 8px; margin-bottom: 15px; font-weight: bold; font-size: 0.9rem; color: var(--text-sub); justify-content: center; }
        .checkbox-container input { width: 18px; height: 18px; accent-color: var(--primary); }
        .error-msg { color: var(--danger); margin-top: 10px; font-weight: bold; display: none; background: var(--danger-light); padding: 10px; border-radius: 8px; font-size: 0.85rem; }
        .success-msg { color: var(--success); margin-top: 10px; font-weight: bold; display: none; background: var(--success-light); padding: 10px; border-radius: 8px; font-size: 0.85rem; }

        #app-content { display: none; }

        /* General UI */
        .flex { display: flex; } .gap-2 { gap: 0.5rem; } .w-full { width: 100%; }
        .font-black { font-weight: 900; } .font-bold { font-weight: 700; }
        .text-sm { font-size: 0.875rem; } .text-xs { font-size: 0.75rem; } .text-xl { font-size: 1.25rem; }
        .hidden { display: none !important; }
        .icon { width: 22px; height: 22px; stroke: currentColor; stroke-width: 2.5; fill: none; stroke-linecap: round; stroke-linejoin: round; }

        header { background: var(--bg-surface); padding: 1rem; position: sticky; top: 0; z-index: 20; border-bottom: 1px solid var(--border); display: flex; justify-content: space-between; align-items: center; box-shadow: 0 2px 4px rgba(0,0,0,0.02); }
        #loading-indicator { position: fixed; top: 0; left: 0; right: 0; height: 3px; background: #e2e8f0; z-index: 300; }
        #loading-bar { height: 100%; background: var(--primary); width: 0%; transition: width 0.3s; }
        
        .stats-container { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 10px; padding: 1rem; }
        .stat-box { background: var(--bg-surface); padding: 1rem 0.5rem; border-radius: var(--radius); text-align: center; border: 2px solid transparent; transition: transform 0.1s; box-shadow: 0 1px 3px rgba(0,0,0,0.05); }
        .stat-box:active { transform: scale(0.95); }
        .stat-box.debtors { background: var(--danger-light); color: var(--danger); }
        .stat-box.clear { background: var(--success-light); color: var(--success); }
        
        .reminder-section { margin: 0 1rem; background: var(--bg-surface); padding: 1rem; border-radius: var(--radius); border: 1px solid var(--border); }
        .reminder-item { display: flex; justify-content: space-between; align-items: center; padding: 0.75rem; background: var(--bg-body); border-radius: 10px; margin-bottom: 0.5rem; }
        
        .search-bar { margin: 1rem; position: relative; }
        .search-bar input { width: 100%; padding: 1rem 1rem 1rem 3rem; border-radius: var(--radius); border: 2px solid var(--border); font-weight: bold; font-size: 1rem; }
        .search-bar .icon { position: absolute; right: 1rem; top: 1rem; color: var(--text-sub); }
        
        /* Grid Layout */
        #customers-list { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; padding: 0 1rem 20px 1rem; }
        @media (min-width: 768px) { #customers-list { grid-template-columns: repeat(5, 1fr); } }

        /* Card Style - UPDATED FOR COLORING */
        .customer-card {
            background: var(--bg-surface); padding: 1rem 0.5rem; border-radius: var(--radius);
            display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center;
            border: 2px solid transparent; /* Default no border */
            box-shadow: 0 1px 3px rgba(0,0,0,0.05); transition: transform 0.1s; cursor: pointer;
            height: 100%; min-height: 140px;
        }
        .customer-card:active { transform: scale(0.98); background: #f1f5f9; }
        
        /* TINTED COLORS FOR WARNING/DANGER */
        .customer-card.danger { 
            border: 2px solid #ef4444; 
            background: #fee2e2; 
        }
        .customer-card.warning { 
            border: 2px solid #f97316; 
            background: #ffedd5; 
        }
        
        /* Avatar */
        .customer-avatar {
            width: 50px; height: 50px; border-radius: 50%; 
            background: #e0e7ff; color: #4338ca;
            display: flex; flex-direction: column; 
            justify-content: center; align-items: center; 
            margin-bottom: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            line-height: 1;
        }

        .card-name { font-weight: 900; font-size: 1rem; margin-bottom: 5px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; width: 100%; }
        .card-balance { font-weight: 900; font-size: 1.3rem; letter-spacing: -0.5px; color: var(--text-main); }
        .card-balance span { font-size: 0.7rem; color: var(--text-sub); }
        
        /* Tabs System */
        .tabs-wrapper { padding: 10px 1rem 0 1rem; display: flex; gap: 10px; margin-bottom: 10px; }
        .tab-btn { flex: 1; padding: 12px; border-radius: 12px; border: none; font-weight: 900; cursor: pointer; background: white; color: var(--text-sub); border: 2px solid var(--border); transition: 0.2s; display: flex; align-items: center; justify-content: center; gap: 8px; }
        .tab-btn.active.shop { background: var(--primary); color: white; border-color: var(--primary); }
        .tab-btn.active.library { background: var(--purple); color: white; border-color: var(--purple); }

        .fab-btn { position: fixed; bottom: 1.5rem; left: 1.5rem; right: 1.5rem; background: var(--primary); color: white; padding: 1.2rem; border-radius: 1rem; font-weight: 900; font-size: 1.1rem; display: flex; justify-content: center; align-items: center; gap: 0.5rem; box-shadow: 0 10px 15px -3px rgba(37, 99, 235, 0.4); border: none; z-index: 30; }
        .fab-btn:active { transform: scale(0.95); }
        .fab-btn.library-mode { background: var(--purple); }
        
        .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.6); z-index: 50; backdrop-filter: blur(4px); display: flex; justify-content: center; align-items: flex-end; }
        .modal-content { background: #f8fafc; width: 100%; max-width: 600px; border-radius: 2rem 2rem 0 0; padding: 1.5rem; max-height: 95vh; overflow-y: auto; animation: slideUp 0.3s cubic-bezier(0.16, 1, 0.3, 1); display: flex; flex-direction: column; gap: 1rem; }
        @keyframes slideUp { from { transform: translateY(100%); } to { transform: translateY(0); } }
        
        input, select { width: 100%; padding: 1rem; border-radius: 12px; border: 2px solid var(--border); font-size: 1rem; font-weight: bold; background: #fff; }
        input:focus { border-color: var(--primary); }
        .btn-primary { background: var(--primary); color: white; padding: 1rem; border-radius: 12px; font-weight: bold; border: none; width: 100%; }
        .btn-secondary { background: #e2e8f0; color: var(--text-main); padding: 1rem; border-radius: 12px; font-weight: bold; border: none; width: 100%; }
        
        .history-section { display: flex; flex-direction: column; gap: 10px; margin-top: 10px; }
        .trans-card { background: white; padding: 1.2rem; border-radius: 16px; box-shadow: 0 2px 5px rgba(0,0,0,0.03); display: flex; justify-content: space-between; align-items: center; border-right: 6px solid transparent; }
        .trans-take { border-right-color: var(--danger); }
        .trans-give { border-right-color: var(--success); }
        .trans-info { display: flex; flex-direction: column; gap: 4px; }
        .trans-note { font-weight: 900; font-size: 1rem; color: var(--text-main); }
        .trans-date { font-size: 0.75rem; color: var(--text-sub); font-weight: bold; display: flex; align-items: center; gap: 4px; }
        .trans-amount { text-align: left; min-width: 90px; display: flex; flex-direction: column; align-items: flex-end; }
        .amount-val { font-size: 1.3rem; font-weight: 900; letter-spacing: -0.5px; }
        .amount-label { font-size: 0.7rem; font-weight: bold; padding: 2px 8px; border-radius: 6px; margin-top: 2px; }
        .trans-take .amount-val { color: var(--danger); }
        .trans-take .amount-label { background: var(--danger-light); color: var(--danger); }
        .trans-give .amount-val { color: var(--success); }
        .trans-give .amount-label { background: var(--success-light); color: var(--success); }
        .stat-list-item { display: flex; justify-content: space-between; align-items: center; padding: 1rem; border-bottom: 1px solid var(--border); cursor: pointer; background: white; }

        .task-item { background: white; padding: 1rem; border-radius: 12px; margin-bottom: 8px; display: flex; justify-content: space-between; align-items: center; border: 1px solid var(--border); transition: 0.2s; }
        .task-item.done { background: #f0fdf4; border-color: #86efac; }
        .task-item.done .task-text { text-decoration: line-through; opacity: 0.6; }
        .task-text { font-weight: bold; font-size: 1rem; flex: 1; margin-left: 10px; }
        .task-actions { display: flex; gap: 8px; }
        .btn-icon-small { padding: 6px; border-radius: 8px; border: none; cursor: pointer; display: flex; align-items: center; justify-content: center; }

        /* Badge */
        .badge-container { position: relative; display: inline-block; }
        .notification-badge { position: absolute; top: -5px; right: -5px; background-color: var(--danger); color: white; border-radius: 50%; width: 18px; height: 18px; display: flex; justify-content: center; align-items: center; font-size: 10px; font-weight: bold; border: 2px solid white; display: none; }

        #print-area { display: none; }
        @media print {
            body * { visibility: hidden; }
            #print-area, #print-area * { visibility: visible; }
            #print-area { display: block; position: absolute; left: 0; top: 0; width: 100%; background: white; padding: 20px; direction: rtl; }
            table { width: 100%; border-collapse: collapse; margin-top: 20px; }
            th, td { border: 1px solid #000; padding: 10px; text-align: right; font-size: 14px; }
            th { background-color: #f0f0f0; }
            .print-header { text-align: center; margin-bottom: 30px; border-bottom: 2px solid #000; padding-bottom: 10px; }
            .print-total { margin-top: 20px; font-weight: bold; text-align: left; font-size: 18px; }
        }
    </style>
</head>
<body>

    <div id="loading-indicator"><div id="loading-bar"></div></div>

    <!-- Auth Screen -->
    <div id="auth-screen">
        <div id="login-view" class="auth-card">
            <h1 class="font-black text-xl" style="color: var(--primary); margin-bottom: 20px;">تسجيل الدخول</h1>
            <input type="email" id="login-email" class="auth-input" placeholder="البريد الإلكتروني" dir="ltr">
            <input type="password" id="login-pass" class="auth-input" placeholder="كلمة المرور" dir="ltr">
            <div class="checkbox-container"><input type="checkbox" id="remember-me" checked><label for="remember-me">تذكرني</label></div>
            <button onclick="performLogin()" class="auth-btn">دخول</button>
            <div onclick="showView('signup')" class="auth-link">إنشاء حساب جديد</div>
            <div onclick="showView('reset')" class="auth-link" style="color: var(--text-sub);">نسيت كلمة السر؟</div>
            <p id="login-msg" class="error-msg"></p>
        </div>
        <div id="signup-view" class="auth-card" style="display:none;">
            <h1 class="font-black text-xl" style="color: var(--primary); margin-bottom: 20px;">حساب جديد</h1>
            <input type="email" id="signup-email" class="auth-input" placeholder="البريد الإلكتروني" dir="ltr">
            <input type="password" id="signup-pass" class="auth-input" placeholder="كلمة المرور" dir="ltr">
            <button onclick="performSignup()" class="auth-btn">تسجيل</button>
            <div onclick="showView('login')" class="auth-link">لديك حساب؟ تسجيل الدخول</div>
            <p id="signup-msg" class="error-msg"></p>
        </div>
        <div id="reset-view" class="auth-card" style="display:none;">
            <h1 class="font-black text-xl" style="color: var(--primary); margin-bottom: 20px;">استعادة الحساب</h1>
            <input type="email" id="reset-email" class="auth-input" placeholder="البريد الإلكتروني" dir="ltr">
            <button onclick="performReset()" class="auth-btn">إرسال الرابط</button>
            <div onclick="showView('login')" class="auth-link">رجوع</div>
            <p id="reset-msg" class="error-msg"></p>
            <p id="reset-success" class="success-msg"></p>
        </div>
    </div>

    <!-- App Content -->
    <div id="app-content">
        <header>
            <div style="display: flex; align-items: center; gap: 10px;">
                <h1 class="font-black text-xl" style="color: var(--text-main); margin:0;" id="app-title">تتبع الكريدي</h1>
            </div>
            <div class="flex gap-2">
                <button onclick="openGeneralReminders()" class="badge-container" style="background: var(--bg-body); padding: 8px; border-radius: 50%; border: 1px solid var(--border); cursor:pointer;">
                    <svg class="icon" viewBox="0 0 24 24"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path><path d="M13.73 21a2 2 0 0 1-3.46 0"></path></svg>
                    <span id="header-reminder-count" class="notification-badge">0</span>
                </button>
                <button onclick="openPrintModal()" style="background: var(--bg-body); padding: 8px; border-radius: 50%; border: 1px solid var(--border); cursor:pointer;"><svg class="icon" viewBox="0 0 24 24"><polyline points="6 9 6 2 18 2 18 9"></polyline><path d="M6 18H4a2 2 0 0 1-2-2v-5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2h-2"></path><rect x="6" y="14" width="12" height="8"></rect></svg></button>
                <button onclick="openSettings()" style="background: var(--bg-body); padding: 8px; border-radius: 50%; border: 1px solid var(--border); cursor:pointer;"><svg class="icon" viewBox="0 0 24 24"><path d="M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73-.73l.22-.39a2 2 0 0 0-.73-2.73l-.15-.08a2 2 0 0 1-1-1.74v-.47a2 2 0 0 1 1-1.74l.15-.09a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z"/><circle cx="12" cy="12" r="3"/></svg></button>
                <button onclick="logout()" style="background: #fee2e2; color: #dc2626; padding: 8px; border-radius: 50%; border: none; cursor:pointer;"><svg class="icon" viewBox="0 0 24 24"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" x2="9" y1="12" y2="12"/></svg></button>
            </div>
        </header>

        <!-- TABS -->
        <div class="tabs-wrapper">
            <button class="tab-btn active shop" id="tab-shop" onclick="switchTab('customers')">
                <svg class="icon" viewBox="0 0 24 24"><path d="M6 2 3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4Z"/><path d="M3 6h18"/><path d="M16 10a4 4 0 0 1-8 0"/></svg>
                كريدي المحل
            </button>
            <button class="tab-btn" id="tab-lib" onclick="switchTab('library')">
                <svg class="icon" viewBox="0 0 24 24"><path d="M4 19.5v-15A2.5 2.5 0 0 1 6.5 2H20v20H6.5a2.5 2.5 0 0 1 0-5H20"/></svg>
                كريدي المكتبة
            </button>
        </div>

        <div class="stats-container">
            <div class="stat-box total" onclick="openStatsModal('debtors')">
                <div class="text-xs font-bold text-sub">المجموع</div>
                <div id="stat-total" class="font-black text-xl" dir="ltr">...</div>
            </div>
            <div class="stat-box debtors" onclick="openStatsModal('debtors')">
                <div class="text-xs font-bold">مدينون</div>
                <div id="stat-debtors" class="font-black text-xl">...</div>
            </div>
            <div class="stat-box clear" onclick="openStatsModal('clear')">
                <div class="text-xs font-bold">خالصون</div>
                <div id="stat-clear" class="font-black text-xl">...</div>
            </div>
        </div>

        <div class="reminder-section">
            <div class="flex justify-between items-center" style="margin-bottom: 10px;">
                <h3 class="font-bold text-sm text-sub flex items-center gap-2">
                    <svg class="icon" style="width:16px;" viewBox="0 0 24 24"><path d="M6 2 3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4Z"/><path d="M3 6h18"/><path d="M16 10a4 4 0 0 1-8 0"/></svg> نواقص (للمتجر)
                </h3>
            </div>
            <div class="flex gap-2" style="margin-bottom: 10px;">
                <input id="rem-name" placeholder="المنتج" style="padding: 0.6rem;">
                <input id="rem-count" type="number" placeholder="#" style="width: 60px; padding: 0.6rem;">
                <button onclick="addReminder()" style="background: var(--primary); color: white; border-radius: 10px; padding: 0 12px; border:none; cursor:pointer;"><svg class="icon" viewBox="0 0 24 24"><path d="M5 12h14M12 5v14"/></svg></button>
            </div>
            <div id="reminders-list"></div>
        </div>

        <div class="search-bar">
            <input id="search-input" type="text" placeholder="بحث عن زبون..." onkeyup="renderCustomers()">
            <svg class="icon" viewBox="0 0 24 24"><circle cx="11" cy="11" r="8"/><path d="m21 21-4.3-4.3"/></svg>
        </div>

        <div id="customers-list"></div>

        <button id="main-fab" class="fab-btn" onclick="openModal('modal-add')">
            <svg class="icon" viewBox="0 0 24 24"><path d="M16 21v-2a4 4 0 0 0-4-4H6a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M22 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>
            زبون جديد
        </button>
    </div>

    <!-- === MODALS === -->

    <!-- General Reminders Modal -->
    <div id="modal-general-reminders" class="modal-overlay hidden">
        <div class="modal-content" style="background: white; max-height: 85vh;">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-black">تذكيرات ومهام</h2>
                <button onclick="closeModal('modal-general-reminders')" class="btn-icon-small" style="background:#f1f5f9;"><svg class="icon" viewBox="0 0 24 24"><path d="M18 6 6 18"/><path d="m6 6 12 12"/></svg></button>
            </div>
            <div class="flex gap-2 mb-4">
                <input id="new-gen-task" placeholder="اكتب التذكير هنا..." style="flex:1;">
                <button onclick="addGeneralReminder()" class="btn-primary" style="width:auto; padding: 0 15px;">إضافة</button>
            </div>
            <div id="general-reminders-list" style="flex:1; overflow-y:auto; padding-bottom: 20px;"></div>
        </div>
    </div>

    <!-- Print Options Modal -->
    <div id="modal-print" class="modal-overlay hidden">
        <div class="modal-content" style="background: white; max-width: 400px;">
            <h2 class="text-xl font-black mb-4">خيارات الطباعة</h2>
            <div class="flex flex-col gap-3">
                <button onclick="generateReport('all')" class="btn-primary" style="background: #0f172a;">طباعة تقرير الديون الشامل</button>
                <div style="border-top:1px solid #eee; margin:10px 0;"></div>
                <label class="text-sm font-bold text-sub">أو طباعة عمليات شهر محدد:</label>
                <input type="month" id="print-month" class="auth-input">
                <button onclick="generateReport('month')" class="btn-primary">طباعة عمليات الشهر</button>
                <button onclick="closeModal('modal-print')" class="btn-secondary">إلغاء</button>
            </div>
        </div>
    </div>

    <!-- Hidden Print Area -->
    <div id="print-area"></div>

    <!-- Add Customer Modal -->
    <div id="modal-add" class="modal-overlay hidden">
        <div class="modal-content" style="background: white;">
            <h2 class="text-xl font-black">إضافة زبون <span id="add-mode-title" style="font-size:0.7em; color:var(--primary);">(محل)</span></h2>
            <input id="new-name" placeholder="الاسم الكامل">
            <input id="new-type" placeholder="نوع الكريدي (اختياري)">
            <input id="new-amount" type="number" inputmode="numeric" placeholder="المبلغ الأولي">
            <div style="margin-bottom: 10px;">
                <label style="font-size:0.8rem; font-weight:bold; color:var(--text-sub); display:block; margin-bottom:5px;">تاريخ (اختياري)</label>
                <input id="new-date" type="date">
            </div>
            <div class="flex gap-2" style="margin-top: 10px;">
                <button onclick="saveCustomer()" class="btn-primary">حفظ في السحابة</button>
                <button onclick="closeModal('modal-add')" class="btn-secondary">إلغاء</button>
            </div>
        </div>
    </div>

    <!-- Customer Details Modal -->
    <div id="modal-details" class="modal-overlay hidden">
        <div class="modal-content">
            <div class="flex justify-between items-start" style="background: white; padding: 1rem; margin: -1.5rem -1.5rem 0 -1.5rem; border-bottom: 1px solid var(--border); position: sticky; top: -1.5rem; z-index: 10;">
                <div>
                    <h2 id="d-name" class="text-xl font-black" style="margin:0;"></h2>
                    <span id="d-type" class="text-xs font-bold" style="color: var(--primary);"></span>
                </div>
                <button onclick="closeModal('modal-details')" style="background:#f1f5f9; padding:8px; border-radius:50%; border:none; cursor:pointer;">
                    <svg class="icon" style="width:20px;" viewBox="0 0 24 24"><path d="M18 6 6 18"/><path d="m6 6 12 12"/></svg>
                </button>
            </div>
            <div style="background: var(--text-main); color: white; padding: 1.5rem; border-radius: 1.5rem; text-align: center; box-shadow: 0 4px 10px rgba(0,0,0,0.1); margin-top: 10px;">
                <div class="text-xs font-bold opacity-70">المبلغ الإجمالي</div>
                <div id="d-balance" class="font-black" style="font-size: 2.5rem;" dir="ltr">0</div>
            </div>
            <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px;">
                <button id="btn-take" onclick="setTransMode('take')" class="btn-action" style="background: var(--danger-light); color: var(--danger); padding: 1rem; border-radius: 12px; font-weight: 900; border: 2px solid var(--danger);">عليه (كريدي)</button>
                <button id="btn-give" onclick="setTransMode('give')" class="btn-action" style="background: white; color: var(--text-main); padding: 1rem; border-radius: 12px; font-weight: 900; border: 1px solid var(--border);">له (دفع)</button>
            </div>
            <div class="flex gap-2">
                <input id="t-amount" type="number" inputmode="numeric" placeholder="المبلغ" style="flex: 1;">
                <input id="t-note" type="text" placeholder="ملاحظة" style="flex: 1.5;">
            </div>
            <div style="margin-bottom: 10px;">
                <input id="t-date" type="date" style="width:100%; border:1px solid #e2e8f0; font-size:0.9rem;" title="اختر تاريخ للعملية القديمة">
            </div>
            <button onclick="addTransaction()" class="btn-primary" style="padding: 1rem;">تسجيل العملية</button>
            <div style="margin-top: 10px;">
                <h3 class="font-bold text-sm text-sub" style="margin-bottom: 10px;">سجل العمليات (التفاصيل)</h3>
                <div id="d-history" class="history-section"></div>
            </div>
            <button onclick="requestPinToDelete()" style="color: var(--danger); background: white; padding: 1rem; border-radius: 12px; font-weight: bold; border: 2px solid var(--danger-light); display: flex; justify-content: center; align-items: center; gap: 8px; margin-top: 10px; cursor:pointer;">
                <svg class="icon" viewBox="0 0 24 24"><path d="M3 6h18"/><path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"/><path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"/></svg> حذف الزبون
            </button>
        </div>
    </div>

    <!-- PIN Modal -->
    <div id="modal-pin" class="modal-overlay hidden">
        <div class="modal-content" style="background: white; max-width: 350px;">
            <h2 class="text-xl font-black text-center" style="color: var(--danger);">تأكيد الحذف</h2>
            <p style="text-align:center; color:var(--text-sub); font-size:0.9rem; margin-bottom:15px;">أدخل الرمز السري لإتمام العملية</p>
            <input type="password" id="pin-input" placeholder="الرمز السري" dir="ltr" style="text-align:center; letter-spacing: 5px; font-size: 1.5rem;">
            <button onclick="confirmDeleteCustomer()" class="btn-primary" style="background: var(--danger); margin-top:10px;">حذف نهائي</button>
            <button onclick="closeModal('modal-pin')" class="btn-secondary" style="margin-top:10px;">إلغاء</button>
        </div>
    </div>

    <!-- Delete Reminder Modal -->
    <div id="modal-delete-reminder" class="modal-overlay hidden">
        <div class="modal-content" style="background: white; max-width: 350px;">
            <h2 class="text-xl font-black text-center" style="color: var(--danger);">حذف المنتج</h2>
            <p class="text-center text-sub">هل أنت متأكد من حذف هذا المنتج من القائمة؟</p>
            <div class="flex gap-2 mt-4">
                <button onclick="confirmDeleteRem()" class="btn-primary" style="background:var(--danger)">حذف</button>
                <button onclick="closeModal('modal-delete-reminder')" class="btn-secondary">إلغاء</button>
            </div>
        </div>
    </div>

    <!-- Stats Modal -->
    <div id="modal-stats" class="modal-overlay hidden">
        <div class="modal-content" style="background: white;">
            <div class="flex justify-between items-center">
                <h2 id="stats-title" class="text-xl font-black"></h2>
                <button onclick="closeModal('modal-stats')" style="padding: 8px; cursor:pointer;"><svg class="icon" viewBox="0 0 24 24"><path d="M18 6 6 18"/><path d="m6 6 12 12"/></svg></button>
            </div>
            <div id="stats-list" style="overflow-y: auto; max-height: 60vh;"></div>
        </div>
    </div>

    <!-- Settings Modal -->
    <div id="modal-settings" class="modal-overlay hidden">
        <div class="modal-content" style="background: white;">
            <h2 class="text-xl font-black">الإعدادات</h2>
            <label class="font-bold text-sm">تنبيه برتقالي (أيام)</label>
            <input id="set-warn" type="number">
            <label class="font-bold text-sm">تنبيه أحمر (أيام)</label>
            <input id="set-danger" type="number">
            <button onclick="saveSettings()" class="btn-primary" style="margin-top: 10px;">حفظ التغييرات</button>
        </div>
    </div>

    <!-- Logic -->
    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyAcTJZNbTFrHwuaAqzCkkSm_HUm_ZSlFbk",
            authDomain: "kriiidi.firebaseapp.com",
            projectId: "kriiidi",
            storageBucket: "kriiidi.firebasestorage.app",
            messagingSenderId: "429962127465",
            appId: "1:429962127465:web:f2f4c6ead48a1ef4f5daf8",
            measurementId: "G-833X5K1JSB"
        };

        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();
        const auth = firebase.auth();

        db.enablePersistence().catch(err => console.log(err.code));

        let customers = [], reminders = [], generalReminders = [], settings = {warn: 30, danger: 45}, currentId = null, transMode = 'take';
        let reminderToDelete = null;
        let currentCollection = 'customers'; // Default collection
        let unsubscribeCustomers = null; // To handle listener switching

        // Colors
        const THEMES = {
            shop: {
                primary: '#2563eb',
                gradient: 'linear-gradient(135deg, #2563eb, #1d4ed8)',
                bg: '#f1f5f9'
            },
            library: {
                primary: '#8b5cf6',
                gradient: 'linear-gradient(135deg, #8b5cf6, #7c3aed)',
                bg: '#f3e8ff'
            }
        };

        function applyTheme(themeName) {
            const theme = THEMES[themeName];
            const root = document.documentElement;
            root.style.setProperty('--primary', theme.primary);
            root.style.setProperty('--primary-gradient', theme.gradient);
            root.style.setProperty('--bg-body', theme.bg);
        }

        function showLoading() { document.getElementById('loading-bar').style.width = '70%'; }
        function hideLoading() { document.getElementById('loading-bar').style.width = '100%'; setTimeout(()=> document.getElementById('loading-bar').style.width='0%', 300); }
        function openModal(id) { document.getElementById(id).classList.remove('hidden'); }
        function closeModal(id) { document.getElementById(id).classList.add('hidden'); }
        function showView(view) {
            ['login-view', 'signup-view', 'reset-view'].forEach(v => document.getElementById(v).style.display = 'none');
            document.getElementById(view + '-view').style.display = 'block';
            document.querySelectorAll('.error-msg, .success-msg').forEach(e => e.style.display = 'none');
        }

        // --- AUTH ---
        auth.onAuthStateChanged((user) => {
            if (user) {
                document.getElementById('auth-screen').style.display = 'none';
                document.getElementById('app-content').style.display = 'block';
                initApp();
            } else {
                document.getElementById('auth-screen').style.display = 'flex';
                document.getElementById('app-content').style.display = 'none';
                showView('login');
            }
        });

        function performLogin() {
            const email = document.getElementById('login-email').value;
            const pass = document.getElementById('login-pass').value;
            const remember = document.getElementById('remember-me').checked;
            const msg = document.getElementById('login-msg');
            if(!email || !pass) { msg.innerText = "يرجى ملء البيانات"; msg.style.display = 'block'; return; }
            msg.style.display = 'none'; showLoading();
            const persistence = remember ? firebase.auth.Auth.Persistence.LOCAL : firebase.auth.Auth.Persistence.SESSION;
            auth.setPersistence(persistence).then(() => auth.signInWithEmailAndPassword(email, pass)).catch((error) => {
                hideLoading(); msg.innerText = "خطأ في البريد أو كلمة المرور"; msg.style.display = 'block';
            });
        }

        function performSignup() {
            const email = document.getElementById('signup-email').value;
            const pass = document.getElementById('signup-pass').value;
            const msg = document.getElementById('signup-msg');
            if(!email || !pass) { msg.innerText = "يرجى ملء البيانات"; msg.style.display = 'block'; return; }
            msg.style.display = 'none'; showLoading();
            auth.createUserWithEmailAndPassword(email, pass).catch(e => { hideLoading(); msg.innerText = e.message; msg.style.display = 'block'; });
        }

        function performReset() {
            const email = document.getElementById('reset-email').value;
            const msg = document.getElementById('reset-msg'); const suc = document.getElementById('reset-success');
            if(!email) { msg.innerText = "أدخل البريد الإلكتروني"; msg.style.display = 'block'; return; }
            msg.style.display = 'none'; suc.style.display = 'none'; showLoading();
            auth.sendPasswordResetEmail(email).then(() => { hideLoading(); suc.innerText = "تم إرسال الرابط."; suc.style.display = 'block'; }).catch((error) => { hideLoading(); msg.innerText = "البريد غير مسجل"; msg.style.display = 'block'; });
        }

        function logout() { auth.signOut(); }

        // --- APP LOGIC ---
        function initApp() {
            showLoading();
            loadCollection(); // Load default
            db.collection("reminders").onSnapshot(snap => { reminders = snap.docs.map(doc => ({id: doc.id, ...doc.data()})); renderReminders(); });
            db.collection("general_reminders").orderBy("createdAt", "desc").onSnapshot(snap => {
                generalReminders = snap.docs.map(doc => ({id: doc.id, ...doc.data()}));
                renderGeneralReminders();
            });
            db.collection("config").doc("settings").onSnapshot(doc => { if(doc.exists) settings = doc.data(); else db.collection("config").doc("settings").set(settings); });
        }

        // --- GENERAL REMINDERS LOGIC ---
        function openGeneralReminders() { openModal('modal-general-reminders'); }
        function addGeneralReminder() {
            const text = document.getElementById('new-gen-task').value;
            if(!text) return;
            db.collection('general_reminders').add({ text: text, isDone: false, createdAt: new Date().toISOString() });
            document.getElementById('new-gen-task').value = '';
        }
        function toggleGeneralReminder(id, currentStatus) { db.collection('general_reminders').doc(id).update({ isDone: !currentStatus }); }
        function deleteGeneralReminder(id) { if(confirm('حذف هذا التذكير؟')) { db.collection('general_reminders').doc(id).delete(); } }
        function renderGeneralReminders() {
            const list = document.getElementById('general-reminders-list');
            const badge = document.getElementById('header-reminder-count');
            const activeCount = generalReminders.filter(r => !r.isDone).length;
            badge.innerText = activeCount; badge.style.display = activeCount > 0 ? 'flex' : 'none';
            if(generalReminders.length === 0) { list.innerHTML = '<div style="text-align:center; padding:20px; color:#aaa;">لا توجد تذكيرات</div>'; return; }
            list.innerHTML = generalReminders.map(r => `
                <div class="task-item ${r.isDone ? 'done' : ''}">
                    <div class="task-text">${r.text}</div>
                    <div class="task-actions">
                        <button onclick="toggleGeneralReminder('${r.id}', ${r.isDone})" class="btn-icon-small" style="background:${r.isDone?'#dcfce7':'#f1f5f9'}; color:${r.isDone?'#16a34a':'#64748b'}"><svg class="icon" viewBox="0 0 24 24"><polyline points="20 6 9 17 4 12"></polyline></svg></button>
                        <button onclick="deleteGeneralReminder('${r.id}')" class="btn-icon-small" style="background:#fee2e2; color:#ef4444;"><svg class="icon" viewBox="0 0 24 24"><path d="M3 6h18"/><path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"/><path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"/></svg></button>
                    </div>
                </div>`).join('');
        }

        // --- TABS SYSTEM ---
        function switchTab(collectionName) {
            if(currentCollection === collectionName) return;
            currentCollection = collectionName;
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
            if(collectionName === 'customers') {
                document.getElementById('tab-shop').classList.add('active');
                document.getElementById('main-fab').classList.remove('library-mode');
                document.getElementById('add-mode-title').innerText = '(محل)';
                applyTheme('shop');
            } else {
                document.getElementById('tab-lib').classList.add('active');
                document.getElementById('main-fab').classList.add('library-mode');
                document.getElementById('add-mode-title').innerText = '(مكتبة)';
                applyTheme('library');
            }
            loadCollection();
        }

        function loadCollection() {
            showLoading();
            if(unsubscribeCustomers) unsubscribeCustomers(); // Stop listening to old
            unsubscribeCustomers = db.collection(currentCollection).onSnapshot(snap => {
                customers = snap.docs.map(doc => ({id: doc.id, ...doc.data()}));
                renderAll(); 
                hideLoading();
            });
        }

        function getBal(c) { if(!c.transactions) return 0; return c.transactions.reduce((a,t)=> t.type==='take'?a+t.amount:a-t.amount, 0); }
        function getDays(c) { 
            if(!c.transactions || c.transactions.length === 0) return 0;
            const last = [...c.transactions].reverse().find(t => t.type === 'take');
            if(!last) return 0;
            return Math.floor((new Date() - new Date(last.date)) / (1000 * 60 * 60 * 24));
        }

        function getColorFromName(name) {
            const colors = ['#f87171', '#fb923c', '#fbbf24', '#a3e635', '#34d399', '#22d3ee', '#818cf8', '#e879f9', '#f472b6'];
            let hash = 0; for (let i = 0; i < name.length; i++) hash = name.charCodeAt(i) + ((hash << 5) - hash);
            return colors[Math.abs(hash) % colors.length];
        }

        function renderAll() { renderCustomers(); renderStatsBox(); }
        function renderStatsBox() {
            let t = 0, d = 0, c = 0;
            customers.forEach(cust => { let b = getBal(cust); if(b > 0) { t += b; d++; } else c++; });
            document.getElementById('stat-total').innerText = t.toLocaleString();
            document.getElementById('stat-debtors').innerText = d;
            document.getElementById('stat-clear').innerText = c;
        }

        function renderReminders() {
            const list = document.getElementById('reminders-list');
            list.innerHTML = reminders.map(r => `
                <div class="reminder-item">
                    <div class="font-bold">${r.name} ${r.count ? `<span style="background:#dbeafe;color:#1e40af;padding:2px 6px;border-radius:6px;font-size:0.8rem">${r.count}</span>` : ''}</div>
                    <button onclick="askDeleteReminder('${r.id}')" style="color:#ef4444;border:none;background:none;cursor:pointer;"><svg class="icon" viewBox="0 0 24 24"><path d="M3 6h18"/><path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"/><path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"/></svg></button>
                </div>
            `).join('') || '<div class="text-xs text-sub text-center">القائمة فارغة</div>';
        }

        function renderCustomers() {
            const term = document.getElementById('search-input').value.toLowerCase();
            const list = document.getElementById('customers-list');
            
            // 1. Filter: Match Search AND Balance != 0
            const filtered = customers.filter(c => {
                const matchesSearch = c.name.toLowerCase().includes(term) || (c.type||'').toLowerCase().includes(term);
                const hasBalance = getBal(c) !== 0; 
                // Show if matches search (even if 0 balance) OR has non-zero balance
                if(term) return matchesSearch; 
                return hasBalance;
            })
            // 2. Sort: Highest Days first (descending)
            .sort((a,b) => getDays(b) - getDays(a));

            if(filtered.length === 0) { list.innerHTML = '<div style="text-align:center;padding:2rem;color:var(--text-sub);">لا توجد نتائج</div>'; return; }
            list.innerHTML = filtered.map(c => {
                const bal = getBal(c);
                const days = getDays(c);
                const lastNote = c.transactions.length > 0 ? c.transactions[c.transactions.length-1].note : '';
                const avatarColor = getColorFromName(c.name);
                
                let cls = 'customer-card';
                if(bal > 0) { if(days >= settings.danger) cls += ' danger'; else if(days >= settings.warn) cls += ' warning'; }
                return `
                <div class="${cls}" onclick="openDetails('${c.id}')">
                    <div class="customer-avatar" style="background:${avatarColor}20; color:${avatarColor}">
                        <span style="font-size:1.1rem">${days}</span>
                        <span style="font-size:0.55rem">يوم</span>
                    </div>
                    <div class="card-name">${c.name}</div>
                    <div class="card-balance" style="color:${bal>0?'var(--danger)':'var(--success)'}">${bal.toLocaleString()} <span>د.م</span></div>
                    <div style="font-size:0.7rem; color:var(--text-sub); margin-top:5px; height:15px; overflow:hidden;">${lastNote}</div>
                </div>`;
            }).join('');
        }

        // --- CRUD ---
        function saveCustomer() {
            const n = document.getElementById('new-name').value;
            const t = document.getElementById('new-type').value;
            const a = parseFloat(document.getElementById('new-amount').value);
            const dateInput = document.getElementById('new-date').value;
            
            if(!n) return alert('أدخل الاسم');
            
            // USE TYPE AS NOTE IF AVAILABLE
            const initialNote = t ? t : 'رصيد افتتاحي';
            const transactionDate = dateInput ? new Date(dateInput).toISOString() : new Date().toISOString();

            showLoading();
            db.collection(currentCollection).add({
                name: n, 
                type: t, 
                createdAt: new Date().toISOString(), 
                transactions: a ? [{id:Date.now()+'t', amount:a, type:'take', note: initialNote, date: transactionDate}] : [] 
            })
            .then(() => { 
                hideLoading(); 
                closeModal('modal-add'); 
                document.getElementById('new-name').value = ''; 
                document.getElementById('new-type').value = ''; 
                document.getElementById('new-amount').value = ''; 
                document.getElementById('new-date').value = '';
            });
        }

        function addReminder() {
            const n = document.getElementById('rem-name').value; const c = document.getElementById('rem-count').value;
            if(!n) return;
            db.collection("reminders").add({name:n, count:c});
            document.getElementById('rem-name').value = ''; document.getElementById('rem-count').value = '';
        }

        function askDeleteReminder(id) { reminderToDelete = id; openModal('modal-delete-reminder'); }
        function confirmDeleteRem() { if(reminderToDelete) { db.collection("reminders").doc(reminderToDelete).delete(); closeModal('modal-delete-reminder'); } }

        function openDetails(id) { 
            currentId = id; const c = customers.find(x => x.id === id); if(!c) return; 
            document.getElementById('d-name').innerText = c.name; document.getElementById('d-type').innerText = c.type || 'عام'; 
            document.getElementById('t-date').value = ''; // Reset date
            renderDetails(c); openModal('modal-details'); 
        }

        function renderDetails(c) {
            document.getElementById('d-balance').innerText = getBal(c).toLocaleString();
            document.getElementById('d-history').innerHTML = [...c.transactions].reverse().map(t => `
                <div class="trans-card ${t.type === 'take' ? 'trans-take' : 'trans-give'}">
                    <div class="trans-info"><div class="trans-note">${t.note}</div><div class="trans-date">${new Date(t.date).toLocaleDateString('ar-MA')} | ${new Date(t.date).toLocaleTimeString('ar-MA', {hour:'2-digit', minute:'2-digit'})}</div></div>
                    <div class="trans-amount"><span class="amount-val" dir="ltr">${t.amount.toLocaleString()}</span><span class="amount-label">${t.type === 'take' ? 'عليه ⬆' : 'له ⬇'}</span></div>
                </div>`).join('');
        }
        function setTransMode(m) { transMode = m; const bT = document.getElementById('btn-take'); const bG = document.getElementById('btn-give'); if(m === 'take') { bT.style.borderColor = '#ef4444'; bT.style.background = '#fef2f2'; bT.style.color = '#ef4444'; bG.style.borderColor = '#e2e8f0'; bG.style.background = 'white'; bG.style.color = '#0f172a'; } else { bG.style.borderColor = '#22c55e'; bG.style.background = '#f0fdf4'; bG.style.color = '#22c55e'; bT.style.borderColor = '#e2e8f0'; bT.style.background = 'white'; bT.style.color = '#0f172a'; } }
        
        function addTransaction() {
            const a = parseFloat(document.getElementById('t-amount').value); 
            const n = document.getElementById('t-note').value || (transMode==='take'?'كريدي':'دفعة');
            const dVal = document.getElementById('t-date').value;
            
            if(!a || !currentId) return;
            
            const transDate = dVal ? new Date(dVal).toISOString() : new Date().toISOString();

            const c = customers.find(x => x.id === currentId); 
            const newTrans = [...(c.transactions || []), { 
                id: Date.now().toString(), 
                amount:a, 
                type:transMode, 
                note:n, 
                date: transDate // التاريخ الجديد
            }];
            
            showLoading(); 
            db.collection(currentCollection).doc(currentId).update({ transactions: newTrans }).then(() => { 
                hideLoading(); 
                document.getElementById('t-amount').value = ''; 
                document.getElementById('t-note').value = ''; 
                document.getElementById('t-date').value = '';
            });
        }

        function requestPinToDelete() { document.getElementById('pin-input').value = ''; openModal('modal-pin'); }
        function confirmDeleteCustomer() {
            const enteredPin = document.getElementById('pin-input').value;
            if(enteredPin === '1988') {
                showLoading();
                db.collection(currentCollection).doc(currentId).delete().then(() => { hideLoading(); closeModal('modal-pin'); closeModal('modal-details'); alert('تم الحذف بنجاح.'); });
            } else { alert('خطأ: الرمز السري غير صحيح!'); document.getElementById('pin-input').value = ''; }
        }

        // --- Print Logic ---
        function openPrintModal() { openModal('modal-print'); }
        function generateReport(type) {
            const printArea = document.getElementById('print-area');
            const date = new Date().toLocaleDateString('ar-MA');
            let totalDebt = 0;
            let rows = '';
            let title = '';
            const contextTitle = currentCollection === 'customers' ? 'ديون المحل' : 'ديون المكتبة';

            if (type === 'all') {
                title = `تقرير شامل (${contextTitle})`;
                const debtors = customers.filter(c => getBal(c) > 0);
                debtors.forEach(c => {
                    const b = getBal(c);
                    totalDebt += b;
                    rows += `<tr><td>${c.name}</td><td>${c.type || '-'}</td><td style="font-weight:bold">${b.toLocaleString()}</td></tr>`;
                });
            } else if (type === 'month') {
                const mInput = document.getElementById('print-month').value; 
                if(!mInput) return alert("المرجو تحديد الشهر");
                title = `تقرير شهر ${mInput} (${contextTitle})`;
                customers.forEach(c => {
                    const monthlyTrans = c.transactions.filter(t => t.date.startsWith(mInput));
                    if(monthlyTrans.length > 0) {
                        let took = 0; let paid = 0;
                        monthlyTrans.forEach(t => { if(t.type==='take') took += t.amount; else paid += t.amount; });
                        rows += `<tr><td>${c.name}</td><td>${took.toLocaleString()}</td><td>${paid.toLocaleString()}</td><td>${(took-paid).toLocaleString()}</td></tr>`;
                        totalDebt += (took - paid);
                    }
                });
            }

            let tableHeader = type === 'all' ? `<tr><th>الاسم</th><th>النوع</th><th>المبلغ المتبقي</th></tr>` : `<tr><th>الاسم</th><th>أخذ (كريدي)</th><th>دفع</th><th>الصافي</th></tr>`;
            printArea.innerHTML = `<div class="print-header"><h1>تتبع الكريدي - ${title}</h1><p>تاريخ الطباعة: ${date}</p></div><table><thead>${tableHeader}</thead><tbody>${rows || '<tr><td colspan="3">لا توجد بيانات</td></tr>'}</tbody></table><div class="print-total">المجموع الكلي: ${totalDebt.toLocaleString()} د.م</div>`;
            closeModal('modal-print');
            window.print();
        }

        function openStatsModal(view) {
            const list = document.getElementById('stats-list'); const title = document.getElementById('stats-title'); let data = [];
            if(view === 'debtors') { title.innerText = "قائمة المدينين"; data = customers.filter(c => getBal(c) > 0).sort((a,b) => getBal(b) - getBal(a)); } else { title.innerText = "قائمة الخالصين"; data = customers.filter(c => getBal(c) <= 0); }
            if(data.length === 0) { list.innerHTML = '<div style="padding:1rem;text-align:center;color:#64748b;">القائمة فارغة</div>'; } else { list.innerHTML = data.map(c => `<div class="stat-list-item" onclick="closeModal('modal-stats'); openDetails('${c.id}')"><span class="font-bold">${c.name}</span><span class="font-black" dir="ltr" style="color:${view==='debtors'?'#ef4444':'#22c55e'}">${getBal(c).toLocaleString()}</span></div>`).join(''); }
            openModal('modal-stats');
        }
        function openSettings() { document.getElementById('set-warn').value = settings.warn; document.getElementById('set-danger').value = settings.danger; openModal('modal-settings'); }
        function saveSettings() { const w = parseInt(document.getElementById('set-warn').value); const d = parseInt(document.getElementById('set-danger').value); db.collection("config").doc("settings").set({warn: w, danger: d}); closeModal('modal-settings'); }
        
        document.getElementById('search-input').onkeyup = renderCustomers;
    </script>
</body>
</html>
