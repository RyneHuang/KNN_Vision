# KNN算法可视化教学演示系统 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**目标:** 开发一款基于Web的单页面应用程序，用于辅助高职学生直观理解K-近邻（KNN）算法的核心原理

**架构:** 纯前端单HTML文件应用，使用Plotly.js进行数据可视化，模块化JavaScript实现KNN核心算法和数据处理逻辑

**技术栈:** HTML5, CSS3, JavaScript (ES6+), Plotly.js

---

## 文件结构

```
KNN_Vision/
├── index.html              # 主入口文件，包含所有HTML/CSS/JS
├── SPEC.md                  # 需求规格说明书
├── docs/
│   └── superpower/
│       └── plans/
│           └── 2026-04-06-knn-visualization-teaching-system.md  # 本计划
└── tests/                   # 测试文件目录
```

---

## 任务分解

### Task 1: 项目初始化与SPEC文档

**Files:**
- Create: `SPEC.md`
- Create: `.gitignore`

- [ ] **Step 1: 创建 .gitignore**

```bash
# Git ignore
node_modules/
.DS_Store
*.log
```

- [ ] **Step 2: 初始化Git仓库**

```bash
git init
git add .
git commit -m "chore: initial project setup"
```

- [ ] **Step 3: 创建SPEC.md需求规格文档**

根据需求描述文档，提取关键规格：
- 4个内置数据集：乳腺癌诊断(2特征)、鸢尾花(2特征)、月亮数据集、同心圆
- 归一化开关：Min-Max归一化，公式 X_scaled = (X - X_min) / (X_max - X_min)
- K值范围：1-30，50x50网格决策边界
- 交互功能：点击生成待测点、显示K个最近邻居、投票结果
- 布局：左侧70%主画布，右侧30%控制面板
- 配色：浅红色类别1，浅蓝色类别2，绿色待测点

- [ ] **Step 4: 提交**

```bash
git add SPEC.md .gitignore
git commit -m "docs: add SPEC.md with detailed requirements"
```

---

### Task 2: HTML结构与CSS样式

**Files:**
- Modify: `index.html` (创建基础HTML结构和CSS样式)

- [ ] **Step 1: 创建HTML基础结构**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KNN算法可视化教学演示系统</title>
    <!-- Plotly.js CDN -->
    <script src="https://cdn.plot.ly/plotly-2.27.0.min.js"></script>
    <style>
        /* CSS样式将在后续步骤添加 */
    </style>
</head>
<body>
    <div class="container">
        <!-- 左侧主画布区域 (70%) -->
        <div class="main-canvas">
            <div id="scatter-plot"></div>
            <div id="decision-boundary"></div>
        </div>
        <!-- 右侧控制面板 (30%) -->
        <div class="control-panel">
            <h2>KNN 算法演示</h2>
            <!-- 数据集选择 -->
            <div class="control-group">
                <label>数据集</label>
                <select id="dataset-select">
                    <option value="breast-cancer">乳腺癌诊断数据集</option>
                    <option value="iris">鸢尾花数据集</option>
                    <option value="moons">月亮数据集</option>
                    <option value="circles">同心圆数据集</option>
                </select>
            </div>
            <!-- 归一化开关 -->
            <div class="control-group">
                <label>启用Min-Max归一化</label>
                <input type="checkbox" id="normalization-toggle">
            </div>
            <!-- K值滑块 -->
            <div class="control-group">
                <label>K值: <span id="k-value-display">5</span></label>
                <input type="range" id="k-slider" min="1" max="30" value="5">
            </div>
            <!-- 预测结果 -->
            <div class="control-group" id="prediction-result">
                <h3>预测结果</h3>
                <div id="prediction-details"></div>
            </div>
            <!-- 教学说明 -->
            <div class="control-group">
                <h3>教学说明</h3>
                <div id="teaching-notes"></div>
            </div>
        </div>
    </div>
</body>
</html>
```

- [ ] **Step 2: 添加CSS样式**

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: #f5f5f5;
    color: #333;
}

.container {
    display: flex;
    height: 100vh;
    width: 100vw;
}

.main-canvas {
    flex: 0 0 70%;
    padding: 20px;
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.control-panel {
    flex: 0 0 30%;
    background: #fff;
    padding: 20px;
    box-shadow: -2px 0 10px rgba(0,0,0,0.1);
    overflow-y: auto;
}

.control-group {
    margin-bottom: 20px;
}

.control-group label {
    display: block;
    font-weight: 600;
    margin-bottom: 8px;
}

.control-group select,
.control-group input[type="range"] {
    width: 100%;
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
}

#k-value-display {
    color: #2196F3;
    font-weight: bold;
}

#prediction-details {
    font-size: 14px;
    line-height: 1.6;
}
```

- [ ] **Step 3: 提交**

```bash
git add index.html
git commit -m "feat: add HTML structure and CSS styling"
```

---

### Task 3: 数据集定义与加载

**Files:**
- Modify: `index.html` (添加数据集定义)

- [ ] **Step 1: 添加JavaScript数据集定义**

```javascript
// 数据集定义
const DATASETS = {
    'breast-cancer': {
        name: '乳腺癌诊断数据集',
        // 特征: 平均半径, 平均平滑度
        // 良性肿瘤标记为0，恶性肿瘤标记为1
        data: [
            // 良性 (0)
            {x: 12.5, y: 0.085, label: 0},
            {x: 13.2, y: 0.092, label: 0},
            {x: 14.1, y: 0.088, label: 0},
            {x: 11.8, y: 0.076, label: 0},
            {x: 13.5, y: 0.095, label: 0},
            {x: 14.3, y: 0.090, label: 0},
            {x: 12.1, y: 0.078, label: 0},
            {x: 13.0, y: 0.086, label: 0},
            {x: 15.2, y: 0.098, label: 0},
            {x: 11.5, y: 0.072, label: 0},
            // 恶性 (1)
            {x: 22.5, y: 0.142, label: 1},
            {x: 21.3, y: 0.134, label: 1},
            {x: 23.5, y: 0.156, label: 1},
            {x: 20.8, y: 0.128, label: 1},
            {x: 24.2, y: 0.162, label: 1},
            {x: 19.5, y: 0.118, label: 1},
            {x: 22.8, y: 0.148, label: 1},
            {x: 21.0, y: 0.130, label: 1},
            {x: 23.0, y: 0.152, label: 1},
            {x: 20.5, y: 0.122, label: 1}
        ],
        description: '展示未归一化数据在距离计算中的尺度偏差问题'
    },
    'iris': {
        name: '鸢尾花数据集',
        // 花萼长度, 花萼宽度
        data: [
            // Setosa (0)
            {x: 5.1, y: 3.5, label: 0},
            {x: 4.9, y: 3.0, label: 0},
            {x: 5.0, y: 3.2, label: 0},
            {x: 5.5, y: 3.6, label: 0},
            {x: 4.8, y: 3.1, label: 0},
            {x: 5.2, y: 3.4, label: 0},
            {x: 5.3, y: 3.7, label: 0},
            {x: 4.7, y: 3.2, label: 0},
            {x: 5.8, y: 4.0, label: 0},
            {x: 5.4, y: 3.9, label: 0},
            // Versicolor (1)
            {x: 7.0, y: 3.2, label: 1},
            {x: 6.4, y: 3.2, label: 1},
            {x: 6.9, y: 3.1, label: 1},
            {x: 6.5, y: 2.8, label: 1},
            {x: 6.3, y: 2.9, label: 1},
            {x: 6.6, y: 2.5, label: 1},
            {x: 7.1, y: 3.0, label: 1},
            {x: 6.2, y: 2.2, label: 1},
            {x: 6.5, y: 2.8, label: 1},
            {x: 6.8, y: 3.0, label: 1},
            // Virginica (2)
            {x: 6.3, y: 3.3, label: 2},
            {x: 5.8, y: 2.7, label: 2},
            {x: 7.1, y: 3.0, label: 2},
            {x: 6.3, y: 2.9, label: 2},
            {x: 6.5, y: 3.0, label: 2},
            {x: 7.7, y: 3.8, label: 2},
            {x: 7.7, y: 2.6, label: 2},
            {x: 6.0, y: 2.2, label: 2},
            {x: 6.9, y: 3.2, label: 2},
            {x: 5.6, y: 2.8, label: 2}
        ],
        description: '展示基础分类场景，三种鸢尾花分类'
    },
    'moons': {
        name: '月亮数据集',
        // 生成的月牙形数据
        data: generateMoonsData(),
        description: '展示KNN在处理非凸形状数据分布时的优势'
    },
    'circles': {
        name: '同心圆数据集',
        // 生成的同心圆数据
        data: generateCirclesData(),
        description: '验证KNN算法在局部邻域分类中的表现'
    }
};

// 生成月亮数据集
function generateMoonsData() {
    const data = [];
    // 第一个半月 (标签0)
    for (let i = 0; i < 50; i++) {
        const angle = Math.PI * (i / 50);
        const r = 1 + Math.random() * 0.3;
        data.push({
            x: r * Math.cos(angle) + (Math.random() - 0.5) * 0.1,
            y: r * Math.sin(angle) + (Math.random() - 0.5) * 0.1,
            label: 0
        });
    }
    // 第二个半月 (标签1)
    for (let i = 0; i < 50; i++) {
        const angle = Math.PI * (i / 50);
        const r = 1 + Math.random() * 0.3;
        data.push({
            x: r * Math.cos(angle + Math.PI) + 2 + (Math.random() - 0.5) * 0.1,
            y: r * Math.sin(angle + Math.PI) + 0.5 + (Math.random() - 0.5) * 0.1,
            label: 1
        });
    }
    return data;
}

// 生成同心圆数据集
function generateCirclesData() {
    const data = [];
    // 内圆 (标签0)
    for (let i = 0; i < 50; i++) {
        const angle = 2 * Math.PI * (i / 50);
        const r = 0.5 + Math.random() * 0.2;
        data.push({
            x: r * Math.cos(angle) + (Math.random() - 0.5) * 0.05,
            y: r * Math.sin(angle) + (Math.random() - 0.5) * 0.05,
            label: 0
        });
    }
    // 外圆 (标签1)
    for (let i = 0; i < 50; i++) {
        const angle = 2 * Math.PI * (i / 50);
        const r = 1.5 + Math.random() * 0.3;
        data.push({
            x: r * Math.cos(angle) + (Math.random() - 0.5) * 0.05,
            y: r * Math.sin(angle) + (Math.random() - 0.5) * 0.05,
            label: 1
        });
    }
    return data;
}
```

- [ ] **Step 2: 添加数据集加载和状态管理**

```javascript
// 全局状态
const state = {
    currentDataset: 'breast-cancer',
    normalized: false,
    k: 5,
    testPoint: null,
    neighbors: []
};

// 加载数据集
function loadDataset(datasetKey) {
    const dataset = DATASETS[datasetKey];
    state.currentDataset = datasetKey;
    state.testPoint = null;
    state.neighbors = [];
    updateVisualization();
    updateTeachingNotes(dataset.description);
}
```

- [ ] **Step 3: 提交**

```bash
git add index.html
git commit -m "feat: add dataset definitions and loading logic"
```

---

### Task 4: KNN核心算法实现

**Files:**
- Modify: `index.html` (添加KNN算法)

- [ ] **Step 1: 添加欧氏距离计算函数**

```javascript
/**
 * 计算两个点的欧氏距离
 * @param {Object} p1 - 点1 {x, y}
 * @param {Object} p2 - 点2 {x, y}
 * @returns {number} 欧氏距离
 */
function euclideanDistance(p1, p2) {
    return Math.sqrt(Math.pow(p1.x - p2.x, 2) + Math.pow(p1.y - p2.y, 2));
}
```

- [ ] **Step 2: 添加KNN分类算法**

```javascript
/**
 * K近邻分类算法
 * @param {Object} testPoint - 待分类点 {x, y}
 * @param {Array} trainData - 训练数据 [{x, y, label}, ...]
 * @param {number} k - 近邻数量
 * @returns {Object} {label: 预测标签, neighbors: 最近k个邻居}
 */
function knn_predict(testPoint, trainData, k) {
    // 计算测试点到所有训练点的距离
    const distances = trainData.map(point => ({
        point: point,
        distance: euclideanDistance(testPoint, point)
    }));
    
    // 按距离升序排序
    distances.sort((a, b) => a.distance - b.distance);
    
    // 取前k个最近邻
    const kNeighbors = distances.slice(0, k);
    
    // 多数投票
    const voteCount = {};
    kNeighbors.forEach(neighbor => {
        const label = neighbor.point.label;
        voteCount[label] = (voteCount[label] || 0) + 1;
    });
    
    // 找出得票最多的类别
    let maxVotes = 0;
    let predictedLabel = 0;
    for (const [label, count] of Object.entries(voteCount)) {
        if (count > maxVotes) {
            maxVotes = count;
            predictedLabel = parseInt(label);
        }
    }
    
    return {
        label: predictedLabel,
        neighbors: kNeighbors,
        voteCount: voteCount
    };
}
```

- [ ] **Step 3: 添加归一化算法**

```javascript
/**
 * Min-Max归一化
 * @param {Array} data - 原始数据
 * @returns {Array} 归一化后的数据
 */
function minMaxNormalize(data) {
    const xValues = data.map(p => p.x);
    const yValues = data.map(p => p.y);
    
    const xMin = Math.min(...xValues);
    const xMax = Math.max(...xValues);
    const yMin = Math.min(...yValues);
    const yMax = Math.max(...yValues);
    
    return data.map(p => ({
        x: (p.x - xMin) / (xMax - xMin),
        y: (p.y - yMin) / (yMax - yMin),
        label: p.label
    }));
}
```

- [ ] **Step 4: 提交**

```bash
git add index.html
git commit -m "feat: implement KNN core algorithm with euclidean distance and voting"
```

---

### Task 5: 决策边界绘制

**Files:**
- Modify: `index.html` (添加决策边界绘制)

- [ ] **Step 1: 添加网格预测函数**

```javascript
/**
 * 生成决策边界网格
 * @param {Array} data - 训练数据
 * @param {number} resolution - 网格分辨率 (默认50x50)
 * @param {number} k - K值
 * @returns {Object} {grid: 2D数组, xRange, yRange}
 */
function generateDecisionBoundary(data, resolution = 50, k = 5) {
    const xValues = data.map(p => p.x);
    const yValues = data.map(p => p.y);
    
    const xMin = Math.min(...xValues);
    const xMax = Math.max(...xValues);
    const yMin = Math.min(...yValues);
    const yMax = Math.max(...yValues);
    
    const xStep = (xMax - xMin) / resolution;
    const yStep = (yMax - yMin) / resolution;
    
    const grid = [];
    
    for (let i = 0; i < resolution; i++) {
        const row = [];
        for (let j = 0; j < resolution; j++) {
            const x = xMin + (i + 0.5) * xStep;
            const y = yMin + (j + 0.5) * yStep;
            const prediction = knn_predict({x, y}, data, k);
            row.push(prediction.label);
        }
        grid.push(row);
    }
    
    return {
        grid: grid,
        xRange: [xMin, xMax],
        yRange: [yMin, yMax],
        xStep: xStep,
        yStep: yStep
    };
}
```

- [ ] **Step 2: 添加决策边界可视化函数**

```javascript
/**
 * 绘制决策边界
 */
function plotDecisionBoundary() {
    const dataset = DATASETS[state.currentDataset];
    const data = state.normalized 
        ? minMaxNormalize(dataset.data) 
        : dataset.data;
    
    const {grid, xRange, yRange} = generateDecisionBoundary(data, 50, state.k);
    
    // 创建决策边界热力图
    const boundaryTrace = {
        x: [],
        y: [],
        z: [],
        type: 'heatmap',
        colorscale: [
            [0, 'rgba(135, 206, 250, 0.6)'],  // 浅蓝色
            [0.5, 'rgba(240, 128, 128, 0.3)'], // 中间色
            [1, 'rgba(240, 128, 128, 0.6)']   // 浅红色
        ],
        showscale: false,
        x0: xRange[0],
        dx: (xRange[1] - xRange[0]) / 50,
        y0: yRange[0],
        dy: (yRange[1] - yRange[0]) / 50
    };
    
    // 填充网格数据
    for (let i = 0; i < grid.length; i++) {
        for (let j = 0; j < grid[i].length; j++) {
            boundaryTrace.x.push(xRange[0] + (i + 0.5) * (xRange[1] - xRange[0]) / 50);
            boundaryTrace.y.push(yRange[0] + (j + 0.5) * (yRange[1] - yRange[0]) / 50);
            boundaryTrace.z.push(grid[i][j]);
        }
    }
    
    return boundaryTrace;
}
```

- [ ] **Step 3: 提交**

```bash
git add index.html
git commit -m "feat: add decision boundary generation and rendering"
```

---

### Task 6: Plotly可视化集成

**Files:**
- Modify: `index.html` (集成Plotly可视化)

- [ ] **Step 1: 添加散点图绘制函数**

```javascript
/**
 * 绘制数据点散点图
 */
function plotScatter() {
    const dataset = DATASETS[state.currentDataset];
    const data = state.normalized 
        ? minMaxNormalize(dataset.data) 
        : dataset.data;
    
    // 按类别分组
    const traces = {};
    data.forEach(point => {
        if (!traces[point.label]) {
            traces[point.label] = {x: [], y: [], mode: 'markers', name: `类别 ${point.label}`};
        }
        traces[point.label].x.push(point.x);
        traces[point.label].y.push(point.y);
    });
    
    return Object.values(traces);
}
```

- [ ] **Step 2: 添加完整可视化更新函数**

```javascript
/**
 * 更新所有可视化
 */
function updateVisualization() {
    const dataset = DATASETS[state.currentDataset];
    const processedData = state.normalized 
        ? minMaxNormalize(dataset.data) 
        : dataset.data;
    
    // 绘制决策边界
    const boundaryTrace = plotDecisionBoundary();
    
    // 绘制散点
    const scatterTraces = plotScatter();
    
    const layout = {
        title: `${dataset.name} - KNN决策边界 (K=${state.k})`,
        xaxis: {title: '特征 1'},
        yaxis: {title: '特征 2'},
        dragmode: 'pan',
        margin: {t: 60, r: 20, b: 60, l: 60}
    };
    
    // 如果有待测点，添加预测可视化
    if (state.testPoint) {
        const prediction = knn_predict(state.testPoint, processedData, state.k);
        state.neighbors = prediction.neighbors;
        
        // 添加待测点
        const testPointTrace = {
            x: [state.testPoint.x],
            y: [state.testPoint.y],
            mode: 'markers',
            marker: {symbol: 'star', size: 20, color: 'green'},
            name: '待测点'
        };
        
        // 添加邻居连线
        const neighborTraces = prediction.neighbors.map((n, i) => ({
            x: [state.testPoint.x, n.point.x],
            y: [state.testPoint.y, n.point.y],
            mode: 'lines',
            line: {color: 'green', width: 1},
            opacity: 0.5 - i * 0.05,
            name: `邻居 ${i+1}`,
            showlegend: false
        }));
        
        Plotly.newPlot('scatter-plot', 
            [boundaryTrace, ...scatterTraces, testPointTrace, ...neighborTraces], 
            layout
        );
        
        // 更新预测结果
        updatePredictionResult(prediction);
    } else {
        Plotly.newPlot('scatter-plot', [boundaryTrace, ...scatterTraces], layout);
    }
}
```

- [ ] **Step 3: 提交**

```bash
git add index.html
git commit -m "feat: integrate Plotly.js visualization with scatter plot and decision boundary"
```

---

### Task 7: 用户交互功能

**Files:**
- Modify: `index.html` (添加用户交互事件处理)

- [ ] **Step 1: 添加事件监听器**

```javascript
// 页面加载完成后初始化
document.addEventListener('DOMContentLoaded', function() {
    // 数据集选择
    document.getElementById('dataset-select').addEventListener('change', function(e) {
        loadDataset(e.target.value);
    });
    
    // 归一化开关
    document.getElementById('normalization-toggle').addEventListener('change', function(e) {
        state.normalized = e.target.checked;
        updateVisualization();
    });
    
    // K值滑块
    document.getElementById('k-slider').addEventListener('input', function(e) {
        state.k = parseInt(e.target.value);
        document.getElementById('k-value-display').textContent = state.k;
        updateVisualization();
    });
    
    // 画布点击事件
    document.getElementById('scatter-plot').on('plotly_click', function(data) {
        const x = data.points[0].x;
        const y = data.points[0].y;
        state.testPoint = {x, y};
        updateVisualization();
    });
    
    // 初始加载
    loadDataset('breast-cancer');
});
```

- [ ] **Step 2: 添加预测结果展示函数**

```javascript
/**
 * 更新预测结果面板
 */
function updatePredictionResult(prediction) {
    const resultDiv = document.getElementById('prediction-details');
    
    let html = `<p><strong>预测类别:</strong> ${prediction.label}</p>`;
    html += `<p><strong>投票统计:</strong></p><ul>`;
    for (const [label, count] of Object.entries(prediction.voteCount)) {
        html += `<li>类别 ${label}: ${count} 票</li>`;
    }
    html += `</ul>`;
    html += `<p><strong>最近 ${state.k} 个邻居:</strong></p><ul>`;
    prediction.neighbors.forEach((n, i) => {
        html += `<li>邻居${i+1}: (${n.point.x.toFixed(3)}, ${n.point.y.toFixed(3)}) - 距离: ${n.distance.toFixed(4)}</li>`;
    });
    html += `</ul>`;
    
    resultDiv.innerHTML = html;
}
```

- [ ] **Step 3: 添加教学说明更新函数**

```javascript
/**
 * 更新教学说明
 */
function updateTeachingNotes(description) {
    const notesDiv = document.getElementById('teaching-notes');
    const kValue = state.k;
    
    let notes = `<p>${description}</p>`;
    
    if (kValue === 1) {
        notes += `<p><strong>K=1:</strong> 决策边界非常细碎，容易过拟合。</p>`;
    } else if (kValue <= 5) {
        notes += `<p><strong>K=${kValue}:</strong> 决策边界适中。</p>`;
    } else if (kValue >= 20) {
        notes += `<p><strong>K=${kValue}:</strong> 决策边界平滑但可能欠拟合。</p>`;
    }
    
    notesDiv.innerHTML = notes;
}
```

- [ ] **Step 4: 提交**

```bash
git add index.html
git commit -m "feat: add user interaction handlers for dataset selection, normalization, and K-value"
```

---

### Task 8: 优化与测试验证

**Files:**
- Modify: `index.html` (性能优化和bug修复)

- [ ] **Step 1: 优化决策边界计算性能**

```javascript
// 使用节流优化频繁更新
let updateTimeout = null;

function debounceUpdateVisualization() {
    if (updateTimeout) clearTimeout(updateTimeout);
    updateTimeout = setTimeout(updateVisualization, 100);
}

// 修改K值滑块事件使用节流
document.getElementById('k-slider').addEventListener('input', function(e) {
    state.k = parseInt(e.target.value);
    document.getElementById('k-value-display').textContent = state.k;
    debounceUpdateVisualization();
});
```

- [ ] **Step 2: 添加数据集描述信息**

```javascript
// 数据集中文描述增强
const DATASET_DESCRIPTIONS = {
    'breast-cancer': {
        title: '乳腺癌诊断数据集',
        features: ['平均半径', '平均平滑度'],
        purpose: '展示未归一化数据的尺度偏差问题',
        scaleIssue: true
    },
    'iris': {
        title: '鸢尾花数据集',
        features: ['花萼长度', '花萼宽度'],
        purpose: '展示基础三分类场景',
        scaleIssue: false
    },
    'moons': {
        title: '月亮数据集',
        features: ['X坐标', 'Y坐标'],
        purpose: '展示KNN处理非线性可分数据的能力',
        scaleIssue: false
    },
    'circles': {
        title: '同心圆数据集',
        features: ['X坐标', 'Y坐标'],
        purpose: '验证KNN在局部邻域分类中的表现',
        scaleIssue: false
    }
};
```

- [ ] **Step 3: 提交**

```bash
git add index.html
git commit -m "perf: optimize visualization update with debouncing"
```

---

### Task 9: 最终功能完善

**Files:**
- Modify: `index.html` (完善界面和功能)

- [ ] **Step 1: 添加响应式布局优化**

```css
@media (max-width: 1024px) {
    .container {
        flex-direction: column;
    }
    .main-canvas {
        flex: none;
        height: 60vh;
    }
    .control-panel {
        flex: none;
        height: 40vh;
    }
}
```

- [ ] **Step 2: 添加清空待测点功能**

```javascript
function clearTestPoint() {
    state.testPoint = null;
    state.neighbors = [];
    updateVisualization();
    document.getElementById('prediction-details').innerHTML = '<p>点击画布添加待测点</p>';
}
```

- [ ] **Step 3: 完善教学说明面板**

```javascript
function updateTeachingNotes(datasetKey) {
    const info = DATASET_DESCRIPTIONS[datasetKey];
    const kValue = state.k;
    
    let html = `<h4>${info.title}</h4>`;
    html += `<p><strong>特征:</strong> ${info.features.join(', ')}</p>`;
    html += `<p><strong>目的:</strong> ${info.purpose}</p>`;
    
    if (info.scaleIssue) {
        html += `<p style="color: #e65100;"><strong>注意:</strong> 未归一化时，半径值(10-25)远大于平滑度值(0.07-0.16)，距离计算会被半径主导。</p>`;
    }
    
    // K值影响说明
    html += `<hr><h4>K值对模型的影响</h4>`;
    if (kValue <= 3) {
        html += `<p>K值较小，模型复杂度高，容易过拟合，决策边界细碎。</p>`;
    } else if (kValue <= 10) {
        html += `<p>K值适中，模型复杂度适中，决策边界较为平滑。</p>`;
    } else {
        html += `<p>K值较大，模型简单，决策边界平滑，但可能欠拟合。</p>`;
    }
    
    document.getElementById('teaching-notes').innerHTML = html;
}
```

- [ ] **Step 4: 提交**

```bash
git add index.html
git commit -m "feat: add responsive layout and enhanced teaching notes"
```

---

### Task 10: GitHub提交与发布

**Files:**
- Modify: `README.md`

- [ ] **Step 1: 创建README.md**

```markdown
# KNN算法可视化教学演示系统

一款基于Web的K-近邻(KNN)算法可视化教学工具，帮助学生直观理解数据归一化和K值对模型的影响。

## 功能特性

- 4个内置经典数据集
- Min-Max归一化实时切换
- K值滑块调节(1-30)
- 50x50决策边界网格
- 点击画布交互式预测
- K近邻可视化连线
- 详细投票统计

## 快速开始

直接在浏览器中打开 `index.html` 文件即可运行。

## 数据集

- 乳腺癌诊断数据集 - 展示尺度偏差问题
- 鸢尾花数据集 - 基础三分类
- 月亮数据集 - 非线性可分
- 同心圆数据集 - 局部邻域分类

## 技术栈

- HTML5/CSS3/JavaScript (ES6+)
- Plotly.js 数据可视化

## 项目结构

```
KNN_Vision/
├── index.html    # 主应用文件
├── SPEC.md       # 需求规格说明
└── README.md     # 项目说明
```
```

- [ ] **Step 2: 添加远程仓库并推送**

```bash
# 创建GitHub仓库后执行
git remote add origin https://github.com/YOUR_USERNAME/KNN_Vision.git
git branch -M main
git push -u origin main
```

- [ ] **Step 3: 提交所有更改**

```bash
git add .
git commit -m "feat: complete KNN visualization teaching system"
git push origin main
```

---

## 自检清单

### 需求覆盖检查

| 需求 | 对应任务 |
|------|----------|
| 4个内置数据集 | Task 3 |
| 数据集切换功能 | Task 7 |
| Min-Max归一化 | Task 4 |
| 归一化开关切换 | Task 7 |
| K值滑块1-30 | Task 7 |
| 50x50决策边界 | Task 5 |
| 点击画布预测 | Task 7 |
| K近邻可视化 | Task 6 |
| 投票统计展示 | Task 7 |
| 教学说明面板 | Task 7, Task 9 |
| 左侧70%画布 | Task 2 |
| 右侧30%控制面板 | Task 2 |
| 响应式布局 | Task 9 |
| 欧氏距离计算 | Task 4 |
| 中文注释 | 所有任务 |

### 代码一致性检查

- `state.normalized` - 一致使用
- `state.k` - 一致使用
- `state.testPoint` - 一致使用
- `state.currentDataset` - 一致使用
- `DATASETS` 对象结构一致

---

## 计划执行

**Plan complete and saved to `docs/superpowers/plans/2026-04-06-knn-visualization-teaching-system.md`**

**两种执行方式:**

1. **Subagent-Driven (推荐)** - 每个任务派发独立子代理，任务间审核，快速迭代

2. **Inline Execution** - 本会话内批量执行，附带检查点审核

**选择哪种方式?**
