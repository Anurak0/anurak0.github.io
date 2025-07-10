<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>แอพบันทึกโอที (OT Tracker)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- PDF Generation Libraries -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.8.2/jspdf.plugin.autotable.min.js"></script>
    <!-- Font data for PDF generation (moved to head for earlier loading) -->
    <script src="https://storage.googleapis.com/gemini-pro-temp-exporters/THSarabun-normal.js"></script>
    <style>
        body {
            font-family: 'Sarabun', sans-serif;
        }
        .tab-active {
            background-color: #3b82f6; /* bg-blue-500 */
            color: white;
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center p-4">
    <div class="w-full max-w-4xl mx-auto bg-white rounded-2xl shadow-lg p-6 md:p-8 space-y-6">
        
        <!-- Header -->
        <div class="text-center">
            <h1 class="text-3xl font-bold text-gray-800">แอพบันทึกโอที</h1>
            <p class="text-gray-500 mt-1">บันทึกและติดตามชั่วโมงการทำงานล่วงเวลาของคุณ</p>
            <div id="auth-status" class="mt-2 text-xs text-gray-400">
                <p>สถานะ: กำลังเชื่อมต่อ...</p>
                <p id="user-id-display" class="break-all"></p>
            </div>
        </div>

        <!-- Input Form -->
        <div class="bg-gray-50 p-6 rounded-xl border border-gray-200">
            <h2 class="text-xl font-semibold text-gray-700 mb-4">เพิ่มรายการโอทีใหม่</h2>
            <form id="ot-form" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-5 gap-4 items-end">
                <div class="space-y-1">
                    <label for="ot-date" class="block text-sm font-medium text-gray-600">วันที่</label>
                    <input type="date" id="ot-date" required class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                </div>
                <div class="space-y-1">
                    <label for="start-time" class="block text-sm font-medium text-gray-600">เวลาเริ่มต้น</label>
                    <input type="time" id="start-time" required class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                </div>
                <div class="space-y-1">
                    <label for="end-time" class="block text-sm font-medium text-gray-600">เวลาสิ้นสุด</label>
                    <input type="time" id="end-time" required class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                </div>
                <div class="space-y-1 md:col-span-2 lg:col-span-1">
                    <label for="description" class="block text-sm font-medium text-gray-600">หมายเหตุ</label>
                    <input type="text" id="description" placeholder="เช่น ปิดโปรเจค" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                </div>
                <button type="submit" class="w-full bg-blue-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-blue-600 transition duration-300 lg:col-span-1 h-10">บันทึก</button>
            </form>
            <p id="error-message" class="text-red-500 text-sm mt-2"></p>
        </div>

        <!-- Summaries -->
        <div class="bg-blue-50 p-4 rounded-xl border border-blue-200">
             <h3 class="text-lg font-semibold text-gray-700 mb-3 text-center">สรุปยอดรวมชั่วโมงโอที</h3>
            <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 text-center">
                <div>
                    <p class="text-sm text-gray-600">วันนี้</p>
                    <p id="summary-daily" class="text-2xl font-bold text-blue-600">0.00</p>
                    <p class="text-xs text-gray-500">ชั่วโมง</p>
                </div>
                <div>
                    <p class="text-sm text-gray-600">เดือนนี้</p>
                    <p id="summary-monthly" class="text-2xl font-bold text-blue-600">0.00</p>
                     <p class="text-xs text-gray-500">ชั่วโมง</p>
                </div>
                <div>
                    <p class="text-sm text-gray-600">ปีนี้</p>
                    <p id="summary-yearly" class="text-2xl font-bold text-blue-600">0.00</p>
                     <p class="text-xs text-gray-500">ชั่วโมง</p>
                </div>
            </div>
        </div>

        <!-- OT Records List -->
        <div>
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-semibold text-gray-700">ประวัติการบันทึก</h2>
                <button id="export-pdf-btn" class="bg-green-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-green-600 transition duration-300 flex items-center gap-2 text-sm">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                      <path stroke-linecap="round" stroke-linejoin="round" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" />
                    </svg>
                    Export เป็น PDF
                </button>
            </div>
            <div id="ot-list-container" class="bg-white border border-gray-200 rounded-lg overflow-hidden">
                <div class="hidden md:grid grid-cols-12 gap-4 p-4 bg-gray-50 border-b font-semibold text-sm text-gray-600">
                    <div class="col-span-2">วันที่</div>
                    <div class="col-span-2">เวลา</div>
                    <div class="col-span-2">ชั่วโมง (ชม.)</div>
                    <div class="col-span-5">หมายเหตุ</div>
                    <div class="col-span-1 text-right"></div>
                </div>
                <div id="ot-list" class="divide-y divide-gray-200">
                    <!-- OT records will be inserted here -->
                     <p class="text-center text-gray-500 p-8">ยังไม่มีข้อมูล... ลองเพิ่มรายการโอทีใหม่ดูสิ</p>
                </div>
            </div>
        </div>

    </div>

    <script type="module">
        // Import Firebase modules
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, doc, deleteDoc, query, orderBy, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- Firebase Configuration ---
        const firebaseConfig = typeof __firebase_config !== 'undefined' 
            ? JSON.parse(__firebase_config) 
            : { apiKey: "YOUR_API_KEY", authDomain: "YOUR_AUTH_DOMAIN", projectId: "YOUR_PROJECT_ID" };
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'ot-tracker-app';

        // --- Initialize Firebase ---
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);

        // --- PDF Library ---
        const { jsPDF } = window.jspdf;

        // --- DOM Elements ---
        const otForm = document.getElementById('ot-form');
        const otDateInput = document.getElementById('ot-date');
        const startTimeInput = document.getElementById('start-time');
        const endTimeInput = document.getElementById('end-time');
        const descriptionInput = document.getElementById('description');
        const otList = document.getElementById('ot-list');
        const errorMessage = document.getElementById('error-message');
        const summaryDaily = document.getElementById('summary-daily');
        const summaryMonthly = document.getElementById('summary-monthly');
        const summaryYearly = document.getElementById('summary-yearly');
        const authStatusEl = document.getElementById('auth-status');
        const userIdDisplayEl = document.getElementById('user-id-display');
        const exportPdfBtn = document.getElementById('export-pdf-btn');

        // --- State ---
        let otCollectionRef;
        let unsubscribe;
        let currentRecords = []; // To store records for PDF export

        // --- Authentication ---
        onAuthStateChanged(auth, async user => {
            if (user) {
                const userId = user.uid;
                authStatusEl.querySelector('p').textContent = 'สถานะ: เชื่อมต่อสำเร็จ';
                userIdDisplayEl.textContent = `User ID: ${userId}`;
                otCollectionRef = collection(db, `artifacts/${appId}/users/${userId}/ot_records`);
                
                if (unsubscribe) unsubscribe();

                const q = query(otCollectionRef, orderBy("date", "desc"));
                unsubscribe = onSnapshot(q, (snapshot) => {
                    currentRecords = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                    renderOTList(currentRecords);
                    updateSummaries(currentRecords);
                }, (error) => {
                    console.error("Error fetching data: ", error);
                    errorMessage.textContent = "เกิดข้อผิดพลาดในการดึงข้อมูล";
                });

            } else {
                authStatusEl.querySelector('p').textContent = 'สถานะ: กำลังตรวจสอบสิทธิ์...';
                userIdDisplayEl.textContent = '';
                try {
                    if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                        await signInWithCustomToken(auth, __initial_auth_token);
                    } else {
                        await signInAnonymously(auth);
                    }
                } catch (error) {
                    console.error("Authentication failed:", error);
                    authStatusEl.querySelector('p').textContent = 'สถานะ: การเชื่อมต่อล้มเหลว';
                    errorMessage.textContent = "ไม่สามารถเชื่อมต่อกับบริการได้ โปรดรีเฟรชหน้าจอ";
                }
            }
        });

        // --- Functions ---

        function calculateDuration(startTime, endTime) {
            const start = new Date(`1970-01-01T${startTime}:00`);
            const end = new Date(`1970-01-01T${endTime}:00`);
            let diff = end - start;
            if (diff < 0) diff += 24 * 60 * 60 * 1000;
            return diff / (1000 * 60 * 60);
        }

        function renderOTList(records) {
            otList.innerHTML = '';
            if (records.length === 0) {
                otList.innerHTML = `<p class="text-center text-gray-500 p-8">ยังไม่มีข้อมูล... ลองเพิ่มรายการโอทีใหม่ดูสิ</p>`;
                exportPdfBtn.disabled = true;
                exportPdfBtn.classList.add('opacity-50', 'cursor-not-allowed');
                return;
            }
            
            exportPdfBtn.disabled = false;
            exportPdfBtn.classList.remove('opacity-50', 'cursor-not-allowed');

            records.forEach(record => {
                const duration = record.durationHours || 0;
                const recordEl = document.createElement('div');
                recordEl.className = 'grid grid-cols-6 md:grid-cols-12 gap-4 p-4 items-center text-sm';
                recordEl.innerHTML = `
                    <div class="col-span-6 md:hidden">
                        <p class="font-semibold text-gray-800">${new Date(record.date).toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' })}</p>
                        <p class="text-gray-600">เวลา: ${record.startTime} - ${record.endTime}</p>
                        <p class="text-gray-600">รวม: <span class="font-bold text-blue-600">${duration.toFixed(2)}</span> ชม.</p>
                        <p class="text-gray-500 italic break-words">${record.description || '-'}</p>
                    </div>
                     <div class="col-span-6 md:hidden flex justify-end items-center">
                        <button data-id="${record.id}" class="delete-btn text-red-500 hover:text-red-700 font-semibold py-1 px-2 rounded">ลบ</button>
                    </div>
                    <div class="hidden md:block col-span-2 text-gray-700">${new Date(record.date).toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' })}</div>
                    <div class="hidden md:block col-span-2 text-gray-600">${record.startTime} - ${record.endTime}</div>
                    <div class="hidden md:block col-span-2 font-semibold text-blue-600">${duration.toFixed(2)}</div>
                    <div class="hidden md:block col-span-5 text-gray-600 break-words">${record.description || '-'}</div>
                    <div class="hidden md:block col-span-1 text-right">
                        <button data-id="${record.id}" class="delete-btn text-red-500 hover:text-red-700 transition-colors">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>
                        </button>
                    </div>
                `;
                otList.appendChild(recordEl);
            });
        }

        function updateSummaries(records) {
            const now = new Date();
            const todayStr = now.toISOString().split('T')[0];
            const currentMonth = now.getMonth();
            const currentYear = now.getFullYear();
            let dailyTotal = 0, monthlyTotal = 0, yearlyTotal = 0;
            records.forEach(record => {
                const recordDate = new Date(record.date);
                const duration = record.durationHours || 0;
                if (recordDate.getFullYear() === currentYear) {
                    yearlyTotal += duration;
                    if (recordDate.getMonth() === currentMonth) {
                        monthlyTotal += duration;
                        if (record.date === todayStr) {
                            dailyTotal += duration;
                        }
                    }
                }
            });
            summaryDaily.textContent = dailyTotal.toFixed(2);
            summaryMonthly.textContent = monthlyTotal.toFixed(2);
            summaryYearly.textContent = yearlyTotal.toFixed(2);
        }

        /**
         * Waits for a global variable to be defined.
         * @param {string} variableName The name of the global variable to wait for.
         * @param {number} timeout The maximum time to wait in milliseconds.
         * @returns {Promise<void>} A promise that resolves when the variable is defined.
         */
        function waitForGlobal(variableName, timeout = 3000) {
            return new Promise((resolve, reject) => {
                const startTime = Date.now();
                const interval = setInterval(() => {
                    if (typeof window[variableName] !== 'undefined') {
                        clearInterval(interval);
                        resolve();
                    } else if (Date.now() - startTime > timeout) {
                        clearInterval(interval);
                        reject(new Error(`Timed out waiting for ${variableName}`));
                    }
                }, 100); // Check every 100ms
            });
        }
        
        /**
         * Exports the current OT data to a PDF file.
         */
        async function exportToPDF() {
            errorMessage.textContent = '';

            if (currentRecords.length === 0) {
            
