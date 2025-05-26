<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>الخطة التدريبية الشاملة</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Cairo:wght@400;500;600;700&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/date-fns@2.28.0/date-fns.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-adapter-date-fns@2.0.0/dist/chartjs-adapter-date-fns.bundle.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@3.7.0/dist/chart.min.js"></script>
    <!-- Application Structure Plan: 
        Header: Contains app title and main navigation buttons ("الخطة الأسبوعية", "مكتبة التمارين", "تحليل أداء الأوزان", "تحليل وزن الجسم"). Active section highlighted.
        Conditional Sub-Navigation/Controls: 
            - When "الخطة الأسبوعية" is active: A secondary row appears with day tabs (Saturday-Friday) and the week selector input.
            - Other sections do not display this secondary row.
        Main Content Area: Dynamically updated based on selected main section and sub-section.
        Content Presentation:
            - الخطة الأسبوعية: Day's plan in a card. Exercise table refined.
            - مكتبة التمارين: Exercises as individual cards. Dropdown filter for muscle groups.
            - تحليل أداء الأوزان: Dropdown for exercise, Chart.js line chart, data table in a card.
            - تحليل وزن الجسم: Input form, Chart.js line chart, data table in a card.
        User Flow: Select main section. If plan, select day/week. If library, browse/filter. If analysis, select data.
        Justification: Improved visual hierarchy, contextual controls, card-based layouts for readability and appeal. -->
    <!-- Visualization & Content Choices:
        Workout Plan: Tabular exercises within day card. Weight inputs per exercise/week. Goal: Clear guidance, easy logging.
        Exercise Library: Cards per exercise. New: Muscle group filter. Goal: Easy browsing. Interaction: Filtering.
        Exercise Weight Analysis: Chart.js line chart & table in a card. Goal: Visualize strength progress.
        Body Weight Analysis: Chart.js line chart & table for body weight, refined input form, in a card. Goal: Visualize body weight trends.
        Styling: "Serene Blue & Warm Sand" palette. Unicode icons for minor visual cues.
        Libraries: Vanilla JS, Chart.js, localStorage. -->
    <style>
        body {
            font-family: 'Cairo', 'Inter', sans-serif;
            background-color: #fffbeb; /* Tailwind amber-50 */
            color: #334155; /* Tailwind slate-700 */
        }
        .main-nav-button, .day-nav-button {
            transition: background-color 0.3s ease, color 0.3s ease, box-shadow 0.3s ease;
            border: 1px solid transparent;
        }
        .main-nav-button {
            background-color: #ffffff;
            color: #0369a1; /* sky-700 */
            border-color: #e0f2fe; /* sky-100 */
        }
        .main-nav-button:hover {
            background-color: #f0f9ff; /* sky-50 */
            border-color: #7dd3fc; /* sky-300 */
        }
        .main-nav-button.active, .day-nav-button.active {
            background-color: #0284c7; /* Tailwind sky-600 */
            color: #f0f9ff; /* Tailwind sky-50 */
            border-color: #0284c7; /* sky-600 */
            box-shadow: 0 4px 14px rgba(2, 132, 199, 0.25);
        }
        .day-nav-button {
             background-color: #f0f9ff; /* sky-50 */
             color: #075985; /* sky-800 */
             border-color: #e0f2fe; /* sky-100 */
        }
         .day-nav-button:hover {
            background-color: #e0f2fe; /* sky-100 */
        }

        .content-card {
            background-color: #ffffff; 
            border-radius: 0.75rem; /* rounded-xl */
            padding: 1.5rem; /* p-6 */
            box-shadow: 0 8px 16px rgba(0,0,0,0.05), 0 3px 6px rgba(0,0,0,0.03);
            margin-top: 1.5rem; /* mt-6 */
        }
        .table th, .table td {
            border: 1px solid #e2e8f0; /* Tailwind slate-200 */
            padding: 0.75rem 1rem; 
            text-align: right;
            vertical-align: middle;
        }
        .table th {
            background-color: #f8fafc; /* Tailwind slate-50 */
            color: #1e293b; /* slate-800 */
            font-weight: 600; 
        }
        .input-field {
            padding: 0.625rem 0.75rem; /* py-2.5 px-3 */
            border: 1px solid #cbd5e1; /* border-slate-300 */
            border-radius: 0.375rem; /* rounded-md */
            text-align: center;
            transition: border-color 0.2s ease, box-shadow 0.2s ease;
        }
        .input-field:focus {
            border-color: #0284c7; /* sky-600 */
            box-shadow: 0 0 0 2px rgba(2, 132, 199, 0.2);
            outline: none;
        }
        .weight-input, .week-input { width: 7rem; }
        .body-weight-input { width: 9rem; }
        .date-input { width: 11rem; }

        .exercise-card {
            background-color: #f0f9ff; /* Tailwind sky-50 */
            border: 1px solid #e0f2fe; /* Tailwind sky-100 */
            border-radius: 0.5rem; 
            padding: 1.25rem; 
            margin-bottom: 1.25rem; 
            box-shadow: 0 4px 8px rgba(0,0,0,0.04);
        }
        .chart-container { 
            position: relative; width: 100%; max-width: 700px; margin-left: auto; margin-right: auto; height: 350px; max-height: 450px; padding: 1.5rem; background-color: #fff; border-radius: 0.75rem; box-shadow: 0 6px 12px rgba(0,0,0,0.05);
        }
        @media (min-width: 768px) { .chart-container { height: 400px; } }
        .hidden { display: none !important; }
        .delete-btn {
            background-color: #f43f5e; /* rose-500 */
            color: white;
            padding: 0.375rem 0.75rem;
            border-radius: 0.375rem;
            font-size: 0.875rem;
            cursor: pointer;
            transition: background-color 0.2s ease;
        }
        .delete-btn:hover {
            background-color: #e11d48; /* rose-600 */
        }
        .submit-btn {
            background-color: #0ea5e9; /* sky-500 */
            color: white;
            font-weight: 600;
            padding: 0.625rem 1.25rem;
            border-radius: 0.375rem;
            transition: background-color 0.2s ease;
        }
        .submit-btn:hover {
            background-color: #0284c7; /* sky-600 */
        }
        .section-title-icon {
            margin-left: 0.5rem; /* ml-2 */
            font-size: 1.2em; /* slightly larger icon */
            color: #0ea5e9; /* sky-500 */
        }
    </style>
</head>
<body class="text-slate-700">
    <div class="container mx-auto p-4 sm:p-6 lg:p-8 max-w-7xl">
        <header class="text-center mb-10">
            <h1 class="text-4xl sm:text-5xl font-bold text-sky-700">الخطة التدريبية الشاملة</h1>
            <p class="mt-3 text-lg text-slate-600">خطتك التدريبية، تسجيل الأوزان، تحليل الأداء، ومكتبة التمارين.</p>
        </header>

        <nav class="flex flex-wrap justify-center gap-3 mb-6" id="mainNavigation">
            <button data-section="plan" class="main-nav-button active px-4 py-2.5 sm:px-6 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">الخطة الأسبوعية</button>
            <button data-section="library" class="main-nav-button px-4 py-2.5 sm:px-6 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">مكتبة التمارين</button>
            <button data-section="exerciseAnalysis" class="main-nav-button px-4 py-2.5 sm:px-6 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">تحليل أداء الأوزان</button>
            <button data-section="bodyWeightAnalysis" class="main-nav-button px-4 py-2.5 sm:px-6 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">تحليل وزن الجسم</button>
        </nav>
        
        <div id="planControlsContainer" class="mb-6 hidden">
            <div class="flex flex-col sm:flex-row justify-center items-center gap-4 p-4 bg-sky-50 rounded-lg shadow-sm">
                <div class="flex items-center gap-2">
                    <label for="currentWeek" class="text-sm font-medium text-slate-700">الأسبوع الحالي لتسجيل الأوزان:</label>
                    <input type="number" id="currentWeek" value="1" min="1" class="week-input input-field">
                </div>
            </div>
            <nav class="flex flex-wrap justify-center gap-2 mt-4" id="dayNavigationContainer">
                </nav>
        </div>


        <main id="appContent" class="min-h-[300px]">
            </main>

        <footer class="text-center mt-16 py-8 border-t border-slate-200">
            <p class="text-slate-500">&copy; 2024 خطتك التدريبية. بالتوفيق في رحلتك نحو القوة والصحة!</p>
        </footer>
    </div>

    <script>
        // --- DATA ---
        const initialWorkoutData = [
             {
                dayId: "sat", dayName: "السبت (الصدر)", focus: "الصدر",
                warmUp: ["10 دقائق كارديو.", "5 دقائق إحماء ديناميكي للصدر والأكتاف."],
                exercises: [
                    { id:"sat_ex1", name: "باربيل بنش بريس", sets: "5", reps: "4-6", rest: "2-3 دقائق", notes: "ركز على القوة." },
                    { id:"sat_ex2", name: "دمبل بنش بريس مائل للأعلى", sets: "4", reps: "8-12", rest: "60-90 ثانية", notes: "للصدر العلوي." },
                    { id:"sat_ex3", name: "غطس للصدر أو بنش بريس مائل للأسفل", sets: "3", reps: "8-12", rest: "60-90 ثانية", notes: "ميل الجذع للغطس." },
                    { id:"sat_ex4", name: "تفتيح بالكيبل أو بالدمبل", sets: "3", reps: "10-15", rest: "60 ثانية", notes: "تمدد وعصر." },
                    { id:"sat_ex5", name: "(اختياري) تمرين الضغط", sets: "2", reps: "حتى الفشل", rest: "60 ثانية", notes: "لضخ دم." }
                ],
                coolDown: [ "10 دقائق كارديو خفيف.", "5-10 دقائق إطالات للصدر." ]
            },
            {
                dayId: "sun", dayName: "الأحد (الظهر)", focus: "الظهر",
                warmUp: ["10 دقائق كارديو.", "5 دقائق إحماء ديناميكي للظهر."],
                exercises: [
                    { id:"sun_ex1", name: "سحب البار بالانحناء", sets: "4", reps: "6-10", rest: "90-120 ثانية", notes: "ظهر مستقيم." },
                    { id:"sun_ex2", name: "عقلة أو سحب أمامي", sets: "4", reps: "AMRAP أو 8-12", rest: "60-90 ثانية", notes: "سحب المرفقين." },
                    { id:"sun_ex3", name: "سحب الكيبل الأرضي (ضيق)", sets: "3", reps: "8-12", rest: "60-90 ثانية", notes: "عصر لوحي الكتفين." },
                    { id:"sun_ex4", name: "سحب التي-بار أو دمبل بذراع", sets: "3", reps: "8-12 (لكل ذراع)", rest: "60-90 ثانية", notes: "مدى كامل." },
                    { id:"sun_ex5", name: "سحب الحبل للوجه", sets: "3", reps: "12-15", rest: "60 ثانية", notes: "لصحة الكتف." }
                ],
                coolDown: ["10 دقائق كارديو خفيف.", "5-10 دقائق إطالات للظهر."]
            },
            {
                dayId: "mon", dayName: "الاثنين (الأكتاف)", focus: "الأكتاف",
                warmUp: ["10 دقائق كارديو.", "5 دقائق إحماء ديناميكي للأكتاف."],
                exercises: [
                    { id:"mon_ex1", name: "ضغط الكتف بالبار وقوفاً", sets: "4", reps: "6-10", rest: "90-120 ثانية", notes: "ثبات الجذع." },
                    { id:"mon_ex2", name: "رفرفة جانبية بالدمبل", sets: "4", reps: "10-15", rest: "60 ثانية", notes: "للرأس الجانبي." },
                    { id:"mon_ex3", name: "رفرفة أمامية بالدمبل", sets: "3", reps: "10-12", rest: "60 ثانية", notes: "للرأس الأمامي." },
                    { id:"mon_ex4", name: "رفرفة خلفية بالدمبل بانحناء", sets: "4", reps: "12-15", rest: "60 ثانية", notes: "للرأس الخلفي." },
                    { id:"mon_ex5", name: "(اختياري) تمرين الترابيس", sets: "3", reps: "10-15", rest: "60 ثانية", notes: "رفع الكتفين." }
                ],
                coolDown: ["10 دقائق كارديو خفيف.", "5-10 دقائق إطالات للأكتاف."]
            },
            {
                dayId: "tue", dayName: "الثلاثاء (راحة)", isRestDay: true,
                advice: ["**الاستشفاء**", "**التغذية والبروتين.**", "**الترطيب.**", "**النوم (7-9 ساعات).**", "**راحة نشطة (اختياري).**"]
            },
            {
                dayId: "wed", dayName: "الأربعاء (الأرجل)", focus: "الأرجل",
                warmUp: ["10 دقائق كارديو.", "5-10 دقائق إحماء ديناميكي للأرجل."],
                exercises: [
                    { id:"wed_ex1", name: "سكوات بالبار الخلفي", sets: "4-5", reps: "6-10", rest: "2-3 دقائق", notes: "انزل بعمق." },
                    { id:"wed_ex2", name: "الرفعة الرومانية (RDLs)", sets: "4", reps: "8-12", rest: "90-120 ثانية", notes: "للفخذ الخلفي." },
                    { id:"wed_ex3", name: "دفع الأرجل بجهاز", sets: "4", reps: "10-15", rest: "60-90 ثانية", notes: "لا تقفل الركبة." },
                    { id:"wed_ex4", name: "ثني الأرجل للفخذ الخلفي", sets: "3", reps: "10-15", rest: "60 ثانية", notes: "عزل الخلفية." },
                    { id:"wed_ex5", name: "تمرين السمانة وقوفاً", sets: "4", reps: "12-20", rest: "45-60 ثانية", notes: "مدى كامل." }
                ],
                coolDown: ["10 دقائق كارديو خفيف.", "5-10 دقائق إطالات للأرجل."]
            },
            {
                dayId: "thu", dayName: "الخميس (الذراعان)", focus: "الذراعان",
                warmUp: ["10 دقائق كارديو.", "5 دقائق إحماء ديناميكي للذراعين."],
                exercises: [
                    { type: "title", sectionTitle: "ترايسبس (Triceps):"},
                    { id:"thu_ex1", name: "ضغط بنش قبضة ضيقة", sets: "3-4", reps: "6-10", rest: "60-90 ثانية", notes: "للترايسبس." },
                    { id:"thu_ex2", name: "تمديد ترايسبس دمبل فوق الرأس", sets: "3", reps: "10-15", rest: "60 ثانية", notes: "للرأس الطويل." },
                    { id:"thu_ex3", name: "دفع كيبل بالحبل للأسفل", sets: "3", reps: "10-15", rest: "60 ثانية", notes: "افتح الحبل." },
                    { type: "title", sectionTitle: "بايسبس (Biceps):"},
                    { id:"thu_ex4", name: "مرجحة بار", sets: "3-4", reps: "8-12", rest: "60-90 ثانية", notes: "ثبات المرفقين." },
                    { id:"thu_ex5", name: "مرجحة دمبل تبادل على بنش مائل", sets: "3", reps: "10-12 (لكل ذراع)", rest: "60 ثانية", notes: "تمدد جيد." },
                    { id:"thu_ex6", name: "مرجحة مطرقة بالدمبل", sets: "3", reps: "10-15 (لكل ذراع)", rest: "60 ثانية", notes: "لسماكة الذراع." }
                ],
                coolDown: ["10 دقائق كارديو خفيف.", "5-10 دقائق إطالات للذراعين."]
            },
            {
                dayId: "fri", dayName: "الجمعة (راحة)", isRestDay: true,
                advice: ["**نفس إرشادات يوم الثلاثاء.**", "**الاستشفاء التام.**", "**الاستعداد للأسبوع القادم.**"]
            }
        ];
        
        const exerciseLibraryData = [
            {
                id: "lib_pushup", name: "تمرين الضغط (Push-ups)", muscleGroup: "الصدر", equipment: "وزن الجسم",
                instructions: ["يديك أوسع قليلاً من كتفيك.","جسم مستقيم.","انزل بصدرك نحو الأرض.","ادفع للأعلى."],
                notes: ["ظهر مستقيم.","تحكم بالحركة."]
            },
            {
                id: "lib_squat", name: "سكوات بوزن الجسم", muscleGroup: "الأرجل", equipment: "وزن الجسم",
                instructions: ["قدميك بعرض الكتفين.","ادفع وركيك للخلف وانزل.","ظهر مستقيم وصدر مرفوع.","فخذيك موازيين للأرض.","ادفع بكعبيك للعودة."],
                notes: ["لا تدع ركبتيك تنحنيان للداخل."]
            },
             {
                id: "lib_deadlift", name: "الرفعة الميتة التقليدية", muscleGroup: "الظهر", equipment: "باربيل وأوزان",
                instructions: ["قف والبار فوق منتصف قدميك.","امسك البار خارج ساقيك.","أنزل وركيك، ظهر مستقيم، صدر مرفوع.","ادفع الأرض بقدميك، البار قريب من الجسم.","قف بشكل كامل ومستقيم.","أنزل البار بتحكم."],
                notes: ["تمرين قوي، تعلم الأداء الصحيح أولاً.","ظهر محايد، لا تقوسه."]
            },
            {
                id: "lib_bicep_curl_concentration", name: "مرجحة البايسبس تركيز بالدمبل", muscleGroup: "البايسبس", equipment: "دمبل، بنش",
                instructions: ["اجلس، مرفقك على فخذك الداخلي.","ذراعك تتدلى بشكل مستقيم.","ارفع الدمبل نحو كتفك، عصر البايسبس.","أنزل الدمبل ببطء."],
                notes: ["ثبات الجذع والكتف.","تجنب الزخم."]
            },
            {
                id: "lib_plank", name: "تمرين البلانك", muscleGroup: "الجذع", equipment: "وزن الجسم",
                instructions: ["ساعديك على الأرض، مرفقاك تحت كتفيك.","جسمك خط مستقيم من الرأس للكعبين.","شد عضلات بطنك ومؤخرتك.","حافظ على الوضعية."],
                notes: ["لا تدع وركيك يتدليان.","تنفس بشكل طبيعي."]
            },
            {
                id: "lib_lat_pulldown", name: "سحب أمامي لأسفل (Lat Pulldown)", muscleGroup: "الظهر", equipment: "جهاز سحب أمامي",
                instructions: ["اجلس على الجهاز مع تثبيت ركبتيك تحت الوسادات.","أمسك البار بقبضة واسعة (أوسع من عرض الكتفين).","اسحب البار للأسفل نحو أعلى صدرك، مع ميل الجذع للخلف قليلاً.","ركز على عصر عضلات ظهرك (اللاتس).","عد بالبار للأعلى ببطء وتحكم."],
                notes: ["تجنب استخدام الزخم أو تأرجح الجسم بشكل مفرط.","حافظ على صدرك مرفوعاً طوال الحركة."]
            },
            {
                id: "lib_dumbbell_shoulder_press", name: "ضغط الكتف بالدمبل جلوساً", muscleGroup: "الأكتاف", equipment: "دمبلز، بنش بمسند ظهر",
                instructions: ["اجلس على بنش مع مسند ظهر مستقيم.","أمسك دمبل في كل يد عند مستوى الكتفين، مع توجيه راحتي اليدين للأمام.","ادفع الدمبلز للأعلى حتى تمتد ذراعاك بالكامل تقريباً، دون قفل المرفقين.","أنزل الدمبلز ببطء وتحكم إلى وضع البداية."],
                notes: ["حافظ على ظهرك مسطحاً على المسند.","تحكم في الحركة وتجنب ترك الدمبلز تسقط بسرعة."]
            }
        ];
        // --- END DATA ---

        // --- DOM ELEMENTS ---
        const mainNavigation = document.getElementById('mainNavigation');
        const dayNavigationContainer = document.getElementById('dayNavigationContainer');
        const planControlsContainer = document.getElementById('planControlsContainer'); // Renamed from weekSelectorContainer
        const currentWeekInput = document.getElementById('currentWeek');
        const appContent = document.getElementById('appContent');
        // --- END DOM ELEMENTS ---

        // --- APP STATE ---
        const dayNamesArabic = {
            sat: "السبت", sun: "الأحد", mon: "الاثنين", tue: "الثلاثاء",
            wed: "الأربعاء", thu: "الخميس", fri: "الجمعة"
        };
        let currentSection = 'plan'; 
        let currentDayId = initialWorkoutData[0].dayId;
        let userWeeklyWeights = {}; 
        let userBodyWeightHistory = []; 
        let exercisePerformanceChart = null;
        let bodyWeightChart = null;
        // --- END APP STATE ---

        // --- DATA HANDLING FUNCTIONS ---
        function loadData() {
            const storedWeeklyWeights = localStorage.getItem('userWeeklyWorkoutWeights');
            if (storedWeeklyWeights) userWeeklyWeights = JSON.parse(storedWeeklyWeights);
            
            const storedBodyWeights = localStorage.getItem('userBodyWeightHistory');
            if (storedBodyWeights) userBodyWeightHistory = JSON.parse(storedBodyWeights);
        }

        function saveWeeklyWeight(exerciseId, week, weight) {
            if (!userWeeklyWeights[exerciseId]) userWeeklyWeights[exerciseId] = {};
            if (weight === "" || weight === null || isNaN(parseFloat(weight))) {
                delete userWeeklyWeights[exerciseId][`week${week}`];
            } else {
                userWeeklyWeights[exerciseId][`week${week}`] = parseFloat(weight);
            }
            localStorage.setItem('userWeeklyWorkoutWeights', JSON.stringify(userWeeklyWeights));
        }

        function getWeeklyWeight(exerciseId, week) {
            return userWeeklyWeights[exerciseId] && userWeeklyWeights[exerciseId][`week${week}`] !== undefined 
                   ? userWeeklyWeights[exerciseId][`week${week}`] 
                   : "";
        }

        function saveBodyWeightEntry(date, weight) {
            if (!date || isNaN(parseFloat(weight)) || parseFloat(weight) <= 0) {
                alert("الرجاء إدخال تاريخ ووزن صحيحين.");
                return;
            }
            const existingEntryIndex = userBodyWeightHistory.findIndex(entry => entry.date === date);
            if (existingEntryIndex > -1) {
                userBodyWeightHistory[existingEntryIndex].weight = parseFloat(weight);
            } else {
                 userBodyWeightHistory.push({ date: date, weight: parseFloat(weight), id: Date.now() });
            }
            userBodyWeightHistory.sort((a, b) => new Date(a.date) - new Date(b.date)); 
            localStorage.setItem('userBodyWeightHistory', JSON.stringify(userBodyWeightHistory));
            displayBodyWeightAnalysis(); 
        }
        
        function deleteBodyWeightEntry(entryId) {
            userBodyWeightHistory = userBodyWeightHistory.filter(entry => entry.id !== entryId);
            localStorage.setItem('userBodyWeightHistory', JSON.stringify(userBodyWeightHistory));
            displayBodyWeightAnalysis(); 
        }
        // --- END DATA HANDLING FUNCTIONS ---

        // --- UI RENDERING FUNCTIONS ---
        function renderDayNavigation() {
            dayNavigationContainer.innerHTML = ''; 
            if (currentSection !== 'plan') {
                planControlsContainer.classList.add('hidden');
                return;
            }
            planControlsContainer.classList.remove('hidden');

            initialWorkoutData.forEach((day) => {
                const button = document.createElement('button');
                button.textContent = dayNamesArabic[day.dayId];
                button.classList.add('day-nav-button', 'px-3', 'py-2', 'sm:px-4', 'sm:py-2', 'text-xs', 'sm:text-sm', 'font-semibold', 'rounded-lg', 'shadow-sm');
                button.dataset.dayId = day.dayId;
                if (day.dayId === currentDayId) button.classList.add('active');
                button.addEventListener('click', () => {
                    currentDayId = day.dayId;
                    displayWorkoutPlan(day.dayId);
                    document.querySelectorAll('#dayNavigationContainer .day-nav-button').forEach(btn => btn.classList.remove('active'));
                    button.classList.add('active');
                });
                dayNavigationContainer.appendChild(button);
            });
        }
        
        function buildList(itemsArray, listClasses = "list-disc list-inside space-y-1.5 mb-4 text-slate-600") {
            if (!itemsArray || itemsArray.length === 0) return '';
            let listHtml = `<ul class="${listClasses}">`;
            itemsArray.forEach(item => { listHtml += `<li>${item}</li>`; });
            listHtml += '</ul>';
            return listHtml;
        }
        
        function buildExercisesTable(exercisesArray) {
            if (!exercisesArray || exercisesArray.length === 0) return '<p class="text-slate-500">لا توجد تمارين لهذا اليوم.</p>';
            const selectedWeek = parseInt(currentWeekInput.value) || 1;
            
            let tableHtml = '<div class="overflow-x-auto mt-4"><table class="table w-full min-w-[750px] text-sm"><thead><tr>';
            tableHtml += '<th class="w-2/6">التمرين</th><th>المجموعات</th><th>التكرارات</th><th>الراحة</th>';
            tableHtml += `<th class="w-1/6">الوزن للأسبوع ${selectedWeek} (كغم)</th><th class="w-2/6">ملاحظات</th></tr></thead><tbody>`;

            exercisesArray.forEach((ex) => {
                if (ex.type === 'title') {
                    tableHtml += `<tr><td colspan="6" class="bg-sky-100 text-sky-800 font-bold text-md py-3">${ex.sectionTitle}</td></tr>`;
                } else {
                    const weightValue = getWeeklyWeight(ex.id, selectedWeek);
                    tableHtml += `<tr><td>${ex.name}</td><td>${ex.sets}</td><td>${ex.reps}</td><td>${ex.rest}</td>`;
                    tableHtml += `<td><input type="number" step="0.25" class="weight-input input-field" value="${weightValue}" placeholder="--" onchange="saveWeeklyWeight('${ex.id}', ${selectedWeek}, this.value)"></td>`;
                    tableHtml += `<td class="text-xs text-slate-500">${ex.notes}</td></tr>`;
                }
            });
            tableHtml += '</tbody></table></div>';
            return tableHtml;
        }

        function displayWorkoutPlan(dayId) {
            const dayData = initialWorkoutData.find(d => d.dayId === dayId);
            if (!dayData) { appContent.innerHTML = '<p class="text-center p-10">لم يتم العثور على بيانات لهذا اليوم.</p>'; return; }

            let contentHtml = `<div class="content-card animate-fadeIn">
                <h2 class="text-3xl font-bold mb-4 text-sky-700">${dayData.dayName} <span class="text-lg text-slate-500">(${dayData.focus})</span></h2>`;
            if (dayData.isRestDay) {
                contentHtml += `<h3 class="text-xl font-semibold mt-6 mb-3 text-sky-600"><span class="section-title-icon">🛌</span>نصائح يوم الراحة:</h3>${buildList(dayData.advice)}`;
            } else {
                contentHtml += `<div class="grid md:grid-cols-2 gap-6">
                                    <div>
                                        <h3 class="text-xl font-semibold mt-2 mb-3 text-sky-600"><span class="section-title-icon">🔥</span>الإحماء:</h3>${buildList(dayData.warmUp)}
                                    </div>
                                    <div>
                                        <h3 class="text-xl font-semibold mt-2 mb-3 text-sky-600"><span class="section-title-icon">❄️</span>التهدئة:</h3>${buildList(dayData.coolDown)}
                                    </div>
                               </div>
                                <h3 class="text-2xl font-semibold mt-8 mb-3 text-sky-600"><span class="section-title-icon">🏋️</span>التمارين:</h3>
                                <p class="text-sm text-slate-500 mb-3">سجل أوزانك للأسبوع المحدد في الخانات المخصصة. سيتم الحفظ تلقائياً.</p>
                                ${buildExercisesTable(dayData.exercises)}`;
            }
            contentHtml += `</div>`;
            appContent.innerHTML = contentHtml;
        }

        function displayExerciseLibrary() {
            let contentHtml = `<div class="content-card animate-fadeIn">
                <div class="flex flex-col sm:flex-row justify-between items-center mb-6">
                    <h2 class="text-3xl font-bold text-sky-700 mb-4 sm:mb-0"><span class="section-title-icon">📚</span>مكتبة التمارين</h2>
                    <div class="w-full sm:w-auto">
                        <label for="muscleGroupFilter" class="sr-only">فلتر حسب المجموعة العضلية:</label>
                        <select id="muscleGroupFilter" class="input-field w-full sm:w-56">
                            <option value="all">كل المجموعات العضلية</option>
                            </select>
                    </div>
                </div>
                <p class="text-lg mb-6 text-slate-600">استكشف تمارين متنوعة لتعزيز برنامجك التدريبي.</p>
                <div id="exerciseListContainer" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                    </div>
            </div>`;
            appContent.innerHTML = contentHtml;

            const muscleGroups = [...new Set(exerciseLibraryData.map(ex => ex.muscleGroup))].sort();
            const filterSelect = document.getElementById('muscleGroupFilter');
            muscleGroups.forEach(group => {
                filterSelect.innerHTML += `<option value="${group}">${group}</option>`;
            });

            renderExerciseLibraryCards('all'); // Initial render

            filterSelect.addEventListener('change', (e) => {
                renderExerciseLibraryCards(e.target.value);
            });
        }

        function renderExerciseLibraryCards(selectedGroup) {
            const container = document.getElementById('exerciseListContainer');
            container.innerHTML = ''; // Clear previous cards
            const filteredExercises = selectedGroup === 'all' 
                ? exerciseLibraryData 
                : exerciseLibraryData.filter(ex => ex.muscleGroup === selectedGroup);

            if (filteredExercises.length === 0) {
                container.innerHTML = '<p class="text-slate-500 md:col-span-2 lg:col-span-3 text-center">لا توجد تمارين تطابق هذا الفلتر.</p>';
                return;
            }

            filteredExercises.forEach(ex => {
                let cardHtml = `<div class="exercise-card flex flex-col h-full">
                    <h3 class="text-xl font-semibold text-sky-700 mb-2">${ex.name}</h3>
                    <p class="text-sm text-slate-600 mb-1"><strong>المجموعة العضلية:</strong> ${ex.muscleGroup}</p>
                    <p class="text-sm text-slate-600 mb-4"><strong>المعدات:</strong> ${ex.equipment}</p>
                    <div class="text-sm">
                        <h4 class="font-semibold text-slate-700 mb-1 mt-2">طريقة اللعب:</h4>
                        ${buildList(ex.instructions, "list-decimal list-inside space-y-1 mb-3 text-slate-600 text-xs")}
                        <h4 class="font-semibold text-slate-700 mb-1 mt-2">ملاحظات هامة:</h4>
                        ${buildList(ex.notes, "list-disc list-inside space-y-1 text-slate-600 text-xs")}
                    </div>
                </div>`;
                container.innerHTML += cardHtml;
            });
        }
        
        function displayExercisePerformanceAnalysis() {
            let contentHtml = `<div class="content-card animate-fadeIn">
                <h2 class="text-3xl font-bold mb-6 text-sky-700"><span class="section-title-icon">📊</span>تحليل أداء الأوزان</h2>
                <p class="text-lg mb-6 text-slate-600">اختر تمريناً لعرض تطور الأوزان التي رفعتها عبر الأسابيع.</p>
                <div class="mb-6">
                    <label for="exerciseAnalysisSelect" class="block text-sm font-medium text-slate-700 mb-1">اختر التمرين:</label>
                    <select id="exerciseAnalysisSelect" class="input-field mt-1 block w-full pl-3 pr-10 py-2 text-base sm:text-sm rounded-md shadow-sm">
                        <option value="">-- اختر تمريناً --</option>`;
            const allExercises = [];
            initialWorkoutData.forEach(day => {
                if (day.exercises) {
                    day.exercises.forEach(ex => {
                        if (ex.type !== 'title' && !allExercises.find(e => e.id === ex.id)) {
                            allExercises.push({id: ex.id, name: ex.name, day: dayNamesArabic[day.dayId]});
                        }
                    });
                }
            });
            allExercises.sort((a,b) => a.name.localeCompare(b.name));
            allExercises.forEach(ex => { contentHtml += `<option value="${ex.id}">${ex.name} (${ex.day})</option>`; });
            contentHtml += `</select></div>
                <div id="exerciseChartContainerWrapper" class="chart-container mb-8 hidden"><canvas id="exercisePerformanceChartEl"></canvas></div>
                <div id="exercisePerformanceTableContainer" class="overflow-x-auto hidden"></div>
            </div>`;
            appContent.innerHTML = contentHtml;

            document.getElementById('exerciseAnalysisSelect').addEventListener('change', function() {
                const selectedExerciseId = this.value;
                if (selectedExerciseId) {
                    renderExercisePerformanceChartAndTable(selectedExerciseId);
                } else {
                    if (exercisePerformanceChart) exercisePerformanceChart.destroy();
                    document.getElementById('exerciseChartContainerWrapper').classList.add('hidden');
                    document.getElementById('exercisePerformanceTableContainer').classList.add('hidden').innerHTML = '';
                }
            });
        }

        function renderExercisePerformanceChartAndTable(exerciseId) {
            const exerciseData = userWeeklyWeights[exerciseId];
            const chartWrapper = document.getElementById('exerciseChartContainerWrapper');
            const tableContainer = document.getElementById('exercisePerformanceTableContainer');
            
            if (!exerciseData || Object.keys(exerciseData).length === 0) {
                chartWrapper.classList.add('hidden');
                tableContainer.innerHTML = '<p class="text-slate-500 text-center py-4">لا توجد بيانات أوزان مسجلة لهذا التمرين.</p>';
                tableContainer.classList.remove('hidden');
                if (exercisePerformanceChart) exercisePerformanceChart.destroy();
                return;
            }
            chartWrapper.classList.remove('hidden');
            tableContainer.classList.remove('hidden');

            const weeks = Object.keys(exerciseData).map(k => parseInt(k.replace('week', ''))).sort((a, b) => a - b);
            const weights = weeks.map(w => exerciseData[`week${w}`]);
            const labels = weeks.map(w => `أسبوع ${w}`);

            if (exercisePerformanceChart) exercisePerformanceChart.destroy();
            exercisePerformanceChart = new Chart(document.getElementById('exercisePerformanceChartEl').getContext('2d'), {
                type: 'line', data: { labels, datasets: [{ label: 'الوزن (كغم)', data: weights, borderColor: '#0284c7', tension: 0.1, fill: true, backgroundColor: 'rgba(2, 132, 199, 0.1)', pointBackgroundColor: '#0369a1' }] },
                options: { responsive: true, maintainAspectRatio: false, scales: { y: { beginAtZero: false, title: {display: true, text:'الوزن (كغم)', font: { family: "'Cairo', sans-serif" } }, ticks: { font: { family: "'Cairo', sans-serif" } } }, x: {title: {display: true, text:'الأسبوع', font: { family: "'Cairo', sans-serif" } }, ticks: { font: { family: "'Cairo', sans-serif" } }} }, plugins: { legend: { labels: { font: { family: "'Cairo', sans-serif" } } }, tooltip: { bodyFont: { family: "'Cairo', sans-serif" }, titleFont: { family: "'Cairo', sans-serif" }, callbacks: { label: function(context) { return `${context.dataset.label}: ${context.parsed.y} كغم`; } } } } }
            });

            let tableHtml = '<h4 class="text-lg font-semibold mb-3 text-slate-700">ملخص الأوزان:</h4><table class="table w-full sm:w-auto mx-auto"><thead><tr><th>الأسبوع</th><th>الوزن (كغم)</th></tr></thead><tbody>';
            weeks.forEach((w, i) => { tableHtml += `<tr><td>أسبوع ${w}</td><td>${weights[i]}</td></tr>`; });
            tableHtml += '</tbody></table>';
            tableContainer.innerHTML = tableHtml;
        }

        function displayBodyWeightAnalysis() {
            let contentHtml = `<div class="content-card animate-fadeIn">
                <h2 class="text-3xl font-bold mb-6 text-sky-700"><span class="section-title-icon">⚖️</span>تحليل وزن الجسم وتتبع التقدم</h2>
                <p class="text-lg mb-6 text-slate-600">سجل وزن جسمك بانتظام وتتبع تقدمك نحو أهدافك الصحية.</p>
                
                <form id="bodyWeightForm" class="mb-8 p-6 border border-sky-100 rounded-lg bg-sky-50 grid grid-cols-1 sm:grid-cols-3 gap-4 items-end shadow-sm">
                    <div>
                        <label for="bodyWeightDate" class="block text-sm font-medium text-slate-700 mb-1">التاريخ:</label>
                        <input type="date" id="bodyWeightDate" required class="date-input input-field w-full">
                    </div>
                    <div>
                        <label for="bodyWeightInput" class="block text-sm font-medium text-slate-700 mb-1">وزن الجسم (كغم):</label>
                        <input type="number" step="0.1" id="bodyWeightInput" required placeholder="مثال: 75.5" class="body-weight-input input-field w-full">
                    </div>
                    <button type="submit" class="submit-btn h-fit w-full sm:w-auto">حفظ الوزن</button>
                </form>

                <h3 class="text-xl font-semibold mt-8 mb-4 text-sky-600">تطور وزن الجسم:</h3>
                <div id="bodyWeightChartContainerWrapper" class="chart-container mb-8 ${userBodyWeightHistory.length === 0 ? 'hidden' : ''}"><canvas id="bodyWeightChartEl"></canvas></div>
                
                <h3 class="text-xl font-semibold mt-8 mb-4 text-sky-600">سجل وزن الجسم:</h3>
                <div id="bodyWeightTableContainer" class="overflow-x-auto"></div>
            </div>`;
            appContent.innerHTML = contentHtml;
            
            renderBodyWeightChartAndTable();

            document.getElementById('bodyWeightForm').addEventListener('submit', function(event) {
                event.preventDefault();
                const date = document.getElementById('bodyWeightDate').value;
                const weight = document.getElementById('bodyWeightInput').value;
                saveBodyWeightEntry(date, weight);
                this.reset(); 
            });
        }

        function renderBodyWeightChartAndTable() {
            const chartWrapper = document.getElementById('bodyWeightChartContainerWrapper');
            const tableContainer = document.getElementById('bodyWeightTableContainer');

            if (!userBodyWeightHistory || userBodyWeightHistory.length === 0) {
                if(chartWrapper) chartWrapper.classList.add('hidden');
                if(tableContainer) tableContainer.innerHTML = '<p class="text-slate-500 text-center py-4">لا توجد بيانات وزن جسم مسجلة حتى الآن. ابدأ بإضافة إدخال جديد.</p>';
                if (bodyWeightChart) { bodyWeightChart.destroy(); bodyWeightChart = null; }
                return;
            }
            if(chartWrapper) chartWrapper.classList.remove('hidden');
            
            userBodyWeightHistory.sort((a, b) => new Date(a.date) - new Date(b.date)); 

            const chartDataPoints = userBodyWeightHistory.map(entry => {
                const parts = entry.date.split('-');
                return {
                    x: new Date(parseInt(parts[0]), parseInt(parts[1]) - 1, parseInt(parts[2])),
                    y: entry.weight
                };
            });

            if (bodyWeightChart) bodyWeightChart.destroy();
            const canvasEl = document.getElementById('bodyWeightChartEl');
            if (!canvasEl) return; 
            const ctx = canvasEl.getContext('2d');
            
            bodyWeightChart = new Chart(ctx, {
                type: 'line',
                data: {
                    datasets: [{
                        label: 'وزن الجسم (كغم)', 
                        data: chartDataPoints, 
                        borderColor: '#0284c7', 
                        tension: 0.1, 
                        fill: true,
                        backgroundColor: 'rgba(2, 132, 199, 0.1)', 
                        pointBackgroundColor: '#0369a1'
                    }]
                },
                options: {
                    responsive: true, maintainAspectRatio: false,
                    scales: {
                        y: { 
                            beginAtZero: false, 
                            title: { display: true, text: 'وزن الجسم (كغم)', font: { family: "'Cairo', sans-serif" } }, 
                            ticks: { font: { family: "'Cairo', sans-serif" } } 
                        },
                        x: { 
                            type: 'time', 
                            title: { display: true, text: 'التاريخ', font: { family: "'Cairo', sans-serif" } }, 
                            time: { 
                                unit: 'day',
                                displayFormats: { day: 'MMM d, yy' }
                            }, 
                            ticks: { font: { family: "'Cairo', sans-serif" }, source: 'auto', major: { enabled: true } } 
                        }
                    },
                    plugins: { 
                        legend: { labels: { font: { family: "'Cairo', sans-serif" } } },
                        tooltip: { 
                            bodyFont: { family: "'Cairo', sans-serif" }, titleFont: { family: "'Cairo', sans-serif" },
                            callbacks: { 
                                title: (tooltipItems) => { 
                                    if (tooltipItems && tooltipItems[0] && tooltipItems[0].parsed) {
                                        return dateFns.format(new Date(tooltipItems[0].parsed.x), 'PPP', { locale: dateFns.locale.arSA || dateFns.locale.enUS }); 
                                    }
                                    return '';
                                },
                                label: function(context) {
                                    return `${context.dataset.label}: ${context.parsed.y.toFixed(1)} كغم`;
                                }
                            } 
                        } 
                    }
                }
            });
            
            let tableHtml = '<table class="table w-full min-w-[450px] text-sm"><thead><tr><th>التاريخ</th><th>وزن الجسم (كغم)</th><th>إجراء</th></tr></thead><tbody>';
            [...userBodyWeightHistory].reverse().forEach(entry => {
                const dateParts = entry.date.split('-');
                const displayDate = new Date(parseInt(dateParts[0]), parseInt(dateParts[1]) - 1, parseInt(dateParts[2]));
                const formattedDate = displayDate.toLocaleDateString('ar-EG-u-nu-latn', { year: 'numeric', month: 'long', day: 'numeric' });
                tableHtml += `<tr><td>${formattedDate}</td><td>${entry.weight.toFixed(1)}</td><td><button class="delete-btn" onclick="deleteBodyWeightEntry(${entry.id})">حذف</button></td></tr>`;
            });
            tableHtml += '</tbody></table>';
            if(tableContainer) tableContainer.innerHTML = tableHtml;
        }
        
        // --- NAVIGATION & INITIALIZATION ---
        function setActiveMainTab(section) {
            document.querySelectorAll('#mainNavigation .main-nav-button').forEach(btn => {
                btn.classList.remove('active');
                if (btn.dataset.section === section) btn.classList.add('active');
            });
        }

        mainNavigation.addEventListener('click', (event) => {
            if (event.target.tagName === 'BUTTON') {
                const section = event.target.dataset.section;
                if (section && section !== currentSection) {
                    currentSection = section;
                    setActiveMainTab(section);
                    if (exercisePerformanceChart) { exercisePerformanceChart.destroy(); exercisePerformanceChart = null; }
                    if (bodyWeightChart) { bodyWeightChart.destroy(); bodyWeightChart = null; }

                    if (section === 'plan') {
                        renderDayNavigation(); // This will also show planControlsContainer
                        displayWorkoutPlan(currentDayId); 
                    } else {
                        planControlsContainer.classList.add('hidden'); // Hide for other sections
                        if (section === 'library') displayExerciseLibrary();
                        else if (section === 'exerciseAnalysis') displayExercisePerformanceAnalysis();
                        else if (section === 'bodyWeightAnalysis') displayBodyWeightAnalysis();
                    }
                }
            }
        });
        
        currentWeekInput.addEventListener('change', () => {
            if (currentSection === 'plan') displayWorkoutPlan(currentDayId);
        });
        currentWeekInput.addEventListener('input', () => { // For immediate update
             if (currentSection === 'plan') displayWorkoutPlan(currentDayId);
        });


        document.addEventListener('DOMContentLoaded', () => {
            loadData();
            renderDayNavigation(); // Initial render for day tabs and plan controls
            
            // Set initial view based on currentSection
            if (currentSection === 'plan') {
                if (initialWorkoutData.length > 0) {
                    displayWorkoutPlan(currentDayId);
                    const firstDayButton = dayNavigationContainer.querySelector(`[data-day-id="${currentDayId}"]`);
                    if (firstDayButton) firstDayButton.classList.add('active');
                }
            } else {
                planControlsContainer.classList.add('hidden'); // Ensure hidden if not plan
                if (currentSection === 'library') displayExerciseLibrary();
                else if (currentSection === 'exerciseAnalysis') displayExercisePerformanceAnalysis();
                else if (currentSection === 'bodyWeightAnalysis') displayBodyWeightAnalysis();
            }
            setActiveMainTab(currentSection); // Ensure correct main tab is active
        });

        const styleSheet = document.createElement("style");
        styleSheet.type = "text/css";
        styleSheet.innerText = `
            @keyframes fadeIn {
                from { opacity: 0; transform: translateY(20px); }
                to { opacity: 1; transform: translateY(0); }
            }
            .animate-fadeIn { animation: fadeIn 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94) forwards; }
        `;
        document.head.appendChild(styleSheet);

    </script>
</body>
</html>
