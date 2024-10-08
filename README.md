## 介绍
全新的高颜值/高性能的AI模型接口管理与分发系统

> [!NOTE]
> 本项目在 [NewAPI](https://github.com/Calcium-Ion/new-api) & [OneAPI](https://github.com/songquanpeng/one-api) 的基础上进行二次开发

> [!IMPORTANT]
> 使用者必须在遵循上游AI模型服务提供商以及**法律法规**的情况下使用，不得用于非法用途。
> 本项目仅供个人学习使用，不保证稳定性，且不提供任何技术支持。
> 根据[《生成式人工智能服务管理暂行办法》](http://www.cac.gov.cn/2023-07/13/c_1690898327029107.htm)的要求，请勿对中国地区公众提供一切未经备案的生成式人工智能服务。

> [!WARNING]
> 本系统为闭源免授权使用，仅供个人学习使用，请勿用于任何商业用途。

## 差异说明
> 本系统与NewAPI的差异主要在于：

- [x] 整体界面风格的差异
- [x] 增加开发文档独立页支持
- [x] 增加服务监控页配置支持
- [x] 增加LinuxDO的第三方登录支持
- [x] 优化部分界面元素的边距避免挤在一起
- [x] 优化第三方登录部分逻辑
- [x] 用户注册时间支持
- [x] 优化数据操作按钮位置避免在页面底部
- [x] 在线充值关闭时不显示对应模块
- [x] 新版本在线检测支持
- [x] 清空本地缓存支持
- [x] 重构移动端菜单为弹出侧边栏展示
- [x] 增加C#的[trueai-org/midjourney-proxy](https://github.com/trueai-org/midjourney-proxy)支持
- [x] 增加用户管理列表邮箱显示
- [x] 增加登录/对话/绘画IP记录
- [x] 增加签到功能支持
- [x] 增加易支付自定义渠道支持
- [x] 增加MJ渠道模型的relax/fast/turbo区分
- [x] 增加全站模型汇率配置
- [x] 增加模型价格页计量单位与货币单位切换显示
- [x] 增加模型价格页倍率切换显示
- [x] 增加模型价格页模型信息（模型图标、介绍、标签、归属厂商、归属厂商图标）展示支持
- [x] 增加模型厂商滚动选项栏分组
- [x] 增加模型价格页厂商切换时显示对应的介绍与公告
- [x] 屏蔽敏感词触发响应错误包含具体敏感字词信息
- [x] 增加敏感词触发风控记录支持
- [x] 增加全局转发API调用速率限制支持，环境变量：`GLOBAL_RELAY_RATE_LIMIT_ENABLE`=false、`GLOBAL_RELAY_RATE_LIMIT`=180、`GLOBAL_RELAY_RATE_LIMIT_DURATION`=180
- [ ] 增加第三方AI客户端动态嵌入支持
- [ ] 增加Flux API的支持
- [ ] 增加StabilityAI API的支持
- [ ] 增加可灵API的支持
- [ ] 增加对话模型请求字段过滤支持

## 使用说明

在您的服务器新建`docker-compose.yml`文件，内容如下：
```
version: '3.4'

services:
  voapi:
    image: licoy/voapi:latest
    container_name: voapi
    restart: always
    command: --log-dir /app/logs
    ports:
      - "3000:3000"
    volumes:
      - ./data:/data
      - ./logs:/app/logs
    extra_hosts:
      - "host.docker.internal:host-gateway"
    environment:
      - SQL_DSN=root:123456@tcp(host.docker.internal:3306)/voapi?charset=utf8mb4&parseTime=True&loc=Local  # 修改此行，或注释掉以使用 SQLite 作为数据库
      - REDIS_CONN_STRING=redis://redis
      - SESSION_SECRET=random_string  # 启动前必须手动修改此值为随机字符串
      - TZ=Asia/Shanghai

    depends_on:
      - redis
    healthcheck:
      test: [ "CMD-SHELL", "wget -q -O - http://localhost:3000/api/status | grep -o '\"success\":\\s*true' | awk -F: '{print $2}'" ]
      interval: 30s
      timeout: 10s
      retries: 3

  redis:
    image: redis:latest
    container_name: redis
    restart: always

```

启动服务：
```
docker-compose up -d
```
访问`http://ip:3000`即可看到登录界面，输入账号密码即可登录（默认账号：root，默认密码：123456）。


若有版本更新，您可以通过以下命令更新并重启服务：
```
docker-compose pull && docker-compose up -d
```

## 首页模板配置属性
- `name`：`string` 标题
- `desc`：`string` 描述
- `bgImg`：`string` 首页第一栏右侧图片
- `notes`：`array` 打字机说明列表
    - `string` 说明文字
- `hideAiModels`：`bool` 是否隐藏AI模型
- `aiTitle`：`string` AI模型标题
- `aiModels`：`array` AI模型列表
    - `name`：`string` AI模型名称
    - `link`：`string` AI模型描述
    - `img`：`string` AI模型图片
- `hidePlatforms`：`bool` 是否隐藏平台
- `platformsTitle`：`string` 平台标题
- `platforms`：`array` 平台列表
    - `name`：`string` 平台名称
    - `link`：`string` 平台描述
    - `img`：`string` 平台图片
- `stepsImg`：`string` 步骤左侧图片
- `steps`：`array` 步骤列表
    - `name`：`string` 步骤名称
    - `desc`：`string` 步骤描述
-  `qas`：`array` 问答列表
    - `q`：`string` 问题
    - `a`：`string` 答案

## 模型信息/厂商信息示例
- 模型信息
```json
{
	"gpt-3.5-turbo": {
		"note": "这是OpenAI的模型，介绍可以自定义 **支持markdown**",
		"icon": "/index/platforms/voapi-p-3.png",
		"tags": "对话|低价|OpenAI",
		"group": "openai"
	},
	"gpt-4": {
		"note": "这是OpenAI的模型，介绍可以自定义 **支持markdown**",
		"icon": "/index/ai/voapi-1.ico",
		"tags": "对话|评分高|OpenAI",
		"group": "openai"
	}
}
```
- 模型厂商信息
```json
{
	"openai": {
		"name": "OpenAI",
		"icon": "/index/ai/voapi-1.ico",
		"desc": "OpenAI介绍 **支持markdown**",
		"notice": "OpenAI当前状态正常，你可以 [点此查看](https://status.openai.com)"
	}
}
```    

## 加群交流
> 您也可以扫描下方二维码加入QQ交流群以获取最新的开发动态和相关事件
> 
> 点击链接加入群聊：https://qm.qq.com/q/QFibUxMPEQ

<img width="235" alt="image" src="https://github.com/user-attachments/assets/d4798bae-03e5-4fce-aa03-e6dadd9ac748">


## 界面截图
### PC端
- 首页
![](./images/1.png)
- 首页（亮色模式）
![](./images/2.png)
- 配置页面
![](./images/4.png)
- 配置页面
![](./images/5.png)
### 移动端
- 首页
![](./images/3.png)

