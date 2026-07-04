# 🗺️ Gaode Map MCP Server

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue)](https://www.python.org/)
[![FastMCP](https://img.shields.io/badge/FastMCP-0.x-green)](https://github.com/kukapay/fastmcp)
[![License](https://img.shields.io/badge/License-GPL%203.0-blue)](LICENSE)

一个基于 [FastMCP](https://github.com/kukapay/fastmcp) 构建的 **高德地图 MCP 服务器**，将高德地图 Web API 封装为 MCP 工具，可被任何 MCP 客户端（如 Claude Desktop、VS Code + Continue / Cline、LangChain Agent 等）直接调用。

> 💡 部分命名方式和逻辑借鉴了 [qiao101660/gaode_mcp](https://github.com/qiao101660/gaode_mcp)，在此致谢。

---

## ✨ 已实现的工具

| 工具名 | 功能 | 核心输入 |
|---|---|---|
| `geocoding` | 地理编码：地址 → 城市编码 / 区域编码 / 经纬度 | 结构化地址 + 可选城市 |
| `public_transit_route_planning` | 公共交通路线规划（地铁/公交/步行/出租） | 起终点经纬度 + 城市编码 |
| `bicycle_route_schedule` | 自行车骑行路线规划 | 起终点经纬度 |
| `automobile_driving_route_schedule` | 驾车路线规划（支持策略/途经点/车牌限行） | 起终点经纬度 + 可选参数 |
| `keyword_search_of_poi` | 关键字搜索 POI（美食/购物/景点等） | 关键字 + 可选区域 |
| `poi_around_given_location` | 周边 POI 搜索 | 中心点经纬度 + 半径 + 类型 |

---

## 📁 项目结构

```
.
├── .env                      # API Key 和接口 URL 配置
├── gaode_map_config.py       # 全局配置（超时、请求头）
├── gaode_map_mcp.py          # MCP 服务器主入口（含所有工具定义）
└── README.md
```

---

## 🔧 配置

### 1. 获取高德地图 API Key

前往 [高德开放平台](https://lbs.amap.com/) 注册并创建应用，获取 **Web 服务** 类型的 API Key。

### 2. 克隆仓库

```bash
git clone https://github.com/ColtM1873/Gaode-Map-MCP
cd Gaode-Map-MCP
```

### 3. 安装依赖

```bash
pip install fastmcp httpx python-dotenv
```

### 4. 配置 `.env`

编辑 `.env` 文件，填入你的 API Key：

```ini
GAODE_MAP_API_KEY="你的API Key，不加引号也可以"
```

> 其余 URL 一般无需修改，除非高德 API 地址发生变更。

---

## 🚀 使用方式

### 方式一：VS Code / Cline / Continue 等 MCP 客户端

在客户端的 MCP 配置中添加：

```json
{
  "mcpServers": {
    "gaode_map": {
      "command": "python",
      "args": ["/path/to/your/gaode_map_mcp.py"],
      "transport": "stdio"
    }
  }
}
```

> **注意：** 把 `/path/to/your/gaode_map_mcp.py` 替换为你本机上的实际路径。

配置后重启客户端，即可在对话中调用高德地图相关能力。

---

### 方式二：Claude Desktop

编辑 Claude Desktop 的 MCP 配置文件（通常在 `~/Library/Application Support/Claude/claude_desktop_config.json` 或 `%APPDATA%\Claude\claude_desktop_config.json`）：

```json
{
  "mcpServers": {
    "gaode_map": {
      "command": "python",
      "args": ["/path/to/your/gaode_map_mcp.py"],
      "transport": "stdio"
    }
  }
}
```

---

### 方式三：LangChain Agent

```python
from langchain_mcp_adapters.client import MultiServerMCPClient

map_client = MultiServerMCPClient(
    {
        "map_consult": {
            "command": "python",
            "args": ["/path/to/your/gaode_map_mcp.py"],
            "transport": "stdio"
        },
    }
)

# 获取工具列表
tools = await map_client.get_tools()

# 在 Agent 中使用
from langgraph.prebuilt import create_react_agent
agent = create_react_agent(llm, tools)
```

---

## 📝 工具详解

### 1. `geocoding` — 地理编码

将结构化地址解析为城市编码（citycode）、区域编码（adcode）和经纬度（location）。

```
输入: address="北京市朝阳区阜通东大街6号", city="北京"
输出: citycode="010", adcode="110101", location="116.482086,39.990496"
```

> ⚡ 其他路线规划工具都依赖经纬度，不确定时请先用此工具查询。

---

### 2. `public_transit_route_planning` — 公共交通路线规划

规划地铁、公交、步行及少量出租车的组合出行方案。

- `city1` / `city2` 必须传 **citycode**（如北京是 `"010"`），拿不准可通过 `geocoding` 获取。
- 返回每条路线的总距离（米）、时间成本（秒）、换乘费用（元）、首末班车时间等。

---

### 3. `bicycle_route_schedule` — 骑行路线规划

自行车骑行路线规划，返回距离（米）、耗时（秒）、分段步骤等。

---

### 4. `automobile_driving_route_schedule` — 驾车路线规划

支持三种策略：
| strategy | 含义 |
|---|---|
| `0` | 速度优先（不一定最短） |
| `1` | 费用优先（不走收费路段） |
| `2` | 常规最快（默认，综合距离/耗时） |

还支持途经点（`waypoints`）、车牌号（`plate`，用于判断限行）、POI ID 等高级参数。

---

### 5. `keyword_search_of_poi` — 关键字搜索 POI

按关键字搜索兴趣点，返回 POI 的 ID、类型、名称、地址及商业信息（美食、购物、玩乐等）。

---

### 6. `poi_around_given_location` — 周边 POI 搜索

以指定经纬度为中心，搜索半径范围内的兴趣点。支持的 POI 类型包括：

| typecode | 类型 |
|---|---|
| `050000` | 餐饮服务 |
| `060000` | 购物服务 |
| `080000` | 体育休闲 |
| `100000` | 住宿服务 |
| `110000` | 风景名胜 |
| `140000` | 科教文化 |

---

## 🧪 命令行直接运行

```bash
python gaode_map_mcp.py
```

将启动 MCP 服务器并监听 stdio，可用于调试或直接与 MCP 客户端对接。

---

## ⭐ 如果觉得好用……

如果这个项目对你有帮助，欢迎给一个 **Star** ⭐，这是对我最大的鼓励！

---

## 📈 Star 历史

[![Star History Chart](https://api.star-history.com/svg?repos=ColtM1873/Gaode-Map-MCP&type=Date)](https://star-history.com/#ColtM1873/Gaode-Map-MCP&Date)

> 图表由 [star-history.com](https://star-history.com) 提供。仓库刚上线时还没有数据，过段时间再回来看曲线吧～

---

## 📄 License

GPL-3.0 ©
```

---
