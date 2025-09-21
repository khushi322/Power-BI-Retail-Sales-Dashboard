<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Sales Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        /* Chosen Palette: Warm Neutrals */
        /* Application Structure Plan: A responsive dashboard layout featuring a main header, a top row for 4 key KPIs, and a central grid for 4 primary charts. A left-aligned sidebar contains interactive filters (Region, Category) to dynamically update all visualizations. This structure prioritizes immediate insight via KPIs and deep exploration through filtered charts, offering a more intuitive user flow than a static report. */
        /* Visualization & Content Choices: 
            - KPIs: Goal=Inform. Method=Dynamic text blocks. Interaction=Updates on filter. Justification=Quick, high-level overview.
            - Sales Trend: Goal=Show Change. Method=Chart.js Line Chart. Interaction=Tooltips. Justification=Clearly shows performance over time.
            - Profit by Category: Goal=Compare. Method=Chart.js Bar Chart. Interaction=Tooltips. Justification=Direct comparison of category profitability.
            - Sales by Region: Goal=Compare Proportions. Method=Chart.js Doughnut Chart. Interaction=Tooltips. Justification=Visually engaging way to see regional sales distribution.
            - Profit by Sub-Category: Goal=Organize/Rank. Method=Chart.js Horizontal Bar Chart. Interaction=Tooltips. Justification=Effectively highlights best and worst performers, including negative profits.
            - All interactions are handled via JavaScript filtering and Chart.js updates.
        */
        /* CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. */
        body {
            background-color: #F8F7F4; /* Warm neutral background */
            font-family: 'Inter', sans-serif;
            color: #333;
        }
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        .chart-container {
            position: relative;
            height: 40vh;
            width: 100%;
            max-height: 400px;
        }
        .kpi-card {
            background-color: #FFFFFF;
            border-radius: 0.75rem;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
            transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
        }
        .kpi-card:hover {
            transform: translateY(-4px);
            box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
        }
        .chart-card {
            background-color: #FFFFFF;
            border-radius: 0.75rem;
            padding: 1.5rem;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
        }
        .filter-select {
            background-color: #FFFFFF;
        }
    </style>
</head>
<body class="p-4 sm:p-6 lg:p-8">
    <div class="max-w-7xl mx-auto">
        <!-- Header -->
        <header class="mb-8">
            <h1 class="text-4xl font-bold text-gray-800">Retail Sales Dashboard</h1>
            <p class="text-gray-500 mt-1">An interactive analysis of the Superstore dataset.</p>
        </header>

        <!-- Main Content -->
        <div class="flex flex-col lg:flex-row gap-8">
            <!-- Filters Sidebar -->
            <aside class="w-full lg:w-1/5">
                <div class="p-6 bg-white rounded-xl shadow-md space-y-4">
                    <h3 class="text-xl font-bold text-gray-700">Filters</h3>
                    <div>
                        <label for="regionFilter" class="block text-sm font-medium text-gray-600 mb-1">Region</label>
                        <select id="regionFilter" class="w-full p-2 border border-gray-300 rounded-lg filter-select focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                            <option value="All">All Regions</option>
                        </select>
                    </div>
                    <div>
                        <label for="categoryFilter" class="block text-sm font-medium text-gray-600 mb-1">Category</label>
                        <select id="categoryFilter" class="w-full p-2 border border-gray-300 rounded-lg filter-select focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                            <option value="All">All Categories</option>
                        </select>
                    </div>
                </div>
            </aside>

            <!-- Dashboard Grid -->
            <main class="w-full lg:w-4/5">
                <!-- KPIs -->
                <section class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
                    <div class="kpi-card p-6">
                        <h4 class="text-gray-500 font-semibold">Total Sales</h4>
                        <p id="totalSales" class="text-4xl font-bold mt-2 text-blue-600">$0</p>
                    </div>
                    <div class="kpi-card p-6">
                        <h4 class="text-gray-500 font-semibold">Total Profit</h4>
                        <p id="totalProfit" class="text-4xl font-bold mt-2 text-green-600">$0</p>
                    </div>
                    <div class="kpi-card p-6">
                        <h4 class="text-gray-500 font-semibold">Total Quantity</h4>
                        <p id="totalQuantity" class="text-4xl font-bold mt-2 text-purple-600">0</p>
                    </div>
                    <div class="kpi-card p-6">
                        <h4 class="text-gray-500 font-semibold">Profit Margin</h4>
                        <p id="profitMargin" class="text-4xl font-bold mt-2 text-orange-600">0%</p>
                    </div>
                </section>

                <!-- Charts -->
                <section class="grid grid-cols-1 xl:grid-cols-2 gap-8">
                    <div class="chart-card">
                        <h3 class="text-xl font-bold text-gray-700 mb-4">Sales Trend Over Time</h3>
                        <div class="chart-container">
                            <canvas id="salesTrendChart"></canvas>
                        </div>
                    </div>
                    <div class="chart-card">
                        <h3 class="text-xl font-bold text-gray-700 mb-4">Profit by Category</h3>
                        <div class="chart-container">
                            <canvas id="profitByCategoryChart"></canvas>
                        </div>
                    </div>
                    <div class="chart-card">
                        <h3 class="text-xl font-bold text-gray-700 mb-4">Sales by Region</h3>
                        <div class="chart-container">
                            <canvas id="salesByRegionChart"></canvas>
                        </div>
                    </div>
                    <div class="chart-card">
                        <h3 class="text-xl font-bold text-gray-700 mb-4">Profit by Sub-Category</h3>
                        <div class="chart-container">
                            <canvas id="profitBySubCategoryChart"></canvas>
                        </div>
                    </div>
                </section>
            </main>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function () {
            // A sample of data extracted from SampleSuperstore.csv for this demo.
            // In a real application, this would be loaded via an API.
            const salesData = [
                { "Order Date": "2016-11-08", "Region": "South", "Category": "Furniture", "Sub-Category": "Bookcases", "Sales": 261.96, "Quantity": 2, "Profit": 41.9136 },
                { "Order Date": "2016-11-08", "Region": "South", "Category": "Furniture", "Sub-Category": "Chairs", "Sales": 731.94, "Quantity": 3, "Profit": 219.582 },
                { "Order Date": "2016-06-12", "Region": "West", "Category": "Office Supplies", "Sub-Category": "Labels", "Sales": 14.62, "Quantity": 2, "Profit": 6.8714 },
                { "Order Date": "2015-10-11", "Region": "South", "Category": "Furniture", "Sub-Category": "Tables", "Sales": 957.5775, "Quantity": 5, "Profit": -383.031 },
                { "Order Date": "2015-10-11", "Region": "South", "Category": "Office Supplies", "Sub-Category": "Storage", "Sales": 22.368, "Quantity": 2, "Profit": 2.5164 },
                { "Order Date": "2014-06-09", "Region": "West", "Category": "Furniture", "Sub-Category": "Furnishings", "Sales": 48.86, "Quantity": 7, "Profit": 14.1694 },
                { "Order Date": "2014-06-09", "Region": "West", "Category": "Office Supplies", "Sub-Category": "Art", "Sales": 7.28, "Quantity": 4, "Profit": 1.9656 },
                { "Order Date": "2014-06-09", "Region": "West", "Category": "Technology", "Sub-Category": "Phones", "Sales": 907.152, "Quantity": 4, "Profit": 90.7152 },
                { "Order Date": "2017-06-17", "Region": "Central", "Category": "Office Supplies", "Sub-Category": "Binders", "Sales": 18.504, "Quantity": 3, "Profit": 5.7825 },
                { "Order Date": "2017-06-17", "Region": "Central", "Category": "Office Supplies", "Sub-Category": "Appliances", "Sales": 114.9, "Quantity": 5, "Profit": 34.47 },
                { "Order Date": "2017-06-17", "Region": "Central", "Category": "Furniture", "Sub-Category": "Tables", "Sales": 1706.184, "Quantity": 9, "Profit": 85.3092 },
                { "Order Date": "2017-06-17", "Region": "Central", "Category": "Technology", "Sub-Category": "Phones", "Sales": 911.424, "Quantity": 4, "Profit": 68.3568 },
                { "Order Date": "2016-12-25", "Region": "East", "Category": "Office Supplies", "Sub-Category": "Binders", "Sales": 3.296, "Quantity": 2, "Profit": 1.1206 },
                { "Order Date": "2016-12-25", "Region": "East", "Category": "Furniture", "Sub-Category": "Chairs", "Sales": 332.96, "Quantity": 2, "Profit": 103.2176 },
                { "Order Date": "2014-11-15", "Region": "East", "Category": "Technology", "Sub-Category": "Accessories", "Sales": 230.98, "Quantity": 5, "Profit": 57.745 }
            ];

            // In a real scenario, you'd process the full CSV. For this demo, let's expand the sample data to make it more representative.
            const fullDataset = [...salesData, ...salesData.map(d => ({...d, Sales: d.Sales * 0.8, Profit: d.Profit * 0.7, "Order Date": "2015-05-20"})), ...salesData.map(d => ({...d, Sales: d.Sales * 1.2, Profit: d.Profit * 1.3, "Order Date": "2016-02-10"})), ...salesData.map(d => ({...d, Sales: d.Sales * 1.5, Profit: d.Profit * 1.8, "Order Date": "2017-09-05"}))];

            // Populate filters
            const regions = [...new Set(fullDataset.map(d => d.Region))];
            const categories = [...new Set(fullDataset.map(d => d.Category))];
            const regionFilter = document.getElementById('regionFilter');
            const categoryFilter = document.getElementById('categoryFilter');
            regions.forEach(r => regionFilter.innerHTML += `<option value="${r}">${r}</option>`);
            categories.forEach(c => categoryFilter.innerHTML += `<option value="${c}">${c}</option>`);
            
            // Chart instances
            let salesTrendChart, profitByCategoryChart, salesByRegionChart, profitBySubCategoryChart;

            const formatCurrency = (value) => new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', minimumFractionDigits: 0, maximumFractionDigits: 0 }).format(value);

            function updateDashboard() {
                const selectedRegion = regionFilter.value;
                const selectedCategory = categoryFilter.value;

                const filteredData = fullDataset.filter(d => 
                    (selectedRegion === 'All' || d.Region === selectedRegion) &&
                    (selectedCategory === 'All' || d.Category === selectedCategory)
                );

                // Update KPIs
                const totalSales = filteredData.reduce((sum, d) => sum + d.Sales, 0);
                const totalProfit = filteredData.reduce((sum, d) => sum + d.Profit, 0);
                const totalQuantity = filteredData.reduce((sum, d) => sum + d.Quantity, 0);
                const profitMargin = totalSales > 0 ? (totalProfit / totalSales) * 100 : 0;

                document.getElementById('totalSales').textContent = formatCurrency(totalSales);
                document.getElementById('totalProfit').textContent = formatCurrency(totalProfit);
                document.getElementById('totalQuantity').textContent = totalQuantity.toLocaleString();
                document.getElementById('profitMargin').textContent = `${profitMargin.toFixed(1)}%`;

                // Update Charts
                updateSalesTrendChart(filteredData);
                updateProfitByCategoryChart(filteredData);
                updateSalesByRegionChart(filteredData);
                updateProfitBySubCategoryChart(filteredData);
            }

            function createChart(ctx, config) {
                return new Chart(ctx, {
                    ...config,
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        plugins: {
                            legend: {
                                position: 'bottom',
                                labels: {
                                    padding: 20
                                }
                            },
                            tooltip: {
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
                        ...config.options
                    }
                });
            }

            function updateSalesTrendChart(data) {
                const salesByMonth = data.reduce((acc, d) => {
                    const month = d['Order Date'].substring(0, 7);
                    acc[month] = (acc[month] || 0) + d.Sales;
                    return acc;
                }, {});
                const sortedMonths = Object.keys(salesByMonth).sort();
                const labels = sortedMonths;
                const chartData = sortedMonths.map(month => salesByMonth[month]);

                if (!salesTrendChart) {
                    const ctx = document.getElementById('salesTrendChart').getContext('2d');
                    salesTrendChart = createChart(ctx, {
                        type: 'line',
                        data: { labels, datasets: [{ label: 'Sales', data: chartData, borderColor: '#3B82F6', tension: 0.1, fill: false }] }
                    });
                } else {
                    salesTrendChart.data.labels = labels;
                    salesTrendChart.data.datasets[0].data = chartData;
                    salesTrendChart.update();
                }
            }

            function updateProfitByCategoryChart(data) {
                const profitByCategory = data.reduce((acc, d) => {
                    acc[d.Category] = (acc[d.Category] || 0) + d.Profit;
                    return acc;
                }, {});
                const labels = Object.keys(profitByCategory);
                const chartData = Object.values(profitByCategory);

                if (!profitByCategoryChart) {
                    const ctx = document.getElementById('profitByCategoryChart').getContext('2d');
                    profitByCategoryChart = createChart(ctx, {
                        type: 'bar',
                        data: { labels, datasets: [{ label: 'Profit', data: chartData, backgroundColor: ['#10B981', '#F59E0B', '#6366F1'] }] }
                    });
                } else {
                    profitByCategoryChart.data.labels = labels;
                    profitByCategoryChart.data.datasets[0].data = chartData;
                    profitByCategoryChart.update();
                }
            }
            
            function updateSalesByRegionChart(data) {
                const salesByRegion = data.reduce((acc, d) => {
                    acc[d.Region] = (acc[d.Region] || 0) + d.Sales;
                    return acc;
                }, {});
                const labels = Object.keys(salesByRegion);
                const chartData = Object.values(salesByRegion);

                if (!salesByRegionChart) {
                    const ctx = document.getElementById('salesByRegionChart').getContext('2d');
                    salesByRegionChart = createChart(ctx, {
                        type: 'doughnut',
                        data: { labels, datasets: [{ label: 'Sales', data: chartData, backgroundColor: ['#EF4444', '#3B82F6', '#F59E0B', '#10B981'] }] },
                        options: {
                           plugins: {
                                tooltip: {
                                    callbacks: {
                                        label: function(context) {
                                            let label = context.label || '';
                                            if (label) { label += ': '; }
                                            if (context.parsed !== null) {
                                                label += formatCurrency(context.parsed);
                                            }
                                            return label;
                                        }
                                    }
                                }
                           }
                        }
                    });
                } else {
                    salesByRegionChart.data.labels = labels;
                    salesByRegionChart.data.datasets[0].data = chartData;
                    salesByRegionChart.update();
                }
            }

            function updateProfitBySubCategoryChart(data) {
                const profitBySubCategory = data.reduce((acc, d) => {
                    acc[d['Sub-Category']] = (acc[d['Sub-Category']] || 0) + d.Profit;
                    return acc;
                }, {});

                const sortedSubCategories = Object.entries(profitBySubCategory).sort(([, a], [, b]) => a - b);
                const labels = sortedSubCategories.map(item => item[0]);
                const chartData = sortedSubCategories.map(item => item[1]);
                
                if (!profitBySubCategoryChart) {
                    const ctx = document.getElementById('profitBySubCategoryChart').getContext('2d');
                    profitBySubCategoryChart = createChart(ctx, {
                        type: 'bar',
                        data: { labels, datasets: [{ label: 'Profit', data: chartData, backgroundColor: chartData.map(v => v < 0 ? '#EF4444' : '#10B981') }] },
                        options: { indexAxis: 'y' }
                    });
                } else {
                    profitBySubCategoryChart.data.labels = labels;
                    profitBySubCategoryChart.data.datasets[0].data = chartData;
                    profitBySubCategoryChart.data.datasets[0].backgroundColor = chartData.map(v => v < 0 ? '#EF4444' : '#10B981');
                    profitBySubCategoryChart.update();
                }
            }

            // Initial Load
            updateDashboard();

            // Event Listeners
            regionFilter.addEventListener('change', updateDashboard);
            categoryFilter.addEventListener('change', updateDashboard);
        });
    </script>
</body>
</html>
