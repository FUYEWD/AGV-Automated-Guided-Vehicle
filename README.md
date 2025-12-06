# 🤖 AGV 無人搬運車系統 - 跨品牌協同架構

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)

一個支援**跨品牌協同**的智能無人搬運車系統。解決現實場景中多品牌機器人共用空間時的衝突問題，透過統一通訊協議實現異構機器人的協同導航。

## 💡 設計理念

### 問題場景
在餐廳、醫院、倉庫等場景，經常出現多品牌機器人共用空間的情況：
- **現況**：不同品牌機器人各自為政，只能靠感測器互相避障
- **問題**：感測器失效時會卡在走廊互相等待，造成交通癱瘓
- **根因**：缺乏統一的通訊協議和協調機制

### 解決方案
本專案實現 **跨品牌機器人協同框架（Cross-Brand Robot Coordination Framework）**：
```
┌─────────────────────────────────────────────────────┐
│         統一協調層 (Universal Coordination Layer)      │
│  - 路權仲裁  - 路徑預約  - 衝突解決  - 優先級管理    │
└─────────────────────────────────────────────────────┘
         ↓              ↓              ↓
    品牌A機器人      品牌B機器人      本專案AGV
    (適配器接入)    (適配器接入)     (原生支援)
```

## ✨ 核心特性

### 🌐 跨品牌協同
- **統一通訊協議**：基於 MQTT/DDS 的輕量級協議，任何品牌都可接入
- **品牌適配器**：為主流品牌（普渡、擎朗等）提供即插即用適配器
- **路權管理**：動態分配走廊路權，避免死鎖
- **路徑預約系統**：機器人提前廣播路徑意圖，預防衝突

### 🚦 智能交通管理
- **虛擬交通燈**：在關鍵路口實施通行管制
- **動態優先級**：緊急任務、低電量等獲得優先通行權
- **死鎖檢測**：自動識別並解決多車死鎖
- **避讓協商**：機器人之間協商最優避讓方案

### 🎯 原有功能
- SLAM 建圖與定位
- 自主導航與路徑規劃
- 動態避障
- 電量管理
- Web 監控介面


  模擬器連結:https://agv000.yay.boo/

## 🏗️ 系統架構

```
agv-system/
├── src/
│   ├── coordination_layer/          # ⭐ 協調層（核心）
│   │   ├── protocol/                # 統一通訊協議
│   │   ├── traffic_manager/         # 交通管理
│   │   ├── conflict_resolver/       # 衝突解決
│   │   ├── path_reservation/        # 路徑預約
│   │   └── deadlock_detector/       # 死鎖檢測
│   ├── brand_adapters/              # ⭐ 品牌適配器
│   │   ├── pudu_adapter/            # 普渡機器人
│   │   ├── keenon_adapter/          # 擎朗機器人
│   │   ├── bear_adapter/            # 熊貓機器人
│   │   └── generic_adapter/         # 通用適配器
│   ├── agv_control/                 # 運動控制
│   ├── agv_navigation/              # 導航
│   ├── agv_perception/              # 感知
│   └── agv_communication/           # 通訊模組
├── protocols/                       # ⭐ 協議規範文檔
├── config/
├── simulation/
│   └── multi_brand_scenario/        # ⭐ 多品牌仿真場景
└── docs/
    ├── coordination_protocol.md     # 協調協議文檔
    └── brand_integration_guide.md   # 品牌接入指南
```

## 📡 跨品牌通訊協議

### 協議設計原則
1. **輕量級**：最小化通訊開銷
2. **即時性**：<100ms 延遲
3. **容錯性**：支援部分節點離線
4. **安全性**：防止惡意機器人

### 消息類型

```python
# 1. 心跳消息（每秒廣播）
{
  "type": "heartbeat",
  "robot_id": "brand_a_robot_01",
  "brand": "pudu",
  "position": {"x": 10.5, "y": 20.3, "theta": 1.57},
  "velocity": {"vx": 0.5, "vy": 0, "omega": 0},
  "status": "moving",
  "battery": 75
}

# 2. 路徑預約（規劃路徑時發送）
{
  "type": "path_reservation",
  "robot_id": "brand_b_robot_02",
  "path": [
    {"x": 10, "y": 20, "t": 1670000000},
    {"x": 11, "y": 20, "t": 1670000002},
    {"x": 12, "y": 20, "t": 1670000004}
  ],
  "priority": "normal",
  "task_type": "delivery"
}

# 3. 路權請求（衝突時發送）
{
  "type": "right_of_way_request",
  "robot_id": "our_agv_01",
  "location": {"x": 15, "y": 25},
  "conflicting_with": ["brand_a_robot_01"],
  "reason": "low_battery",  // 或 "emergency", "blocked"
  "priority": "high"
}

# 4. 協商響應
{
  "type": "negotiation_response",
  "robot_id": "brand_a_robot_01",
  "action": "yield",  // 或 "proceed", "wait", "reroute"
  "estimated_clear_time": 5  // 秒
}
```

## 🚀 快速開始

### 1. 基礎安裝

```bash
git clone https://github.com/yourusername/agv-system.git
cd agv-system
pip3 install -r requirements.txt
colcon build
source install/setup.bash
```

### 2. 啟動協調服務器

```bash
# 啟動 MQTT 協調服務器
ros2 launch coordination_layer coordination_server.launch.py

# 啟動交通管理器
ros2 run coordination_layer traffic_manager
```

### 3. 多品牌仿真場景

```bash
# 啟動包含3個品牌機器人的餐廳場景
ros2 launch agv_simulation restaurant_multi_brand.launch.py

# 場景包含：
# - 2台普渡送餐機器人
# - 1台擎朗機器人
# - 2台本專案AGV
# 在狹窄走廊中模擬你描述的場景
```

### 4. 接入現有品牌機器人

```bash
# 為品牌機器人部署適配器（在其控制電腦上）
cd brand_adapters/pudu_adapter
python3 adapter_node.py --robot-ip 192.168.1.100 --mqtt-broker 192.168.1.10

# 適配器會：
# 1. 訂閱品牌機器人的位置和狀態
# 2. 將其轉換為統一協議並廣播
# 3. 接收協調指令並轉換為品牌機器人的控制指令
```

## 🎮 實際案例演示

### 場景：餐廳走廊衝突

```python
# 運行你描述的場景測試
ros2 run coordination_layer corridor_deadlock_test.py

# 測試流程：
# 1. 兩台不同品牌機器人從兩端駛入窄走廊
# 2. 傳統方式：互相檢測，嘗試避障，最後卡住 ❌
# 3. 協同方式：
#    - 機器人A提前廣播路徑
#    - 機器人B檢測到衝突
#    - 協調層判定A優先（基於任務急迫性）
#    - B主動退讓到側邊等待區
#    - A通過後B繼續前進 ✅
```

## 📊 協調策略

### 1. 優先級定義
```python
PRIORITY_LEVELS = {
    "emergency": 100,      # 緊急任務（如醫療配送）
    "low_battery": 80,     # 低電量返回充電
    "occupied": 70,        # 載有物品/客人
    "normal": 50,          # 一般任務
    "idle": 10            # 空閒狀態
}
```

### 2. 衝突解決規則
1. **優先級高者優先**
2. **先到先得**：同優先級時，先預約路徑者優先
3. **就近避讓**：距離避讓點近的機器人避讓
4. **成本最小化**：選擇總體延遲最小的方案

### 3. 死鎖處理
```python
# 檢測到循環等待時
if detect_deadlock(robots_in_conflict):
    # 強制一台機器人後退
    victim = select_victim(robots, criteria="lowest_priority")
    send_command(victim, "reverse", distance=2.0)
```

## 🔌 品牌適配器開發

### 為新品牌開發適配器

```python
from coordination_layer import RobotAdapter

class MyBrandAdapter(RobotAdapter):
    def __init__(self, robot_ip):
        super().__init__()
        self.robot = MyBrandAPI(robot_ip)
    
    def get_position(self):
        """獲取機器人位置"""
        pos = self.robot.get_current_pose()
        return {"x": pos.x, "y": pos.y, "theta": pos.theta}
    
    def get_status(self):
        """獲取機器人狀態"""
        return self.robot.get_status()
    
    def execute_command(self, command):
        """執行協調指令"""
        if command.action == "yield":
            self.robot.stop()
        elif command.action == "reroute":
            self.robot.set_goal(command.alternative_path)
```

## 📱 監控介面

Web 介面新增**協同視圖**：
- 🗺️ 所有品牌機器人的實時位置
- 🛣️ 預約路徑可視化
- ⚠️ 衝突事件記錄
- 📊 協調效率統計
- 🎛️ 手動介入控制

訪問 `http://localhost:5000/coordination`

## 🧪 測試場景

```bash
# 1. 走廊對峙測試
ros2 launch tests corridor_conflict_test.launch.py

# 2. 十字路口測試
ros2 launch tests intersection_test.launch.py

# 3. 大規模壓力測試（10+ 機器人）
ros2 launch tests stress_test.launch.py

# 4. 單點故障測試（協調服務器掉線）
ros2 launch tests fault_tolerance_test.launch.py
```

## 🌍 開放標準倡議

我們呼籲建立 **機器人共空間協同標準（Robot Shared-Space Coordination Standard）**：

### 願景
- 讓不同品牌機器人能安全高效地共享空間
- 推動行業採用統一協調協議
- 開源所有協議規範和參考實現

### 加入我們
- ⭐ Star 本專案支持開放標準
- 🤝 貢獻你的品牌適配器
- 📢 向你的機器人廠商推薦此協議
- 📝 參與協議規範討論

## 📚 文檔

- [協調協議詳細規範](docs/coordination_protocol.md)
- [品牌接入指南](docs/brand_integration_guide.md)
- [衝突解決算法](docs/conflict_resolution.md)
- [安全性與隱私](docs/security.md)
- [FAQ - 為什麼需要跨品牌協同？](docs/faq.md)

## 🤝 貢獻

特別歡迎：
- 新品牌機器人的適配器
- 更優的協調算法
- 實際場景測試數據
- 協議改進建議

## 🎯 開發路線圖

- [x] 基礎協調協議設計
- [x] MQTT 通訊層實現
- [x] 基本衝突檢測與解決
- [ ] 主流品牌適配器（普渡、擎朗、熊貓）
- [ ] 機器學習增強的衝突預測
- [ ] V2X 整合（與智慧建築系統協同）
- [ ] 提交 IEEE/ISO 標準提案

## 📄 授權

MIT License - 促進開放標準的採用

## 🙏 致謝

感謝所有在現實場景中發現問題並推動解決方案的人，這個專案的靈感來自於火鍋店送餐機器人"卡住"的真實案例。


**「讓機器人學會禮讓，而不是對峙」**

⭐ 如果你也曾見過機器人卡在走廊，請給個 Star 支持開放協同標準！
