# Nutrition Diet Helper
面向大众的膳食营养智能分析网页，用户输入自己的饮食内容、身体基础数据、减脂/增肌目标，由AI估算热量、碳水、蛋白、脂肪，生成饮食改善建议。

<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI膳食营养师</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#22c55e',
                        secondary: '#0ea5e9',
                    },
                    fontFamily: {
                        sans: ['Inter', 'system-ui', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .card-shadow {
                box-shadow: 0 10px 25px -5px rgba(34, 197, 94, 0.1);
            }
            .bg-gradient-food {
                background: linear-gradient(135deg, #22c55e 0%, #10b981 100%);
            }
        }
    </style>
</head>
<body class="bg-green-50 min-h-screen">
    <div class="max-w-4xl mx-auto px-4 py-8">
        <!-- 头部 -->
        <header class="text-center mb-10">
            <div class="inline-flex items-center justify-center w-16 h-16 rounded-full bg-gradient-food text-white text-2xl mb-4">
                <i class="fa fa-cutlery"></i>
            </div>
            <h1 class="text-[clamp(1.8rem,4vw,2.5rem)] font-bold text-gray-800">AI膳食营养师</h1>
            <p class="text-gray-500 mt-2">输入你的身体信息，智能生成个性化一日食谱</p >
        </header>

        <div class="grid md:grid-cols-2 gap-6">
            <!-- 表单区域 -->
            <div class="bg-white rounded-2xl p-6 card-shadow">
                <h2 class="text-lg font-semibold text-gray-700 mb-4 flex items-center">
                    <i class="fa fa-user-circle text-primary mr-2"></i>个人信息
                </h2>
                <form id="dietForm" class="space-y-4">
                    <div>
                        <label class="block text-sm text-gray-600 mb-1">性别</label>
                        <select id="gender" class="w-full border border-gray-200 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-primary/50">
                            <option value="male">男</option>
                            <option value="female">女</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-sm text-gray-600 mb-1">年龄（岁）</label>
                        <input type="number" id="age" min="10" max="100" value="25" class="w-full border border-gray-200 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-primary/50">
                    </div>
                    <div>
                        <label class="block text-sm text-gray-600 mb-1">身高（cm）</label>
                        <input type="number" id="height" min="100" max="250" value="170" class="w-full border border-gray-200 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-primary/50">
                    </div>
                    <div>
                        <label class="block text-sm text-gray-600 mb-1">体重（kg）</label>
                        <input type="number" id="weight" min="30" max="300" value="60" class="w-full border border-gray-200 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-primary/50">
                    </div>
                    <div>
                        <label class="block text-sm text-gray-600 mb-1">活动水平</label>
                        <select id="activity" class="w-full border border-gray-200 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-primary/50">
                            <option value="1.2">久坐（几乎不运动）</option>
                            <option value="1.375">轻度活动（每周运动1-3次）</option>
                            <option value="1.55">中度活动（每周运动3-5次）</option>
                            <option value="1.725">高强度活动（每周运动6-7次）</option>
                            <option value="1.9">极高强度（体力工作/专业训练）</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-sm text-gray-600 mb-1">忌口 / 偏好（例如：不吃辣、素食、乳糖不耐）</label>
                        <textarea id="avoid" rows="2" placeholder="无" class="w-full border border-gray-200 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-primary/50"></textarea>
                    </div>
                    <button type="submit" class="w-full bg-gradient-food text-white py-3 rounded-lg font-medium hover:opacity-90 transition-all">
                        <i class="fa fa-magic mr-1"></i>生成AI膳食方案
                    </button>
                </form>
            </div>

            <!-- 结果展示区域 -->
            <div class="bg-white rounded-2xl p-6 card-shadow">
                <h2 class="text-lg font-semibold text-gray-700 mb-4 flex items-center">
                    <i class="fa fa-list-alt text-primary mr-2"></i>膳食报告
                </h2>
                <div id="resultBox" class="text-gray-500">
                    <div class="py-10 text-center">
                        <i class="fa fa-spoon text-4xl text-gray-200 mb-3"></i>
                        <p>填写信息后点击生成<br>为你定制营养食谱</p >
                    </div>
                </div>
            </div>
        </div>

        <footer class="mt-10 text-center text-sm text-gray-400">
            <p>AI膳食营养师 · 仅供健康参考，不能替代医师与临床营养师建议</p >
            <p class="mt-1">GitHub Pages 静态演示版</p >
        </footer>
    </div>

    <script>
        const form = document.getElementById('dietForm');
        const resultBox = document.getElementById('resultBox');

        // Mifflin-St Jeor 基础代谢公式
        function calcBmr(gender, age, height, weight) {
            if(gender === 'male'){
                return 10 * weight + 6.25 * height - 5 * age + 5;
            }else{
                return 10 * weight + 6.25 * height - 5 * age - 161;
            }
        }

        function generateDietPlan(tdee, avoid) {
            const carbs = Math.round(tdee * 0.5 / 4);
            const protein = Math.round(tdee * 0.25 / 4);
            const fat = Math.round(tdee * 0.25 / 9);

            const mealTemplates = [
                {
                    breakfast: "全麦面包2片 + 水煮蛋1个 + 无糖豆浆一杯 + 小番茄几颗",
                    lunch: "杂粮饭一碗 + 清蒸鱼 + 清炒西兰花 + 菌菇汤",
                    dinner: "红薯半块 + 鸡胸肉沙拉（生菜黄瓜）+ 凉拌木耳",
                    snack: "一个苹果 / 一小把原味坚果"
                },
                {
                    breakfast: "燕麦牛奶粥 + 鸡蛋 + 蓝莓一小盒",
                    lunch: "荞麦面 + 瘦牛肉 + 青菜 + 豆腐",
                    dinner: "玉米一根 + 虾仁滑蛋 + 炒油麦菜",
                    snack: "无糖酸奶"
                },
                {
                    breakfast: "玉米 + 茶叶蛋 + 纯牛奶",
                    lunch: "糙米饭 + 去皮鸡腿肉 + 冬瓜 + 炒菠菜",
                    dinner: "山药 + 清蒸虾 + 凉拌海带丝",
                    snack: "橙子一个"
                }
            ]
            const randomMeal = mealTemplates[Math.floor(Math.random() * mealTemplates.length)];
            let tip = "";
            if(avoid && avoid.trim() !== "无" && avoid.trim() !== ""){
                tip = `<p class="text-amber-600 mt-2"><i class="fa fa-exclamation-circle"></i> 已记录忌口：${avoid}</p >`;
            }

            return `
                <div class="space-y-4">
                    <div class="bg-green-50 p-3 rounded-lg">
                        <div class="grid grid-cols-2 gap-2 text-sm">
                            <div>BMI：<span class="font-semibold text-gray-800" id="bmiVal"></span></div>
                            <div>每日总热量：<span class="font-semibold text-gray-800">${Math.round(tdee)} kcal</span></div>
                            <div>碳水：<span class="font-semibold">${carbs} g</span></div>
                            <div>蛋白质：<span class="font-semibold">${protein} g</span></div>
                            <div>脂肪：<span class="font-semibold">${fat} g</span></div>
                        </div>
                    </div>
                    ${tip}
                    <div>
                        <h3 class="font-medium text-gray-700">🌞 早餐</h3>
                        <p class="text-gray-600">${randomMeal.breakfast}</p >
                    </div>
                    <div>
                        <h3 class="font-medium text-gray-700">☀️ 午餐</h3>
                        <p class="text-gray-600">${randomMeal.lunch}</p >
                    </div>
                    <div>
                        <h3 class="font-medium text-gray-700">🌙 晚餐</h3>
                        <p class="text-gray-600">${randomMeal.dinner}</p >
                    </div>
                    <div>
                        <h3 class="font-medium text-gray-700">🍎 加餐推荐</h3>
                        <p class="text-gray-600">${randomMeal.snack}</p >
                    </div>
                    <div class="border-t pt-3 text-sm text-gray-500">
                        💡 小贴士：三餐定时，少油少糖，每天饮水1500-2000ml；食谱仅为参考，如有慢病请遵医嘱。
                    </div>
                </div>
            `;
        }

        form.addEventListener('submit', function(e){
            e.preventDefault();
            const gender = document.getElementById('gender').value;
            const age = Number(document.getElementById('age').value);
            const height = Number(document.getElementById('height').value);
            const weight = Number(document.getElementById('weight').value);
            const activity = Number(document.getElementById('activity').value);
            const avoid = document.getElementById('avoid').value.trim();

            const bmr = calcBmr(gender, age, height, weight);
            const tdee = bmr * activity;
            const bmi = (weight / ((height/100)**2)).toFixed(1);

            resultBox.innerHTML = generateDietPlan(tdee, avoid);
            document.getElementById('bmiVal').innerText = bmi;
        })
    </script>
</body>
</html>
