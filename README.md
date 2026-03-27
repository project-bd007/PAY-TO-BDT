# PAY-TO-BDT
<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Earn Pro - Mood Menu</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <style>
        :root { --primary: #0088cc; --accent: #ff9f43; --bg: #0f172a; --card: #1e293b; --text: #f8fafc; }
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: var(--bg); color: var(--text); margin: 0; padding: 15px; }
        
        /* প্রোফাইল কার্ড স্টাইল */
        .profile-card { background: linear-gradient(135deg, var(--primary), #005580); padding: 25px; border-radius: 25px; text-align: center; box-shadow: 0 10px 25px rgba(0,0,0,0.3); margin-bottom: 20px; }
        .user-pic { width: 75px; height: 75px; border-radius: 50%; border: 3px solid rgba(255,255,255,0.2); margin-bottom: 10px; }
        .balance-val { font-size: 32px; font-weight: bold; margin: 5px 0; }
        .stat-row { display: flex; justify-content: space-around; margin-top: 15px; font-size: 12px; opacity: 0.9; }

        /* মুড মেনু গ্রিড */
        .menu-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 25px; }
        .menu-item { background: var(--card); padding: 20px; border-radius: 20px; text-align: center; border: 1px solid rgba(255,255,255,0.05); transition: 0.3s; }
        .menu-item:active { transform: scale(0.95); }
        .menu-item span { display: block; font-size: 28px; margin-bottom: 8px; }
        .menu-item h4 { margin: 0; font-size: 14px; }
        .menu-item small { font-size: 10px; color: #94a3b8; }

        /* উইথড্র সেকশন */
        .withdraw-card { background: var(--card); padding: 20px; border-radius: 25px; margin-top: 10px; }
        select, input { width: 100%; padding: 12px; margin: 10px 0; border-radius: 12px; border: 1px solid #334155; background: #0f172a; color: white; box-sizing: border-box; }
        .action-btn { width: 100%; padding: 15px; border-radius: 15px; border: none; font-weight: bold; cursor: pointer; transition: 0.3s; }
        .btn-active { background: var(--primary); color: white; }
        .btn-locked { background: #475569; color: #94a3b8; cursor: not-allowed; }
        .error-note { color: #fb7185; font-size: 11px; margin-bottom: 10px; text-align: center; }
    </style>
</head>
<body>

    <div class="profile-card">
        <img src="https://via.placeholder.com/100" class="user-pic" id="u-pic">
        <div style="font-size: 14px; opacity: 0.8;">মোট উপার্জন</div>
        <div class="balance-val">৳ <span id="balance">0.00</span></div>
        <div class="stat-row">
            <div>রেফার: <span id="ref-count">0</span> / 3</div>
            <div>আজকের জিমেইল: <span id="g-limit">25</span></div>
        </div>
    </div>

    <div class="menu-grid">
        <div class="menu-item" onclick="gmailTask()">
            <span>📧</span>
            <h4>জিমেইল টাস্ক</h4>
            <small>৳ ১০.০০ / প্রতি কাজ</small>
        </div>
        <div class="menu-item" onclick="visitWeb()">
            <span>🌐</span>
            <h4>ওয়েবসাইট ভিজিট</h4>
            <small>১৫ সেকেন্ড (৳ ১.৫০)</small>
        </div>
        <div class="menu-item" onclick="watchAds()">
            <span>📺</span>
            <h4>ভিডিও অ্যাডস</h4>
            <small>প্রতিদিন ১৫টি (৳২.০০) </small>
        </div>
        <div class="menu-item" onclick="copyRefer()">
            <span>🔗</span>
            <h4>রেফার লিঙ্ক</h4>
            <small>বোনাস ৳ ১০.০০</small>
        </div>
    </div>

    <div class="withdraw-card">
        <h3 style="margin-top: 0; font-size: 16px;">টাকা উত্তোলন (Withdraw)</h3>
        <div id="lock-text" class="error-note">⚠️ কমপক্ষে ৩টি রেফার ছাড়া উইথড্র লক থাকবে।</div>
        
        <select id="method">
            <option value="Bkash">বিকাশ (Bkash)</option>
            <option value="Nagad">নগদ (Nagad)</option>
            <option value="Recharge">মোবাইল রিচার্জ</option>
            <option value="Binance">বাইনান্স (Binance ID)</option>
        </select>
        <input type="text" id="acc-no" placeholder="নাম্বার বা বাইনান্স আইডি দিন">
        <button id="w-btn" class="action-btn btn-locked" disabled onclick="handleWithdraw()">উত্তোলন করুন</button>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, doc, getDoc, setDoc, updateDoc, increment } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

        // [আপনার Firebase সেটিংস এখানে বসান]
        const firebaseConfig = {
            apiKey: "**[আপনার API KEY]**",
            authDomain: "**[আপনার প্রোজেক্ট ডোমেইন]**",
            projectId: "**[আপনার প্রোজেক্ট আইডি]**",
            storageBucket: "**[আপনার স্টোরেজ লিঙ্ক]**",
            messagingSenderId: "**[আপনার আইডি]**",
            appId: "**[আপনার অ্যাপ আইডি]**"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        let tg = window.Telegram.WebApp;
        tg.expand();

        const user = tg.initDataUnsafe.user || { id: "test_user", first_name: "Guest", photo_url: "" };
        const userId = user.id.toString();

        async function syncData() {
            const userRef = doc(db, "users", userId);
            const userSnap = await getDoc(userRef);

            if (userSnap.exists()) {
                const data = userSnap.data();
                document.getElementById("balance").innerText = data.balance.toFixed(2);
                document.getElementById("ref-count").innerText = data.referCount || 0;
                document.getElementById("g-limit").innerText = 25 - (data.gmailCount || 0);
                if(user.photo_url) document.getElementById("u-pic").src = user.photo_url;

                // ৩টি রেফারের শর্ত চেক
                if((data.referCount || 0) >= 3) {
                    const btn = document.getElementById("w-btn");
                    btn.disabled = false;
                    btn.className = "action-btn btn-active";
                    document.getElementById("lock-text").style.display = "none";
                }
            } else {
                // নতুন ইউজার রেজিস্টার এবং রেফারেল চেক
                const urlParams = new URLSearchParams(window.location.search);
                const inviterId = urlParams.get('start');
                
                await setDoc(userRef, { name: user.first_name, balance: 0, referCount: 0, gmailCount: 0, adsCount: 0 });
                
                if(inviterId) {
                    const inviterRef = doc(db, "users", inviterId);
                    await updateDoc(inviterRef, { 
                        balance: increment(10.00), 
                        referCount: increment(1) 
                    });
                }
            }
        }

        window.gmailTask = async function() {
            let res = prompt("আপনার তৈরি করা জিমেইল এবং পাসওয়ার্ড দিন (User:Pass):");
            if(res) {
                const userRef = doc(db, "users", userId);
                await updateDoc(userRef, { balance: increment(5.00), gmailCount: increment(1) });
                alert("সাফল্যের সাথে জমা হয়েছে! ৫ টাকা যোগ হয়েছে।");
                syncData();
            }
        };

        window.visitWeb = function() {
            window.open("**[আপনার ওয়েবসাইট লিঙ্ক]**", "_blank");
            alert("১৫ সেকেন্ড অপেক্ষা করুন...");
            setTimeout(async () => {
                const userRef = doc(db, "users", userId);
                await updateDoc(userRef, { balance: increment(2.00) });
                alert("২ টাকা যোগ হয়েছে!");
                syncData();
            }, 15000);
        };

        window.watchAds = function() {
            alert("অ্যাড লোড হচ্ছে...");
            setTimeout(async () => {
                const userRef = doc(db, "users", userId);
                await updateDoc(userRef, { balance: increment(1.50) });
                alert("১.৫০ টাকা যোগ হয়েছে!");
                syncData();
            }, 10000);
        };

        window.copyRefer = function() {
            const link = `https://t.me/**[আপনার_বট_ইউজারনেম]**?start=${userId}`;
            navigator.clipboard.writeText(link);
            alert("আপনার রেফারেল লিঙ্ক কপি হয়েছে!");
        };

        window.handleWithdraw = function() {
            alert("উইথড্র রিকোয়েস্ট সফল! এডমিন চেক করে পেমেন্ট করবে।");
        };

        syncData();
    </script>
</body>
</html>
