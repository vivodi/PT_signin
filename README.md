> [!NOTE]
> 若图片显示异常，访问[备用链接](https://blog.3628688.xyz/index.php/2022/07/25/利用阿里云实现pt自动签到和数据统计/)

## 🌀简介

本项目能每天自动签到 PT 站点并统计上传下载量和分享率等数据，支持大多数国内站点和部分国外站点，通过微信或 Telegram 接收签到结果。

本项目使用[阿里云函数计算](https://www.aliyun.com/product/fc)运行，每天签到一次的情况下月费不超过0.1元。

![introduction](img/introduction-1.jpg)
![introduction](img/introduction-2.jpg)

## ⚠️使用须知

> [!NOTE]
> 本项目基于上游项目 [flexget_qbittorrent_mod](https://github.com/madwind/flexget_qbittorrent_mod)，所有功能均由上游项目提供。上游项目的自动运行需能运行 Python 或 Docker 的服务器，本项目为不具条件的用户提供了 Serverless 解决方案。满足条件的用户可考虑不使用本项目而直接使用上游项目。

> [!CAUTION]
> 尽管发生可能性极小，仍应注意使用本项目的风险因素。本项目不负责使用本项目可能造成的任何损失，以下为风险因素：
> 1. PT 站点禁止脚本登录。若上游开发者发现此类站点，会立即移除相关代码。但若上游开发者未立即进行更改，你的 PT 站点账号有被封风险。
> 2. 上游项目包含恶意代码。若 PT 站点网页变化，上游项目代码需更新才可保证功能正常，故本项目每次运行都会调用上游最新代码。若上游包含恶意代码，你的 PT 站点账号有被盗风险。若具有相关知识，你可以通过复刻上游项目后审查上游代码更改再并入你的复刻消除此风险，参见[创建函数](#41-创建函数)。
> 3. 平台服务器遭入侵。若阿里云或 GitHub 相关服务器遭入侵，你的 PT 站点账号有被盗风险。

> [!TIP]
> 编写不易，点击右上角↗ ⭐Star 给项目涨涨热度😍😍
>
> 赠人玫瑰，手有余香。⭐Star 本项目还可防丢失哦😉😉

## 🧲使用

### １．获取百度智能云文字识别应用参数

依[指引](https://cloud.baidu.com/doc/OCR/s/dk3iqnq51)获取 `AppID`、`API Key` 与 `Secret Key`。

### ２．配置企业微信或 Telegram bot 推送

#### 企业微信
[获取企业微信参数](https://work.weixin.qq.com/api/doc/90000/90135/90665)
微信关注微工作台接收消息则不必安装企业微信。

#### Telegram bot
[Bots：开发人员指南](https://core.telegram.org/bots#3-how-do-i-create-a-bot)
因 Telegram 限制，bot 无法主动发起聊天，需要你先向 bot 发一条消息。

### ３．创建存储库、添加配置文件并获取令牌

#### 3.1 创建 GitHub 存储库

![repo](img/usage-3.1-1.png)

![repo](img/usage-3.1-2.png)

#### 3.2 添加配置文件

![repo](img/usage-3.2-1.png)

![repo](img/usage-3.2-2.png)

填写 `config.yml` 时，具体站点的配置规范见 [config_example.yml](https://github.com/madwind/flexget_qbittorrent_mod/blob/master/config_example.yml#L575)，还可参见上游项目 wiki：[签到](https://github.com/IvonWei/flexget_qbittorrent_mod/wiki/auto_sign_in) 与 [推送](https://github.com/IvonWei/flexget_qbittorrent_mod/wiki/wecom)。

```yml
tasks:
  sign_in:
    auto_sign_in:
      user-agent: Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Mobile Safari/537.36
      # 推荐值：5。多线程各站点日志会乱序，如需查看日志设为 1。默认 1
      # 太小程序运行时间延长，增加阿里云 CU 时间，导致费用增加，太大增加资源消耗
      max_workers: 5
      # 设为 no 跳过未读信息获取。默认 no
      get_messages: no
      # 设为 no 跳过获取统计。默认 yes
      get_details: yes
      # 百度智能云文字识别应用参数
      aipocr:
        app_id: ''
        api_key: ''
        secret_key: ''
      sites:
        # 站名: cookie 
        1ptba: xxxxxxxx
        # 部分站点 cookie 过期太快，使用模拟登陆
        filelist:
          login:
            username: xxxxxxxx
            password: xxxxxxxx
    accept_all: yes
    seen:
      fields:
        - title
    notify:
      task:
        always_send: true
        message: |+
          {%- if task.accepted -%}
          {%- for group in task.accepted|groupby('task') -%}
          FlexGet has just signed in {{ group.list|length }} sites for task {{ group.grouper }}:
          {%- for entry in group.list %}
          {{ loop.index }}: {{ entry.title }} {{ entry.result }}
          {%- endfor -%}
          {%- endfor -%}
          {%- endif -%}
          {%- if task.failed %}
          {% for group in task.failed|groupby('task') %}
          The following sites have failed for task {{ group.grouper }}:
          {%- for entry in group.list %}
          {{ loop.index }}: {{ entry.title }} {{ entry.result }} Reason: {{entry.reason|d('unknown')}}
          {%- endfor -%}
          {%- endfor -%}
          {%- endif -%}
          {%- for group in task.entries|groupby('task') %}
          {% for entry in group.list %}
          {%- if entry.messages|d('') %}
          Messages:
          {{ entry.title }} {{ entry.messages }}
          {% endif %}
          {%- endfor -%}
          {%- endfor -%}
        # 填写要使用的推送渠道，移除不需要的
        via:
          - wecom:
              corp_id: ''
              corp_secret: ''
              agent_id: ''
              to_user: ''
              image: details_report.png
          # Telegram 配置参见 https://flexget.com/en/Plugins/Notifiers/telegram
          - telegram:
              # 需替换成你的
              bot_token: 5247092814:AAHmk7qdOMXEtZtGrnrzNxcoOazKNM51Atg
              images:
                - details_report.png
              recipients:
                # Chat ID 通过 @raw_data_bot (https://t.me/raw_data_bot) 获取
                - chat_id: 1694213419
```

![repo](img/usage-3.2-2.png)

#### 3.3 获取 GitHub 个人访问令牌

![token](img/usage-3.3-1.png)

![token](img/usage-3.3-2.png)

![token](img/usage-3.3-3.png)

![token](img/usage-3.3-4.png)

![token](img/usage-3.3-5.png)

![token](img/usage-3.3-6.png)

### ４．配置阿里云函数计算

#### 4.1 创建函数

打开[阿里云函数计算（香港）](https://fcnext.console.aliyun.com/cn-hongkong/functions)创建函数。可自行选择区域，但选择中国大陆地区会导致被 [GFW](https://zh.wikipedia.org/wiki/%E9%98%B2%E7%81%AB%E9%95%BF%E5%9F%8E) 封锁的站点（如 U2 遭 DNS 污染）签到失败及 Telegram 推送失败。

![ali](img/usage-4.1-1.png)

> [!NOTE]
> 1. 建议选择阿里云支持的最新的 Python 版本，上游项目会积极适配最新的 Python 版本并放弃支持较旧的。
> 2. 选用较高规格的 vCPU、内存及硬盘会造成不必要的费用，务必依下图配置。
> 3. 环境变量配置说明：
>
> 变量|值
> ---|---
> PLUGIN_REPO|`madwind/flexget_qbittorrent_mod`：直接使用上游开发者存储库，始终为最新<br>`lhllhx/flexget_qbittorrent_mod`：使用 `lhllhx` 复刻，滞后最新版数小时，无其他区别<br>`owner/repo`：使用你自己的复刻，可安全审查上游开发者存储库更改后再并入你的复刻以防恶意代码窃取你的 PT 站点账密。须为公开存储库，程序优先使用 `master` 分支，不存在则使用默认分支
> CONFIG_REPO|`owner/flexget-config`：GitHub 用户名/前文创建的存储库名，程序使用默认分支
> GITHUB_TOKEN|前文生成的令牌

![ali](img/usage-4.1-2.png)

#### 4.2 部署代码

将编辑框中示例代码替换为[此文件](https://github.com/lhllhx/PT_signin/raw/AliYun/index.py)中的代码，然后点击“部署代码”。

![ali](img/usage-4.2-1.png)

#### 4.3 安装依赖

![ali](img/usage-4.3-1.png)

![ali](img/usage-4.3-2.png)

复制以下内容
```pip-requirements
aiohttp[speedups]
baidu-aip
chardet
flexget[telegram]
fuzzywuzzy
pandas[plot]
```

> [!NOTE]
> “兼容运行时”和“构建环境”的 Python 版本应与函数一致。

![ali](img/usage-4.3-3.png)

层创建完成后为函数选上：

![ali](img/usage-4.3-4.png)

#### 4.4 测试函数

> [!NOTE]
> 1. 使用配置文件中直接指定 `chat_id` 的 Telegram bot 推送，需曾向 bot 发送过一条消息，对发送消息的时间没有要求，比如可以是1年前。使用非直接指定 `chat_id` 的 Telegram bot 推送，需恰好在程序运行前发送过一条消息，保守估计需在30分钟内，对于非直接指定的若删除 `db-config.sqlite` 还需重新发送。
> 2. 要验证是否签到成功，应以微信或 Telegram 中的消息为准。详细运行日志在前文创建的 GitHub 存储库中的 `flexget.log` 文件中。阿里云此次运行结果的“日志输出”中仅有被截断的部分日志，要在阿里云中查看此次运行完整日志或查看触发器触发运行的日志，需在“日志”中开通功能，可能会额外计费，没有必要开通。
> 3. 因 FlexGet 程序不支持在同一 Python 进程中多次调用（即 `flexget.main(['execute']);flexget.main(['execute'])`），而阿里云函数计算短时间内多次运行会使用同一实例，要在短时间内多次测试函数可点击“部署代码”强制使用新实例（否则两次运行需间隔数分钟）。
> 4. 因上游项目限制，一天只能推送一次，如果想在运行成功后重复测试，删除前文创建的 GitHub 存储库中的 `db-config.sqlite` 文件。

![ali](img/usage-4.4-1.png)

#### 4.5 添加定时触发器

![ali](img/usage-4.5-1.png)

![ali](img/usage-4.5-2.png)

#### 4.6 费用与充值

目前，每天签到一次的情况下月费不超过0.1元。

![ali](img/usage-4.6-1.png)

![ali](img/usage-4.6-2.png)

## ✨更新

程序后续运行出错可能是因上游项目引入新的依赖或不再支持较旧的 Python 版本，通过以下步骤解决：

### １．更新 Python 版本

![update](img/update-1-1.png)

> [!NOTE]
> 应选择阿里云函数计算支持的最新 Python 版本

![update](img/update-1-2.png)

### ２．更新依赖

![update](img/update-2-1.png)

复制以下内容
```pip-requirements
aiohttp[speedups]
baidu-aip
chardet
flexget[telegram]
fuzzywuzzy
pandas[plot]
```

> [!NOTE]
> “兼容运行时”和“构建环境”的 Python 版本应与函数一致。

![update](img/usage-4.3-3.png)

层创建完成后为函数选上新的层版本：

![update](img/update-2-3.png)

### ３．更新 index.py

将编辑框中所有代码替换为[此文件](https://github.com/lhllhx/PT_signin/raw/AliYun/index.py)中的代码，然后点击“部署代码”。

![update](img/update-3-1.png)

## 💕感谢

本项目基于以下项目建立：

- [FlexGet](https://github.com/flexget/flexget)
- [flexget_qbittorrent_mod](https://github.com/IvonWei/flexget_qbittorrent_mod)

## ⭐Stargazers

[![Stargazers over time](https://starchart.cc/lhllhx/PT_signin.svg)](https://starchart.cc/lhllhx/PT_signin)
