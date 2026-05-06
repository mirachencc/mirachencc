---
name: travel-planner
description: 帮用户规划多天旅行行程。当用户说"帮我规划旅行"、"我想去XX"、"安排XX天行程"、"旅游攻略"、"行程规划"时触发。收集需求后自动完成信息检索、动线规划、餐厅推荐，输出完整结构化行程表。
---

# 旅行行程规划助手

## 前置要求
- **OpenClaw浏览器**：必须已安装并配置OpenClaw浏览器工具
- **浏览器状态**：使用前确保浏览器已启动：`openclaw browser start`
- **登录状态**：建议提前登录小红书账号以获得完整搜索功能

## 语言规则
检测用户输入语言，用相同语言输出所有内容。无法识别时默认使用中文。

## 触发条件
当用户提到旅行、出行、行程规划、去某地玩、攻略等相关内容时启动此流程。

---

## 第一步：收集用户需求

主动向用户提问，**一次性**发送以下所有问题，不要分多次问：

```
你好！我来帮你规划一份省心又好玩的旅行行程 🗺️

请告诉我以下信息：

1. 目的地是哪里？（国家/城市，可以是大范围，例如"斯里兰卡"或"泰国北部"）
2. 出行天数？（例如：7天6夜）
3. 从哪里出发？（用于安排第一天交通参考）
4. 旅行风格偏好？（可多选）
   - 🏛️ 人文历史   🌿 自然风光   🏖️ 海滩度假
   - 🍜 美食探索   📸 网红打卡   🎯 体验项目（冲浪/safari等）
5. 行程节奏偏好？
   - 🐢 轻松型（每天2-3个地点，留白时间多）
   - 🚶 标准型（每天3-4个地点）
   - 🏃 特种兵型（每天5+个地点，把景点榨干）
6. 有没有特别想去的地方，或者一定要避开的？（选填）
```

收到用户回答后，**直接进入第二步**，不需要再次确认。

> ✅ 第一步完成。不再追问，立即执行第二步。

---

## 第二步：多源检索攻略信息

> ⚠️ **执行规则：每个信息源只搜索一次，不重复执行。搜索完成后立即进入下一步，不回头补充。**

### 2.1 Google搜索（主要信息源，使用OpenClaw浏览器）

**重要说明**：小红书内容主要为图片形式，无法通过文本提取获取详细攻略信息。因此主要依赖Google搜索获取结构化信息。

使用OpenClaw浏览器CLI命令进行搜索，按以下顺序执行：

1. **打开Google搜索页面**：
   ```bash
   openclaw browser open "https://www.google.com/search?q={目的地}+{天数}天+行程+攻略+2025"
   ```
   - 等待页面加载（约3-5秒）
   
   **内容提取方法**：
   
   a) **提取AI概览和搜索结果**：
   ```bash
   openclaw browser evaluate --fn "() => {
     // 提取Google搜索结果中的结构化信息
     const text = document.body.innerText.substring(0, 10000);
     
     // 分析目的地相关城市
     const cityPatterns = {
       '日本': ['东京', '大阪', '京都', '奈良', '神户', '富士山', '箱根', '镰仓', '横滨'],
       '泰国': ['曼谷', '清迈', '普吉岛', '芭堤雅', '苏梅岛'],
       '韩国': ['首尔', '釜山', '济州岛'],
       '斯里兰卡': ['科伦坡', '康提', '努沃勒埃利耶', '埃拉', '雅拉', '加勒', '尼甘布']
     };
     
     // 根据目的地选择城市列表
     let cities = [];
     for (const country in cityPatterns) {
       if (text.includes(country)) {
         cities = cityPatterns[country].filter(city => text.includes(city));
         break;
       }
     }
     
     // 提取景点信息
     const attractionPattern = /[\\u4e00-\\u9fffA-Za-z]{2,20}(?:寺|宫|公园|山|湖|海滩|古城|塔|博物馆)/g;
     const attractions = text.match(attractionPattern) || [];
     
     // 提取行程天数
     let daysSuggested = {天数};
     const dayMatch = text.match(/(\\d+)[\\s\\-]+(?:天|日)(?:\\s*行程|\\s*游|\\s*攻略)/);
     if (dayMatch) daysSuggested = parseInt(dayMatch[1]);
     
     // 提取交通信息
     const transportKeywords = ['新干线', 'JR', '地铁', '巴士', '包车', '火车', '飞机'];
     const transportInfo = transportKeywords.filter(keyword => text.includes(keyword));
     
     return JSON.stringify({
       source: 'Google搜索',
       cities: cities,
       attractions: [...new Set(attractions)].slice(0, 15),
       daysSuggested: daysSuggested,
       transportKeywords: transportInfo,
       excerpt: text.substring(0, 1000).replace(/\\s+/g, ' ')
     }, null, 2);
   }"
   ```
   
   b) **获取页面快照**（备用）：
   ```bash
   openclaw browser snapshot --limit 800
   ```
   
   **从提取的内容中分析：**
   - 推荐城市和景点（高频出现的名称）
   - 行程路线建议
   - 交通方式信息
   - 实用贴士

2. **小红书搜索结果页深度提取**：
   ```bash
   openclaw browser open "https://www.xiaohongshu.com/search_result?keyword={目的地}+{天数}天+旅行攻略"
   sleep 3
   openclaw browser evaluate --fn "function() {
     const text = document.body.innerText;
     const lines = text.split('\\n').filter(l => l.trim().length > 5);
     
     // 提取相关笔记标题
     const relevantLines = lines.filter(line => 
       line.includes('{目的地}') && 
       (line.includes('天') || line.includes('行程') || line.includes('攻略'))
     );
     
     const titles = relevantLines.slice(0, 15).map(line => line.substring(0, 150));
     
     // 提取预算信息
     const budgets = [];
     const budgetRegex = /人均[^\\n]{0,20}\\d{3,5}/g;
     let match;
     while ((match = budgetRegex.exec(text)) !== null) {
       budgets.push(match[0]);
     }
     
     // 提取高频城市（基于常见目的地）
     const commonCities = {
       '日本': ['东京', '大阪', '京都', '奈良', '富士山', '箱根', '镰仓', '北海道'],
       '泰国': ['曼谷', '清迈', '普吉岛', '芭堤雅', '苏梅岛'],
       '韩国': ['首尔', '釜山', '济州岛'],
       '斯里兰卡': ['科伦坡', '康提', '努沃勒埃利耶', '埃拉', '雅拉', '加勒', '尼甘布']
     };
     
     let mentionedCities = [];
     for (const country in commonCities) {
       if (text.includes(country)) {
         mentionedCities = commonCities[country].filter(city => text.includes(city));
         break;
       }
     }
     
     // 分析路线类型
     let routeType = '';
     if (text.includes('阪进东出') || text.includes('大阪进东京出')) {
       routeType = '阪进东出（大阪进东京出）';
     } else if (text.includes('东进阪出') || text.includes('东京进大阪出')) {
       routeType = '东进阪出（东京进大阪出）';
     }
     
     return JSON.stringify({
       source: '小红书搜索结果页',
       noteCount: relevantLines.length,
       titles: titles,
       budgets: [...new Set(budgets)].slice(0, 5),
       mentionedCities: mentionedCities,
       routeType: routeType,
       analysis: '从小红书标题中提取预算、路线、热门城市信息',
       sampleTitles: titles.slice(0, 5)
     }, null, 2);
   }"
   ```

### 2.2 城市间交通查询（使用Google搜索）

如果目的地涉及多个城市，使用Google搜索：
```bash
openclaw browser open "https://www.google.com/search?q={目的地}+城市间交通+火车+大巴+时间+2025"
```
或
```bash
openclaw browser open "https://www.google.com/search?q={出发城市}+到+{下一城市}+交通+时间+价格"
```

从搜索结果中提取：交通方式、大约时长、费用参考、是否需要提前预订。

**提取函数示例**：
```bash
openclaw browser evaluate --fn "() => {
  const text = document.body.innerText.substring(0, 5000);
  return {
    route: '{出发城市} → {下一城市}',
    transport: text.match(/(?:交通|方式)[：:]?\\s*([^\\n]{10,50})/)?.[1] || '未知',
    duration: text.match(/(?:时间|车程|需要)[：:]?\\s*([^\\n]{10,30})/)?.[1] || '未知',
    price: text.match(/(?:价格|费用|票价)[：:]?\\s*([^\\n]{10,50})/)?.[1] || '未知',
    booking: text.includes('提前预订') ? '建议提前预订' : '可现场购买'
  };
}"
```

### 2.3 使用evaluate进行智能内容提取

#### 专用提取函数库：
创建可重用的JavaScript提取函数，通过`openclaw browser evaluate`执行：

```javascript
// Google搜索结果提取函数（主要信息源）
function extractGoogleTravelInfo() {
  const text = document.body.innerText.substring(0, 15000);
  const results = {
    cities: [],
    attractions: [],
    itinerary: [],
    transport: [],
    tips: [],
    budget: ''
  };
  
  // 提取城市信息（基于常见目的地）
  const commonCities = {
    '日本': ['东京', '大阪', '京都', '奈良', '神户', '富士山', '箱根', '镰仓', '横滨'],
    '泰国': ['曼谷', '清迈', '普吉岛', '芭堤雅', '苏梅岛', '甲米'],
    '韩国': ['首尔', '釜山', '济州岛', '仁川'],
    '越南': ['河内', '胡志明市', '岘港', '下龙湾', '会安'],
    '斯里兰卡': ['科伦坡', '康提', '努沃勒埃利耶', '埃拉', '雅拉', '加勒', '尼甘布']
  };
  
  // 识别目的地并提取相关城市
  for (const country in commonCities) {
    if (text.includes(country)) {
      results.cities = commonCities[country].filter(city => text.includes(city));
      break;
    }
  }
  
  // 提取景点信息
  const attractionPattern = /[\\u4e00-\\u9fffA-Za-z]{2,20}(?:寺|宫|公园|山|湖|海滩|古城|塔|博物馆|乐园|神社)/g;
  const attractions = text.match(attractionPattern) || [];
  results.attractions = [...new Set(attractions)].slice(0, 20);
  
  // 提取行程安排
  const itineraryPattern = /(?:第[一二三四五六七八九十]天|Day\\s*\\d+)[^\\n]{20,150}/g;
  results.itinerary = text.match(itineraryPattern) || [];
  
  // 提取交通信息
  const transportKeywords = ['新干线', 'JR', '地铁', '巴士', '包车', '火车', '飞机', '出租车', '租车'];
  results.transport = transportKeywords.filter(keyword => text.includes(keyword));
  
  // 提取小贴士
  const tipLines = text.split('\\n').filter(line => 
    line.includes('注意') || line.includes('建议') || line.includes('贴士') || 
    line.includes('提醒') || line.includes('必备') || line.includes('重要')
  );
  results.tips = tipLines.slice(0, 10);
  
  // 提取预算信息
  const budgetMatch = text.match(/(?:预算|花费|费用|人均)[：:]?\\s*([^\\n]{10,80})/);
  results.budget = budgetMatch ? budgetMatch[1] : '';
  
  return results;
}

// 小红书标题提取函数（仅作参考）
function extractXiaohongshuTitles() {
  const titles = [];
  const elements = document.querySelectorAll('.title, h3, [class*=\"title\"], .note-title');
  
  elements.forEach(el => {
    const title = el.innerText.trim();
    if (title && title.length > 5 && titles.length < 15) {
      titles.push(title.substring(0, 150));
    }
  });
  
  return {
    source: '小红书标题参考',
    count: titles.length,
    titles: titles,
    note: '小红书内容为图片形式，无法提取详细攻略信息，仅作热门程度参考'
  };
}
          content: content.substring(0, 500),
          likes: parseInt(likes) || 0,
          time: time,
          relevance: (title + content).includes('攻略') || (title + content).includes('行程') ? 1 : 0
        });
      }
    }
  });
  
  // 方法2：如果没找到卡片，提取整个页面文本
  if (results.length === 0) {
    const fullText = document.body.innerText;
    // 提取可能包含行程信息的部分
    const lines = fullText.split('\n').filter(line => 
      line.includes('天') || line.includes('行程') || line.includes('攻略') || 
      line.includes('景点') || line.includes('推荐')
    );
    lines.forEach((line, index) => {
      if (index < 20) {
        results.push({
          type: '文本行',
          content: line.trim()
        });
      }
    });
  }
  
  return JSON.stringify(results, null, 2);
}

// Google Maps餐厅提取函数
function extractGoogleMapsRestaurants() {
  const restaurants = [];
  const results = document.querySelectorAll('[role="article"], .section-result, .place-result');
  
  results.forEach((result, index) => {
    if (index < 10) {
      const text = result.innerText || '';
      const nameMatch = text.match(/^[A-Za-z\u4e00-\u9fff][^\n\d]{2,50}/m);
      const ratingMatch = text.match(/(\d\.\d)\(/);
      const priceMatch = text.match(/(Rs\s*[\d,]+|人均\s*[¥￥]\s*[\d,]+)/);
      
      if (nameMatch) {
        restaurants.push({
          name: nameMatch[0].trim(),
          rating: ratingMatch ? ratingMatch[1] : '未知',
          price: priceMatch ? priceMatch[1] : '未知',
          excerpt: text.substring(0, 200).replace(/\n/g, ' '),
          index: index + 1
        });
      }
    }
  });
  
  return JSON.stringify(restaurants, null, 2);
}

// 通用旅游信息提取函数
function extractTravelInfo() {
  const info = {
    cities: [],
    attractions: [],
    daysSuggested: null,
    tips: []
  };
  
  const text = document.body.innerText.substring(0, 10000);
  
  // 提取城市名（中文城市名模式）
  const cityMatches = text.match(/[北上广深杭成武南重天西郑长哈沈大济厦福]\S{1,5}市|[A-Z][a-z]+(?:\s+[A-Z][a-z]+)*/g);
  if (cityMatches) {
    info.cities = [...new Set(cityMatches)].slice(0, 10);
  }
  
  // 提取景点（包含常见景点后缀）
  const attractionMatches = text.match(/[\u4e00-\u9fffA-Za-z]{2,30}(?:景区|公园|寺庙|海滩|山|岛|湖|博物馆|遗址|广场)/g);
  if (attractionMatches) {
    info.attractions = [...new Set(attractionMatches)].slice(0, 15);
  }
  
  // 提取天数建议
  const dayMatch = text.match(/(\d+)[\s\-~]+(?:天|日)(?:\s*行程|\s*游)/);
  if (dayMatch) {
    info.daysSuggested = parseInt(dayMatch[1]);
  }
  
  // 提取小贴士
  const tipLines = text.split('\n').filter(line => 
    line.includes('注意') || line.includes('建议') || line.includes('贴士') || 
    line.includes('提醒') || line.includes('禁忌')
  );
  info.tips = tipLines.slice(0, 10);
  
  return JSON.stringify(info, null, 2);
}
```

#### 执行提取的命令：
```bash
# 小红书内容提取
openclaw browser evaluate --fn "(${extractXiaohongshuContent.toString()})();"

# Google Maps餐厅提取  
openclaw browser evaluate --fn "(${extractGoogleMapsRestaurants.toString()})();"

# 通用旅游信息提取
openclaw browser evaluate --fn "(${extractTravelInfo.toString()})();"
```

#### 信息整合流程：
1. **执行提取函数**获取结构化JSON数据
2. **解析JSON结果**，提取关键信息
3. **去重和排序**（按相关性、评分、点赞数等）
4. **生成初步行程框架**

#### 整合结果：
将以上提取的信息整合，得出：
- 推荐游览城市列表及顺序
- 每个城市建议停留天数  
- 核心景点清单（标注游览时长，例如：狮子岩 约3-4小时）
- 城市间交通建议

---

## 第三步：基于提取信息的智能深度搜索

使用第二步提取的结构化信息，进行**自动化深度搜索**：

### 3.1 景点详情批量搜索
对提取的核心景点列表，自动批量搜索：

```bash
# 景点详情搜索函数
function searchAttractionDetails(attractions) {
  const results = [];
  attractions.forEach((attraction, index) => {
    if (index < 8) { // 限制数量
      // 打开新标签搜索
      console.log(`搜索景点: ${attraction}`);
      // 实际执行：openclaw browser open "https://www.google.com/search?q=${encodeURIComponent(attraction)}+开放时间+门票"
      
      // 提取信息函数
      const extractDetails = () => {
        const text = document.body.innerText;
        const hoursMatch = text.match(/(?:开放|营业)时间[：:]?\s*([^\n]{10,50})/);
        const priceMatch = text.match(/(?:门票|价格)[：:]?\s*([^\n]{10,50})/);
        const timeMatch = text.match(/(?:建议|游览)时间[：:]?\s*([^\n]{10,50})/);
        
        return {
          attraction: attraction,
          openingHours: hoursMatch ? hoursMatch[1] : '未知',
          ticketPrice: priceMatch ? priceMatch[1] : '未知',
          suggestedTime: timeMatch ? timeMatch[1] : '2-3小时',
          source: window.location.href
        };
      };
      
      // 这里实际会执行 evaluate
      results.push({
        attraction: attraction,
        status: '待搜索'
      });
    }
  });
  return results;
}
```

**执行策略：**
1. 对前5个核心景点进行详情搜索
2. 每个搜索在新标签页打开
3. 使用`evaluate`提取结构化信息
4. 汇总所有景点详情

### 3.2 城市信息深度搜索
基于提取的城市列表，搜索每个城市的详细信息：

```bash
# 对每个城市执行
openclaw browser evaluate --fn "() => {
  const cities = ${JSON.stringify(extractedCities)}; // 从第二步获取
  const cityInfo = [];
  
  cities.forEach(city => {
    // 实际会执行：openclaw browser open "https://www.google.com/search?q=${city}+旅游攻略+必去"
    const extractCityInfo = () => {
      const text = document.body.innerText.substring(0, 5000);
      const attractions = text.match(/[\\u4e00-\\u9fffA-Za-z]{2,30}(?:景点|景区|公园|博物馆)/g) || [];
      const bestSeason = text.match(/(?:最佳|适合)季节[：:]?\\s*([^\\n]{10,50})/);
      const specials = text.match(/(?:特色|必吃|必玩)[：:]?\\s*([^\\n]{10,100})/);
      
      return {
        city: city,
        topAttractions: [...new Set(attractions)].slice(0, 5),
        bestSeason: bestSeason ? bestSeason[1] : '全年',
        specialActivities: specials ? specials[1] : '未知',
        source: window.location.href
      };
    };
    
    cityInfo.push({ city: city, status: '待搜索' });
  });
  
  return JSON.stringify(cityInfo, null, 2);
}"
```

### 3.3 智能交通规划搜索
基于城市顺序，自动搜索交通方案：

```bash
# 交通规划函数
function searchTransportation(citySequence) {
  const routes = [];
  
  for (let i = 0; i < citySequence.length - 1; i++) {
    const from = citySequence[i];
    const to = citySequence[i + 1];
    
    // 搜索交通信息
    const query = `${from} 到 ${to} 交通 时间 价格`;
    console.log(`搜索交通: ${query}`);
    
    // 实际执行：openclaw browser open "https://www.google.com/search?q=${encodeURIComponent(query)}"
    
    const extractTransport = () => {
      const text = document.body.innerText;
      const transportMatch = text.match(/(?:交通方式|乘坐)[：:]?\\s*([^\\n]{10,50})/);
      const timeMatch = text.match(/(?:时间|时长|需要)[：:]?\\s*([^\\n]{10,50})/);
      const priceMatch = text.match(/(?:价格|费用|大概)[：:]?\\s*([^\\n]{10,50})/);
      
      return {
        route: `${from} → ${to}`,
        transportation: transportMatch ? transportMatch[1] : '汽车/火车',
        duration: timeMatch ? timeMatch[1] : '未知',
        cost: priceMatch ? priceMatch[1] : '未知',
        bookingTip: text.includes('提前预订') ? '建议提前预订' : '现场购买'
      };
    };
    
    routes.push({
      from: from,
      to: to,
      query: query,
      status: '待搜索'
    });
  }
  
  return routes;
}
```

---

## 第四步：自动化执行流程示例

### 完整执行流程示例（斯里兰卡7天行程）：

```bash
# 1. 启动浏览器
openclaw browser start

# 2. 小红书初步搜索
openclaw browser open "https://www.xiaohongshu.com/search_result?keyword=斯里兰卡+7天+旅行攻略"

# 3. 提取小红书内容
openclaw browser evaluate --fn "() => {
  ${extractXiaohongshuContent.toString()}
  return extractXiaohongshuContent();
}"

# 4. 解析提取结果，获取城市和景点列表
# 假设提取到：cities = ['科伦坡', '康提', '努沃勒埃利耶', '埃拉', '雅拉', '美瑞莎']
# attractions = ['佛牙寺', '皇家植物园', '霍顿平原', '小亚当峰', '雅拉国家公园']

# 5. 景点详情搜索（示例：佛牙寺）
openclaw browser open "https://www.google.com/search?q=佛牙寺+开放时间+门票价格+游览时长"
openclaw browser evaluate --fn "() => {
  const text = document.body.innerText;
  return {
    attraction: '佛牙寺',
    openingHours: text.match(/(?:开放|营业)时间[：:]?\\s*([^\\n]{10,50})/)?.[1] || '未知',
    ticketPrice: text.match(/(?:门票|价格)[：:]?\\s*([^\\n]{10,50})/)?.[1] || '未知',
    suggestedTime: text.match(/(?:建议|游览)时间[：:]?\\s*([^\\n]{10,50})/)?.[1] || '2-3小时'
  };
}"

# 6. 城市信息搜索（示例：康提）
openclaw browser open "https://www.google.com/search?q=康提+旅游攻略+必去景点"
openclaw browser evaluate --fn "() => {
  const text = document.body.innerText.substring(0, 5000);
  return {
    city: '康提',
    topAttractions: (text.match(/[\\u4e00-\\u9fffA-Za-z]{2,30}(?:景点|景区|公园|博物馆)/g) || []).slice(0, 5),
    bestSeason: text.match(/(?:最佳|适合)季节[：:]?\\s*([^\\n]{10,50})/)?.[1] || '全年',
    specialActivities: text.match(/(?:特色|必吃|必玩)[：:]?\\s*([^\\n]{10,100})/)?.[1] || '未知'
  };
}"

# 7. 交通搜索（示例：科伦坡→康提）
openclaw browser open "https://www.google.com/search?q=科伦坡+到+康提+交通+时间+价格"
openclaw browser evaluate --fn "() => {
  const text = document.body.innerText;
  return {
    route: '科伦坡 → 康提',
    transportation: text.match(/(?:交通方式|乘坐)[：:]?\\s*([^\\n]{10,50})/)?.[1] || '汽车/火车',
    duration: text.match(/(?:时间|时长|需要)[：:]?\\s*([^\\n]{10,50})/)?.[1] || '未知',
    cost: text.match(/(?:价格|费用|大概)[：:]?\\s*([^\\n]{10,50})/)?.[1] || '未知'
  };
}"

# 8. Google Maps餐厅搜索（示例：康提餐厅）
openclaw browser open "https://www.google.com/maps/search/康提+餐厅"
openclaw browser evaluate --fn "() => {
  ${extractGoogleMapsRestaurants.toString()}
  return extractGoogleMapsRestaurants();
}"
```

### 执行优化策略：
1. **并行搜索**：可以同时打开多个标签页进行不同搜索
2. **结果缓存**：将提取的结果保存到临时文件供后续使用
3. **错误重试**：如果某个搜索失败，自动重试或使用备用关键词
4. **进度跟踪**：记录已完成的搜索和待完成的搜索

---

## 第五步：生成框架行程

基于第二、三、四步提取的信息，生成 DAY1 到最后一天的**框架行程**，每天包含：
- 所在城市（来自提取的城市列表）
- 主要景点（来自提取的景点列表，根据用户节奏偏好筛选）
- 住宿推荐区域（基于城市信息搜索）
- 交通安排（基于交通搜索结果）

**生成算法：**
```javascript
function generateItinerary(extractedData, userPreferences) {
  const { cities, attractions, transportInfo, restaurantInfo } = extractedData;
  const { pace, style, days } = userPreferences;
  
  // 根据天数分配城市
  const daysPerCity = Math.max(1, Math.floor(days / cities.length));
  
  // 生成每日行程
  const itinerary = [];
  let day = 1;
  let cityIndex = 0;
  
  while (day <= days && cityIndex < cities.length) {
    const city = cities[cityIndex];
    const cityAttractions = attractions.filter(a => 
      a.includes(city) || attractionsForCity[city]?.includes(a)
    ).slice(0, pace === '轻松型' ? 2 : pace === '标准型' ? 3 : 4);
    
    itinerary.push({
      day: day,
      city: city,
      attractions: cityAttractions,
      transport: transportInfo.find(t => t.to === city),
      restaurants: restaurantInfo.filter(r => r.location?.includes(city)).slice(0, 2)
    });
    
    day++;
    if (day % daysPerCity === 0) cityIndex++;
  }
  
  return itinerary;
}
```

框架行程生成后，**直接进入第六步细化**，不需要等待用户确认。

---

## 第六步：逐天细化行程+餐厅推荐

对每一天，按以下步骤处理：

### 6.1 景点时间安排
根据景点游览时长，合理分配上午/下午/傍晚时段：
- 早上出发时间一般为 08:00-09:00
- 景点之间预留 30-45 分钟交通时间
- 每天结束时间不超过 21:00

### 6.2 餐厅推荐（使用OpenClaw浏览器）

对每天的午餐和晚餐位置，使用OpenClaw浏览器搜索Google Maps：

```bash
# 搜索当天景点附近的餐厅
openclaw browser open "https://www.google.com/maps/search/{当天主要景点}+附近+餐厅"
```

**执行步骤：**
1. 打开Google Maps搜索页面
2. 等待页面加载完成（约3-5秒）
3. **提取餐厅信息**：
   ```bash
   openclaw browser evaluate --fn "() => {
     const results = document.querySelectorAll('[role=\"article\"], .section-result, [data-tooltip]');
     let restaurants = [];
     results.forEach((result, index) => {
       if (index < 5) { // 只取前5个结果
         const text = result.innerText || result.textContent || '';
         const nameMatch = text.match(/^[A-Za-z][^\\n\\d]{2,50}/);
         const ratingMatch = text.match/(\\d\\.\\d)\\(/);
         const priceMatch = text.match(/Rs\\s*([\\d,]+)/);
         
         if (nameMatch) {
           restaurants.push({
             name: nameMatch[0].trim(),
             rating: ratingMatch ? ratingMatch[1] : '未知',
             price: priceMatch ? 'Rs ' + priceMatch[1] : '未知',
             description: text.substring(0, 150)
           });
         }
       }
     });
     return JSON.stringify(restaurants, null, 2);
   }"
   ```
4. 如果evaluate失败，使用快照：`openclaw browser snapshot --limit 800`

**筛选标准：**
- 距离当天行程景点或下一个目的地 **30分钟以内**
- Google Maps 评分 **4.0 以上**（从快照中识别评分）
- 每餐推荐 **2-3家**，不要太多
- 包含：餐厅名称（中英文）、菜系、人均价格、推荐理由

**备用方案（小红书餐厅推荐）：**
```bash
openclaw browser open "https://www.xiaohongshu.com/search_result?keyword={城市}+餐厅+推荐"
```
- 如果小红书有登录状态，可以获取更多本地人推荐

### 6.3 住宿推荐

每天行程结尾推荐住宿区域和1-2家酒店：

**使用浏览器搜索住宿信息：**
```bash
openclaw browser open "https://www.google.com/search?q={城市}+推荐住宿区域+旅游"
```
或
```bash
openclaw browser open "https://www.google.com/maps/search/{城市}+酒店"
```

**从搜索结果中提取：**
1. 住宿区域建议（如：康提湖附近、美瑞莎海滩沿线等）
2. 1-2家具体酒店名称
3. 大概价位范围（人民币）
4. 推荐理由（位置便利性、景观、设施等）

---

## 第七步：输出完整行程表

所有天数细化完成后，输出完整的 Markdown 格式行程表。

> ⚠️ **执行规则：行程表输出完成后，立即停止所有操作，等待用户反馈。不主动继续执行任何步骤，不自动优化或补充内容。**

### 输出格式模板：

```markdown
# 🗺️ {目的地} {天数}天行程规划

## 行程概览
- 📍 目的地：{目的地}
- 📅 行程天数：{天数}
- ✈️ 出发城市：{出发城市}
- 🎯 旅行风格：{风格偏好}
- 💡 行程亮点：{用一句话概括这次行程的特色}

---

## DAY 1｜{城市名} — {当天主题，例如"落地探索古城"}

**住宿区域推荐：** {区域名} | 推荐酒店：{酒店名}（约 ¥{价格}/晚）

| 时间 | 地点 | 分类 | 交通 | 预估费用 | 备注 |
|------|------|------|------|----------|------|
| 09:00 | {景点名称（英文名）} | 🏛️ 景点 | {交通方式+时长} | {门票价格} | {推荐说明，例如：建议2小时，注意着装要求} |
| 12:30 | {餐厅名称（英文名）} | 🍽️ 午餐 | 步行5分钟 | 人均 ¥{价格} | {菜系+推荐理由} |
| 14:00 | {景点名称} | 📸 景点 | {交通方式} | {费用} | {说明} |
| 18:30 | {餐厅名称} | 🍽️ 晚餐 | {交通} | 人均 ¥{价格} | {推荐理由} |

**🚗 今日交通小结：** {今天主要交通方式和费用}

**💡 今日贴士：** {1-2条重要注意事项}

---

## DAY 2｜{城市} — {主题}
{重复以上格式}

---

## 💰 行程预算参考

| 项目 | 预估费用（人民币）|
|------|----------------|
| 国际机票 | ¥{价格区间} |
| 城市间交通 | ¥{总计} |
| 住宿（{天数}晚）| ¥{总计} |
| 景点门票 | ¥{总计} |
| 餐饮 | ¥{总计} |
| 其他（签证/小费等）| ¥{估算} |
| **合计** | **¥{总计}** |

---

## 📋 出行必知

- 🛂 签证：{签证要求和办理方式}
- 💊 健康：{疫苗/防疫建议}
- 💰 货币：{当地货币+汇率参考}
- 📱 通讯：{SIM卡/流量建议}
- 🌡️ 气候：{旅行季节建议}
- ⚠️ 注意事项：{文化禁忌/安全提示}
```

---

## 重要执行原则

### 防止循环执行
1. **每步只执行一次**：每个步骤执行完成后立即进入下一步，不重复执行
2. **明确终止条件**：输出完整行程表后任务结束，停止所有操作
3. **不主动优化**：除非用户明确要求修改，否则不自动重新生成或补充内容
4. **搜索次数上限**：每个关键词只搜索一次，搜索失败则切换备选方案，不反复重试

### 浏览器使用规范
1. **使用OpenClaw浏览器CLI**：所有网页搜索必须使用`openclaw browser`命令
2. **等待页面加载**：执行`openclaw browser open`后等待3-5秒让页面完全加载
3. **获取页面内容**：使用`openclaw browser snapshot`获取页面可访问内容
4. **多标签管理**：可以同时打开多个标签页进行并行搜索

### 信息质量原则
1. **真实性优先**：所有景点、餐厅、交通信息都必须来自实际搜索结果，不要凭记忆编造数据
2. **时效性**：优先使用最近6个月内的攻略内容
3. **素人优先**：小红书搜索时优先非广告的普通用户分享
4. **距离合理**：餐厅和下一个景点的距离不超过30分钟车程
5. **时间合理**：每天行程不要过于紧凑，景点间预留交通缓冲时间
6. **费用透明**：尽量给出实际价格范围，用当地货币标注并换算人民币参考

### 故障处理
1. **搜索失败处理**：如果某个网页无法访问，自动切换备选搜索源
2. **登录要求处理**：如果网站需要登录（如小红书），尝试使用已登录状态或切换到Google搜索
3. **网络超时处理**：如果命令执行超时，重试一次或切换到备用方案
4. **内容提取失败**：如果无法从快照中提取足够信息，尝试截图分析或使用其他关键词搜索

---

## 信息源说明与优化策略

### 主要信息源
1. **Google搜索**（核心信息源）
   - ✅ 可提取：行程框架、景点信息、交通方式、实用贴士
   - ✅ 优势：文本内容丰富，有AI概览功能，无需登录
   - ✅ 可靠性：高，信息更新及时
   - 📊 提取深度：深度（可获取详细攻略内容）

2. **Google Maps**（地点和餐厅信息）
   - ✅ 可提取：餐厅评分、价格、菜系、用户评价、地点详情
   - ✅ 优势：实时数据，用户评价真实，信息准确
   - ✅ 可靠性：高，数据来自实际用户
   - 📊 提取深度：深度（结构化数据）

### 重要辅助信息源
3. **小红书**（中国最大旅行信息平台）
   - ✅ **优势**：素人分享多，广告比例低，内容真实实用
   - ✅ **可提取**：笔记标题、作者、发布时间、互动数据、预算范围、行程天数、路线类型
   - ⚠️ **限制**：详细攻略内容在图片中，需要OCR识别
   - 📊 **提取深度**：
     - **文本层**：中（可获取元数据和部分文本）
     - **图片层**：需要OCR工具支持
   - 💡 **使用价值**：预算参考、热门路线、真实用户经验

### OCR增强选项
如需提取小红书图片中的详细攻略内容，可安装OCR工具：

```bash
# 方案A：安装Tesseract OCR（推荐）
brew install tesseract tesseract-lang
pip3 install pytesseract pillow

# 方案B：使用在线OCR API
# 需要申请百度/腾讯/Google OCR API key
```

**OCR提取流程**：
1. 使用`openclaw browser screenshot`截图小红书页面
2. 使用OCR工具识别图片中的文字
3. 提取行程详情、景点推荐、实用贴士
4. 整合到行程规划中

### 混合信息源策略
根据可用工具选择最佳组合：

| 场景 | 推荐信息源组合 | 说明 |
|------|----------------|------|
| **无OCR工具** | Google搜索(主) + Google Maps(辅) + 小红书标题参考 | 基础可靠方案 |
| **有OCR工具** | 小红书(主) + Google搜索(辅) + Google Maps(辅) | 获取最真实用户分享 |
| **深度规划** | 小红书(OCR) + Google搜索 + Google Maps + 其他平台 | 最全面信息收集 |

### 小红书文本提取优化
即使没有OCR，通过以下方法可提取更多信息：

1. **深度DOM遍历**：提取所有文本节点和属性
2. **点击笔记详情**：尝试获取更多页面内容
3. **解析JSON数据**：查找页面内嵌的结构化数据
4. **多页面采集**：采集多个笔记标题和元数据

**实际可提取内容示例**：
- `国庆人均6500日本7天6晚行程攻略及费用`（标题）
- `七想想八想想 2025-10-13 96`（作者、时间、互动）
- `预算：4984-6500元`（费用范围）
- `路线：阪进东出`（行程类型）
- `特色：不早起不赶路`（行程风格）

### OCR提取实现示例
如果安装了OCR工具，可使用以下流程：

```python
#!/usr/bin/env python3
import pytesseract
from PIL import Image
import subprocess
import json
import os

def extract_xiaohongshu_with_ocr():
    """使用OCR提取小红书攻略内容"""
    
    # 1. 截图小红书页面
    print("正在截图小红书页面...")
    result = subprocess.run(
        ["openclaw", "browser", "screenshot"],
        capture_output=True,
        text=True
    )
    
    # 解析截图路径
    screenshot_path = None
    for line in result.stdout.split('\n'):
        if line.startswith('MEDIA:'):
            screenshot_path = line.replace('MEDIA:', '').strip()
            break
    
    if not screenshot_path or not os.path.exists(screenshot_path):
        print("截图失败")
        return None
    
    print(f"截图保存至: {screenshot_path}")
    
    # 2. 使用OCR识别文字
    try:
        image = Image.open(screenshot_path)
        text = pytesseract.image_to_string(image, lang='chi_sim+eng')
        
        # 3. 提取行程相关信息
        extracted_info = {
            'destination': None,
            'days': None,
            'budget': None,
            'itinerary': [],
            'tips': []
        }
        
        # 分析OCR识别结果
        lines = text.split('\n')
        for line in lines:
            line = line.strip()
            if not line:
                continue
                
            # 提取目的地
            if '日本' in line or '泰国' in line or '韩国' in line:
                extracted_info['destination'] = line
                
            # 提取天数
            import re
            day_match = re.search(r'(\d+)[\s\-~]+天', line)
            if day_match:
                extracted_info['days'] = int(day_match.group(1))
                
            # 提取预算
            budget_match = re.search(r'(?:人均|预算|花费|费用)[：:]?\s*[¥￥]?\s*(\d{3,5})', line)
            if budget_match:
                extracted_info['budget'] = budget_match.group(1)
                
            # 提取行程安排
            if 'Day' in line or 'DAY' in line or '第' in line and '天' in line:
                extracted_info['itinerary'].append(line)
                
            # 提取小贴士
            if '注意' in line or '建议' in line or '贴士' in line or '提醒' in line:
                extracted_info['tips'].append(line)
        
        return extracted_info
        
    except Exception as e:
        print(f"OCR处理失败: {e}")
        return None

# 使用示例
if __name__ == "__main__":
    info = extract_xiaohongshu_with_ocr()
    if info:
        print(json.dumps(info, ensure_ascii=False, indent=2))
```

### 技能执行建议
根据用户环境和需求，动态选择最佳策略：

1. **询问用户偏好**：
   ```
   请问你希望主要参考哪个平台的信息？
   - A. 小红书（素人分享真实，但需要OCR工具）
   - B. Google搜索（信息全面，无需额外工具）
   - C. 混合使用（综合各平台优势）
   ```

2. **检测环境能力**：
   ```bash
   # 自动检测是否支持OCR
   which tesseract && echo "OCR可用" || echo "OCR不可用"
   ```

3. **自适应执行**：
   - 如果OCR可用且用户偏好小红书 → 使用OCR提取详细攻略
   - 如果OCR不可用 → 使用Google搜索为主，小红书标题参考为辅
   - 如果用户无明确偏好 → 使用混合策略，以Google搜索为核心

---

## 第八步：截图与OCR增强能力

### 8.1 截图功能集成

OpenClaw浏览器提供强大的截图功能，可用于获取页面完整内容：

#### 基本截图命令：
```bash
# 1. 普通截图（当前视图）
openclaw browser screenshot

# 2. 全页面截图（滚动截图）
openclaw browser screenshot --full-page

# 3. 指定元素截图
openclaw browser screenshot --selector ".note-item"

# 4. 截图保存路径
# 截图自动保存到：~/.openclaw/media/browser/{uuid}.jpg
# 返回格式：MEDIA:/path/to/screenshot.jpg
```

#### 截图在行程规划中的应用：

**应用场景1：小红书搜索结果页截图**
```bash
# 打开小红书搜索页面
openclaw browser open "https://www.xiaohongshu.com/search_result?keyword={目的地}+{天数}天+攻略"

# 等待页面加载
sleep 3

# 截图保存
openclaw browser screenshot --full-page
# 返回：MEDIA:/Users/mirachen/.openclaw/media/browser/9156eb94-6bc2-4444-90df-6a840ba7fc01.jpg
```

**应用场景2：Google Maps餐厅列表截图**
```bash
# 搜索餐厅
openclaw browser open "https://www.google.com/maps/search/{城市}+餐厅"

# 等待加载
sleep 3

# 截图餐厅列表
openclaw browser screenshot --selector '[role="article"]'
```

**应用场景3：景点详情页截图**
```bash
# 搜索景点详情
openclaw browser open "https://www.google.com/search?q={景点}+开放时间+门票"

# 截图关键信息区域
openclaw browser screenshot --selector "#search"
```

### 8.2 OCR识别集成

#### 前提条件：安装OCR工具
```bash
# 安装Tesseract OCR（开源免费）
brew install tesseract tesseract-lang

# 安装Python OCR库
pip3 install pytesseract pillow
```

#### OCR识别函数库：

```python
#!/usr/bin/env python3
"""
OCR工具库 - 集成到travel-planner技能中
"""

import pytesseract
from PIL import Image
import subprocess
import os
import json
import re

class XiaohongshuOCR:
    """小红书OCR识别类"""
    
    def __init__(self):
        self.screenshot_dir = os.path.expanduser("~/.openclaw/media/browser")
        
    def take_screenshot(self, url, wait_seconds=3):
        """截图指定网页"""
        print(f"截图网页: {url}")
        
        # 打开网页
        result = subprocess.run(
            ["openclaw", "browser", "open", url],
            capture_output=True,
            text=True
        )
        
        # 等待页面加载
        import time
        time.sleep(wait_seconds)
        
        # 截图
        result = subprocess.run(
            ["openclaw", "browser", "screenshot", "--full-page"],
            capture_output=True,
            text=True
        )
        
        # 解析截图路径
        screenshot_path = None
        for line in result.stdout.split('\n'):
            if line.startswith('MEDIA:'):
                screenshot_path = line.replace('MEDIA:', '').strip()
                break
        
        if screenshot_path and os.path.exists(screenshot_path):
            print(f"✅ 截图成功: {screenshot_path}")
            print(f"   文件大小: {os.path.getsize(screenshot_path) / 1024:.1f} KB")
            return screenshot_path
        else:
            print("❌ 截图失败")
            return None
    
    def ocr_recognize(self, image_path, language='chi_sim+eng'):
        """OCR识别图片文字"""
        if not os.path.exists(image_path):
            print(f"❌ 图片不存在: {image_path}")
            return None
        
        try:
            # 打开图片
            image = Image.open(image_path)
            print(f"图片尺寸: {image.size}")
            
            # OCR识别
            print(f"OCR识别中（语言: {language}）...")
            text = pytesseract.image_to_string(image, lang=language)
            
            print(f"✅ 识别完成，共 {len(text)} 字符")
            return text
            
        except Exception as e:
            print(f"❌ OCR识别失败: {e}")
            return None
    
    def analyze_xiaohongshu_content(self, ocr_text):
        """分析小红书OCR识别结果"""
        if not ocr_text:
            return None
        
        # 提取日本相关行
        japan_lines = []
        for line in ocr_text.split('\n'):
            line = line.strip()
            if line and '日本' in line and len(line) > 10:
                japan_lines.append(line)
        
        # 提取预算信息
        budgets = []
        budget_patterns = [
            r'人均[^\\n]{0,20}\\d{3,5}',
            r'预算[^\\n]{0,20}\\d{3,5}',
            r'花费[^\\n]{0,20}\\d{3,5}',
            r'¥\\s*\\d{3,5}',
            r'￥\\s*\\d{3,5}'
        ]
        
        for pattern in budget_patterns:
            matches = re.findall(pattern, ocr_text)
            budgets.extend(matches)
        
        # 提取城市信息
        cities = ['东京', '大阪', '京都', '奈良', '富士山', '箱根', '镰仓', '北海道', '冲绳']
        found_cities = []
        for city in cities:
            if city in ocr_text:
                found_cities.append(city)
        
        # 提取行程天数
        days = []
        day_patterns = [
            r'\\d+[\\s\\-~]+天',
            r'\\d+[\\s\\-~]+日',
            r'第[一二三四五六七八九十]+天'
        ]
        
        for pattern in day_patterns:
            matches = re.findall(pattern, ocr_text)
            days.extend(matches)
        
        # 提取路线类型
        route_type = ''
        if '阪进东出' in ocr_text or '大阪进东京出' in ocr_text:
            route_type = '阪进东出（大阪进东京出）'
        elif '东进阪出' in ocr_text or '东京进大阪出' in ocr_text:
            route_type = '东进阪出（东京进大阪出）'
        
        return {
            'total_chars': len(ocr_text),
            'japan_lines': japan_lines[:20],
            'budgets': list(set(budgets))[:10],
            'cities': found_cities,
            'days': list(set(days))[:5],
            'route_type': route_type,
            'sample_text': ocr_text[:1000]
        }
    
    def process_xiaohongshu_search(self, keyword):
        """完整的小红书搜索处理流程"""
        print(f"搜索小红书: {keyword}")
        
        # 1. 构建URL
        url = f"https://www.xiaohongshu.com/search_result?keyword={keyword}"
        
        # 2. 截图
        screenshot_path = self.take_screenshot(url, wait_seconds=5)
        if not screenshot_path:
            return None
        
        # 3. OCR识别
        ocr_text = self.ocr_recognize(screenshot_path)
        if not ocr_text:
            return None
        
        # 4. 分析内容
        analysis = self.analyze_xiaohongshu_content(ocr_text)
        
        # 5. 保存结果
        result_file = screenshot_path.replace('.jpg', '_analysis.json').replace('.png', '_analysis.json')
        with open(result_file, 'w', encoding='utf-8') as f:
            json.dump({
                'keyword': keyword,
                'screenshot': screenshot_path,
                'ocr_text_length': len(ocr_text),
                'analysis': analysis
            }, f, ensure_ascii=False, indent=2)
        
        print(f"✅ 分析结果已保存: {result_file}")
        return analysis

# 使用示例
if __name__ == "__main__":
    ocr = XiaohongshuOCR()
    
    # 测试小红书搜索
    result = ocr.process_xiaohongshu_search("日本+7天+行程")
    
    if result:
        print("\\n📊 小红书OCR分析结果:")
        print(f"  总字符数: {result['total_chars']}")
        print(f"  日本相关行: {len(result['japan_lines'])}")
        print(f"  预算范围: {', '.join(result['budgets']) if result['budgets'] else '未找到'}")
        print(f"  提到城市: {', '.join(result['cities']) if result['cities'] else '未找到'}")
        print(f"  行程天数: {', '.join(result['days']) if result['days'] else '未找到'}")
        print(f"  路线类型: {result['route_type']}")
        
        if result['japan_lines']:
            print("\\n📝 小红书内容摘要:")
            for i, line in enumerate(result['japan_lines'][:10], 1):
                print(f"  {i}. {line}")
```

### 8.3 集成到技能流程中

#### 更新后的第二步（支持OCR）：

```bash
# 检测OCR是否可用
if command -v tesseract &> /dev/null; then
    echo "✅ OCR工具可用，使用截图+OCR提取小红书内容"
    
    # 使用OCR提取小红书
    python3 -c "
    from xiaohongshu_ocr import XiaohongshuOCR
    ocr = XiaohongshuOCR()
    result = ocr.process_xiaohongshu_search('{目的地}+{天数}天+行程')
    
    if result:
        print('小红书OCR提取结果:')
        print(f'预算范围: {result[\"budgets\"]}')
        print(f'热门城市: {result[\"cities\"]}')
        print(f'路线类型: {result[\"route_type\"]}')
    "
else
    echo "⚠️ OCR工具不可用，使用文本提取"
    
    # 使用文本提取（原有方法）
    openclaw browser evaluate --fn "function() {
        const text = document.body.innerText;
        const lines = text.split('\\n').filter(l => l.trim().length > 5);
        
        const relevantLines = lines.filter(line => 
            line.includes('{目的地}') && 
            (line.includes('天') || line.includes('行程') || line.includes('攻略'))
        );
        
        const titles = relevantLines.slice(0, 15).map(line => line.substring(0, 150));
        
        return JSON.stringify({
            source: '小红书文本提取',
            titles: titles,
            note: 'OCR不可用，仅提取文本内容'
        }, null, 2);
    }"
fi
```

#### 技能执行流程优化：

**有OCR时的流程：**
1. 小红书搜索 → 截图 → OCR识别 → 详细内容提取
2. Google搜索 → 补充信息
3. Google Maps → 餐厅信息
4. 整合所有信息生成行程

**无OCR时的流程：**
1. 小红书搜索 → 文本提取 → 标题和元数据
2. Google搜索 → 主要信息源
3. Google Maps → 餐厅信息
4. 整合信息生成行程

### 8.4 截图管理

#### 截图文件管理：
```bash
# 查看所有截图
ls -lh ~/.openclaw/media/browser/*.jpg

# 按时间排序
ls -lt ~/.openclaw/media/browser/*.jpg | head -10

# 清理旧截图（保留最近10个）
cd ~/.openclaw/media/browser
ls -t *.jpg | tail -n +11 | xargs rm -f
```

#### 截图命名规范：
建议为截图添加有意义的名称：
```bash
# 截图并重命名
openclaw browser screenshot
# 获取截图路径后重命名
mv ~/.openclaw/media/browser/xxx.jpg ~/.openclaw/media/browser/日本7天行程_小红书搜索.jpg
```

### 8.5 性能优化建议

1. **缓存OCR结果**：相同关键词的搜索结果可以缓存
2. **批量处理**：多个截图可以批量OCR识别
3. **图片预处理**：裁剪、调整对比度提高OCR准确率
4. **并行处理**：多个截图可以并行OCR识别

### 技术限制说明
1. **图片内容无法提取**：`openclaw browser evaluate`只能提取HTML中的文本内容，无法OCR识别图片文字
2. **需要登录的内容受限**：某些网站需要登录才能查看完整内容
3. **页面结构变化**：网站改版可能导致提取函数失效
4. **网络访问限制**：某些网站可能限制自动化访问
5. **OCR准确率**：图片质量、字体、背景影响OCR识别准确率

### 技能优化建议
1. **优先使用Google搜索**作为主要信息源
2. **加强Google Maps搜索**获取餐厅和地点信息
3. **建立常见目的地知识库**减少对实时搜索的依赖
4. **定期更新提取函数**适应网站改版
5. **集成OCR能力**提升小红书内容提取质量

### 版本更新记录
- **v2.2** (2026-03-26)：集成截图和OCR能力，支持小红书图片内容提取
- **v2.1** (2026-03-26)：调整信息源策略，明确小红书仅作参考，主要依赖Google搜索
- **v2.0** (2026-03-26)：支持Google Maps实时餐厅搜索，增强内容提取功能
- **v1.0** (初始版本)：基础行程规划功能
