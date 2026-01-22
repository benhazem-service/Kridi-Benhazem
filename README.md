<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>تتبع الكريدي (تصميم الدردشة)</title>
    
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
            --teal: #0d9488; --teal-dark: #115e59;
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

        /* Password Wrapper */
        .password-wrapper { position: relative; width: 100%; margin-bottom: 15px; }
        .password-wrapper .auth-input { margin-bottom: 0; padding-left: 45px; } 
        .password-toggle { position: absolute; left: 15px; top: 50%; transform: translateY(-50%); cursor: pointer; color: var(--text-sub); padding: 5px; z-index: 10; }

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
        
        /* Reminder Accordion Style */
        .reminder-section { margin: 0 1rem; background: var(--bg-surface); border-radius: var(--radius); border: 1px solid var(--border); overflow: hidden; transition: 0.3s; box-shadow: 0 2px 5px rgba(0,0,0,0.02); }
        .reminder-header { padding: 1rem; background: var(--bg-surface); cursor: pointer; display: flex; justify-content: space-between; align-items: center; transition: background 0.2s; }
        .reminder-header:active { background: #f8fafc; }
        .reminder-body { padding: 1rem; border-top: 1px solid var(--border); background: #fdfdfd; display: none; }
        .reminder-item { display: flex; justify-content: space-between; align-items: center; padding: 0.75rem; background: white; border: 1px solid #e2e8f0; border-radius: 10px; margin-bottom: 0.5rem; }
        .count-badge { background: var(--text-sub); color: white; border-radius: 12px; padding: 2px 8px; font-size: 0.75rem; font-weight: bold; margin-right: 8px; transition: 0.3s; }
        .count-badge.has-items { background: var(--danger); }
        .arrow-icon { transition: transform 0.3s; }
        .arrow-icon.open { transform: rotate(180deg); }

        .search-bar { margin: 1rem; position: relative; }
        .search-bar input { width: 100%; padding: 1rem 1rem 1rem 3rem; border-radius: var(--radius); border: 2px solid var(--border); font-weight: bold; font-size: 1rem; }
        .search-bar .icon { position: absolute; right: 1rem; top: 1rem; color: var(--text-sub); }
        
        /* --- LAYOUTS --- */
        /* Standard Grid for Shop/Lib */
        .customers-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; padding: 0 1rem 20px 1rem; }
        @media (min-width: 768px) { .customers-grid { grid-template-columns: repeat(5, 1fr); } }

        /* Shared List (Vertical) */
        .shared-list { display: flex; flex-direction: column; gap: 10px; padding: 0 1rem 20px 1rem; }

        /* Card Style (Standard) */
        .customer-card { background: var(--bg-surface); padding: 1rem 0.5rem; border-radius: var(--radius); display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center; border: 2px solid transparent; box-shadow: 0 1px 3px rgba(0,0,0,0.05); transition: transform 0.1s; cursor: pointer; height: 100%; min-height: 140px; }
        .customer-card:active { transform: scale(0.98); background: #f1f5f9; }
        .customer-card.danger { border-color: #fca5a5; background: #fff1f2; }
        .customer-card.warning { border-color: #fdba74; background: #fff7ed; }
        
        /* Card Avatar */
        .customer-avatar { width: 50px; height: 50px; border-radius: 50%; background: #e0e7ff; color: #4338ca; display: flex; flex-direction: column; justify-content: center; align-items: center; margin-bottom: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); line-height: 1; }
        .card-name { font-weight: 900; font-size: 1rem; margin-bottom: 5px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; width: 100%; }
        .card-balance { font-weight: 900; font-size: 1.3rem; letter-spacing: -0.5px; color: var(--text-main); }
        .card-balance span { font-size: 0.7rem; color: var(--text-sub); }

        /* === NEW: SHARED CHAT CARD === */
        .shared-card {
            background: white; border-radius: 16px; padding: 15px; 
            border: 1px solid var(--border); box-shadow: 0 2px 4px rgba(0,0,0,0.03);
            display: flex; justify-content: space-between; align-items: center; cursor: pointer;
            transition: 0.2s; position: relative;
        }
        .shared-card:active { transform: scale(0.98); background: #f8fafc; }
        
        .shared-content { flex: 1; min-width: 0; }
        .shared-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 6px; }
        .shared-name { font-weight: 900; font-size: 1.1rem; color: var(--text-main); }
        .shared-time { font-size: 0.75rem; color: var(--text-sub); }
        
        .shared-last-msg {
            font-size: 0.9rem; color: var(--text-sub);
            white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
            display: flex; align-items: center; gap: 5px;
        }
        .shared-last-msg.new { color: var(--text-main); font-weight: bold; }
        .msg-sender { font-weight: bold; color: var(--primary); }

        .shared-actions { margin-left: 15px; display: flex; align-items: center; }
        .btn-hide-chat {
            padding: 8px; border-radius: 50%; border: none; background: #f1f5f9; color: var(--text-sub); cursor: pointer;
        }
        .btn-hide-chat:active { background: #e2e8f0; }

        /* Hidden Section */
        .hidden-section { margin: 20px 1rem; border-top: 1px dashed var(--border); padding-top: 20px; }
        .hidden-header { font-size: 0.9rem; color: var(--text-sub); font-weight: bold; margin-bottom: 10px; display: flex; align-items: center; gap: 8px; cursor: pointer; }
        
        /* Tabs System */
        .tabs-wrapper { padding: 10px 1rem 0 1rem; display: flex; gap: 8px; margin-bottom: 10px; overflow-x: auto; }
        .tab-btn { position: relative; flex: 1; min-width: 100px; padding: 12px; border-radius: 12px; border: none; font-weight: 900; cursor: pointer; background: white; color: var(--text-sub); border: 2px solid var(--border); transition: 0.2s; display: flex; align-items: center; justify-content: center; gap: 6px; font-size: 0.9rem; }
        .tab-btn.active.shop { background: var(--primary); color: white; border-color: var(--primary); }
        .tab-btn.active.library { background: var(--purple); color: white; border-color: var(--purple); }
        .tab-btn.active.shared { background: var(--teal); color: white; border-color: var(--teal); }
        .tab-badge { background: rgba(0,0,0,0.1); color: inherit; font-size: 0.7rem; padding: 2px 6px; border-radius: 10px; min-width: 20px; text-align: center; display: none; }
        .tab-btn.active .tab-badge { background: rgba(255,255,255,0.3); color: white; }

        .fab-btn { position: fixed; bottom: 1.5rem; left: 1.5rem; right: 1.5rem; background: var(--primary); color: white; padding: 1.2rem; border-radius: 1rem; font-weight: 900; font-size: 1.1rem; display: flex; justify-content: center; align-items: center; gap: 0.5rem; box-shadow: 0 10px 15px -3px rgba(37, 99, 235, 0.4); border: none; z-index: 30; }
        .fab-btn:active { transform: scale(0.95); }
        .fab-btn.library-mode { background: var(--purple); }
        .fab-btn.shared-mode { background: var(--teal); }
        
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
        .badge-container { position: relative; display: inline-block; }
        .notification-badge { position: absolute; top: -5px; right: -5px; background-color: var(--danger); color: white; border-radius: 50%; width: 18px; height: 18px; display: flex; justify-content: center; align-items: center; font-size: 10px; font-weight: bold; border: 2px solid white; display: none; }
        #name-suggestions { background: #fff1f2; border: 2px solid #fecaca; border-radius: 12px; padding: 10px; margin-bottom: 10px; display: none; }
        .suggestion-item { padding: 8px; color: #b91c1c; font-weight: bold; font-size: 0.9rem; border-bottom: 1px solid #fee2e2; display: flex; justify-content: space-between; align-items: center; cursor: pointer; }
        .suggestion-item:last-child { border-bottom: none; }
        .id-box { background: #1e293b; color: #fff; padding: 10px; border-radius: 8px; font-family: monospace; font-size: 0.85rem; letter-spacing: 1px; margin-top: 5px; word-break: break-all; text-align: center; user-select: all; cursor: pointer; border: 1px dashed #475569; }
        .id-box:active { background: #0f172a; }

        /* Shared Notes UI */
        .shared-msg { padding: 10px 14px; border-radius: 12px; margin-bottom: 12px; position: relative; max-width: 85%; line-height: 1.4; word-wrap: break-word; }
        .msg-me { background: #dcfce7; color: #14532d; margin-right: auto; border-bottom-right-radius: 2px; border: 1px solid #86efac; }
        .msg-partner { background: #fee2e2; color: #7f1d1d; margin-left: auto; border-bottom-left-radius: 2px; border: 1px solid #fca5a5; }
        .msg-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 4px; font-size: 0.7rem; opacity: 0.8; }
        .msg-text { font-weight: bold; font-size: 1rem; }
        .delete-note-btn { position: absolute; top: -8px; left: -8px; background: #ef4444; color: white; border-radius: 50%; width: 22px; height: 22px; border: 2px solid white; cursor: pointer; display: flex; justify-content: center; align-items: center; box-shadow: 0 2px 5px rgba(0,0,0,0.2); }

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
            <div class="password-wrapper">
                <input type="password" id="login-pass" class="auth-input" placeholder="كلمة المرور" dir="ltr">
                <span onclick="togglePassword('login-pass')" class="password-toggle">
                   <svg class="icon" viewBox="0 0 24 24" width="20" height="20"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path><circle cx="12" cy="12" r="3"></circle></svg>
                </span>
            </div>
            <div class="checkbox-container"><input type="checkbox" id="remember-me" checked><label for="remember-me">تذكرني</label></div>
            <button onclick="performLogin()" class="auth-btn">دخول</button>
            <div onclick="showView('signup')" class="auth-link">إنشاء حساب جديد</div>
            <div onclick="showView('reset')" class="auth-link" style="color: var(--text-sub);">نسيت كلمة السر؟</div>
            <p id="login-msg" class="error-msg"></p>
        </div>
        <div id="signup-view" class="auth-card" style="display:none;">
            <h1 class="font-black text-xl" style="color: var(--primary); margin-bottom: 20px;">حساب جديد</h1>
            <input type="text" id="signup-name" class="auth-input" placeholder="اسم المستخدم (اسم المتجر)">
            <input type="email" id="signup-email" class="auth-input" placeholder="البريد الإلكتروني" dir="ltr">
            <div class="password-wrapper">
                <input type="password" id="signup-pass" class="auth-input" placeholder="كلمة المرور (6 أرقام)" dir="ltr">
                <span onclick="togglePassword('signup-pass')" class="password-toggle">
                   <svg class="icon" viewBox="0 0 24 24" width="20" height="20"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path><circle cx="12" cy="12" r="3"></circle></svg>
                </span>
            </div>
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
                <button onclick="openSettings()" style="background: var(--bg-body); padding: 8px; border-radius: 50%; border: 1px solid var(--border); cursor:pointer;"><svg class="icon" viewBox="0 0 24 24"><path d="M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73.73l.22-.39a2 2 0 0 0-.73-2.73l-.15-.08a2 2 0 0 1-1-1.74v-.47a2 2 0 0 1 1-1.74l.15-.09a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z"/><circle cx="12" cy="12" r="3"/></svg></button>
                <button onclick="logout()" style="background: #fee2e2; color: #dc2626; padding: 8px; border-radius: 50%; border: none; cursor:pointer;"><svg class="icon" viewBox="0 0 24 24"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" x2="9" y1="12" y2="12"/></svg></button>
            </div>
        </header>

        <!-- TABS -->
        <div class="tabs-wrapper">
            <button class="tab-btn active shop" id="tab-shop" onclick="switchTab('customers')">
                <svg class="icon" viewBox="0 0 24 24"><path d="M6 2 3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4Z"/><path d="M3 6h18"/><path d="M16 10a4 4 0 0 1-8 0"/></svg>
                كريدي المحل
                <span id="badge-shop" class="tab-badge">0</span>
            </button>
            <button class="tab-btn" id="tab-lib" onclick="switchTab('library')">
                <svg class="icon" viewBox="0 0 24 24"><path d="M4 19.5v-15A2.5 2.5 0 0 1 6.5 2H20v20H6.5a2.5 2.5 0 0 1 0-5H20"/></svg>
                كريدي المكتبة
                <span id="badge-lib" class="tab-badge">0</span>
            </button>
            <button class="tab-btn" id="tab-shared" onclick="switchTab('shared_ledgers')">
                <svg class="icon" viewBox="0 0 24 24"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path><circle cx="9" cy="7" r="4"></circle><path d="M23 21v-2a4 4 0 0 0-3-3.87"></path><path d="M16 3.13a4 4 0 0 1 0 7.75"></path></svg>
                مشترك
                <span id="badge-shared" class="tab-badge">0</span>
            </button>
        </div>

        <div class="stats-container" id="stats-section">
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

        <!-- Collapsible Reminder Section -->
        <div class="reminder-section" id="reminder-section" style="display:block;">
            <div class="reminder-header" onclick="toggleReminders()">
                <div class="flex gap-2 items-center">
                    <span class="font-black text-main">نواقص (للمتجر)</span>
                    <span id="shortage-count" class="count-badge">0</span>
                </div>
                <svg class="icon arrow-icon" id="rem-arrow" viewBox="0 0 24 24"><polyline points="6 9 12 15 18 9"></polyline></svg>
            </div>
            
            <div id="reminder-body" class="reminder-body">
                <div class="flex gap-2" style="margin-bottom: 10px;">
                    <input id="rem-name" placeholder="المنتج" style="padding: 0.6rem;">
                    <input id="rem-count" type="number" placeholder="#" style="width: 60px; padding: 0.6rem;">
                    <button onclick="addReminder()" style="background: var(--primary); color: white; border-radius: 10px; padding: 0 12px; border:none; cursor:pointer;"><svg class="icon" viewBox="0 0 24 24"><path d="M5 12h14M12 5v14"/></svg></button>
                </div>
                <div id="reminders-list"></div>
            </div>
        </div>

        <div class="search-bar">
            <input id="search-input" type="text" placeholder="بحث عن زبون..." onkeyup="renderCustomers()">
            <svg class="icon" viewBox="0 0 24 24"><circle cx="11" cy="11" r="8"/><path d="m21 21-4.3-4.3"/></svg>
        </div>

        <div id="customers-list"></div>

        <!-- HIDDEN SHARED SECTION (Container for archived chats) -->
        <div id="hidden-shared-container" style="display:none;">
            <div onclick="toggleHiddenShared()" style="text-align:center; padding:15px; cursor:pointer; color:var(--text-sub); border-top:1px dashed var(--border); margin-top:20px; font-weight:bold;">
                 الأرشيف (مخفي) <span id="hidden-count"></span> 
                 <svg class="icon" style="width:16px; vertical-align:middle;" viewBox="0 0 24 24"><polyline points="6 9 12 15 18 9"></polyline></svg>
            </div>
            <div id="hidden-shared-list" style="display:none;"></div>
        </div>

        <button id="main-fab" class="fab-btn" onclick="openMainAction()">
            <svg class="icon" viewBox="0 0 24 24"><path d="M16 21v-2a4 4 0 0 0-4-4H6a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M22 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>
            <span id="fab-text">زبون جديد</span>
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

    <!-- Add Customer Modal -->
    <div id="modal-add" class="modal-overlay hidden">
        <div class="modal-content" style="background: white;">
            <h2 class="text-xl font-black">إضافة زبون <span id="add-mode-title" style="font-size:0.7em; color:var(--primary);">(محل)</span></h2>
            
            <input id="new-name" placeholder="الاسم الكامل" oninput="checkSimilarNames()" autocomplete="off">
            <div id="name-suggestions"></div>

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
    
    <!-- Connect Shared Account Modal -->
    <div id="modal-connect" class="modal-overlay hidden">
        <div class="modal-content" style="background: white;">
            <h2 class="text-xl font-black">ربط حساب مشترك</h2>
            <p style="font-size:0.9rem; color:var(--text-sub); margin-bottom:15px;">أدخل اسم الشريك والمعرف الرقمي (ID) الخاص به.</p>
            <input id="connect-name" placeholder="اسم الشريك (للتذكير)">
            <input id="connect-id" placeholder="المعرف الرقمي للشريك (User UID)" style="font-family:monospace;">
            <div class="flex gap-2" style="margin-top: 10px;">
                <button onclick="createSharedLedger()" class="btn-primary" style="background:var(--teal);">ربط الحساب</button>
                <button onclick="closeModal('modal-connect')" class="btn-secondary">إلغاء</button>
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

            <!-- Normal Balance Section (Hidden in Shared Mode) -->
            <div id="balance-section" style="background: var(--text-main); color: white; padding: 1.5rem; border-radius: 1.5rem; text-align: center; box-shadow: 0 4px 10px rgba(0,0,0,0.1); margin-top: 10px;">
                <div class="text-xs font-bold opacity-70">المبلغ الإجمالي</div>
                <div id="d-balance" class="font-black" style="font-size: 2.5rem;" dir="ltr">0</div>
            </div>

            <!-- Normal Action Buttons (Hidden in Shared Mode) -->
            <div id="actions-section" style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 10px;">
                <button id="btn-take" onclick="setTransMode('take')" class="btn-action" style="background: var(--danger-light); color: var(--danger); padding: 1rem; border-radius: 12px; font-weight: 900; border: 2px solid var(--danger);">عليه (كريدي)</button>
                <button id="btn-give" onclick="setTransMode('give')" class="btn-action" style="background: white; color: var(--text-main); padding: 1rem; border-radius: 12px; font-weight: 900; border: 1px solid var(--border);">له (دفع)</button>
            </div>

            <div class="flex gap-2" style="margin-top: 10px;">
                <input id="t-amount" type="number" inputmode="numeric" placeholder="المبلغ" style="flex: 1;">
                <input id="t-note" type="text" placeholder="ملاحظة" style="flex: 1.5;">
            </div>
            <div style="margin-bottom: 10px; margin-top:5px;">
                <input id="t-date" type="date" style="width:100%; border:1px solid #e2e8f0; font-size:0.9rem;" title="اختر تاريخ للعملية القديمة">
            </div>
            
            <button id="btn-add-trans" onclick="addTransaction()" class="btn-primary" style="padding: 1rem;">تسجيل العملية</button>
            
            <div style="margin-top: 10px;">
                <h3 class="font-bold text-sm text-sub" style="margin-bottom: 10px;">السجل</h3>
                <div id="d-history" class="history-section"></div>
            </div>
            <button id="btn-delete-customer" onclick="requestPinToDelete()" style="color: var(--danger); background: white; padding: 1rem; border-radius: 12px; font-weight: bold; border: 2px solid var(--danger-light); display: flex; justify-content: center; align-items: center; gap: 8px; margin-top: 10px; cursor:pointer;">
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
            
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-black">الإعدادات</h2>
                <button onclick="closeModal('modal-settings')" class="btn-icon-small" style="background:#f1f5f9;"><svg class="icon" viewBox="0 0 24 24"><path d="M18 6 6 18"/><path d="m6 6 12 12"/></svg></button>
            </div>

            <label class="font-bold text-sm">تنبيه برتقالي (أيام)</label>
            <input id="set-warn" type="number">
            <label class="font-bold text-sm">تنبيه أحمر (أيام)</label>
            <input id="set-danger" type="number">
            
            <div style="border-top:1px solid #eee; margin:15px 0;"></div>
            
            <!-- Custom Delete PIN Section -->
            <label class="font-bold text-sm">رمز الحذف (PIN) الجديد:</label>
            <input id="set-pin" type="text" placeholder="الافتراضي 1988" maxlength="6">

            <div style="border-top:1px solid #eee; margin:15px 0;"></div>
            <!-- Digital ID Section -->
            <div class="settings-info" style="text-align: center;">
                <label style="font-weight:bold; color:var(--text-sub); display:block; margin-bottom:5px;">المعرف الرقمي الخاص بك (ID)</label>
                <div id="user-digital-id" class="id-box" onclick="copyMyID()">...</div>
                <p style="font-size:0.7rem; color:var(--text-sub); margin-top:5px;">اضغط على الرقم لنسخه</p>
            </div>

            <!-- Direct Link Button -->
            <button onclick="openModal('modal-connect')" class="btn-secondary" style="margin-top:10px; border:2px dashed var(--teal); color:var(--teal);">ربط حساب شريك</button>

            <!-- Admin Actions -->
            <div id="admin-actions" style="display:none; text-align:center; margin-top:15px; padding-top:15px; border-top:1px solid #eee;">
                <p style="color:var(--text-sub); font-size:0.8rem; margin-bottom:10px;">إدارة البيانات القديمة</p>
                <button onclick="claimLegacyData()" class="btn-secondary" style="border: 2px dashed var(--primary); color: var(--primary);">ربط البيانات القديمة بحسابي</button>
                <p id="claim-status" style="font-size:0.8rem; color:green; margin-top:5px;"></p>
            </div>

            <button onclick="saveSettings()" class="btn-primary" style="margin-top: 10px;">حفظ التغييرات</button>
        </div>
    </div>

    <!-- Hidden Print Area -->
    <div id="print-area"></div>

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

        let customers = [], reminders = [], generalReminders = [], settings = {warn: 30, danger: 45, deletePin: '1988'}, currentId = null, transMode = 'take';
        let reminderToDelete = null;
        let currentCollection = 'customers'; 
        let unsubscribeCustomers = null; 
        let currentUser = null;

        const ADMIN_EMAIL = 'benhazem.service@gmail.com'; 

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
            },
            shared: {
                primary: '#14b8a6',
                gradient: 'linear-gradient(135deg, #14b8a6, #0f766e)',
                bg: '#f0fdfa'
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
        function closeModal(id) { 
            document.getElementById(id).classList.add('hidden'); 
            if(id === 'modal-add') {
                document.getElementById('name-suggestions').style.display = 'none';
                document.getElementById('new-name').value = '';
                document.getElementById('new-type').value = '';
                document.getElementById('new-amount').value = '';
                document.getElementById('new-date').value = '';
            }
            if(id === 'modal-connect') {
                document.getElementById('connect-name').value = '';
                document.getElementById('connect-id').value = '';
            }
        }
        function showView(view) {
            ['login-view', 'signup-view', 'reset-view'].forEach(v => document.getElementById(v).style.display = 'none');
            document.getElementById(view + '-view').style.display = 'block';
            document.querySelectorAll('.error-msg, .success-msg').forEach(e => e.style.display = 'none');
        }

        function togglePassword(fieldId) {
            const input = document.getElementById(fieldId);
            if(input.type === 'password') input.type = 'text';
            else input.type = 'password';
        }

        // --- AUTH ---
        auth.onAuthStateChanged((user) => {
            if (user) {
                currentUser = user;
                document.getElementById('auth-screen').style.display = 'none';
                document.getElementById('app-content').style.display = 'block';
                if(user.email === ADMIN_EMAIL) {
                    document.getElementById('admin-actions').style.display = 'block';
                }
                initApp();
            } else {
                currentUser = null;
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
            const name = document.getElementById('signup-name').value;
            const email = document.getElementById('signup-email').value;
            const pass = document.getElementById('signup-pass').value;
            const msg = document.getElementById('signup-msg');
            
            if(!name || !email || !pass) { msg.innerText = "يرجى ملء جميع البيانات"; msg.style.display = 'block'; return; }
            msg.style.display = 'none'; showLoading();
            
            auth.createUserWithEmailAndPassword(email, pass)
                .then((userCredential) => {
                    return userCredential.user.updateProfile({ displayName: name });
                })
                .catch(e => { 
                    hideLoading(); 
                    msg.innerText = e.message; 
                    msg.style.display = 'block'; 
                });
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
            setupTabCounters();
            loadCollection(); 
            
            db.collection("reminders").where("userId", "==", currentUser.uid).onSnapshot(snap => { 
                reminders = snap.docs.map(doc => ({id: doc.id, ...doc.data()})); renderReminders(); 
            });
            
            db.collection("general_reminders").where("userId", "==", currentUser.uid).orderBy("createdAt", "desc").onSnapshot(snap => {
                generalReminders = snap.docs.map(doc => ({id: doc.id, ...doc.data()}));
                renderGeneralReminders();
            });

            db.collection("user_settings").doc(currentUser.uid).onSnapshot(doc => { 
                if(doc.exists) {
                    settings = { ...settings, ...doc.data() }; 
                } else {
                    db.collection("user_settings").doc(currentUser.uid).set(settings);
                }
            });
        }

        // --- GENERAL REMINDERS LOGIC ---
        function openGeneralReminders() { openModal('modal-general-reminders'); }
        function addGeneralReminder() {
            const text = document.getElementById('new-gen-task').value;
            if(!text) return;
            db.collection('general_reminders').add({ text: text, isDone: false, createdAt: new Date().toISOString(), userId: currentUser.uid });
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

        // --- TAB COUNTERS ---
        function setupTabCounters() {
            db.collection('customers').where("userId", "==", currentUser.uid).onSnapshot(snap => {
                const count = snap.size;
                const el = document.getElementById('badge-shop');
                el.innerText = count;
                el.style.display = count > 0 ? 'inline-block' : 'none';
            });
            db.collection('library').where("userId", "==", currentUser.uid).onSnapshot(snap => {
                const count = snap.size;
                const el = document.getElementById('badge-lib');
                el.innerText = count;
                el.style.display = count > 0 ? 'inline-block' : 'none';
            });
            db.collection('shared_ledgers').where("participants", "array-contains", currentUser.uid).onSnapshot(snap => {
                const count = snap.size;
                const el = document.getElementById('badge-shared');
                el.innerText = count;
                el.style.display = count > 0 ? 'inline-block' : 'none';
            });
        }

        // --- TABS SYSTEM ---
        function switchTab(collectionName) {
            if(currentCollection === collectionName) return;
            currentCollection = collectionName;
            
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('main-fab').classList.remove('library-mode', 'shared-mode');
            
            const reminderSection = document.querySelector('.reminder-section');
            const statsSection = document.getElementById('stats-section');
            const hiddenSection = document.getElementById('hidden-shared-container');

            // Reset UI states
            if (hiddenSection) hiddenSection.style.display = 'none';

            if(collectionName === 'customers') {
                document.getElementById('tab-shop').classList.add('active');
                document.getElementById('add-mode-title').innerText = '(محل)';
                document.getElementById('fab-text').innerText = 'زبون جديد';
                reminderSection.style.display = 'block'; 
                statsSection.style.display = 'grid';
                applyTheme('shop');
            } else if (collectionName === 'library') {
                document.getElementById('tab-lib').classList.add('active');
                document.getElementById('main-fab').classList.add('library-mode');
                document.getElementById('add-mode-title').innerText = '(مكتبة)';
                document.getElementById('fab-text').innerText = 'زبون جديد';
                reminderSection.style.display = 'none'; 
                statsSection.style.display = 'grid';
                applyTheme('library');
            } else if (collectionName === 'shared_ledgers') {
                document.getElementById('tab-shared').classList.add('active');
                document.getElementById('main-fab').classList.add('shared-mode');
                document.getElementById('add-mode-title').innerText = '(مشترك)';
                document.getElementById('fab-text').innerText = 'ربط حساب';
                reminderSection.style.display = 'none'; 
                statsSection.style.display = 'none'; 
                if(hiddenSection) hiddenSection.style.display = 'block'; // Show hidden container for shared
                applyTheme('shared');
            }
            loadCollection();
        }

        function openMainAction() {
            if(currentCollection === 'shared_ledgers') {
                openModal('modal-connect');
            } else {
                openModal('modal-add');
            }
        }

        function loadCollection() {
            showLoading();
            if(unsubscribeCustomers) unsubscribeCustomers(); 
            
            // CLEAR DATA
            customers = [];
            renderAll();

            if (currentCollection === 'shared_ledgers') {
                unsubscribeCustomers = db.collection('shared_ledgers')
                    .where("participants", "array-contains", currentUser.uid)
                    .onSnapshot(snap => {
                        customers = snap.docs.map(doc => ({id: doc.id, ...doc.data()}));
                        renderAll(); 
                        hideLoading();
                    });
            } else {
                unsubscribeCustomers = db.collection(currentCollection)
                    .where("userId", "==", currentUser.uid)
                    .onSnapshot(snap => {
                        customers = snap.docs.map(doc => ({id: doc.id, ...doc.data()}));
                        renderAll(); 
                        hideLoading();
                    });
            }
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
            const badge = document.getElementById('shortage-count');
            
            badge.innerText = reminders.length;
            if(reminders.length > 0) badge.classList.add('has-items');
            else badge.classList.remove('has-items');

            if(reminders.length === 0) {
                list.innerHTML = '<div style="text-align:center; padding:10px; color:#aaa; font-size:0.8rem;">لا توجد نواقص حالياً</div>';
                return;
            }

            list.innerHTML = reminders.map(r => `
                <div class="reminder-item">
                    <div class="font-bold">${r.name} ${r.count ? `<span style="background:#dbeafe;color:#1e40af;padding:2px 6px;border-radius:6px;font-size:0.8rem">${r.count}</span>` : ''}</div>
                    <button onclick="askDeleteReminder('${r.id}')" style="color:#ef4444;border:none;background:none;cursor:pointer;"><svg class="icon" viewBox="0 0 24 24"><path d="M3 6h18"/><path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"/><path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"/></svg></button>
                </div>
            `).join('');
        }

        function toggleReminders() {
            const body = document.getElementById('reminder-body');
            const arrow = document.getElementById('rem-arrow');
            if(body.style.display === 'block') {
                body.style.display = 'none';
                arrow.classList.remove('open');
            } else {
                body.style.display = 'block';
                arrow.classList.add('open');
            }
        }

        function toggleHiddenShared() {
            const list = document.getElementById('hidden-shared-list');
            list.style.display = list.style.display === 'none' ? 'grid' : 'none';
        }

        // --- SHARED HIDE/UNHIDE LOGIC ---
        function hideSharedLedger(id) {
             db.collection('shared_ledgers').doc(id).update({ isHidden: true });
        }
        
        function unhideSharedLedger(id) {
             db.collection('shared_ledgers').doc(id).update({ isHidden: false });
        }


        function renderCustomers() {
            const term = document.getElementById('search-input').value.toLowerCase();
            const list = document.getElementById('customers-list');
            const hiddenContainer = document.getElementById('hidden-shared-container');
            const hiddenList = document.getElementById('hidden-shared-list');
            
            // Default filtering
            let filtered = customers.filter(c => {
                const matchesSearch = c.name.toLowerCase().includes(term) || (c.type||'').toLowerCase().includes(term);
                const hasBalance = getBal(c) !== 0; 
                if(term) return matchesSearch; 
                return hasBalance;
            })
            .sort((a,b) => getDays(b) - getDays(a));

            if(currentCollection === 'shared_ledgers') {
                list.className = 'shared-grid';
                hiddenList.className = 'shared-grid';
                
                // Split active vs hidden
                const activeShared = filtered.filter(c => !c.isHidden);
                const hiddenShared = filtered.filter(c => c.isHidden);
                
                // Render Active
                if(activeShared.length === 0) list.innerHTML = '<div style="text-align:center;padding:2rem;color:var(--text-sub);">لا توجد حسابات نشطة</div>';
                else list.innerHTML = activeShared.map(c => renderPartnerCard(c)).join('');

                // Render Hidden
                if(hiddenShared.length > 0) {
                    hiddenContainer.style.display = 'block';
                    document.getElementById('hidden-count').innerText = `(${hiddenShared.length})`;
                    hiddenList.innerHTML = hiddenShared.map(c => renderPartnerCard(c, true)).join('');
                } else {
                    hiddenContainer.style.display = 'none';
                }

            } else {
                // NORMAL MODE
                hiddenContainer.style.display = 'none';
                list.className = 'customers-grid';
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
        }

        // Helper to render shared card
        function renderPartnerCard(c, isHidden = false) {
             const bal = getBal(c);
             const lastTrans = c.transactions.length > 0 ? c.transactions[c.transactions.length-1] : null;
             
             let lastMsg = 'لا توجد رسائل';
             let lastTime = '';
             
             if(lastTrans) {
                 const isMe = lastTrans.by === currentUser.uid;
                 const sender = isMe ? 'أنت:' : 'هو:';
                 lastMsg = `<span class="msg-sender">${sender}</span> ${lastTrans.note}`;
                 lastTime = new Date(lastTrans.date).toLocaleTimeString('ar-MA', {hour:'2-digit', minute:'2-digit'});
             }

             // Determine hide/unhide button
             const actionBtn = isHidden 
                ? `<button onclick="event.stopPropagation(); unhideSharedLedger('${c.id}')" class="btn-hide-chat" title="استرجاع"><svg class="icon" viewBox="0 0 24 24"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path><circle cx="12" cy="12" r="3"></circle></svg></button>`
                : `<button onclick="event.stopPropagation(); hideSharedLedger('${c.id}')" class="btn-hide-chat" title="إخفاء"><svg class="icon" viewBox="0 0 24 24"><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"></path><line x1="1" y1="1" x2="23" y2="23"></line></svg></button>`;

             return `
             <div class="shared-card" onclick="openDetails('${c.id}')" style="opacity:${isHidden ? '0.7' : '1'}">
                 <div class="shared-content">
                     <div class="shared-header">
                         <div class="shared-name">${c.name}</div>
                         <div class="shared-time">${lastTime}</div>
                     </div>
                     <div class="shared-last-msg ${lastTrans && lastTrans.by !== currentUser.uid ? 'new' : ''}">
                         ${lastMsg}
                     </div>
                 </div>
                 <div class="shared-actions">
                     ${actionBtn}
                 </div>
             </div>`;
        }


        // --- NAME DUPLICATION CHECK ---
        function checkSimilarNames() {
            const input = document.getElementById('new-name').value.trim().toLowerCase();
            const suggestionsBox = document.getElementById('name-suggestions');
            suggestionsBox.innerHTML = '';
            if(input.length < 2) { suggestionsBox.style.display = 'none'; return; }
            const matches = customers.filter(c => c.name.toLowerCase().includes(input));
            if(matches.length > 0) {
                suggestionsBox.style.display = 'block';
                matches.forEach(c => {
                    const div = document.createElement('div');
                    div.className = 'suggestion-item';
                    div.innerHTML = `<span>موجود: ${c.name}</span> <span style="font-size:0.7rem">(${getBal(c)})</span>`;
                    div.onclick = () => { closeModal('modal-add'); openDetails(c.id); };
                    suggestionsBox.appendChild(div);
                });
            } else { suggestionsBox.style.display = 'none'; }
        }

        // --- CRUD with USER ID ---
        function saveCustomer() {
            const n = document.getElementById('new-name').value;
            const t = document.getElementById('new-type').value;
            const a = parseFloat(document.getElementById('new-amount').value);
            const dateInput = document.getElementById('new-date').value;
            if(!n) return alert('أدخل الاسم');
            const initialNote = t ? t : 'رصيد افتتاحي';
            const transactionDate = dateInput ? new Date(dateInput).toISOString() : new Date().toISOString();
            showLoading();
            db.collection(currentCollection).add({
                name: n, type: t, createdAt: new Date().toISOString(), userId: currentUser.uid, 
                transactions: a ? [{id:Date.now()+'t', amount:a, type:'take', note: initialNote, date: transactionDate}] : [] 
            }).then(() => { hideLoading(); closeModal('modal-add'); });
        }

        // --- SHARED LEDGER LOGIC ---
        function createSharedLedger() {
            const name = document.getElementById('connect-name').value;
            const partnerId = document.getElementById('connect-id').value.trim();
            if(!name || !partnerId) return alert('يرجى إدخال البيانات');
            if(partnerId === currentUser.uid) return alert('لا يمكنك ربط الحساب بنفسك!');
            showLoading();
            db.collection('shared_ledgers').add({
                name: name, participants: [currentUser.uid, partnerId], createdAt: new Date().toISOString(), transactions: [], isHidden: false // Default not hidden
            }).then(() => { hideLoading(); closeModal('modal-connect'); alert('تم إنشاء الحساب المشترك بنجاح!'); }).catch(e => { hideLoading(); alert('خطأ: ' + e.message); });
        }

        function addReminder() {
            const n = document.getElementById('rem-name').value; const c = document.getElementById('rem-count').value;
            if(!n) return;
            db.collection("reminders").add({name:n, count:c, userId: currentUser.uid});
            document.getElementById('rem-name').value = ''; document.getElementById('rem-count').value = '';
        }

        function askDeleteReminder(id) { reminderToDelete = id; openModal('modal-delete-reminder'); }
        function confirmDeleteRem() { if(reminderToDelete) { db.collection("reminders").doc(reminderToDelete).delete(); closeModal('modal-delete-reminder'); } }

        function openDetails(id) { 
            currentId = id; const c = customers.find(x => x.id === id); if(!c) return; 
            document.getElementById('d-name').innerText = c.name; document.getElementById('d-type').innerText = c.type || 'عام'; 
            document.getElementById('t-date').value = ''; 
            
            const amountInput = document.getElementById('t-amount');
            const actionsSection = document.getElementById('actions-section');
            const balanceSection = document.getElementById('balance-section');
            const addBtn = document.getElementById('btn-add-trans');
            const deleteBtn = document.getElementById('btn-delete-customer');

            if(currentCollection === 'shared_ledgers') {
                amountInput.style.display = 'none';
                actionsSection.style.display = 'none';
                balanceSection.style.display = 'none';
                deleteBtn.style.display = 'none'; 
                addBtn.innerText = 'إرسال ملاحظة';
                document.getElementById('t-note').placeholder = 'اكتب رسالة...';
            } else {
                amountInput.style.display = 'block';
                actionsSection.style.display = 'grid';
                balanceSection.style.display = 'block';
                deleteBtn.style.display = 'flex';
                addBtn.innerText = 'تسجيل العملية';
                document.getElementById('t-note').placeholder = 'ملاحظة';
            }
            renderDetails(c); openModal('modal-details'); 
        }

        function renderDetails(c) {
            if(currentCollection !== 'shared_ledgers') { document.getElementById('d-balance').innerText = getBal(c).toLocaleString(); }
            document.getElementById('d-history').innerHTML = [...c.transactions].reverse().map(t => {
                if (currentCollection === 'shared_ledgers') {
                    const isMe = t.by === currentUser.uid;
                    const msgClass = isMe ? 'msg-me' : 'msg-partner';
                    const deleteBtn = isMe ? `<button onclick="deleteSharedNote('${t.id}')" class="delete-note-btn">&times;</button>` : '';
                    return `
                        <div class="shared-msg ${msgClass}">
                            ${deleteBtn}
                            <div class="msg-header"><span>${isMe ? 'أنا' : 'الشريك'}</span><span>${new Date(t.date).toLocaleTimeString('ar-MA', {hour:'2-digit', minute:'2-digit'})}</span></div>
                            <div class="msg-text">${t.note}</div>
                        </div>`;
                } else {
                    return `
                    <div class="trans-card ${t.type === 'take' ? 'trans-take' : 'trans-give'}">
                        <div class="trans-info"><div class="trans-note">${t.note}</div><div class="trans-date">${new Date(t.date).toLocaleDateString('ar-MA')}</div></div>
                        <div class="trans-amount"><span class="amount-val" dir="ltr">${t.amount.toLocaleString()}</span><span class="amount-label">${t.type === 'take' ? 'عليه ⬆' : 'له ⬇'}</span></div>
                    </div>`;
                }
            }).join('');
        }
        
        function setTransMode(m) { transMode = m; const bT = document.getElementById('btn-take'); const bG = document.getElementById('btn-give'); if(m === 'take') { bT.style.borderColor = '#ef4444'; bT.style.background = '#fef2f2'; bT.style.color = '#ef4444'; bG.style.borderColor = '#e2e8f0'; bG.style.background = 'white'; bG.style.color = '#0f172a'; } else { bG.style.borderColor = '#22c55e'; bG.style.background = '#f0fdf4'; bG.style.color = '#22c55e'; bT.style.borderColor = '#e2e8f0'; bT.style.background = 'white'; bT.style.color = '#0f172a'; } }
        
        function addTransaction() {
            const n = document.getElementById('t-note').value;
            let a = 0;
            if(currentCollection !== 'shared_ledgers') { a = parseFloat(document.getElementById('t-amount').value); if(!a && !n) return; } else { if(!n) return; }
            const dVal = document.getElementById('t-date').value;
            const transDate = dVal ? new Date(dVal).toISOString() : new Date().toISOString();
            const c = customers.find(x => x.id === currentId); 
            const newTrans = [...(c.transactions || []), { id: Date.now().toString(), amount: a || 0, type:transMode, note:n, date: transDate, by: currentUser.uid }];
            
            showLoading(); 
            // Update logic - force isHidden to false when adding msg
            const updateData = { transactions: newTrans };
            if(currentCollection === 'shared_ledgers') { updateData.isHidden = false; }

            db.collection(currentCollection).doc(currentId).update(updateData).then(() => { 
                hideLoading(); document.getElementById('t-amount').value = ''; document.getElementById('t-note').value = ''; document.getElementById('t-date').value = '';
            });
        }

        function deleteSharedNote(transId) {
            if(!confirm("حذف هذه الملاحظة؟")) return;
            const c = customers.find(x => x.id === currentId);
            const newTrans = c.transactions.filter(t => t.id !== transId);
            showLoading();
            db.collection(currentCollection).doc(currentId).update({ transactions: newTrans }).then(() => { hideLoading(); });
        }

        function requestPinToDelete() { document.getElementById('pin-input').value = ''; openModal('modal-pin'); }
        function confirmDeleteCustomer() {
            const enteredPin = document.getElementById('pin-input').value;
            const correctPin = settings.deletePin || '1988'; 
            if(enteredPin === correctPin) {
                showLoading();
                db.collection(currentCollection).doc(currentId).delete().then(() => { hideLoading(); closeModal('modal-pin'); closeModal('modal-details'); alert('تم الحذف بنجاح.'); });
            } else { alert('خطأ: الرمز السري غير صحيح!'); document.getElementById('pin-input').value = ''; }
        }

        async function claimLegacyData() {
            if(!currentUser || currentUser.email !== ADMIN_EMAIL) return;
            const status = document.getElementById('claim-status');
            status.innerText = "جاري المعالجة...";
            try {
                const cols = ['customers', 'library', 'reminders', 'general_reminders'];
                let count = 0;
                for (const colName of cols) {
                    const snap = await db.collection(colName).get();
                    const batch = db.batch();
                    let batchCount = 0;
                    snap.forEach(doc => { const data = doc.data(); if (!data.userId) { batch.update(doc.ref, { userId: currentUser.uid }); batchCount++; count++; } });
                    if (batchCount > 0) await batch.commit();
                }
                status.innerText = `تم ربط ${count} سجل بحسابك بنجاح!`; setTimeout(() => window.location.reload(), 2000);
            } catch (e) { status.innerText = "حدث خطأ: " + e.message; }
        }

        function copyMyID() {
            const idText = document.getElementById('user-digital-id').innerText;
            if(idText) { navigator.clipboard.writeText(idText).then(() => { alert("تم نسخ المعرف الرقمي: " + idText); }).catch(err => { console.error('Failed to copy: ', err); }); }
        }

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
        function openSettings() {
            if(currentUser) { document.getElementById('user-digital-id').innerText = currentUser.uid; }
            document.getElementById('set-warn').value = settings.warn; 
            document.getElementById('set-danger').value = settings.danger;
            document.getElementById('set-pin').value = settings.deletePin || '';
            openModal('modal-settings'); 
        }
        function saveSettings() { 
            const w = parseInt(document.getElementById('set-warn').value); const d = parseInt(document.getElementById('set-danger').value); const p = document.getElementById('set-pin').value.trim();
            db.collection("user_settings").doc(currentUser.uid).set({ warn: w, danger: d, deletePin: p || '1988' }); 
            closeModal('modal-settings'); 
        }
        document.getElementById('search-input').onkeyup = renderCustomers;
    </script>
</body>
</html>
