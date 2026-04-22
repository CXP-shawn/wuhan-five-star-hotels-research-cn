# 调研方法、来源层级、交叉验证规则与已知局限

## 1. 调研工作流

1. **候选清单构建**(2026-04-22):
   - 抓取 **湖北省文化和旅游厅** 《星级旅游饭店名单》http://wlt.hubei.gov.cn/bsfw/bmcxfw/xjfd/(2024-12-27 更新)。
   - 抓取 **武汉市文化和旅游局** 文旅名录-星级饭店 https://wlj.wuhan.gov.cn/bsfw_27/wlml/lvfd/ 。
   - 抓取 **Booking.com 武汉尊贵型酒店** https://www.booking.com/fivestars/city/cn/wuhan.zh-cn.html 获取国际品牌定位五星的酒店列表(含万豪、丽思卡尔顿、费尔蒙、希尔顿等)。
   - 抓取 **携程武汉五星酒店筛选页** https://m.ctrip.com/webapp/hotel/wuhan477/star5 获取中文主流 OTA 排名。
   - 用 `web_search` 进行多轮关键词搜索(如 "武汉 五星级酒店 名单"、"武汉 万豪 希尔顿 洲际 凯悦"、"Wuhan 5 star hotels" 等)进一步补全候选。

2. **逐家深度调研**:
   - 对每家候选酒店,调用品牌官网(shangri-la.com、marriott.com.cn 等)+ Booking / 携程详情页 + web_search 专题提问。
   - 将抓取到的原始内容保存至 `raw/`(版本控制之外的本地镜像),结构化后存入 `data/hotels.json`。

3. **结构化 + 渲染**:
   - 先填充结构化 JSON(客房数、开业年、地址等字段),再以模板 Markdown 渲染生成 `hotels/<slug>/README.md`,避免幻觉。

4. **GitHub 提交**:
   - 中文 Markdown 文件全部以 **UTF-8** 编码,通过 `base64 -w 0`(Linux `coreutils`)编码为 base64 后 PUT 到 GitHub Contents API。
   - 每次 PUT 后回读并校验中文字符(如 "武汉")是否正确解码,发现乱码立即用原始 UTF-8 重新生成 base64 再推。

## 2. 来源层级(Priority)

| 层级 | 类型 | 代表站点 |
| --- | --- | --- |
| L1 | 品牌官网 | shangri-la.com、marriott.com.cn、hilton.com.cn、ihg.com、hyatt.com、all.accor.com、wandahotels.com、vanke-hotels.com、marcopolohotels.com 等 |
| L2 | 主流 OTA | booking.com、ctrip.com(携程)、trip.com、fliggy.com(飞猪) |
| L3 | 政府 / 主流媒体 | wlt.hubei.gov.cn、wlj.wuhan.gov.cn、sw.wuhan.gov.cn、wuchang.gov.cn、jianghan.gov.cn、people.com.cn、prnasia.com、travelweekly.com |
| L4 | 口碑 / 百科 | tripadvisor.com、dianping.com、xiaohongshu.com、zhihu.com、wikipedia.org、baike.com |

L1 在客房数、官网、开业年、品牌归属字段上作为首要来源;L2 / L3 用于交叉验证;L4 用于获取口碑 / 差评归纳与历史背景。

## 3. 交叉验证规则

- 每家酒店至少要有 **3 个独立来源**(L1 / L2 / L3 / L4 中至少跨 2 层)才被收录。
- 关键数值字段(客房数、开业年、地址门牌)当多来源出现冲突时:
  - 优先采信 L1 / L3(官方、品牌官网、政府)。
  - 若仍不一致,则在档案中 **同时保留两组数字** 并注明来源,不做选择性删除。
- 价格区间基于本次运行中实际观察到的挂牌价与促销价,不做跨日回溯,并在档案中注明 `pricing_asof: 2026-04-22`。

## 4. 冲突处理的实例

- **武汉卓尔万豪酒店** 位置:
  - 原候选清单基于 Booking slug 曾误标为 "汉口王家墩",交叉核对万豪国际官网 + Booking 地图 + Trip.com 详情后,修正为 **东西湖区宏图大道 8 号**(接近天河机场)。
- **武汉万科瞻云酒店** 行政区:
  - 原候选清单曾将其标为 "武昌东湖",TripAdvisor 原曲目为 "The Yun Hotel Hankou",经比对万科官方 + nerago 详情 + TripAdvisor,实际为 **江汉区常青路 177 号**(2023 年由原武汉晟云 / 君澜换牌)。
- **江城明珠豪生大酒店** 品牌归属:
  - 不同来源分别标注为 "豪生(温德姆)" 与 "美爵(雅高)",本次调研在公开网页上未找到该酒店当前官网的权威确认,档案中保留两种标注并建议读者以酒店当日名片为准。
- **武汉丽思卡尔顿酒店**:
  - TripAdvisor 页面显示过 "暂停营业" 提示,品牌官网页面访问时出现 404。档案中已注明需要向酒店客服确认最新营业状态,不做主观替换。

## 5. 已知局限

1. 本次调研未覆盖 **筹建中或仅有框架协议** 但尚未公开开业信息的酒店(如传闻中的武汉 JW 万豪、武汉凯宾斯基、武汉瑞吉等)。如有后续开业,欢迎补充。
2. 部分 **政务接待型酒店**(洪山宾馆、东湖宾馆、光谷金盾等)对外公开信息有限,部分字段标注为 "暂无公开数据"。
3. 价格区间基于单次 OTA 快照,**不代表实际成交均价**。做 ADR / RevPAR 测算需要接入 STR / 盈蝶 / 华住 Dashboard 等授权数据源。
4. 大众点评 / 小红书 / 知乎的差评与舆情归纳以定性文本为主,未做结构化 NPS 计算。

## 6. 编码与写入规范

- 所有 Markdown 文件为 **UTF-8** 编码,每次 PUT GitHub Contents API 之前由 shell `base64 -w 0` 直接从 UTF-8 文件生成 base64,禁止经过 latin1 中转。
- PUT 后对每家酒店的 README 回读一次,校验中文字符("武汉"、"酒店"等)是否正确解码。
- 如发现乱码,不做反向解码,直接用原始 UTF-8 字符串重新 PUT。

## 7. 数据冻结日期

本仓库所有事实性字段采集于 **2026-04-22** 前后的联网搜索与抓取;价格、开业状态、品牌归属可能随时间变化,请以最新官网 / OTA 为准。
