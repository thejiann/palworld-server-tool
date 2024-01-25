# palworld-server-tool

专注于管理 PalWorld 专用服务器的工具，有命令行版和 REST 服务版

## 配置文件

第一次运行会自动生成 config.yaml 文件到可执行文件目录

```yaml
host: 127.0.0.1:25575
password:
timeout: 10
```

## REST 服务

REST 服务除了提供操作外，还额外增加了一个 sqlite 数据库，用来存历史玩家数据，并且每五分钟会定时查询一次在线玩家列表，更新最后在线时间。

```bash
./pst-server
```

### 基本 URL

所有 API 请求都发往基本 URL `http://<server-address>:8080`。请将 `<server-address>` 替换为实际的服务器地址。

### 端点

#### 服务器信息

- **端点**: `/server/info`
- **方法**: GET
- **描述**: 检索服务器信息。
- **响应**: 包含服务器信息的 JSON 对象。

#### 玩家列表

- **端点**: `/player`
- **方法**: GET
- **查询参数**:
  - `update`（可选）: 一个布尔值（`"true"`/`"false"`）表示是否从服务器更新玩家数据。
- **描述**: 获取所有玩家的列表，可选择更新其数据。
- **响应**: 包含`name`、`steamid`、`playeruid`、`last_online`和`online`状态的玩家对象的 JSON 数组。

#### 踢出玩家

- **端点**: `/player/:steamid/kick`
- **方法**: POST
- **路径参数**:
  - `steamid`: 要踢出的玩家的 SteamID（其实 PlayerUID 也可以）。
- **描述**: 使用玩家的 SteamID 将玩家从服务器踢出。
- **响应**: 包含表示操作成功或失败的消息的 JSON 对象。

#### 封禁玩家

- **端点**: `/player/:steamid/ban`
- **方法**: POST
- **路径参数**:
  - `steamid`: 要封禁的玩家的 SteamID（其实 PlayerUID 也可以）。
- **描述**: 使用玩家的 SteamID 封禁玩家。
- **响应**: 包含表示操作成功或失败的消息的 JSON 对象。

#### 广播消息

- **端点**: `/broadcast`
- **方法**: POST
- **请求体**:
  - `message`: 要广播的消息。
- **描述**: 向服务器上的所有玩家广播消息。
- **响应**: 包含表示操作成功或失败的消息的 JSON 对象。

#### 关闭服务器

- **端点**: `/server/shutdown`
- **方法**: POST
- **请求体**:
  - `seconds`: 服务器关闭之前的倒计时时间（默认值："60"）。
  - `message`: 关闭前显示的消息。
- **描述**: 安排一个带有自定义倒计时和消息的服务器关闭。
- **响应**: 包含表示操作成功或失败的消息的 JSON 对象。

### 错误处理

在发生错误时，API 将返回一个包含错误消息的 JSON 对象，其中包含一个`error`键。

## 命令行工具

### 玩家

#### 在线玩家列表

```bash
./pst-cli player list
```

> [!WARNING]
> 如果昵称中包含中文，则最后一名玩家信息可能显示不全，我就直接显示 <null/err> 了

```
+-------------------------------------------+
| Pal World 在线玩家列表                    |
+----------+------------+-------------------+
| 昵称     | PLAYERUID  | STEAMID           |
+----------+------------+-------------------+
| 全国可飞 | 357689484  | 76561199164594721 |
| 香菇包子 | 2398722357 | 76561199401662262 |
| 梵音丶   | 2144044083 | 76561198101062108 |
| 狐狸     | 1333009711 | 76561198863159356 |
| 九龙     | 3049571152 | 76561198984756742 |
| DZ       | 850234947  | 76561198260413733 |
| Baoz     | <null/err> | <null/err>        |
+----------+------------+-------------------+
|          | 在线人数   | 7                 |
+----------+------------+-------------------+
```

#### 踢出/封禁玩家

```bash
./pst-cli kick -s <SteamID>
./pst-cli ban -s <SteamID>
```

### 广播

```bash
./pst-cli broadcast -m "<message>"
```

> [!WARNING]
> message 中不能包含中文

### 服务器

#### 关闭服务器

```bash
./pst-cli server shutdown -s <seconds> -m "Server Will Shutdown"
```