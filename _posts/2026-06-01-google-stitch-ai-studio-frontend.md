---
title: "用 Google Stitch + AI Studio 设计地图前端页面"
date: 2026-06-01
categories:
  - 技术
  - AI
tags:
  - Google Stitch
  - Google AI Studio
  - Gemini
  - 地图
  - 前端开发
---

想象一下：你要做一个带地图的网页——某个城市的餐厅推荐、旅行足迹展示、房源分布——但你还不想手写几百行样式和数据映射代码。过去这件事很麻烦：设计地图样式、准备数据、写交互逻辑、调试样式，每个环节都得自己来。

现在有了 **Google Stitch** 和 **Google AI Studio** 这对组合，这件事的流程变成了：

**AI Studio 帮你生成配置 → Stitch 可视化微调 → 前端代码直接跑起来**

几十分钟就能出一个可交互的地图页面。这篇博客介绍它们的用法和组合拳。

## 一、Google Stitch：地图即配置

Google Stitch 是 Google Maps Platform 在 2023 年推出的**声明式地图样式编辑器**。它的核心哲学是：**地图即配置（Map as Configuration）**——你用 JSON 文件定义地图长什么样、显示什么数据、点击发生什么事，Stitch 会帮你渲染出来。

### 工作流

```
配置文件 (JSON) → Stitch Studio 导入 → 生成 Map ID → 前端 SDK 加载
```

### 一个 Stitch 配置长这样

```json
{
  "displayName": "我的餐厅地图",
  "style": {
    "base": "google://light",
    "custom": [
      {
        "featureType": "road",
        "elementType": "geometry",
        "stylers": [{ "color": "#FFFFFF" }, { "weight": 2 }]
      },
      {
        "featureType": "water",
        "elementType": "geometry.fill",
        "stylers": [{ "color": "#A0D8F1" }]
      }
    ]
  },
  "layers": [
    {
      "id": "restaurants",
      "type": "geojson",
      "source": "./data/restaurants.geojson",
      "style": {
        "icon": "restaurant",
        "color": "#FF6B6B",
        "size": "medium"
      },
      "interaction": {
        "click": {
          "action": "showInfoWindow",
          "template": "<h3>{name}</h3><p>{cuisine} · ⭐{rating}</p>"
        }
      }
    }
  ],
  "controls": ["zoom", "fullscreen", "mapType"]
}
```

你定义了三层东西：

- **基础样式（base + custom）**：地图背景色、道路、水体配色
- **数据图层（layers）**：从 GeoJSON 加载餐厅数据，定义图标样式
- **交互行为（interaction）**：点击标记弹出信息窗口

### 关键特性

| 特性 | 说明 |
|------|------|
| **声明式配置** | 用 JSON 定义一切，可版本控制 |
| **实时预览** | Stitch Studio 修改即时反映 |
| **数据绑定** | 连接 BigQuery 或 GeoJSON 数据源 |
| **条件样式** | 基于数据属性动态设置颜色/图标 |
| **团队协作** | 配置可导出分享，支持 Git |

## 二、Google AI Studio：自然语言生成代码

Google AI Studio（现整合为 **Gemini API 开发者平台**）是 Google 基于 Gemini 模型的 **AI 开发和原型设计平台**。

它能做的事情非常直接：**你说需求，它出代码。**

### 快速生成前端页面原型

在 AI Studio 的 Prompt 区域输入：

```
创建一个包含 Google 地图的 HTML 页面：
- 地图中心在东京 (35.6762, 139.6503)
- 三个标记：东京塔（景点）、Sushi Zanmai（餐厅）、Park Hyatt（酒店）
- 点击标记弹出信息窗口
- 现代化 UI，响应式设计
```

AI Studio 会直接输出一个完整的 HTML 文件——包含 CSS、地图初始化、标记和信息窗口——你可以直接复制运行。

### 让 AI Studio 生成 Stitch 配置

更有意思的是，你可以在 AI Studio **直接描述需求生成 Stitch 配置**：

```
帮我生成 Google Stitch 的 JSON 配置文件：
1. 基础样式: dark 风格
2. 数据图层: 从 GeoJSON 加载全球城市地标
3. 条件样式: 根据人口数量，标记大小分 large/medium/small
4. 交互: 点击显示城市名称和人口
5. 控件: 缩放、全屏、图层切换
```

AI 会帮你输出完整的 Stitch JSON，包括 `featureType` 覆盖和条件样式映射。

### AI Studio 的关键能力

| 特性 | 说明 |
|------|------|
| **多模型** | Gemini 1.5 Pro/Flash、Gemini 2.0 |
| **即时运行** | 部分代码可直接在浏览器中预览 |
| **Prompt 模板** | 内置常见场景模板 |
| **多模态** | 支持图片 + 文本描述生成 UI |
| **导出** | 一键导出到 Colab 或本地 |

## 三、组合工作流：Stitch × AI Studio

这两个工具互补得特别好：

| 环节 | 用 Stitch | 用 AI Studio |
|------|-----------|-------------|
| 样式设计 | 定义地图样式和数据图层 | 生成初始配置模板 |
| 代码生成 | 提供 Map ID 给前端 | 生成前端页面代码 |
| 调试迭代 | 实时预览地图效果 | 快速调整 Prompt 优化输出 |
| 原型开发 | 精细控制地图行为 | 快速搭建 UI 框架 |

### 推荐流程

```
Step 1: [AI Studio] → 描述需求 → 生成 Stitch 配置 + 前端代码框架
Step 2: [Stitch Studio] → 导入配置 → 微调样式 → 获取 Map ID
Step 3: [AI Studio] → 根据 Map ID → 完善前端集成代码
Step 4: [本地开发] → 部署完整应用
```

### 实战：餐厅推荐地图

**Step 1 - 在 AI Studio 生成 Stitch 配置：**

```
生成 Google Stitch 配置用于餐厅推荐地图：
- 基础样式: 明亮风格
- 三个图层：餐厅（红）、咖啡馆（橙）、酒吧（紫）
- 点击显示营业时间、评分、人均消费
- 支持按评分 >4.0 高亮
- 控制组件：缩放、图层切换、全屏
```

**Step 2 - 导入 Stitch Studio 获取 Map ID**

**Step 3 - 在 AI Studio 生成前端代码：**

```
基于 Stitch 配置 (Map ID: abc123)，生成完整前端页面：
- 顶部搜索栏
- 左侧筛选面板（价格范围、评分）
- 地图占 70% 宽度
- 点击标记在右侧显示详细卡片
- Material Design 风格
```

**Step 4 - 完整的 HTML 代码：**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Stitch × AI Studio 示例</title>
  <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500&display=swap" rel="stylesheet">
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: 'Roboto', sans-serif; display: flex; height: 100vh; }
    #sidebar {
      width: 320px; padding: 20px; background: #f8f9fa;
      overflow-y: auto;
    }
    #map-container { flex: 1; }
    #map { width: 100%; height: 100%; }
    .filter-group { margin-bottom: 20px; }
    .filter-group h3 {
      color: #5f6368; font-size: 14px;
      font-weight: 500; margin-bottom: 8px;
    }
    #search-input {
      width: 100%; padding: 10px;
      border: 1px solid #dadce0; border-radius: 8px;
      font-size: 14px;
    }
  </style>
</head>
<body>
  <div id="sidebar">
    <h2 style="margin-bottom: 20px;">🍽️ 美食地图</h2>
    <div class="filter-group">
      <h3>🔍 搜索</h3>
      <input id="search-input" placeholder="输入城市名..." />
    </div>
    <div class="filter-group">
      <h3>💰 价格范围</h3>
      <select id="price-filter">
        <option value="all">全部</option>
        <option value="$">$ 实惠</option>
        <option value="$$">$$ 中等</option>
        <option value="$$$">$$$ 高端</option>
      </select>
    </div>
    <div class="filter-group">
      <h3>⭐ 最低评分</h3>
      <input type="range" id="rating-filter" min="0" max="5" step="0.5" value="0" />
      <span id="rating-value">0.0</span>
    </div>
    <div id="detail-card"></div>
  </div>
  <div id="map-container">
    <div id="map"></div>
  </div>

  <script>
    const MAP_ID = 'YOUR_STITCH_MAP_ID';

    const map = new google.maps.Map(document.getElementById('map'), {
      mapId: MAP_ID,
      center: { lat: 35.6762, lng: 139.6503 },
      zoom: 12,
    });

    // 使用 Stitch 定义的数据图层
    const restaurantLayer = map.getFeatureLayer('restaurants');
    const cafeLayer = map.getFeatureLayer('cafes');
    const barLayer = map.getFeatureLayer('bars');

    // 筛选逻辑
    document.getElementById('price-filter')
      .addEventListener('change', applyFilters);
    document.getElementById('rating-filter')
      .addEventListener('input', applyFilters);

    function applyFilters() {
      const price = document.getElementById('price-filter').value;
      const rating = parseFloat(
        document.getElementById('rating-filter').value
      );
      document.getElementById('rating-value').textContent = rating.toFixed(1);

      // Stitch 支持通过条件样式实现筛选
      restaurantLayer.style = {
        filter: `price == "${price}" || "${price}" == "all"`,
      };
    }
  </script>
  <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_KEY&map_ids=${MAP_ID}"></script>
</body>
</html>
```

## 四、最佳实践

### 先用 AI Studio 探索

不要直接从 Stitch Studio 开始。先在 AI Studio 里描述你的应用场景，让 AI 生成 **初步的 Stitch 配置 + 前端代码框架**，迭代 Prompt 直到满意再进入下一步。

### 用 Stitch Studio 精细调校

把 AI 生成的配置导入 Stitch Studio，可视化地微调颜色、图标、交互，最终导出 Map ID。

### AI Studio 生成集成代码

最后回到 AI Studio，把 Map ID 嵌入，生成完整的前端页面。

### 注意事项

| 注意点 | 说明 |
|--------|------|
| **API 密钥** | 前端不要暴露 API Key，用域名限制或代理 |
| **GeoJSON 大小** | 单图层建议不超过 10MB |
| **AI 输出验证** | AI 生成的坐标、数据引用需要人工检查 |
| **成本控制** | 动态数据加载会增加调用次数，使用 Tileset |

## 五、总结

Google Stitch 和 AI Studio 的组合，让地图前端开发从「手工配置+手写代码」变成了 **「自然语言描述 → AI 生成 → 可视化微调 → 一键部署」**。

- Stitch 负责地图的**长相和行为**，用 JSON 声明式配置
- AI Studio 负责帮你**生成这些配置和前端代码**，用自然语言驱动
- 两者结合，从零到可运行的地图页面，可能只需要一两个小时

这对组合尤其适合那些「不需要 100% 定制、但需要快速出原型」的场景——个人项目、Demo、数据分析的展示页面。下次要做地图页面，不妨试试这条路。
