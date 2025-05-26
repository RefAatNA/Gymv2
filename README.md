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
        :root {
            --bg-primary: #fffbeb; /* amber-50 */
            --bg-secondary: #ffffff;
            --bg-accent: #f0f9ff; /* sky-50 */
            --bg-accent-hover: #e0f2fe; /* sky-100 */
            --text-primary: #334155; /* slate-700 */
            --text-secondary: #1e293b; /* slate-800 */
            --text-accent: #0369a1; /* sky-700 */
            --text-accent-hover: #075985; /* sky-800 */
            --border-primary: #e2e8f0; /* slate-200 */
            --border-accent: #e0f2fe; /* sky-100 */
            --border-accent-hover: #7dd3fc; /* sky-300 */
            --brand-primary: #0284c7; /* sky-600 */
            --brand-secondary: #0ea5e9; /* sky-500 */
            --brand-text: #f0f9ff; /* sky-50 */
            --danger-primary: #f43f5e; /* rose-500 */
            --danger-hover: #e11d48; /* rose-600 */
            --shadow-color-light: rgba(0,0,0,0.05);
            --shadow-color-medium: rgba(0,0,0,0.03);
            --shadow-color-brand: rgba(2, 132, 199, 0.25);
        }

        .dark {
            --bg-primary: #1e293b; /* slate-800 */
            --bg-secondary: #334155; /* slate-700 */
            --bg-accent: #0f172a; /* slate-900 */
            --bg-accent-hover: #1e293b; /* slate-800 */
            --text-primary: #cbd5e1; /* slate-300 */
            --text-secondary: #f1f5f9; /* slate-100 */
            --text-accent: #7dd3fc; /* sky-300 */
            --text-accent-hover: #e0f2fe; /* sky-100 */
            --border-primary: #475569; /* slate-600 */
            --border-accent: #334155; /* slate-700 */
            --border-accent-hover: #0ea5e9; /* sky-500 */
            --brand-primary: #38bdf8; /* sky-400 */
            --brand-secondary: #7dd3fc; /* sky-300 */
            --brand-text: #0f172a; /* slate-900 */
            --danger-primary: #fb7185; /* rose-400 */
            --danger-hover: #f43f5e; /* rose-500 */
            --shadow-color-light: rgba(255,255,255,0.05);
            --shadow-color-medium: rgba(255,255,255,0.03);
            --shadow-color-brand: rgba(56, 189, 248, 0.25);
        }

        body {
            font-family: 'Cairo', 'Inter', sans-serif;
            background-color: var(--bg-primary);
            color: var(--text-primary);
            transition: background-color 0.3s ease, color 0.3s ease;
        }
        .main-nav-button, .day-nav-button {
            transition: background-color 0.3s ease, color 0.3s ease, box-shadow 0.3s ease;
            border: 1px solid transparent;
        }
        .main-nav-button {
            background-color: var(--bg-secondary);
            color: var(--text-accent);
            border-color: var(--border-accent);
        }
        .main-nav-button:hover {
            background-color: var(--bg-accent);
            border-color: var(--border-accent-hover);
        }
        .main-nav-button.active, .day-nav-button.active {
            background-color: var(--brand-primary);
            color: var(--brand-text);
            border-color: var(--brand-primary);
            box-shadow: 0 4px 14px var(--shadow-color-brand);
        }
        .day-nav-button {
             background-color: var(--bg-accent);
             color: var(--text-accent-hover);
             border-color: var(--border-accent);
        }
         .day-nav-button:hover {
            background-color: var(--bg-accent-hover);
        }

        .content-card {
            background-color: var(--bg-secondary); 
            border-radius: 0.75rem; /* rounded-xl */
            padding: 1.5rem; /* p-6 */
            box-shadow: 0 8px 16px var(--shadow-color-light), 0 3px 6px var(--shadow-color-medium);
            margin-top: 1.5rem; /* mt-6 */
            transition: background-color 0.3s ease, box-shadow 0.3s ease;
        }
        .table th, .table td {
            border: 1px solid var(--border-primary);
            padding: 0.75rem 1rem; 
            text-align: right;
            vertical-align: middle;
            transition: border-color 0.3s ease, background-color 0.3s ease;
        }
        .table th {
            background-color: var(--bg-accent);
            color: var(--text-secondary);
            font-weight: 600; 
        }
        .input-field {
            padding: 0.625rem 0.75rem; /* py-2.5 px-3 */
            border: 1px solid var(--border-primary);
            border-radius: 0.375rem; /* rounded-md */
            text-align: center;
            transition: border-color 0.2s ease, box-shadow 0.2s ease, background-color 0.3s ease, color 0.3s ease;
            background-color: var(--bg-secondary);
            color: var(--text-primary);
        }
        .input-field:focus {
            border-color: var(--brand-primary);
            box-shadow: 0 0 0 2px var(--shadow-color-brand);
            outline: none;
        }
        .weight-input, .week-input { width: 7rem; }
        .body-weight-input { width: 9rem; }
        .date-input { width: 11rem; }

        .exercise-card {
            background-color: var(--bg-accent);
            border: 1px solid var(--border-accent);
            border-radius: 0.5rem; 
            padding: 1.25rem; 
            margin-bottom: 1.25rem; 
            box-shadow: 0 4px 8px var(--shadow-color-light);
            transition: background-color 0.3s ease, border-color 0.3s ease, box-shadow 0.3s ease;
        }
        .exercise-card:hover {
            border-color: var(--border-accent-hover);
            box-shadow: 0 6px 12px var(--shadow-color-light);
        }
        .chart-container { 
            position: relative; 
            width: 100%; 
            max-width: 700px; 
            margin-left: auto; 
            margin-right: auto; 
            height: 350px; 
            max-height: 450px; 
            padding: 1.5rem; 
            background-color: var(--bg-secondary); 
            border-radius: 0.75rem; 
            box-shadow: 0 6px 12px var(--shadow-color-light);
            transition: background-color 0.3s ease, box-shadow 0.3s ease;
        }
        @media (min-width: 768px) { .chart-container { height: 400px; } }
        .hidden { display: none !important; }
        .delete-btn {
            background-color: var(--danger-primary);
            color: white;
            padding: 0.375rem 0.75rem;
            border-radius: 0.375rem;
            font-size: 0.875rem;
            cursor: pointer;
            transition: background-color 0.2s ease;
        }
        .delete-btn:hover {
            background-color: var(--danger-hover);
        }
        .submit-btn {
            background-color: var(--brand-secondary);
            color: white;
            font-weight: 600;
            padding: 0.625rem 1.25rem;
            border-radius: 0.375rem;
            transition: background-color 0.2s ease;
        }
        .submit-btn:hover {
            background-color: var(--brand-primary);
        }
        .section-title-icon {
            margin-left: 0.5rem; /* ml-2 */
            font-size: 1.2em; /* slightly larger icon */
            color: var(--brand-secondary);
        }
        
        /* New UI Components */
        .theme-toggle {
            position: fixed;
            top: 1rem;
            left: 1rem;
            z-index: 50;
            background-color: var(--bg-secondary);
            color: var(--text-accent);
            border: 1px solid var(--border-accent);
            border-radius: 9999px;
            width: 2.5rem;
            height: 2.5rem;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            box-shadow: 0 2px 8px var(--shadow-color-light);
            transition: all 0.3s ease;
        }
        .theme-toggle:hover {
            background-color: var(--bg-accent);
            transform: scale(1.05);
        }
        
        .badge {
            display: inline-flex;
            align-items: center;
            padding: 0.25rem 0.75rem;
            border-radius: 9999px;
            font-size: 0.75rem;
            font-weight: 600;
            margin-right: 0.5rem;
        }
        .badge-primary {
            background-color: var(--brand-primary);
            color: var(--brand-text);
        }
        .badge-secondary {
            background-color: var(--bg-accent);
            color: var(--text-accent);
            border: 1px solid var(--border-accent);
        }
        
        .progress-container {
            width: 100%;
            height: 0.5rem;
            background-color: var(--bg-accent);
            border-radius: 9999px;
            overflow: hidden;
            margin: 0.5rem 0;
        }
        .progress-bar {
            height: 100%;
            background-color: var(--brand-primary);
            border-radius: 9999px;
            transition: width 0.5s ease;
        }
        
        .tooltip {
            position: relative;
            display: inline-block;
        }
        .tooltip .tooltip-text {
            visibility: hidden;
            background-color: var(--bg-secondary);
            color: var(--text-primary);
            text-align: center;
            border-radius: 0.375rem;
            padding: 0.5rem;
            position: absolute;
            z-index: 1;
            bottom: 125%;
            left: 50%;
            transform: translateX(-50%);
            opacity: 0;
            transition: opacity 0.3s;
            box-shadow: 0 2px 8px var(--shadow-color-light);
            border: 1px solid var(--border-primary);
            width: max-content;
            max-width: 250px;
        }
        .tooltip:hover .tooltip-text {
            visibility: visible;
            opacity: 1;
        }
        
        .animate-fadeIn {
            animation: fadeIn 0.5s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body>
    <!-- Theme Toggle Button -->
    <button id="themeToggle" class="theme-toggle" aria-label="تبديل وضع الإضاءة">
        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="sun-icon">
            <circle cx="12" cy="12" r="5"></circle>
            <line x1="12" y1="1" x2="12" y2="3"></line>
            <line x1="12" y1="21" x2="12" y2="23"></line>
            <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
            <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
            <line x1="1" y1="12" x2="3" y2="12"></line>
            <line x1="21" y1="12" x2="23" y2="12"></line>
            <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line>
            <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
        </svg>
        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="moon-icon hidden">
            <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
        </svg>
    </button>

    <div class="container mx-auto p-4 sm:p-6 lg:p-8 max-w-7xl">
        <header class="text-center mb-10">
            <h1 class="text-4xl sm:text-5xl font-bold text-sky-700">الخطة التدريبية الشاملة</h1>
            <p class="mt-3 text-lg text-slate-600">خطتك التدريبية، تسجيل الأوزان، تحليل الأداء، ومكتبة التمارين.</p>
        </header>

        <!-- Mobile Menu Button (visible on small screens) -->
        <div class="md:hidden flex justify-center mb-4">
            <button id="mobileMenuButton" class="px-4 py-2 bg-sky-500 text-white rounded-lg shadow-sm flex items-center">
                <span>القائمة</span>
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="mr-2">
                    <line x1="3" y1="12" x2="21" y2="12"></line>
                    <line x1="3" y1="6" x2="21" y2="6"></line>
                    <line x1="3" y1="18" x2="21" y2="18"></line>
                </svg>
            </button>
        </div>

        <nav class="flex flex-wrap justify-center gap-3 mb-6 md:flex hidden" id="mainNavigation">
            <button data-section="plan" class="main-nav-button active px-4 py-2.5 sm:px-6 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">الخطة الأسبوعية</button>
            <button data-section="library" class="main-nav-button px-4 py-2.5 sm:px-6 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">مكتبة التمارين</button>
            <button data-section="exerciseAnalysis" class="main-nav-button px-4 py-2.5 sm:px-6 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">تحليل أداء الأوزان</button>
            <button data-section="bodyWeightAnalysis" class="main-nav-button px-4 py-2.5 sm:px-6 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">تحليل وزن الجسم</button>
            <button data-section="goals" class="main-nav-button px-4 py-2.5 sm:px-6 sm:py-3 text-sm sm:text-base font-semibold rounded-lg shadow-sm">الأهداف والإنجازات</button>
        </nav>
        
        <div id="planControlsContainer" class="mb-6 hidden">
            <div class="flex flex-col sm:flex-row justify-center items-center gap-4 p-4 rounded-lg shadow-sm" style="background-color: var(--bg-accent);">
                <div class="flex items-center gap-2">
                    <label for="currentWeek" class="text-sm font-medium">الأسبوع الحالي لتسجيل الأوزان:</label>
                    <input type="number" id="currentWeek" value="1" min="1" class="week-input input-field">
                </div>
                <div class="flex items-center gap-2 mt-3 sm:mt-0">
                    <button id="prevWeekBtn" class="px-3 py-1.5 rounded-lg" style="background-color: var(--bg-secondary); color: var(--text-accent); border: 1px solid var(--border-accent);">
                        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <polyline points="15 18 9 12 15 6"></polyline>
                        </svg>
                    </button>
                    <span class="text-sm font-medium">الأسبوع</span>
                    <button id="nextWeekBtn" class="px-3 py-1.5 rounded-lg" style="background-color: var(--bg-secondary); color: var(--text-accent); border: 1px solid var(--border-accent);">
                        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                            <polyline points="9 18 15 12 9 6"></polyline>
                        </svg>
                    </button>
                </div>
            </div>
            <nav class="flex flex-wrap justify-center gap-2 mt-4" id="dayNavigationContainer">
                </nav>
        </div>


        <main id="appContent" class="min-h-[300px]">
            </main>

        <!-- Goals Section Template (Hidden by default) -->
        <div id="goalsTemplate" class="hidden">
            <div class="content-card animate-fadeIn">
                <h2 class="text-3xl font-bold mb-6 text-sky-700"><span class="section-title-icon">🎯</span>الأهداف والإنجازات</h2>
                <p class="text-lg mb-6 text-slate-600">حدد أهدافك وتتبع إنجازاتك لتحفيز نفسك على الاستمرار.</p>
                
                <div class="mb-8">
                    <h3 class="text-xl font-semibold mb-4 text-sky-600">أهدافك الحالية</h3>
                    <div id="currentGoalsList" class="space-y-4">
                        <!-- Goals will be inserted here -->
                    </div>
                    
                    <div class="mt-6 p-5 border border-sky-100 rounded-lg bg-sky-50" style="background-color: var(--bg-accent); border-color: var(--border-accent);">
                        <h4 class="font-semibold mb-3">إضافة هدف جديد</h4>
                        <form id="newGoalForm" class="space-y-4">
                            <div>
                                <label for="goalType" class="block text-sm font-medium mb-1">نوع الهدف:</label>
                                <select id="goalType" class="input-field w-full" required>
                                    <option value="weight">وزن تمرين</option>
                                    <option value="bodyWeight">وزن الجسم</option>
                                    <option value="consistency">المواظبة</option>
                                </select>
                            </div>
                            <div id="exerciseSelectContainer" class="hidden">
                                <label for="goalExercise" class="block text-sm font-medium mb-1">التمرين:</label>
                                <select id="goalExercise" class="input-field w-full">
                                    <option value="">-- اختر تمريناً --</option>
                                </select>
                            </div>
                            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                                <div>
                                   
