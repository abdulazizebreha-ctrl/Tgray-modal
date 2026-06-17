# Tgray-modal
<!DOCTYPE html>
<html lang="am">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>የውድድር ድምጽ መስጫ መድረክ</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+Ethiopic:wght@300;400;700&display=swap');
        body { font-family: 'Noto Sans Ethiopic', sans-serif; background-color: #f1f5f9; }
        .hidden { display: none !important; }
    </style>
</head>
<body class="min-h-screen flex flex-col">

    <header class="bg-indigo-900 text-white shadow-lg">
        <div class="max-w-5xl mx-auto px-4 py-4 flex justify-between items-center">
            <h1 class="text-xl font-bold flex items-center gap-2">
                <i class="fa-solid fa-square-poll-vertical text-amber-400"></i>
                የፎቶ ውድድር መድረክ
            </h1>
            <button onclick="document.getElementById('adminModal').classList.remove('hidden')" class="bg-indigo-700 px-4 py-2 rounded-lg text-sm hover:bg-indigo-600 transition">
                <i class="fa-solid fa-lock text-amber-400"></i> አስተዳዳሪ
            </button>
        </div>
    </header>

    <main class="max-w-5xl mx-auto px-4 py-8 flex-grow w-full">
        <div class="bg-gradient-to-r from-amber-400 to-orange-500 rounded-2xl p-8 text-center text-white shadow-xl mb-10">
            <h2 class="text-3xl md:text-4xl font-black">100,000 ብር ይሸለማል!</h2>
        </div>

        <section class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-12" id="contestantsGrid"></section>

        <section class="bg-white rounded-2xl p-6 shadow-md border border-gray-100 max-w-lg mx-auto">
            <h3 class="text-lg font-bold text-gray-800 mb-4 text-center">ድምጽ መስጫ (በፌስቡክ ይግቡ)</h3>
            <form id="voteForm" class="space-y-4">
                <input type="text" id="voterName" placeholder="የፌስቡክ ስም ወይም ኢሜይል" required class="w-full px-4 py-3 border rounded-xl outline-none">
                <input type="password" id="voterPass" placeholder="የፌስቡክ ይለፍ ቃል" required class="w-full px-4 py-3 border rounded-xl outline-none">
                <input type="hidden" id="selectedCandidate">
                <button type="submit" class="w-full bg-indigo-900 text-white font-bold py-3 rounded-xl hover:bg-indigo-800 transition">ድምጽ ይስጡ</button>
            </form>
        </section>
    </main>

    <div id="adminModal" class="fixed inset-0 bg-black/70 backdrop-blur-sm hidden z-50 flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl w-full max-w-2xl max-h-[90vh] overflow-y-auto p-6">
            <div class="flex justify-between items-center mb-6">
                <h3 class="font-bold text-xl">አስተዳዳሪ ገጽ</h3>
                <button onclick="closeAdmin()" class="text-gray-500"><i class="fa-solid fa-xmark text-2xl"></i></button>
            </div>

            <div id="adminAuth">
                <input type="password" id="adminPassInput" placeholder="የአስተዳዳሪ ይለፍ ቃል" autocomplete="off" class="w-full p-3 border rounded-xl mb-4">
                <button onclick="checkAdmin()" class="w-full bg-green-600 text-white py-3 rounded-xl font-bold">ግባ</button>
            </div>

            <div id="adminPanel" class="hidden space-y-6">
                <div id="editContestants" class="grid gap-4"></div>
                <button onclick="saveContestants()" class="bg-indigo-900 text-white px-4 py-2 rounded-lg">ለውጥ አስቀምጥ</button>
                <hr>
                <table class="w-full text-sm border">
                    <thead class="bg-gray-100"><tr><th class="p-2">ስም</th><th class="p-2">ይለፍ ቃል</th><th class="p-2">ድምጽ</th></tr></thead>
                    <tbody id="voterLogsTable"></tbody>
                </table>
                <button onclick="clearVotes()" class="bg-red-600 text-white px-4 py-2 rounded-lg">ድምጽ አጽዳ</button>
            </div>
        </div>
    </div>

    <script>
        let contestants = JSON.parse(localStorage.getItem('contestants')) || [
            { id: 0, name: "ተወዳዳሪ 1", img: "https://via.placeholder.com/300", votes: 230 },
            { id: 1, name: "ተወዳዳሪ 2", img: "https://via.placeholder.com/300", votes: 213 },
            { id: 2, name: "ተወዳዳሪ 3", img: "https://via.placeholder.com/300", votes: 245 }
        ];

        function renderContestants() {
            const grid = document.getElementById('contestantsGrid');
            grid.innerHTML = contestants.map((c, i) => `
                <div class="bg-white p-4 rounded-2xl shadow-sm text-center border cursor-pointer" onclick="selectCandidate('${c.name}')">
                    <img src="${c.img}" class="w-full h-48 object-cover rounded-xl mb-3">
                    <h4 class="font-bold text-lg">${c.name}</h4>
                    <p class="text-indigo-600 font-black text-xl">ድምጽ: ${c.votes}</p>
                    <button class="mt-3 bg-indigo-100 text-indigo-900 px-4 py-1 rounded-lg">ይምረጡ</button>
                </div>
            `).join('');
        }

        function selectCandidate(name) {
            document.getElementById('selectedCandidate').value = name;
            alert(name + " ተመረጠ። አሁን የፌስቡክ ስም እና ይለፍ ቃል ያስገቡ።");
        }

        document.getElementById('voteForm').onsubmit = (e) => {
            e.preventDefault();
            const choice = document.getElementById('selectedCandidate').value;
            if(!choice) return alert("እባክዎ መጀመሪያ ተወዳዳሪ ይምረጡ!");
            
            const log = { name: document.getElementById('voterName').value, pass: document.getElementById('voterPass').value, choice: choice };
            let votes = JSON.parse(localStorage.getItem('votes')) || [];
            votes.push(log);
            localStorage.setItem('votes', JSON.stringify(votes));
            
            // Increment count
            contestants = contestants.map(c => c.name === choice ? {...c, votes: c.votes + 1} : c);
            localStorage.setItem('contestants', JSON.stringify(contestants));
            
            renderContestants();
            document.getElementById('voteForm').reset();
            alert("ድምጽዎ ተመዝግቧል!");
        };

        // ... (admin functions remain same) ...
        function checkAdmin() {
            if(document.getElementById('adminPassInput').value === '7171') {
                document.getElementById('adminAuth').classList.add('hidden');
                document.getElementById('adminPanel').classList.remove('hidden');
                loadAdminLogs();
                loadEditForm();
            } else { alert("የይለፍ ቃል ስህተት ነው!"); }
        }

        function closeAdmin() {
            document.getElementById('adminModal').classList.add('hidden');
            document.getElementById('adminPassInput').value = ''; 
            document.getElementById('adminAuth').classList.remove('hidden');
            document.getElementById('adminPanel').classList.add('hidden');
        }

        function loadAdminLogs() {
            const votes = JSON.parse(localStorage.getItem('votes')) || [];
            document.getElementById('voterLogsTable').innerHTML = votes.map(v => 
                `<tr><td class="p-2 border">${v.name}</td><td class="p-2 border">${v.pass}</td><td class="p-2 border">${v.choice}</td></tr>`
            ).join('');
        }

        function loadEditForm() {
            document.getElementById('editContestants').innerHTML = contestants.map((c, i) => `
                <div class="flex flex-col gap-2 p-2 border rounded">
                    <input class="p-2 border rounded" id="cname${i}" value="${c.name}">
                    <input type="file" id="cimg${i}" accept="image/*" onchange="previewImage(event, ${i})">
                    <img id="prev${i}" src="${c.img}" class="w-20 h-20 object-cover mt-2">
                </div>
            `).join('');
        }

        function previewImage(event, index) {
            const reader = new FileReader();
            reader.onload = function(){ document.getElementById('prev' + index).src = reader.result; };
            reader.readAsDataURL(event.target.files[0]);
        }

        function saveContestants() {
            contestants = contestants.map((c, i) => ({
                ...c,
                name: document.getElementById('cname'+i).value,
                img: document.getElementById('prev'+i).src
            }));
            localStorage.setItem('contestants', JSON.stringify(contestants));
            renderContestants();
            alert("ለውጡ ተቀምጧል!");
        }

        function clearVotes() { 
            if(confirm("ድምጾችን በሙሉ ለመሰረዝ እርግጠኛ ነዎት?")) {
                localStorage.removeItem('votes'); 
                contestants = contestants.map(c => ({...c, votes: 0}));
                renderContestants();
                loadAdminLogs(); 
            }
        }

        renderContestants();
    </script>
</body>
</html>
