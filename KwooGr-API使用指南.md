# KwooGr-API使用指南

## 基本调用步骤

1. 获取访问密钥。
2. 调用API获得模型回复。

## 获取访问密钥

1. 登录 https://www.tgkwai.com/ 获取 `access token`（形如 `sk-xxxx`），**Access Token 只展示一次，请妥善保存**。
2. 调用时在WebSocket 参数中增加：
   - WebSocket 参数: `?x-token=sk-xxxx`

## WebSocket连接

**请求方法**: WebSocket connect （**协议选择wss://而不是https://**）

**请求路径**: `/api/v1/growth_model/predict`

**说明**：先创建连接再执行后续命令，连接终止由服务端进行。

**请求头**：`X-Token: Bearer sk-xxxx`如果无法使用自定义header（如浏览器的websocket api），可以采用param的方式，`?x-token=sk-xxxx`

**请求参数**:

- x-token：http header或param的x-token形式任选其一即可

**请求示例**
```
wscat -c "wss://api.tgkwai.com/api/v1/growth_model/predict?x-token=sk-xxxx"
```

**响应参数**:

- data: 会话号session_id

**响应示例**:

```text
 {
   "code":200,
	"msg":"OK",
	"data": <一个uuid>
 }
```

**错误响应**:
- 400-输入参数有误-null
- 403-权限问题-null
- 500-服务端错误-null
## 生长大模型预测

**请求方法**: WebSocket send

**请求路径**: `/api/v1/growth_model/predict`

**说明**：本部分使用**流式输出** ，即会有连续的多条发送，发送完毕后由服务器断开WebSocket连接。

**请求头**：`X-Token: Bearer sk-xxxx`如果无法使用自定义header（如浏览器的websocket api），可以采用param的方式，`?x-token=sk-xxxx`

**请求参数**:

- session_id（string）: WebSocket连接时获取

- data（object）：要传送的所有参数，详细格式见[info.json](./info.json)的`request_varibles`部分。

**请求示例**:

```json
{
    "session_id":"9c5250e2-ea45-4809-b72e-f6b1dae23a44",
    "data":    
    {
        "CROP": "Wheat",
        "VARIETY": "Winter_wheat_107",
        "INITIAL_DATE": "2020-01-15",
        "PREDICT_DATE": "2020-04-15",
        "LOCATION":"黑龙江建三江七星农场",
        "SM0": 0.57,
        "SMW": 0.3,
        "CRAIRC": 0.05,
        "SOPE": 0.55,
        "KSUB": 0.37,
        "WAV": 1.0,
        "TBASEM": 0.0,
        "TEFFMX": 30.0,
        "TSUMEM": 120.0,
        "TSUM1": 1031.0,
        "TSUM2": 928.0,
        "DVSI": 0.0,
        "TDWI": 50.0,
        "LVLAIEM": 0.03445,
        "SPAN": 31.3,
        "TBASE": 0.0,
        "CVL": 0.685,
        "CVO": 0.709,
        "CVR": 0.694,
        "CVS": 0.662,
        "RML": 0.03,
        "RMO": 0.01,
        "RMR": 0.015,
        "RMS": 0.015,
        "CFET": 1.0,
        "RDI": 10.0,
        "RRI": 1.2,
        "RDMCR": 125.0,
        "SMLIM": 0.3,
        "SSMAX": 0.0,
        "N": [
            {
                "date": "2020-01-16",
                "amount": 10.0
            }
        ],
        "P": [],
        "K": [],
        "NH4": [],
        "NO3": [],
        "K2O": [],
        "P2O5": [],
        "KH2PO4": []
    }
}

```

**响应数据** ：

- content（object）：模型返回的内容。主要有以下三类，**保证content是个dict，且key只会是以下三种之一**：

    - phase：模型当前的处理阶段

    - tools_result：JSON格式的预测工具输出结果字符串，详细格式见“大模型集成 —— 生长大模型前端显示模板”，tool_output_keys部分和response_varibles部分。

    - chunk：模型端大模型流式输出的当前chunk，服务端终止链接前会返回一个内容为[DONE]的chunk

- session_id（string）：当前会话的ID

- is_finished（boolean）：本次会话是否结束

**响应示例** :

```json
{
    "code": 200,
    "data": {
        "session_id": "4000d0e5-6fe9-4c19-8afc-4f5763942669",
        "content": {
            "phase": "大模型解析中"
        },
        "is_finished": false
    },
    "msg": "成功"
}

{
  "code": 200,
  "data": {
    "session_id": "4000d0e5-6fe9-4c19-8afc-4f5763942669",
    "content": {
      "tools_result": {
        "LVLAI": [
          0.367393493652344, 0.36751174926758, 0.3673095703125,
          ...
        ],
        "RTD": [
          3.2910451889038086, 3.291064739227295, 3.291069269180298,
          ...
        ],
        "PLTBAG": [
          26272.4140625, 26272.705078125, 26271.99609375, 26272.3984375,
          ...
        ],
        "PLTCT": [
          156.9857635498047, 156.99171447753906, 156.9761199951172,
          ...
        ],
        "LVTW": [
          7060.01953125, 7060.02099609375, 7060.0400390625, 7060.01953125,
          ...
        ],
        "RTTW": [
          3034.408447265625, 3034.372314453125, 3034.425048828125,
          ...
        ]
      }
    },
    "is_finished": false
  },
  "msg": "成功"
}


{
    "code": 200,
    "data": {
        "session_id": "4000d0e5-6fe9-4c19-8afc-4f5763942669",
        "content": {
            "chunk": "信息"
        },
        "is_finished": false
    },
    "msg": "成功"
}

{
    "code": 200,
    "data": {
        "session_id": "4000d0e5-6fe9-4c19-8afc-4f5763942669",
        "content": {
            "chunk": "[DONE]"
        },
        "is_finished": true
    },
    "msg": "成功"
}

```



**错误响应**:

- 400 - session_id不能为空 - null

- 400 - session_id不匹配 - null

- 500 - 生长大模型故障 - null
