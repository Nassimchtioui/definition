<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>لوحة تحكم مشروع بيع الساعات</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm Neutrals with Teal Accent -->
    <!-- Application Structure Plan: The application is designed as an interactive dashboard. It starts with overall summary KPIs for a quick overview. The main section allows users to select a specific stage (1-5) via buttons, which dynamically updates KPI cards and a cost breakdown donut chart for that stage. This provides detailed, focused analysis. Below this, a comprehensive line and bar chart visualizes the growth trends (capital, profit, sales, costs) across all five stages, telling the story of the project's progression. This structure separates the high-level summary from the granular details, allowing for intuitive exploration and a clear understanding of both individual stage performance and overall project growth. -->
    <!-- Visualization & Content Choices: 
        - Overall KPIs (HTML): Goal: Inform. Method: Large text numbers. Justification: Provides immediate, high-level project totals.
        - Stage Selector (HTML Buttons): Goal: Interact/Filter. Method: Buttons. Justification: Simple, clear navigation between stages.
        - Stage-Specific KPIs (HTML Cards): Goal: Inform. Method: Styled cards. Justification: Displays key metrics for the selected stage clearly.
        - Cost Breakdown (Chart.js Donut): Goal: Compare Proportions. Method: Donut Chart. Justification: Effectively shows the percentage contribution of each cost component (purchase, delivery, ads) for a given stage.
        - Growth Trends (Chart.js Bar/Line Combo): Goal: Show Change/Trends. Method: Combined Bar (Sales/Costs) and Line (Profit/Capital) chart. Justification: Powerfully visualizes the project's growth trajectory across all stages, comparing multiple key metrics simultaneously.
        - Assumptions (HTML Table): Goal: Inform. Method: Simple table. Justification: Clearly lists the foundational constants of the plan.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Cairo', sans-serif;
            background-color: #f8fafc; /* slate-50 */
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 800px;
            margin-left: auto;
            margin-right: auto;
            height: 320px;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 400px;
            }
        }
        .kpi-card {
            background-color: white;
            border-radius: 0.75rem;
            padding: 1.5rem;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
            transition: transform 0.2s, box-shadow 0.2s;
            border: 1px solid #e5e7eb; /* gray-200 */
        }
        .stage-btn {
            transition: all 0.2s;
        }
        .stage-btn.active {
            background-color: #0d9488; /* teal-600 */
            color: white;
            transform: translateY(-2px);
            box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
        }
    </style>
</head>
<body class="bg-slate-50 text-gray-800">

    <div class="container mx-auto p-4 sm:p-6 lg:p-8">
        
        <header class="text-center mb-8">
            <h1 class="text-3xl md:text-4xl font-bold text-teal-700">تحليل خطة مشروع بيع الساعات</h1>
            <p class="mt-2 text-lg text-gray-600">لوحة تحكم تفاعلية لعرض وتتبع نمو المشروع عبر 5 مراحل</p>
        </header>

        <section id="summary" class="mb-8 p-6 bg-white rounded-xl shadow-md border border-gray-200">
            <h2 class="text-2xl font-bold text-center mb-4 text-gray-700">نظرة شاملة على المشروع</h2>
            <div class="grid grid-cols-2 md:grid-cols-4 gap-4 text-center">
                <div>
                    <p class="text-gray-500 text-sm md:text-base">إجمالي الربح الصافي المتوقع</p>
                    <p id="totalProfit" class="text-2xl md:text-3xl font-bold text-green-600"></p>
                </div>
                <div>
                    <p class="text-gray-500 text-sm md:text-base">إجمالي المبيعات المتوقعة</p>
                    <p id="totalSales" class="text-2xl md:text-3xl font-bold text-blue-600"></p>
                </div>
                <div>
                    <p class="text-gray-500 text-sm md:text-base">إجمالي الساعات المباعة</p>
                    <p id="totalWatches" class="text-2xl md:text-3xl font-bold text-purple-600"></p>
                </div>
                <div>
                    <p class="text-gray-500 text-sm md:text-base">العائد على الاستثمار الأولي</p>
                    <p id="roi" class="text-2xl md:text-3xl font-bold text-orange-500"></p>
                </div>
            </div>
        </section>

        <main class="bg-white p-4 sm:p-6 rounded-xl shadow-md border border-gray-200 mb-8">
            <div class="text-center mb-6">
                 <h2 class="text-2xl font-bold text-gray-700 mb-2">استكشاف مراحل المشروع</h2>
                 <p class="text-gray-600 max-w-2xl mx-auto">هذا القسم يسمح لك باستكشاف تفاصيل كل مرحلة على حدة. اختر مرحلة من الأزرار أدناه لعرض مؤشراتها الرئيسية وتوزيع تكاليفها في المخططات.</p>
            </div>
            <div id="stage-selector" class="flex flex-wrap justify-center gap-2 mb-6"></div>
            
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div class="lg:col-span-2 grid grid-cols-2 gap-4">
                    <div class="kpi-card col-span-2 sm:col-span-1">
                        <h3 class="font-bold text-gray-500 mb-2">رأس المال</h3>
                        <p id="kpi-capital" class="text-3xl font-bold text-teal-600"></p>
                    </div>
                    <div class="kpi-card col-span-2 sm:col-span-1">
                        <h3 class="font-bold text-gray-500 mb-2">الربح الصافي</h3>
                        <p id="kpi-profit" class="text-3xl font-bold text-green-600"></p>
                    </div>
                    <div class="kpi-card">
                        <h3 class="font-bold text-gray-500 mb-2">إجمالي المبيعات</h3>
                        <p id="kpi-sales" class="text-3xl font-bold text-blue-600"></p>
                    </div>
                    <div class="kpi-card">
                        <h3 class="font-bold text-gray-500 mb-2">التكلفة الإجمالية</h3>
                        <p id="kpi-costs" class="text-3xl font-bold text-red-500"></p>
                    </div>
                    <div class="kpi-card col-span-2">
                        <h3 class="font-bold text-gray-500 mb-2">عدد الساعات المشتراة</h3>
                        <p id="kpi-watches" class="text-3xl font-bold text-purple-600"></p>
                    </div>
                </div>
                <div class="lg:col-span-1 kpi-card flex flex-col justify-center items-center h-full min-h-[300px]">
                    <h3 class="font-bold text-gray-600 mb-2 text-center">توزيع تكاليف المرحلة</h3>
                    <div class="w-full h-full max-h-[300px]">
                        <canvas id="cost-breakdown-chart"></canvas>
                    </div>
                </div>
            </div>
        </main>

        <section class="bg-white p-4 sm:p-6 rounded-xl shadow-md border border-gray-200 mb-8">
            <div class="text-center mb-6">
                <h2 class="text-2xl font-bold text-gray-700 mb-2">تتبع نمو المشروع</h2>
                <p class="text-gray-600 max-w-2xl mx-auto">يعرض هذا المخطط تطور المؤشرات المالية الرئيسية عبر المراحل الخمس، مما يوضح قصة نمو المشروع من البداية وحتى تحقيق الأهداف.</p>
            </div>
            <div class="chart-container">
                <canvas id="growth-chart"></canvas>
            </div>
        </section>

        <section class="bg-white p-6 rounded-xl shadow-md border border-gray-200">
            <h2 class="text-2xl font-bold text-center mb-4 text-gray-700">الافتراضات الأساسية للخطة</h2>
             <div class="overflow-x-auto">
                <table class="min-w-full text-center">
                    <thead class="bg-gray-100">
                        <tr>
                            <th class="p-3 font-semibold text-sm text-gray-600 uppercase">البند</th>
                            <th class="p-3 font-semibold text-sm text-gray-600 uppercase">القيمة</th>
                        </tr>
                    </thead>
                    <tbody class="divide-y divide-gray-200">
                        <tr><td class="p-3">سعر بيع الساعة</td><td class="p-3 font-mono">249 درهم</td></tr>
                        <tr><td class="p-3">تكلفة شراء الساعة</td><td class="p-3 font-mono">139 درهم</td></tr>
                        <tr><td class="p-3">تكلفة البوكس</td><td class="p-3 font-mono">18 درهم</td></tr>
                        <tr><td class="p-3">تكلفة النقل (لكل مرحلة)</td><td class="p-3 font-mono">50 درهم</td></tr>
                        <tr><td class="p-3">تكلفة التوصيل (لكل ساعة)</td><td class="p-3 font-mono">35 درهم</td></tr>
                    </tbody>
                </table>
            </div>
        </section>

        <footer class="text-center mt-8 text-gray-500">
            <p>تم تطوير لوحة التحكم هذه لعرض وتحليل خطة المشروع بشكل تفاعلي.</p>
        </footer>
    </div>

    <script>
        const projectData = [
            { stage: 1, capital: 1000, watches: 5, adCost: 100 },
            { stage: 2, capital: 1135, watches: 6, adCost: 120 },
            { stage: 3, capital: 1414, watches: 8, adCost: 150 },
            { stage: 4, capital: 1882, watches: 11, adCost: 180 },
            { stage: 5, capital: 2589, watches: 15, adCost: 220 },
        ];

        const watchPurchaseCost = 139;
        const boxCost = 18;
        const transportCost = 50;
        const deliveryCostPerWatch = 35;
        const salePricePerWatch = 249;

        const calculatedData = projectData.map(d => {
            const purchaseAndBoxCost = d.watches * (watchPurchaseCost + boxCost);
            const totalDeliveryCost = d.watches * deliveryCostPerWatch;
            const totalCost = purchaseAndBoxCost + transportCost + totalDeliveryCost + d.adCost;
            const totalSales = d.watches * salePricePerWatch;
            const netProfit = totalSales - totalCost;
            const newCapital = d.capital + netProfit;
            
            return {
                ...d,
                purchaseAndBoxCost,
                totalDeliveryCost,
                totalCost,
                totalSales,
                netProfit,
                newCapital,
            };
        });

        let costBreakdownChart;
        let growthChart;
        let currentStageIndex = 0;

        const formatCurrency = (value) => {
            return new Intl.NumberFormat('ar-AE', { style: 'currency', currency: 'AED', minimumFractionDigits: 0 }).format(value);
        };

        function updateStageDetails(index) {
            currentStageIndex = index;
            const data = calculatedData[index];

            document.getElementById('kpi-capital').textContent = formatCurrency(data.capital);
            document.getElementById('kpi-profit').textContent = formatCurrency(data.netProfit);
            document.getElementById('kpi-sales').textContent = formatCurrency(data.totalSales);
            document.getElementById('kpi-costs').textContent = formatCurrency(data.totalCost);
            document.getElementById('kpi-watches').textContent = `${data.watches} ساعات`;
            
            updateActiveButton();
            renderCostBreakdownChart(data);
        }

        function updateActiveButton() {
            const buttons = document.querySelectorAll('.stage-btn');
            buttons.forEach((btn, idx) => {
                if (idx === currentStageIndex) {
                    btn.classList.add('active');
                } else {
                    btn.classList.remove('active');
                }
            });
        }

        function renderCostBreakdownChart(data) {
            const ctx = document.getElementById('cost-breakdown-chart').getContext('2d');
            if (costBreakdownChart) {
                costBreakdownChart.destroy();
            }
            costBreakdownChart = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: ['شراء الساعات والبوكس', 'تكلفة التوصيل', 'تكلفة الإعلان', 'تكلفة النقل'],
                    datasets: [{
                        data: [data.purchaseAndBoxCost, data.totalDeliveryCost, data.adCost, transportCost],
                        backgroundColor: ['#34d399', '#60a5fa', '#a78bfa', '#f87171'],
                        borderColor: '#ffffff',
                        borderWidth: 2,
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'bottom',
                            labels: {
                                font: { family: 'Cairo', size: 12 },
                                color: '#4b5563',
                                padding: 15,
                            }
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    if (context.parsed !== null) {
                                        label += formatCurrency(context.parsed);
                                    }
                                    return label;
                                }
                            },
                            bodyFont: { family: 'Cairo' },
                            titleFont: { family: 'Cairo' },
                        }
                    }
                }
            });
        }

        function renderGrowthChart() {
            const ctx = document.getElementById('growth-chart').getContext('2d');
            const labels = calculatedData.map(d => `المرحلة ${d.stage}`);
            
            growthChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [
                        {
                            type: 'line',
                            label: 'الربح الصافي',
                            data: calculatedData.map(d => d.netProfit),
                            borderColor: '#16a34a', // green-600
                            backgroundColor: '#16a34a',
                            tension: 0.3,
                            yAxisID: 'y',
                            borderWidth: 3,
                            pointBackgroundColor: '#ffffff',
                            pointBorderColor: '#16a34a',
                            pointHoverRadius: 7,
                            pointRadius: 5
                        },
                        {
                            type: 'line',
                            label: 'رأس المال الجديد',
                            data: calculatedData.map(d => d.newCapital),
                            borderColor: '#0d9488', // teal-600
                            backgroundColor: '#0d9488',
                            tension: 0.3,
                            yAxisID: 'y',
                            borderWidth: 3,
                            pointBackgroundColor: '#ffffff',
                            pointBorderColor: '#0d9488',
                            pointHoverRadius: 7,
                            pointRadius: 5
                        },
                        {
                            type: 'bar',
                            label: 'إجمالي المبيعات',
                            data: calculatedData.map(d => d.totalSales),
                            backgroundColor: 'rgba(59, 130, 246, 0.6)', // blue-500
                            borderColor: 'rgba(59, 130, 246, 1)',
                            borderWidth: 1,
                            yAxisID: 'y',
                        },
                        {
                            type: 'bar',
                            label: 'التكلفة الإجمالية',
                            data: calculatedData.map(d => d.totalCost),
                            backgroundColor: 'rgba(239, 68, 68, 0.6)', // red-500
                            borderColor: 'rgba(239, 68, 68, 1)',
                            borderWidth: 1,
                            yAxisID: 'y',
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: {
                            grid: { display: false },
                            ticks: { font: { family: 'Cairo' }, color: '#4b5563' }
                        },
                        y: {
                            beginAtZero: true,
                            grid: {
                                color: '#e5e7eb', // gray-200
                                borderDash: [2, 4],
                            },
                            ticks: {
                                font: { family: 'Cairo' },
                                color: '#4b5563',
                                callback: function(value) {
                                    return formatCurrency(value);
                                }
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            position: 'top',
                            labels: { font: { family: 'Cairo' }, color: '#4b5563' }
                        },
                        tooltip: {
                            mode: 'index',
                            intersect: false,
                            bodyFont: { family: 'Cairo' },
                            titleFont: { family: 'Cairo' },
                            callbacks: {
                                label: function(context) {
                                    let label = context.dataset.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    if (context.parsed.y !== null) {
                                        label += formatCurrency(context.parsed.y);
                                    }
                                    return label;
                                }
                            }
                        }
                    },
                    interaction: {
                        mode: 'index',
                        intersect: false,
                    },
                }
            });
        }
        
        function initializeSummary() {
            const totalProfit = calculatedData.reduce((acc, curr) => acc + curr.netProfit, 0);
            const totalSales = calculatedData.reduce((acc, curr) => acc + curr.totalSales, 0);
            const totalWatches = calculatedData.reduce((acc, curr) => acc + curr.watches, 0);
            const initialCapital = calculatedData[0].capital;
            const roi = (totalProfit / initialCapital) * 100;

            document.getElementById('totalProfit').textContent = formatCurrency(totalProfit);
            document.getElementById('totalSales').textContent = formatCurrency(totalSales);
            document.getElementById('totalWatches').textContent = `${totalWatches} ساعات`;
            document.getElementById('roi').textContent = `${roi.toFixed(1)}%`;
        }

        document.addEventListener('DOMContentLoaded', () => {
            const stageSelector = document.getElementById('stage-selector');
            calculatedData.forEach((data, index) => {
                const button = document.createElement('button');
                button.textContent = `المرحلة ${data.stage}`;
                button.className = 'stage-btn px-4 py-2 bg-white border border-gray-300 rounded-lg text-gray-700 font-semibold hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-teal-500';
                button.onclick = () => updateStageDetails(index);
                stageSelector.appendChild(button);
            });
            
            initializeSummary();
            updateStageDetails(0);
            renderGrowthChart();
        });
    </script>
</body>
</html>

