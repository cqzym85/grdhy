ysp-live v3.2 使用说明

央视频道直播 · 7 天回看 · Logo 匹配 · EPG 自动缓存

64 路频道 · 纯标准库 · M3U / TXT / XMLTV / JSON · DIYP · 酷9 · APTV · TVBox

---

目录

1. 快速开始
2. 订阅地址
3. EPG 电子节目单
4. 回看 / 时移
5. 频道列表
6. 接口速查
7. 部署与目录
8. 常见问题

---

1. 快速开始

三步跑起来，然后把订阅地址填进播放器即可。

① 启动服务

```bash
python3 ysp-live.py 6803
```

端口可省略，默认 6803。启动后浏览器打开 http://你的IP:6803/ 可看到首页。

② 选择你的播放器

播放器 订阅地址 说明
酷9 / DIYP http://你的IP:6803/all.m3u M3U 格式，内置 catchup 参数
酷9 / DIYP（备选） http://你的IP:6803/all.txt TXT 格式，EPG 指向 JSON 接口
APTV / TVBox http://你的IP:6803/aptv.m3u 带 catchup-source 模板

③ 配置 EPG

DIYP / 酷9 在「EPG 地址」中填入下面这一条即可（推荐）：

```
http://你的IP:6803/epg_diyp?ch={name}&date={date}
```

💡 {name} 和 {date} 是播放器会自动替换的占位符，原样保留，不要改成真实频道名。

---

2. 订阅地址

三个入口，按播放器类型选择。

/all.m3u ⭐ 推荐

面向 酷9 / DIYP。每行包含 tvg-id、tvg-name、tvg-logo、group-title、catchup 等属性。

```
http://你的IP:6803/all.m3u
```

分组：央视频道 · 卫视频道 · 其他频道

/aptv.m3u

面向 APTV / TVBox。额外写入回看模板：

```
catchup-source="http://你的IP:6803/replay/{slug}.m3u8?start={start|10}&stop={end|10}"
```

```
http://你的IP:6803/aptv.m3u
```

/all.txt

DIYP 通用 TXT 格式，频道名与 URL 用逗号分隔，URL 上带 logo= 与 tvg-url= 参数。

```
http://你的IP:6803/all.txt
```

⚠️ 把 你的IP 替换为运行本服务机器的局域网 IP（如 192.168.1.100）。若播放器与服务在同一台机器，可用 127.0.0.1。

---

3. EPG 电子节目单

服务启动后每 4 小时自动更新一次，缓存保留 7 天。

接口 格式 用途
/epg_diyp?ch={name}&date={date} JSON DIYP / 酷9 专用，单频道单日，体积小、响应快
/epg_diyp.xml XMLTV 完整节目单，频道 id 使用 slug，含多个 display-name
/epg.xml XMLTV 原始 EPG 缓存文件，直接透传上游数据
/api.php?action=epg_status JSON 查看 EPG 加载状态、频道数、节目数

DIYP JSON 返回示例

```json
{
  "epg_data": [
    {
      "start": "2025-01-01 19:00:00",
      "end":   "2025-01-01 19:30:00",
      "title": "新闻联播",
      "desc":  ""
    }
  ]
}
```

自定义 EPG 源

在脚本同目录新建 epg.txt，每行一个 XMLTV 地址，# 开头为注释：

```
# 每行一个 EPG 源，按顺序尝试
http://epg.51zmt.top:8000/e.xml
http://另一个源/epg.xml
```

文件不存在时使用内置默认源。

⚠️ 刚启动没有 EPG？ 首次需要下载并解析完整 XMLTV，通常要几十秒。稍等片刻后访问 /api.php?action=epg_status 查看 status 是否为 ready。

---

4. 回看 / 时移

三种入口路径，参数名互相兼容，按播放器习惯选用。

入口 参数写法 适用
/PLTV/{slug}.m3u8 ?start=…&stop=… 通用，DIYP 默认
/TVOD/{slug}.m3u8 ?playseek=YYYYMMDDHHMMSS-YYYYMMDDHHMMSS 酷9 / DIYP
/replay/{slug}.m3u8 ?start=…&stop=…（10 位秒级时间戳） APTV / TVBox

支持的参数名

项 值
开始时间 start · starttime · startTime · begin · t0 · stime
结束时间 stop · end · endtime · endTime · t1 · etime
区间写法 playseek= · timeline= · time= · range=（格式 A-B）

支持的时间格式

· 20250101190000 — 14 位 YYYYMMDDHHMMSS
· 202501011900 — 12 位 YYYYMMDDHHMM
· 1735732800 — 10 位 Unix 秒
· 1735732800000 — 13 位 Unix 毫秒
· 2025-01-01 19:00:00 / 2025-01-01T19:00:00Z — 标准时间串

示例

```
# 通用入口（秒级时间戳）
http://你的IP:6803/PLTV/cctv1.m3u8?start=1735732800&stop=1735736400

# 酷9 / DIYP 风格
http://你的IP:6803/TVOD/cctv1.m3u8?playseek=20250101190000-20250101200000

# APTV / TVBox 风格
http://你的IP:6803/replay/cctv1.m3u8?start=1735732800&stop=1735736400
```

⚠️ 限制： 单次回看区间最长 24 小时；可回看范围为最近 7 天；结束时间会自动收敛到「当前时间 − 90 秒」，避免请求尚未生成的分片。

---

5. 频道列表

共 64 路。链接可直接测试其直播地址。

央视频道

频道 slug 直播地址
CCTV-1 综合 cctv1 /PLTV/cctv1.m3u8
CCTV-2 财经 cctv2 /PLTV/cctv2.m3u8
CCTV-3 综艺 cctv3 /PLTV/cctv3.m3u8
CCTV-4 中文国际 cctv4 /PLTV/cctv4.m3u8
CCTV-5 体育 cctv5 /PLTV/cctv5.m3u8
CCTV-5+ 体育赛事 cctv5p /PLTV/cctv5p.m3u8
CCTV-6 电影 cctv6 /PLTV/cctv6.m3u8
CCTV-7 国防军事 cctv7 /PLTV/cctv7.m3u8
CCTV-8 电视剧 cctv8 /PLTV/cctv8.m3u8
CCTV-9 纪录 cctv9 /PLTV/cctv9.m3u8
CCTV-10 科教 cctv10 /PLTV/cctv10.m3u8
CCTV-11 戏曲 cctv11 /PLTV/cctv11.m3u8
CCTV-12 社会与法 cctv12 /PLTV/cctv12.m3u8
CCTV-13 新闻 cctv13 /PLTV/cctv13.m3u8
CCTV-14 少儿 cctv14 /PLTV/cctv14.m3u8
CCTV-15 音乐 cctv15 /PLTV/cctv15.m3u8
CCTV-16 奥林匹克 cctv16 /PLTV/cctv16.m3u8
CCTV-16 4K cctv164k /PLTV/cctv164k.m3u8
CCTV-17 农业农村 cctv17 /PLTV/cctv17.m3u8
CCTV-4K 超高清 cctv4k /PLTV/cctv4k.m3u8
CCTV-8K 超高清 cctv8k /PLTV/cctv8k.m3u8
CCTV 风云剧场 cctvfyjc /PLTV/cctvfyjc.m3u8
CCTV 第一剧场 cctvdyjc /PLTV/cctvdyjc.m3u8
CCTV 怀旧剧场 cctvhjjc /PLTV/cctvhjjc.m3u8
CGTN cgtn /PLTV/cgtn.m3u8
CGTN 法语 cgtnfr /PLTV/cgtnfr.m3u8
CGTN 俄语 cgtnru /PLTV/cgtnru.m3u8
CGTN 阿拉伯语 cgtnar /PLTV/cgtnar.m3u8
CGTN 西班牙语 cgtnes /PLTV/cgtnes.m3u8
CGTN 纪录 cgtndoc /PLTV/cgtndoc.m3u8

卫视频道

频道 slug 直播地址
北京卫视 bjws /PLTV/bjws.m3u8
江苏卫视 jsws /PLTV/jsws.m3u8
东方卫视 dfws /PLTV/dfws.m3u8
浙江卫视 zjws /PLTV/zjws.m3u8
湖南卫视 hnws /PLTV/hnws.m3u8
湖北卫视 hbws /PLTV/hbws.m3u8
广东卫视 gdws /PLTV/gdws.m3u8
广西卫视 gxws /PLTV/gxws.m3u8
黑龙江卫视 hljws /PLTV/hljws.m3u8
海南卫视 hainanws /PLTV/hainanws.m3u8
重庆卫视 cqws /PLTV/cqws.m3u8
深圳卫视 szws /PLTV/szws.m3u8
四川卫视 scws /PLTV/scws.m3u8
河南卫视 henanws /PLTV/henanws.m3u8
东南卫视 dnws /PLTV/dnws.m3u8
贵州卫视 gzws /PLTV/gzws.m3u8
江西卫视 jxws /PLTV/jxws.m3u8
辽宁卫视 lnws /PLTV/lnws.m3u8
安徽卫视 ahws /PLTV/ahws.m3u8
河北卫视 hebws /PLTV/hebws.m3u8
山东卫视 sdws /PLTV/sdws.m3u8
天津卫视 tjws /PLTV/tjws.m3u8
吉林卫视 jlws /PLTV/jlws.m3u8
陕西卫视 saxws /PLTV/saxws.m3u8
宁夏卫视 nxws /PLTV/nxws.m3u8
内蒙古卫视 nmgws /PLTV/nmgws.m3u8
云南卫视 ynws /PLTV/ynws.m3u8
山西卫视 shanxiws /PLTV/shanxiws.m3u8
甘肃卫视 gsws /PLTV/gsws.m3u8
青海卫视 qhws /PLTV/qhws.m3u8
西藏卫视 xizangws /PLTV/xizangws.m3u8
新疆卫视 xjws /PLTV/xjws.m3u8

其他频道

频道 slug 直播地址
CETV-1 cetv1 /PLTV/cetv1.m3u8
国学频道 guoxue /PLTV/guoxue.m3u8

---

6. 接口速查

全部接口均为 GET，返回带 CORS 头，可跨域调用。

路径 返回 说明
/ HTML 首页（存在 index.html 时优先使用）
/health ok 存活探针
/all.m3u M3U 酷9 / DIYP 订阅
/aptv.m3u M3U APTV / TVBox 订阅
/all.txt TXT DIYP 通用 TXT
/PLTV/{slug}.m3u8 M3U8 直播；带 start/stop 参数则回看
/TVOD/{slug}.m3u8 M3U8 回看（playseek 写法）
/replay/{slug}.m3u8 M3U8 回看（秒级时间戳写法）
/epg_diyp JSON DIYP 标准 EPG 接口
/epg_diyp.xml XMLTV 完整节目单
/epg.xml XMLTV 原始 EPG 缓存
/api.php?action=epg_status JSON EPG 状态
/api.php?format=full JSON 频道列表 + Logo + 线路
/logo/{文件名} 图片 频道台标静态文件
/proxy.php?type=ts&url=… TS 分片代理
/proxy.php?type=m3u8&url=… M3U8 清单代理（自动改写相对路径）
/diag 文本 各频道当前模式与最近错误

EPG 状态返回示例

```json
{
  "status": "ready",
  "source": "cache:20250101",
  "channels": 312,
  "programmes": 28640,
  "update_interval_hours": 4
}
```

---

7. 部署与目录

仅依赖 Python 3 标准库，无需 pip 安装任何包。

目录结构

```
ysp-live/
├── ysp-live.py          # 主程序
├── index.html           # 可选，自定义首页（存在则覆盖内置页）
├── logo/                # 可选，频道台标图片
│   ├── CCTV1.png
│   ├── 湖南卫视.png
│   └── ...
├── epg.txt              # 可选，自定义 EPG 源列表
└── epg_cache/           # 运行时自动生成
    ├── epg_20250101.xml
    └── epg_20250101.json
```

Logo 匹配规则

把台标图片放进 logo/ 目录，支持 png / jpg / jpeg / webp / gif / svg。程序按以下顺序查找：

· 频道 slug（如 cctv1、hnws）
· 别名表（如 cctv1 → CCTV1，cctvfyjc → CCTV风云剧场）
· 频道全名（如 CCTV-1 综合）
· 去掉后缀的短名（如 湖南卫视 → 湖南）

文件名会被归一化处理（忽略空格、连字符、下划线、点号，忽略大小写），所以 CCTV-1.png 与 cctv1.PNG 等效。

运行参数

```bash
python3 ysp-live.py [端口]

# 示例
python3 ysp-live.py          # 默认 6803
python3 ysp-live.py 8080     # 自定义端口
```

服务监听 0.0.0.0，局域网内任意设备均可访问。

关键参数（脚本内可调）

常量 默认值 含义
REFRESH_INTERVAL 15 直播分片刷新间隔（秒）
IDLE_TIMEOUT 120 无访问后停止刷新（秒）
WINDOW 300 时移拉取窗口（秒）
MAX_SEGS 60 分片队列上限
MAX_REPLAY_RANGE 86400 单次回看最长区间（秒）
CATCHUP_DAYS 7 回看天数（写入 M3U）
EPG_UPDATE_INTERVAL 14400 EPG 更新间隔（秒）
EPG_KEEP_DAYS 7 EPG 缓存保留天数

---

8. 常见问题

遇到问题先看 /diag 和 /api.php?action=epg_status。

播放时一直转圈 / 返回 503

首次访问某频道需要拉取首个分片，通常 3–10 秒。若持续 503，打开 /diag 查看该频道的 err 字段：

· 显示 DeadHostError —— 主接口返回了失效 CDN，程序会自动切换到备用线路。
· 显示 HTTPError: HTTP 403 —— 触发限流，稍等片刻重试。
· 显示 empty playlist —— 该频道当前无直播信号。

EPG 显示「暂无节目」

· 确认 EPG 地址填的是 ?ch={name}&date={date}，占位符未改动。
· 访问 /api.php?action=epg_status 确认 status 为 ready。
· 部分频道在 EPG 源中无对应条目，属于上游数据缺失。

回看提示「该时段暂无数据」

· 回看范围仅最近 7 天。
· 结束时间不能超过「当前时间 − 90 秒」。
· 单次区间不超过 24 小时，超长请分段请求。
· 确认频道 slug 拼写正确（见第 5 节）。

Logo 不显示

· 确认 logo/ 目录与 ysp-live.py 同级。
· 启动日志会打印「logo: 已匹配 N/64 个频道」，据此判断匹配情况。
· 新增图片后无需重启，程序会按目录修改时间自动重建索引。

能否公网访问？

脚本本身不带鉴权，直接暴露公网有被滥用的风险。建议通过反向代理（Nginx / Caddy）加一层 Basic Auth，或仅在内网 / 虚拟局域网使用。

---

ysp-live v3.2 · 央视频道直播 + 7 天回看 + Logo + EPG 缓存

本说明为静态文档，可直接用浏览器打开，或作为服务的首页。
