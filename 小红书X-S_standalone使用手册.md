# 小红书 X-S Standalone 使用手册

## 1. 文件说明

本手册对应的文件：

```text
小红书X-S_standalone.js
```

该文件是一个 CommonJS 脚本，用于根据请求 URL、请求体和浏览器环境参数生成小红书请求头标签。

生成的主要字段：

```text
x-s
x-s-common
x-t
x-b3-traceid
x-xray-traceid
x-mns
b1
```

脚本依赖 Node.js 内置模块，不需要执行 `npm install`。

建议运行环境：

```text
Node.js 18 或更高版本
```

---

## 2. 最简单的运行方式

### 2.1 在 PyCharm 中直接运行

打开：

```text
小红书X-S_standalone.js
```

点击 PyCharm 的 Run 按钮。

没有传入命令行参数时，脚本会自动使用内置的 `demoRequest` 生成一组演示标签。

终端中可以看到：

```text
[直接运行] 未传入请求参数，使用内置演示请求生成标签。

========================================
成功生成请求头标签值：
x-s: ...
x-s-common: ...
x-t: ...
x-b3-traceid: ...
x-xray-traceid: ...
x-mns: unload
b1: ...
========================================
```

最后还会输出一份 JSON 格式的完整返回结果。

### 2.2 修改演示请求

直接运行时，演示参数来自文件底部的 `demoRequest`：

```javascript
const demoRequest = {
    url: 'https://edith.xiaohongshu.com/api/sns/web/v1/homefeed',
    method: 'POST',
    data: {
        // 请求体
    },
    a1: '',
    loadts: '',
    webBuild: '6.57.2',
    version: '4.4.3',
    platform: 'Windows',
    userAgent: 'Mozilla/5.0',
    b1: '',
    br: '0',
    dsl: '',
    x10: 0,
    encSsk: '',
};
```

直接修改这个对象，即可调试其他请求。

注意：内置演示请求使用的是占位参数，主要用于确认脚本能够运行和查看标签格式。实际请求应填写与当前登录会话匹配的 `a1`、`loadts`、`b1`、`encSsk` 和 `userAgent`。

---

## 3. Node.js 模块调用

### 3.1 引入模块

```javascript
const {
    sign_headers,
    makeHeaders,
    printHeaders,
} = require('./小红书X-S_standalone.js');
```

可用导出：

| 名称 | 类型 | 说明 |
|---|---|---|
| `sign_headers` | Function | 对外签名入口 |
| `makeHeaders` | Function | 实际生成请求头 |
| `printHeaders` | Function | 单独打印请求头标签 |

`sign_headers` 和 `makeHeaders` 当前执行相同的生成流程。

### 3.2 完整调用示例

```javascript
const {
    sign_headers,
    printHeaders,
} = require('./小红书X-S_standalone.js');

const request = {
    url: 'https://edith.xiaohongshu.com/api/sns/web/v1/homefeed',
    method: 'POST',
    data: {
        cursor_score: '',
        num: 18,
        refresh_type: 1,
        note_index: 35,
        category: 'homefeed.cosmetics_v3',
        search_key: '',
        need_num: 8,
        image_formats: ['jpg', 'webp', 'avif'],
        need_filter_image: false,
    },
    a1: '你的 a1',
    loadts: '你的 loadts',
    webBuild: '6.57.2',
    version: '4.4.3',
    platform: 'Windows',
    userAgent: '你的 User-Agent',
    b1: '你的 b1',
    br: '0',
    dsl: '',
    x10: 0,
    encSsk: '',
    print: false,
};

const headers = sign_headers(request);

printHeaders(headers, { target: 'stderr' });
process.stdout.write(JSON.stringify(headers, null, 2) + '\n');
```

### 3.3 为什么 `console.log(result)` 可能没有输出

脚本为了屏蔽补环境诊断信息，在 Node.js 中会覆盖：

```javascript
console.log
console.error
```

因此调用脚本后继续使用 `console.log(result)`，可能看不到任何内容。

请改用：

```javascript
process.stdout.write(JSON.stringify(result, null, 2) + '\n');
```

或者：

```javascript
printHeaders(result);
```

---

## 4. 命令行 JSON 调用

脚本支持把请求对象作为第一个命令行参数传入。

### 4.1 PowerShell 示例

```powershell
$json = '{"url":"https://edith.xiaohongshu.com/api/sns/web/v1/homefeed","method":"POST","data":{"cursor_score":"","num":18},"a1":"你的 a1","userAgent":"你的 User-Agent","b1":"你的 b1","encSsk":"你的 encSsk"}'

node ".\小红书X-S_standalone.js" $json
```

### 4.2 命令行输出通道

默认情况下：

| 内容 | 输出通道 |
|---|---|
| 人类可读的标签文本 | `stderr` |
| JSON 返回结果 | `stdout` |

这样做是为了保证命令行标准输出仍然是合法 JSON，可以直接交给 Python 或其他程序解析。

如果需要把标签文本强制写入标准输出：

```json
{
  "printTarget": "stdout"
}
```

此时标签文本和 JSON 会混在同一个输出通道中，不建议程序化调用时使用。

### 4.3 关闭标签打印

以下任一参数都可以关闭自动打印：

```json
{
  "print": false
}
```

或者：

```json
{
  "printHeaders": false
}
```

程序化调用建议关闭打印，只读取返回对象：

```powershell
$json = '{"url":"https://edith.xiaohongshu.com/api/sns/web/v1/homefeed","method":"POST","data":{"cursor_score":"","num":18},"a1":"你的 a1","userAgent":"你的 User-Agent","print":false}'

$headers = node ".\小红书X-S_standalone.js" $json
$headers | ConvertFrom-Json
```

---

## 5. Python 调用示例

可以通过 `subprocess` 调用 Node.js 版本。

```python
import json
import subprocess
from pathlib import Path

signer = Path(__file__).with_name('小红书X-S_standalone.js')

payload = {
    'url': 'https://edith.xiaohongshu.com/api/sns/web/v1/homefeed',
    'method': 'POST',
    'data': {
        'cursor_score': '',
        'num': 18,
        'refresh_type': 1,
        'note_index': 35,
        'category': 'homefeed.cosmetics_v3',
        'search_key': '',
        'need_num': 8,
        'image_formats': ['jpg', 'webp', 'avif'],
        'need_filter_image': False,
    },
    'a1': '你的 a1',
    'loadts': '你的 loadts',
    'webBuild': '6.57.2',
    'version': '4.4.3',
    'platform': 'Windows',
    'userAgent': '你的 User-Agent',
    'b1': '你的 b1',
    'br': '0',
    'dsl': '',
    'x10': 0,
    'encSsk': '你的 encSsk',
    'print': False,
}

process = subprocess.run(
    [
        'node',
        str(signer),
        json.dumps(payload, ensure_ascii=False, separators=(',', ':')),
    ],
    capture_output=True,
    text=True,
    encoding='utf-8',
    check=False,
)

if process.returncode != 0:
    raise RuntimeError(process.stdout or process.stderr)

headers = json.loads(process.stdout)
print(headers)
```

如果希望同时看到标签打印文本，可以在解析 JSON 前输出 `process.stderr`：

```python
if process.stderr:
    print(process.stderr)
```

如果启用了 `print`，标签会写入 `stderr`，不会破坏 `stdout` 中的 JSON。

---

## 6. 请求参数说明

### 6.1 基础参数

| 参数 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `url` | string | 空字符串 | 请求地址。可以传完整 URL，也可以传以 `/` 开头的路径 |
| `method` | string | 未使用 | 请求方法，保留用于描述请求 |
| `data` | object/array/string/null | `null` | 参与 `x-s` 计算的请求数据 |
| `a1` | string | 空字符串 | Cookie 中的 `a1` |
| `loadts` | string | 空字符串 | Cookie 中的 `loadts` |
| `webBuild` | string | `6.57.2` | 页面构建版本 |
| `version` | string | `4.4.3` | 签名版本 |
| `platform` | string | `Windows` | 平台标识 |
| `userAgent` | string | 空字符串 | 当前请求使用的 User-Agent |
| `b1` | string | 自动生成 | 浏览器环境指纹值；为空时由脚本生成 |
| `br` | string/number | `0` | 拼接到 `x-s-common` 的环境参数 |
| `dsl` | string | 空字符串 | 拼接到 `x-s-common` 的环境参数 |
| `x10` | number | `0` | `x-s-common` 扩展字段 |
| `encSsk` | string | 空字符串 | Base64 格式的 `encSsk`；存在时生成 `x6/x7` 扩展信息 |

### 6.2 请求数据 `data`

`data` 应与实际请求体保持一致。

对象和数组会通过 `JSON.stringify` 追加到签名内容后面：

```javascript
data: {
    num: 18,
    cursor_score: ''
}
```

字符串会直接追加：

```javascript
data: 'raw request body'
```

如果 `data` 为空：

```javascript
data: null
```

建议：

1. 保持字段名称、字段类型和实际请求一致。
2. 保持字段顺序与实际发送的 JSON 一致。
3. 不要传入了签名字段之外的额外字段。
4. GET 请求通常将查询串放在 `url` 中，`data` 使用 `null`。

### 6.3 打印控制参数

| 参数 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `print` | boolean | `true` | 设为 `false` 时关闭自动打印 |
| `printHeaders` | boolean | `true` | 设为 `false` 时关闭自动打印 |
| `printTarget` | string | `stderr` | 可选 `stderr` 或 `stdout` |

优先级说明：

```text
print === false 或 printHeaders === false
```

只要其中一个为 `false`，就不会自动打印。

### 6.4 调试参数

| 参数 | 类型 | 说明 |
|---|---|---|
| `timestampMs` | number | 固定签名时间戳，便于复现问题 |
| `sequence` | number | 覆盖随机序列参数 |
| `windowPropsLength` | number | 覆盖随机窗口属性数量 |

一般情况下不需要手动设置这些参数。

---

## 7. 返回字段说明

`sign_headers()` 返回一个普通 JavaScript 对象：

```javascript
{
    'x-s': '...',
    'x-s-common': '...',
    'x-t': '...',
    'x-b3-traceid': '...',
    'x-xray-traceid': '...',
    'x-mns': 'unload',
    'b1': '...'
}
```

| 字段 | 说明 |
|---|---|
| `x-s` | 请求签名 |
| `x-s-common` | 公共浏览器环境标签 |
| `x-t` | 毫秒时间戳字符串 |
| `x-b3-traceid` | 请求追踪 ID |
| `x-xray-traceid` | 请求追踪 ID |
| `x-mns` | 当前固定为 `unload` |
| `b1` | 本次使用的浏览器环境指纹 |

生成的返回值可以直接合并到 HTTP 请求头中。

---

## 8. POST 请求示例

```javascript
const { sign_headers } = require('./小红书X-S_standalone.js');

const body = {
    cursor_score: '',
    num: 18,
    refresh_type: 1,
    note_index: 35,
    category: 'homefeed.cosmetics_v3',
    search_key: '',
    need_num: 8,
    image_formats: ['jpg', 'webp', 'avif'],
    need_filter_image: false,
};

const headers = sign_headers({
    url: 'https://edith.xiaohongshu.com/api/sns/web/v1/homefeed',
    method: 'POST',
    data: body,
    a1: '你的 a1',
    loadts: '你的 loadts',
    webBuild: '6.57.2',
    version: '4.4.3',
    platform: 'Windows',
    userAgent: '你的 User-Agent',
    b1: '你的 b1',
    encSsk: '你的 encSsk',
    print: true,
});
```

实际发送时，请求体必须与传给 `data` 的内容一致。

---

## 9. GET 请求示例

GET 请求通常把查询串包含在 URL 中：

```javascript
const headers = sign_headers({
    url: 'https://edith.xiaohongshu.com/api/sns/web/v1/feed?source_note_id=xxx',
    method: 'GET',
    data: null,
    a1: '你的 a1',
    loadts: '你的 loadts',
    webBuild: '6.57.2',
    version: '4.4.3',
    platform: 'Windows',
    userAgent: '你的 User-Agent',
    b1: '你的 b1',
    encSsk: '',
    print: true,
});
```

如果接口要求对查询参数对象签名，则应按实际接口要求把参数对象传入 `data`，不要同时重复拼接。

---

## 10. 单独打印标签

`printHeaders` 可以独立使用：

```javascript
const { printHeaders } = require('./小红书X-S_standalone.js');

const headers = {
    'x-s': 'test',
    'x-s-common': 'test-common',
    'x-t': String(Date.now()),
};

printHeaders(headers, { target: 'stderr' });
```

默认目标：

```text
stderr
```

强制输出到标准输出：

```javascript
printHeaders(headers, { target: 'stdout' });
```

---

## 11. 常见问题

### 11.1 直接运行没有输出

确认当前脚本底部存在直接运行分支：

```javascript
require.main === module
```

如果通过另一个 JS 文件执行：

```javascript
require('./小红书X-S_standalone.js');
```

此时脚本会作为模块加载，不会自动执行演示请求。需要自行调用 `sign_headers()`。

### 11.2 调用后 `console.log(result)` 没有输出

脚本在 Node.js 环境中会屏蔽 `console.log` 和 `console.error`。

使用：

```javascript
process.stdout.write(JSON.stringify(result) + '\n');
```

或者：

```javascript
printHeaders(result);
```

### 11.3 标签已经打印，但 Python 读取 JSON 失败

标签默认写入 `stderr`，JSON 写入 `stdout`，正常情况下不会冲突。

检查是否设置了：

```json
{
  "printTarget": "stdout"
}
```

如果设置了这个参数，标签会混入标准输出。程序化调用时应删除该参数或设置：

```json
{
  "print": false
}
```

### 11.4 `x-s` 生成成功但接口仍返回未登录或签名错误

重点检查：

1. `a1` 是否来自当前登录会话。
2. `loadts` 是否有效。
3. `userAgent` 是否与发送请求时一致。
4. `data` 是否与实际请求体完全一致。
5. JSON 字段顺序是否与实际发送的请求一致。
6. `b1` 是否与当前浏览器环境匹配。
7. Cookie 是否已经过期。
8. 本机时间是否准确。

### 11.5 没有生成 `x6/x7`

当 `encSsk` 为空时，不会生成 `x6/x7` 扩展签名信息。

需要从有效会话的 `x-s` 或相关响应数据中提取 Base64 格式的 `encSsk` 后传入。

### 11.6 Windows 命令行 JSON 转义错误

PowerShell 建议使用单引号包住 JSON：

```powershell
$json = '{"url":"https://example.com/api","data":{"num":1},"print":false}'
node ".\小红书X-S_standalone.js" $json
```

如果 JSON 很长，建议使用独立的 JS 调用文件，或通过 Python `subprocess` 传参。

---

## 12. 使用建议

1. 调试时使用 `print: true` 查看完整标签。
2. 生产环境使用 `print: false`，只返回请求头对象。
3. Python 调用时读取 `stdout` 中的 JSON。
4. 需要排查报错时同时打印 `stderr`。
5. 不要使用内置演示参数直接请求正式接口。
6. 标签、Cookie、`a1`、`b1` 和 `encSsk` 都可能属于敏感信息，不要提交到公开仓库或发送给无关人员。
7. 仅对已获得授权的接口和账号进行调试。

---

## 13. 快速检查清单

调用前确认：

- [ ] Node.js 可以正常运行
- [ ] `url` 与实际请求一致
- [ ] `data` 与实际请求体一致
- [ ] `a1`、`loadts`、`b1` 来自有效会话
- [ ] `userAgent` 与实际发送请求一致
- [ ] 程序化调用设置了 `print: false`
- [ ] Python 从 `stdout` 解析 JSON
- [ ] 需要查看标签时打印 `stderr`

