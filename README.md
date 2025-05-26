<!DOCTYPE html>
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
        Multi-User Simulation: User profile creation/selection modal. Active user ID prefixes localStorage keys.
        Header: App title. User profile display area with "Switch/Create Profile" button.
        Main Navigation: "الخطة الأسبوعية", "مكتبة التمارين", "تحليل أداء الأوزان", "تحليل وزن الجسم".
        Conditional Sub-Navigation (for "الخطة الأسبوعية"): Day tabs, week selector.
        Main Content Area: Dynamically updated.
        Content Presentation:
            - الخطة الأسبوعية: Day's plan in a card. Exercise table with 1RM estimate.
            - مكتبة التمارين: Exercises as cards. Muscle group filter.
            - تحليل أداء الأوزان: Dropdown, Chart.js line chart, data table, 1RM display for latest set.
            - تحليل وزن الجسم: Input form (date, weight, target weight). Chart.js line chart with target line. Data table.
        User Flow: Profile selection/creation -> Main section -> Sub-section/interaction.
        Justification: Robust multi-user (simulated) experience, enhanced analytics, and improved UI clarity. -->
    <!-- Visualization & Content Choices:
        User Profiles: Modal for selection/creation. Data namespaced in localStorage.
        Workout Plan: Tabular exercises. Weight inputs. New: Display estimated 1RM per exercise based on current week's log.
        Exercise Library: Cards, muscle group filter.
        Exercise Weight Analysis: Chart, table. New: Display estimated 1RM for the latest recorded set of the selected exercise.
        Body Weight Analysis: Chart with target line, table. New: Target weight input, progress to target display.
        Styling: "Serene Blue & Warm Sand" palette.
        Libraries: Vanilla JS, Chart.js, localStorage. -->
    <style>
        body { font-family: 'Cairo', 'Inter', sans-serif; background-color: #fffbeb; color: #334155; }
        .main-nav-button, .day-nav-button, .profile-btn { transition: all 0.3s ease; border: 1px solid transparent; }
        .main-nav-button { background-color: #ffffff; color: #0369a1; border-color: #e0f2fe; }
        .main-nav-button:hover { background-color: #f0f9ff; border-color: #7dd3fc; }
        .main-nav-button.active, .day-nav-button.active { background-color: #0284c7; color: #f0f9ff; border-color: #0284c7; box-shadow: 0 4px 14px rgba(2, 132, 199, 0.25); }
        .day-nav-button { background-color: #f0f9ff; color: #075985; border-color: #e0f2fe; }
        .day-nav-button:hover { background-color: #e0f2fe; }
        .content-card { background-color: #ffffff; border-radius: 0.75rem; padding: 1.5rem; box-shadow: 0 8px 16px rgba(0,0,0,0.05), 0 3px 6px rgba(0,0,0,0.03); margin-top: 1.5rem; }
        .table th, .table td { border: 1px solid #e2e8f0; padding: 0.75rem 1rem; text-align: right; vertical-align: middle; }
        .table th { background-color: #f8fafc; color: #1e293b; font-weight: 600; }
        .input-field, .select-field { padding: 0.625rem 0.75rem; border: 1px solid #cbd5e1; border-radius: 0.375rem; text-align: center; transition: border-color 0.2s ease, box-shadow 0.2s ease; }
        .input-field:focus, .select-field:focus { border-color: #0284c7; box-shadow: 0 0 0 2px rgba(2, 132, 199, 0.2); outline: none; }
        .weight-input, .week-input { width: 7rem; }
        .body-weight-input { width: 9rem; }
        .date-input { width: 11rem; }
        .exercise-card { background-color: #f0f9ff; border: 1px solid #e0f2fe; border-radius: 0.5rem; padding: 1.25rem; margin-bottom: 1.25rem; box-shadow: 0 4px 8px rgba(0,0,0,0.04); }
        .chart-container { position: relative; width: 100%; max-width: 700px; margin-left: auto; margin-right: auto; height: 350px; max-height: 450px; padding: 1.5rem; background-color: #fff; border-radius: 0.75rem; box-shadow: 0 6px 12px rgba(0,0,0,0.05); }
        @media (min-width: 768px) { .chart-container { height: 400px; } }
        .hidden { display: none !important; }
        .delete-btn { background-color: #f43f5e; color: white; padding: 0.375rem 0.75rem; border-radius: 0.375rem; font-size: 0.875rem; cursor: pointer; transition: background-color 0.2s ease; }
        .delete-btn:hover { background-color: #e11d48; }
        .submit-btn, .profile-btn { background-color: #0ea5e9; color: white; font-weight: 600; padding: 0.625rem 1.25rem; border-radius: 0.375rem; transition: background-color 0.2s ease; }
        .submit-btn:hover, .profile-btn:hover { background-color: #0284c7; }
        .section-title-icon { margin-left: 0.5rem; font-size: 1.2em; color: #0ea5e9; }
        .modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background-color: rgba(0,0,0,0.5); display: flex; justify-content: center; align-items: center; z-index: 1000; }
        .modal-content { background-color: white; padding: 2rem; border-radius: 0.5rem; box-shadow: 0 10px 25px rgba(0,0,0,0.1); width: 90%; max-width: 500px; }
    </style>
</head>
<body class="text-slate-700">
    <div id="userProfileModalOverlay" class="modal-overlay hidden">
        <div class="modal-content">
            <h2 class="text-2xl font-bold text-sky-700 mb-6 text-center">ملف التعريف</h2>
            <div id="existingProfilesContainer" class="mb-4">
                <label for="profileSelect" class="block text-sm font-medium text-slate-700 mb-1">اختر ملف تعريف موجود:</label>
                <select id="profileSelect" class="select-field w-full mb-2"></select>
                <button id="selectProfileBtn" class="submit-btn w-full mb-4">اختيار الملف الشخصي</button>
            </div>
            <hr class="my-4">
            <div>
                <label for="newProfileName" class="block text-sm font-medium text-slate-700 mb-1">أو قم بإنشاء ملف تعريف جديد:</label>
                <input type="text" id="newProfileName" placeholder="اسم المستخدم الجديد" class="input-field w-full mb-2">
                <button id="createProfileBtn" class="submit-btn w-full bg-emerald-500 hover:bg-emerald-600">إنشاء وحفظ</button>
            </div>
             <button id="closeProfileModalBtn" class="mt-6 text-sm text-slate-500 hover:text-slate-700 w-full text-center">إغلاق (إذا كان هناك مستخدم نشط)</button>
        </div>
    </div>

    <div class="container mx-auto p-4 sm:p-6 lg:p-8 max-w-7xl">
        <header class="text-center mb-6">
            <div class="flex justify-between items-center mb-4">
                <div></div> <h1 class="text-4xl sm:text-5xl font-bold text-sky-700">الخطة التدريبية الشاملة</h1>
                <div id="userDisplayArea" class="text-sm">
                    <span id="currentUserName" class="font-semibold text-sky-600"></span>
                    <button id="switchProfileBtn" class="profile-btn text-xs px-3 py-1 ml-2">تبديل/إنشاء ملف</button>
                </div>
            </div>
            <p class="mt-1 text-lg text-slate-600">خطتك التدريبية، تسجيل الأوزان، تحليل الأداء، ومكتبة التمارين.</p>
        </header>

        <nav class="flex flex-wrap justify-center gap-3 mb-6" id="mainNavigation">
            <button data-section="plan" class="main-nav-button active px-3 py-2.5 sm:px-5 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">الخطة الأسبوعية</button>
            <button data-section="library" class="main-nav-button px-3 py-2.5 sm:px-5 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">مكتبة التمارين</button>
            <button data-section="exerciseAnalysis" class="main-nav-button px-3 py-2.5 sm:px-5 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">تحليل أداء الأوزان</button>
            <button data-section="bodyWeightAnalysis" class="main-nav-button px-3 py-2.5 sm:px-5 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">تحليل وزن الجسم</button>
        </nav>
        
        <div id="planControlsContainer" class="mb-6 hidden">
            <div class="flex flex-col sm:flex-row justify-center items-center gap-4 p-4 bg-sky-50 rounded-lg shadow-sm">
                <div class="flex items-center gap-2">
                    <label for="currentWeek" class="text-sm font-medium text-slate-700">الأسبوع الحالي لتسجيل الأوزان:</label>
                    <input type="number" id="currentWeek" value="1" min="1" class="week-input input-field">
                </div>
            </div>
            <nav class="flex flex-wrap justify-center gap-2 mt-4" id="dayNavigationContainer"></nav>
        </div>

        <main id="appContent" class="min-h-[300px]"></main>

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
            { id: "lib_pushup", name: "تمرين الضغط (Push-ups)", muscleGroup: "الصدر", equipment: "وزن الجسم", instructions: ["يديك أوسع قليلاً من كتفيك.","جسم مستقيم.","انزل بصدرك نحو الأرض.","ادفع للأعلى."], notes: ["ظهر مستقيم.","تحكم بالحركة."] },
            { id: "lib_squat", name: "سكوات بوزن الجسم", muscleGroup: "الأرجل", equipment: "وزن الجسم", instructions: ["قدميك بعرض الكتفين.","ادفع وركيك للخلف وانزل.","ظهر مستقيم وصدر مرفوع.","فخذيك موازيين للأرض.","ادفع بكعبيك للعودة."], notes: ["لا تدع ركبتيك تنحنيان للداخل."] },
            { id: "lib_deadlift", name: "الرفعة الميتة التقليدية", muscleGroup: "الظهر", equipment: "باربيل وأوزان", instructions: ["قف والبار فوق منتصف قدميك.","امسك البار خارج ساقيك.","أنزل وركيك، ظهر مستقيم، صدر مرفوع.","ادفع الأرض بقدميك، البار قريب من الجسم.","قف بشكل كامل ومستقيم.","أنزل البار بتحكم."], notes: ["تمرين قوي، تعلم الأداء الصحيح أولاً.","ظهر محايد، لا تقوسه."] },
            { id: "lib_bicep_curl_concentration", name: "مرجحة البايسبس تركيز بالدمبل", muscleGroup: "البايسبس", equipment: "دمبل، بنش", instructions: ["اجلس، مرفقك على فخذك الداخلي.","ذراعك تتدلى بشكل مستقيم.","ارفع الدمبل نحو كتفك، عصر البايسبس.","أنزل الدمبل ببطء."], notes: ["ثبات الجذع والكتف.","تجنب الزخم."] },
            { id: "lib_plank", name: "تمرين البلانك", muscleGroup: "الجذع", equipment: "وزن الجسم", instructions: ["ساعديك على الأرض، مرفقاك تحت كتفيك.","جسمك خط مستقيم من الرأس للكعبين.","شد عضلات بطنك ومؤخرتك.","حافظ على الوضعية."], notes: ["لا تدع وركيك يتدليان.","تنفس بشكل طبيعي."] },
            { id: "lib_lat_pulldown", name: "سحب أمامي لأسفل (Lat Pulldown)", muscleGroup: "الظهر", equipment: "جهاز سحب أمامي", instructions: ["اجلس على الجهاز مع تثبيت ركبتيك.","أمسك البار بقبضة واسعة.","اسحب البار للأسفل نحو أعلى صدرك.","ركز على عصر عضلات ظهرك.","عد بالبار للأعلى ببطء."], notes: ["تجنب استخدام الزخم.","حافظ على صدرك مرفوعاً."] },
            { id: "lib_dumbbell_shoulder_press", name: "ضغط الكتف بالدمبل جلوساً", muscleGroup: "الأكتاف", equipment: "دمبلز، بنش بمسند", instructions: ["اجلس على بنش بمسند مستقيم.","أمسك دمبل في كل يد عند مستوى الكتفين.","ادفع الدمبلز للأعلى.","أنزل الدمبلز ببطء."], notes: ["حافظ على ظهرك مسطحاً.","تحكم في الحركة."] },
            { id: "lib_leg_press", name: "دفع الأرجل بالجهاز (Leg Press)", muscleGroup: "الأرجل", equipment: "جهاز دفع الأرجل", instructions: ["اجلس على الجهاز وضع قدميك على المنصة بعرض الكتفين.","حرر أقفال الأمان.","أنزل الوزن ببطء عن طريق ثني ركبتيك.","ادفع الوزن للأعلى حتى تمتد ساقاك (دون قفل الركبتين بالكامل)."], notes: ["لا تدع أسفل ظهرك يرتفع عن المقعد.","يمكن تغيير وضع القدمين لاستهداف عضلات مختلفة."] },
            { id: "lib_tricep_dips_bench", name: "غطس الترايسبس على البنش (Bench Dips)", muscleGroup: "الترايسبس", equipment: "بنش أو كرسي ثابت", instructions: ["ضع يديك على حافة البنش خلفك، بعرض الكتفين.","مد ساقيك أمامك (أو اثنهما لتسهيل التمرين).","أنزل جسمك ببطء عن طريق ثني مرفقيك حتى يشكل مرفقاك زاوية 90 درجة تقريباً.","ادفع جسمك للأعلى للعودة لوضع البداية."], notes: ["حافظ على ظهرك قريباً من البنش.","ركز على عمل الترايسبس."] }
        ];
        // --- END DATA ---

        // --- DOM ELEMENTS ---
        const userProfileModalOverlay = document.getElementById('userProfileModalOverlay');
        const profileSelect = document.getElementById('profileSelect');
        const selectProfileBtn = document.getElementById('selectProfileBtn');
        const newProfileNameInput = document.getElementById('newProfileName');
        const createProfileBtn = document.getElementById('createProfileBtn');
        const closeProfileModalBtn = document.getElementById('closeProfileModalBtn');
        const currentUserNameDisplay = document.getElementById('currentUserName');
        const switchProfileBtn = document.getElementById('switchProfileBtn');

        const mainNavigation = document.getElementById('mainNavigation');
        const dayNavigationContainer = document.getElementById('dayNavigationContainer');
        const planControlsContainer = document.getElementById('planControlsContainer');
        const currentWeekInput = document.getElementById('currentWeek');
        const appContent = document.getElementById('appContent');
        // --- END DOM ELEMENTS ---

        // --- APP STATE ---
        const dayNamesArabic = { sat: "السبت", sun: "الأحد", mon: "الاثنين", tue: "الثلاثاء", wed: "الأربعاء", thu: "الخميس", fri: "الجمعة" };
        let currentSection = 'plan'; 
        let currentDayId = initialWorkoutData[0].dayId;
        let userProfiles = [];
        let currentUser = null; // { id: 'uuid', name: 'username' }
        let userWeeklyWeights = {}; 
        let userBodyWeightHistory = []; 
        let userBodyWeightTarget = null;
        let exercisePerformanceChart = null;
        let bodyWeightChart = null;
        // --- END APP STATE ---
        
        // --- USER PROFILE FUNCTIONS ---
        function generateUUID() { return crypto.randomUUID(); }

        function loadUserProfiles() {
            const storedProfiles = localStorage.getItem('workoutApp_userProfiles');
            userProfiles = storedProfiles ? JSON.parse(storedProfiles) : [];
            populateProfileSelect();
        }

        function saveUserProfiles() {
            localStorage.setItem('workoutApp_userProfiles', JSON.stringify(userProfiles));
        }

        function setActiveUser(userId) {
            const user = userProfiles.find(p => p.id === userId);
            if (user) {
                currentUser = user;
                localStorage.setItem('workoutApp_activeUserId', userId);
                currentUserNameDisplay.textContent = `مرحباً, ${currentUser.name}!`;
                userProfileModalOverlay.classList.add('hidden');
                loadUserDataForCurrentUser();
                updateAppDisplay(); // Refresh the entire app view for the new user
            }
        }
        
        function createNewProfile() {
            const name = newProfileNameInput.value.trim();
            if (!name) { alert("الرجاء إدخال اسم للملف الشخصي."); return; }
            if (userProfiles.find(p => p.name.toLowerCase() === name.toLowerCase())) {
                alert("اسم الملف الشخصي هذا موجود بالفعل."); return;
            }
            const newUser = { id: generateUUID(), name: name };
            userProfiles.push(newUser);
            saveUserProfiles();
            populateProfileSelect();
            setActiveUser(newUser.id);
            newProfileNameInput.value = '';
        }

        function populateProfileSelect() {
            profileSelect.innerHTML = '<option value="">-- اختر --</option>';
            userProfiles.forEach(profile => {
                profileSelect.innerHTML += `<option value="${profile.id}">${profile.name}</option>`;
            });
        }
        
        function handleProfileSelection() {
            const selectedId = profileSelect.value;
            if (selectedId) setActiveUser(selectedId);
        }

        function showProfileModal() { userProfileModalOverlay.classList.remove('hidden'); }
        function hideProfileModal() { 
            if (currentUser) { // Only allow closing if a user is active
                userProfileModalOverlay.classList.add('hidden'); 
            } else {
                alert("الرجاء اختيار أو إنشاء ملف تعريف للمتابعة.");
            }
        }
        // --- END USER PROFILE FUNCTIONS ---

        // --- DATA HANDLING (USER-SPECIFIC) ---
        function loadUserDataForCurrentUser() {
            if (!currentUser) return;
            const weeklyWeightsKey = `${currentUser.id}_userWeeklyWorkoutWeights`;
            const bodyWeightKey = `${currentUser.id}_userBodyWeightHistory`;
            const bodyWeightTargetKey = `${currentUser.id}_userBodyWeightTarget`;

            const storedWeeklyWeights = localStorage.getItem(weeklyWeightsKey);
            userWeeklyWeights = storedWeeklyWeights ? JSON.parse(storedWeeklyWeights) : {};
            
            const storedBodyWeights = localStorage.getItem(bodyWeightKey);
            userBodyWeightHistory = storedBodyWeights ? JSON.parse(storedBodyWeights) : [];
            
            const storedTarget = localStorage.getItem(bodyWeightTargetKey);
            userBodyWeightTarget = storedTarget ? parseFloat(storedTarget) : null;
        }

        function saveWeeklyWeight(exerciseId, week, weight) {
            if (!currentUser) return;
            if (!userWeeklyWeights[exerciseId]) userWeeklyWeights[exerciseId] = {};
            if (weight === "" || weight === null || isNaN(parseFloat(weight))) {
                delete userWeeklyWeights[exerciseId][`week${week}`];
            } else {
                userWeeklyWeights[exerciseId][`week${week}`] = parseFloat(weight);
            }
            localStorage.setItem(`${currentUser.id}_userWeeklyWorkoutWeights`, JSON.stringify(userWeeklyWeights));
        }

        function getWeeklyWeight(exerciseId, week) { // No change needed, uses global userWeeklyWeights
            return userWeeklyWeights[exerciseId] && userWeeklyWeights[exerciseId][`week${week}`] !== undefined 
                   ? userWeeklyWeights[exerciseId][`week${week}`] 
                   : "";
        }

        function saveBodyWeightEntry(date, weight) {
            if (!currentUser) return;
            if (!date || isNaN(parseFloat(weight)) || parseFloat(weight) <= 0) {
                alert("الرجاء إدخال تاريخ ووزن صحيحين."); return;
            }
            const existingEntryIndex = userBodyWeightHistory.findIndex(entry => entry.date === date);
            if (existingEntryIndex > -1) {
                userBodyWeightHistory[existingEntryIndex].weight = parseFloat(weight);
            } else {
                 userBodyWeightHistory.push({ date: date, weight: parseFloat(weight), id: Date.now() });
            }
            userBodyWeightHistory.sort((a, b) => new Date(a.date) - new Date(b.date)); 
            localStorage.setItem(`${currentUser.id}_userBodyWeightHistory`, JSON.stringify(userBodyWeightHistory));
            displayBodyWeightAnalysis(); 
        }
        
        function deleteBodyWeightEntry(entryId) {
            if (!currentUser) return;
            userBodyWeightHistory = userBodyWeightHistory.filter(entry => entry.id !== entryId);
            localStorage.setItem(`${currentUser.id}_userBodyWeightHistory`, JSON.stringify(userBodyWeightHistory));
            displayBodyWeightAnalysis(); 
        }

        function saveBodyWeightTarget(targetWeight) {
            if (!currentUser) return;
            if (targetWeight === "" || targetWeight === null || isNaN(parseFloat(targetWeight)) || parseFloat(targetWeight) <=0) {
                userBodyWeightTarget = null;
                localStorage.removeItem(`${currentUser.id}_userBodyWeightTarget`);
            } else {
                userBodyWeightTarget = parseFloat(targetWeight);
                localStorage.setItem(`${currentUser.id}_userBodyWeightTarget`, userBodyWeightTarget.toString());
            }
            if (currentSection === 'bodyWeightAnalysis') displayBodyWeightAnalysis(); // Refresh if on the page
        }
        // --- END DATA HANDLING (USER-SPECIFIC) ---

        // --- UI RENDERING FUNCTIONS ---
        function renderDayNavigation() {
            dayNavigationContainer.innerHTML = ''; 
            if (currentSection !== 'plan' || !currentUser) {
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
        
        function calculate1RMEpley(weight, reps) {
            if (reps == 1) return weight; // Already 1RM
            if (reps <= 0 || weight <= 0) return null;
            return (weight * (1 + reps / 30)).toFixed(1);
        }

        function buildExercisesTable(exercisesArray) {
            if (!exercisesArray || exercisesArray.length === 0) return '<p class="text-slate-500">لا توجد تمارين لهذا اليوم.</p>';
            const selectedWeek = parseInt(currentWeekInput.value) || 1;
            
            let tableHtml = '<div class="overflow-x-auto mt-4"><table class="table w-full min-w-[850px] text-sm"><thead><tr>';
            tableHtml += '<th class="w-1/3">التمرين</th><th>المجموعات</th><th>التكرارات</th><th>الراحة</th>';
            tableHtml += `<th class="w-1/6">الوزن للأسبوع ${selectedWeek} (كغم)</th><th class="w-1/6">تقدير 1RM (كغم)</th><th class="w-1/3">ملاحظات</th></tr></thead><tbody>`;

            exercisesArray.forEach((ex) => {
                if (ex.type === 'title') {
                    tableHtml += `<tr><td colspan="7" class="bg-sky-100 text-sky-800 font-bold text-md py-3">${ex.sectionTitle}</td></tr>`;
                } else {
                    const weightValue = getWeeklyWeight(ex.id, selectedWeek);
                    const repsFor1RM = parseInt(ex.reps.split('-')[0]); // Take the lower bound of reps for 1RM
                    const estimated1RM = weightValue && repsFor1RM ? calculate1RMEpley(parseFloat(weightValue), repsFor1RM) : "--";
                    
                    tableHtml += `<tr><td>${ex.name}</td><td>${ex.sets}</td><td>${ex.reps}</td><td>${ex.rest}</td>`;
                    tableHtml += `<td><input type="number" step="0.25" class="weight-input input-field" value="${weightValue}" placeholder="--" 
                                        onchange="saveWeeklyWeight('${ex.id}', ${selectedWeek}, this.value); displayWorkoutPlan(currentDayId);"></td>`; // Refresh to update 1RM
                    tableHtml += `<td>${estimated1RM}</td>`;
                    tableHtml += `<td class="text-xs text-slate-500">${ex.notes}</td></tr>`;
                }
            });
            tableHtml += '</tbody></table></div>';
            return tableHtml;
        }

        function displayWorkoutPlan(dayId) {
            if (!currentUser) { appContent.innerHTML = '<p class="text-center p-10">الرجاء اختيار أو إنشاء ملف تعريف أولاً.</p>'; return; }
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
                                <p class="text-sm text-slate-500 mb-3">سجل أوزانك للأسبوع المحدد. سيتم الحفظ تلقائياً وعرض تقدير الـ 1RM.</p>
                                ${buildExercisesTable(dayData.exercises)}`;
            }
            contentHtml += `</div>`;
            appContent.innerHTML = contentHtml;
        }

        function displayExerciseLibrary() {
            if (!currentUser) { appContent.innerHTML = '<p class="text-center p-10">الرجاء اختيار أو إنشاء ملف تعريف أولاً.</p>'; return; }
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

            renderExerciseLibraryCards('all'); 

            filterSelect.addEventListener('change', (e) => {
                renderExerciseLibraryCards(e.target.value);
            });
        }

        function renderExerciseLibraryCards(selectedGroup) {
            const container = document.getElementById('exerciseListContainer');
            container.innerHTML = ''; 
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
            if (!currentUser) { appContent.innerHTML = '<p class="text-center p-10">الرجاء اختيار أو إنشاء ملف تعريف أولاً.</p>'; return; }
            let contentHtml = `<div class="content-card animate-fadeIn">
                <h2 class="text-3xl font-bold mb-6 text-sky-700"><span class="section-title-icon">📊</span>تحليل أداء الأوزان</h2>
                <p class="text-lg mb-6 text-slate-600">اختر تمريناً لعرض تطور الأوزان وتقدير الـ 1RM.</p>
                <div class="mb-6">
                    <label for="exerciseAnalysisSelect" class="block text-sm font-medium text-slate-700 mb-1">اختر التمرين:</label>
                    <select id="exerciseAnalysisSelect" class="input-field mt-1 block w-full pl-3 pr-10 py-2 text-base sm:text-sm rounded-md shadow-sm">
                        <option value="">-- اختر تمريناً --</option>`;
            const allExercises = [];
            initialWorkoutData.forEach(day => {
                if (day.exercises) {
                    day.exercises.forEach(ex => {
                        if (ex.type !== 'title' && !allExercises.find(e => e.id === ex.id)) {
                            allExercises.push({id: ex.id, name: ex.name, day: dayNamesArabic[day.dayId], repsFor1RM: parseInt(ex.reps.split('-')[0])});
                        }
                    });
                }
            });
            allExercises.sort((a,b) => a.name.localeCompare(b.name));
            allExercises.forEach(ex => { contentHtml += `<option value="${ex.id}" data-reps="${ex.repsFor1RM}">${ex.name} (${ex.day})</option>`; });
            contentHtml += `</select></div>
                <div id="exerciseChartContainerWrapper" class="chart-container mb-8 hidden"><canvas id="exercisePerformanceChartEl"></canvas></div>
                <div id="exercisePerformanceTableContainer" class="overflow-x-auto hidden"></div>
                <div id="oneRmEstimateDisplay" class="mt-6 p-4 bg-sky-50 rounded-lg text-center hidden">
                    <h4 class="text-lg font-semibold text-sky-700">تقدير أقصى وزن لمرة واحدة (1RM) لآخر تسجيل:</h4>
                    <p id="oneRmValue" class="text-2xl font-bold text-sky-600 mt-1"></p>
                </div>
            </div>`;
            appContent.innerHTML = contentHtml;

            document.getElementById('exerciseAnalysisSelect').addEventListener('change', function() {
                const selectedExerciseId = this.value;
                const selectedOption = this.options[this.selectedIndex];
                const repsFor1RM = selectedOption.dataset.reps ? parseInt(selectedOption.dataset.reps) : null;

                if (selectedExerciseId) {
                    renderExercisePerformanceChartAndTable(selectedExerciseId, repsFor1RM);
                } else {
                    if (exercisePerformanceChart) exercisePerformanceChart.destroy();
                    document.getElementById('exerciseChartContainerWrapper').classList.add('hidden');
                    document.getElementById('exercisePerformanceTableContainer').classList.add('hidden').innerHTML = '';
                    document.getElementById('oneRmEstimateDisplay').classList.add('hidden');
                }
            });
        }

        function renderExercisePerformanceChartAndTable(exerciseId, repsFor1RM) {
            const exerciseData = userWeeklyWeights[exerciseId];
            const chartWrapper = document.getElementById('exerciseChartContainerWrapper');
            const tableContainer = document.getElementById('exercisePerformanceTableContainer');
            const oneRmDisplay = document.getElementById('oneRmEstimateDisplay');
            const oneRmValueEl = document.getElementById('oneRmValue');
            
            if (!exerciseData || Object.keys(exerciseData).length === 0) {
                chartWrapper.classList.add('hidden');
                tableContainer.innerHTML = '<p class="text-slate-500 text-center py-4">لا توجد بيانات أوزان مسجلة لهذا التمرين.</p>';
                tableContainer.classList.remove('hidden');
                oneRmDisplay.classList.add('hidden');
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

            // 1RM Calculation for the latest week
            if (weights.length > 0 && repsFor1RM) {
                const latestWeight = weights[weights.length - 1];
                const estimated1RM = calculate1RMEpley(latestWeight, repsFor1RM);
                if (estimated1RM) {
                    oneRmValueEl.textContent = `${estimated1RM} كغم`;
                    oneRmDisplay.classList.remove('hidden');
                } else {
                    oneRmDisplay.classList.add('hidden');
                }
            } else {
                oneRmDisplay.classList.add('hidden');
            }
        }

        function displayBodyWeightAnalysis() {
            if (!currentUser) { appContent.innerHTML = '<p class="text-center p-10">الرجاء اختيار أو إنشاء ملف تعريف أولاً.</p>'; return; }
            let contentHtml = `<div class="content-card animate-fadeIn">
                <h2 class="text-3xl font-bold mb-6 text-sky-700"><span class="section-title-icon">⚖️</span>تحليل وزن الجسم وتتبع التقدم</h2>
                <p class="text-lg mb-6 text-slate-600">سجل وزن جسمك بانتظام وتتبع تقدمك نحو أهدافك الصحية.</p>
                
                <form id="bodyWeightForm" class="mb-8 p-6 border border-sky-100 rounded-lg bg-sky-50 grid grid-cols-1 md:grid-cols-4 gap-4 items-end shadow-sm">
                    <div>
                        <label for="bodyWeightDate" class="block text-sm font-medium text-slate-700 mb-1">التاريخ:</label>
                        <input type="date" id="bodyWeightDate" required class="date-input input-field w-full">
                    </div>
                    <div>
                        <label for="bodyWeightInputVal" class="block text-sm font-medium text-slate-700 mb-1">وزن الجسم (كغم):</label>
                        <input type="number" step="0.1" id="bodyWeightInputVal" required placeholder="مثال: 75.5" class="body-weight-input input-field w-full">
                    </div>
                    <div>
                        <label for="bodyWeightTargetInput" class="block text-sm font-medium text-slate-700 mb-1">الوزن المستهدف (كغم):</label>
                        <input type="number" step="0.1" id="bodyWeightTargetInput" placeholder="مثال: 70" class="body-weight-input input-field w-full" value="${userBodyWeightTarget || ''}">
                    </div>
                    <button type="submit" class="submit-btn h-fit w-full">حفظ الوزن والهدف</button>
                </form>

                <div id="bodyWeightProgressDisplay" class="mb-6 p-4 bg-emerald-50 rounded-lg text-emerald-700 text-center hidden"></div>

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
                const weight = document.getElementById('bodyWeightInputVal').value;
                const targetWeight = document.getElementById('bodyWeightTargetInput').value;
                
                if (date && weight) { // Only save body weight if date and weight are provided
                    saveBodyWeightEntry(date, weight);
                }
                saveBodyWeightTarget(targetWeight); // Save target weight regardless
                
                // Optionally clear only date and weight after submission, keep target
                document.getElementById('bodyWeightDate').value = '';
                document.getElementById('bodyWeightInputVal').value = '';
            });
        }

        function renderBodyWeightChartAndTable() {
            const chartWrapper = document.getElementById('bodyWeightChartContainerWrapper');
            const tableContainer = document.getElementById('bodyWeightTableContainer');
            const progressDisplay = document.getElementById('bodyWeightProgressDisplay');

            if (!userBodyWeightHistory || userBodyWeightHistory.length === 0) {
                if(chartWrapper) chartWrapper.classList.add('hidden');
                if(tableContainer) tableContainer.innerHTML = '<p class="text-slate-500 text-center py-4">لا توجد بيانات وزن جسم مسجلة حتى الآن. ابدأ بإضافة إدخال جديد.</p>';
                if(progressDisplay) progressDisplay.classList.add('hidden');
                if (bodyWeightChart) { bodyWeightChart.destroy(); bodyWeightChart = null; }
                return;
            }
            if(chartWrapper) chartWrapper.classList.remove('hidden');
            
            userBodyWeightHistory.sort((a, b) => new Date(a.date) - new Date(b.date)); 

            const chartDataPoints = userBodyWeightHistory.map(entry => {
                const parts = entry.date.split('-');
                return { x: new Date(parseInt(parts[0]), parseInt(parts[1]) - 1, parseInt(parts[2])), y: entry.weight };
            });

            const datasets = [{
                label: 'وزن الجسم (كغم)', data: chartDataPoints, borderColor: '#0284c7', tension: 0.1, fill: true,
                backgroundColor: 'rgba(2, 132, 199, 0.1)', pointBackgroundColor: '#0369a1'
            }];

            if (userBodyWeightTarget !== null) {
                datasets.push({
                    label: 'الوزن المستهدف (كغم)',
                    data: chartDataPoints.map(p => ({ x: p.x, y: userBodyWeightTarget })),
                    borderColor: '#10b981', // emerald-500
                    borderDash: [5, 5],
                    pointRadius: 0,
                    fill: false,
                    tension: 0
                });
                const latestWeight = userBodyWeightHistory[userBodyWeightHistory.length - 1].weight;
                const diff = userBodyWeightTarget - latestWeight;
                if (progressDisplay) {
                    if (diff > 0) {
                        progressDisplay.innerHTML = `أنت بحاجة لزيادة <strong class="font-bold">${diff.toFixed(1)} كغم</strong> للوصول إلى هدفك.`;
                    } else if (diff < 0) {
                        progressDisplay.innerHTML = `أنت بحاجة لخسارة <strong class="font-bold">${Math.abs(diff).toFixed(1)} كغم</strong> للوصول إلى هدفك.`;
                    } else {
                        progressDisplay.innerHTML = `<strong class="font-bold">🎉 لقد وصلت إلى وزنك المستهدف!</strong>`;
                    }
                    progressDisplay.classList.remove('hidden');
                }
            } else {
                 if(progressDisplay) progressDisplay.classList.add('hidden');
            }


            if (bodyWeightChart) bodyWeightChart.destroy();
            const canvasEl = document.getElementById('bodyWeightChartEl');
            if (!canvasEl) return; 
            const ctx = canvasEl.getContext('2d');
            
            bodyWeightChart = new Chart(ctx, {
                type: 'line', data: { datasets },
                options: { responsive: true, maintainAspectRatio: false,
                    scales: {
                        y: { beginAtZero: false, title: { display: true, text: 'وزن الجسم (كغم)', font: { family: "'Cairo', sans-serif" } }, ticks: { font: { family: "'Cairo', sans-serif" } } },
                        x: { type: 'time', title: { display: true, text: 'التاريخ', font: { family: "'Cairo', sans-serif" } }, time: { unit: 'day', displayFormats: { day: 'MMM d, yy' } }, ticks: { font: { family: "'Cairo', sans-serif" }, source: 'auto', major: { enabled: true } } }
                    },
                    plugins: { legend: { labels: { font: { family: "'Cairo', sans-serif" } } },
                        tooltip: { bodyFont: { family: "'Cairo', sans-serif" }, titleFont: { family: "'Cairo', sans-serif" },
                            callbacks: { 
                                title: (tooltipItems) => { if (tooltipItems && tooltipItems[0] && tooltipItems[0].parsed) { return dateFns.format(new Date(tooltipItems[0].parsed.x), 'PPP', { locale: dateFns.locale.arSA || dateFns.locale.enUS }); } return ''; },
                                label: function(context) { return `${context.dataset.label}: ${context.parsed.y.toFixed(1)} كغم`; }
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
        
        function updateAppDisplay() { // Central function to refresh view based on current state
            if (!currentUser) {
                showProfileModal();
                appContent.innerHTML = '<p class="text-center p-10 text-lg text-slate-500">الرجاء اختيار أو إنشاء ملف تعريف للمتابعة.</p>';
                planControlsContainer.classList.add('hidden');
                return;
            }

            if (currentSection === 'plan') {
                renderDayNavigation();
                displayWorkoutPlan(currentDayId); 
            } else {
                planControlsContainer.classList.add('hidden');
                if (currentSection === 'library') displayExerciseLibrary();
                else if (currentSection === 'exerciseAnalysis') displayExercisePerformanceAnalysis();
                else if (currentSection === 'bodyWeightAnalysis') displayBodyWeightAnalysis();
            }
             setActiveMainTab(currentSection);
        }


        mainNavigation.addEventListener('click', (event) => {
            if (event.target.tagName === 'BUTTON') {
                const section = event.target.dataset.section;
                if (section && section !== currentSection) {
                    currentSection = section;
                    if (exercisePerformanceChart) { exercisePerformanceChart.destroy(); exercisePerformanceChart = null; }
                    if (bodyWeightChart) { bodyWeightChart.destroy(); bodyWeightChart = null; }
                    updateAppDisplay();
                }
            }
        });
        
        currentWeekInput.addEventListener('change', () => {
            if (currentSection === 'plan') displayWorkoutPlan(currentDayId);
        });
        currentWeekInput.addEventListener('input', () => { 
             if (currentSection === 'plan') displayWorkoutPlan(currentDayId);
        });

        // Profile Modal Listeners
        selectProfileBtn.addEventListener('click', handleProfileSelection);
        createProfileBtn.addEventListener('click', createNewProfile);
        switchProfileBtn.addEventListener('click', showProfileModal);
        closeProfileModalBtn.addEventListener('click', hideProfileModal);


        document.addEventListener('DOMContentLoaded', () => {
            loadUserProfiles();
            const lastActiveUserId = localStorage.getItem('workoutApp_activeUserId');
            if (lastActiveUserId && userProfiles.find(p => p.id === lastActiveUserId)) {
                setActiveUser(lastActiveUserId); // This calls loadUserDataForCurrentUser and updateAppDisplay
            } else {
                showProfileModal();
                updateAppDisplay(); // Will show the prompt if no user
            }
        });

        const styleSheet = document.createElement("style");
        styleSheet.type = "text/css";
        styleSheet.innerText = `
            @keyframes fadeIn { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
            .animate-fadeIn { animation: fadeIn 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94) forwards; }
        `;
        document.head.appendChild(styleSheet);

    </script>
</body>
</html>
