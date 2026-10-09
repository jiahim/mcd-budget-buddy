# 麦当劳穷鬼助手的 MCP 集成

## 实际服务与认证

Server：https://mcp.mcd.cn，Streamable HTTP。客户端请求头 Authorization: Bearer ${MCD_MCP_TOKEN}。示例配置仅为环境变量占位模板，运行时由用户自己的本地连接器提供实际凭据。

本次真实握手协商 protocolVersion=2024-11-05，tools/list 返回35个工具。业务采用当次工具schema，不依赖固定全部工具数量。

## 实际调用流程

1. query-nearby-stores：beType=1、searchType=2、city=深圳、keyword=金丰城，取得目标店1421053。
2. query-meals：storeCode=1421053、orderType=1、beType=1；当前返回119个菜单映射项。到店自取不传beCode。
3. query-store-coupons：同一上下文，成功但无可用券。本次不领取新券，也不编造券。
4. query-meal-detail：获取需要的套餐round/choice和默认份量，显式构造选配；不改变用户未请求的特制项。
5. calculate-price：传入同一门店上下文及items/roundList。没有传withOrder，没有创建订单。
6. 本地scripts/meal-math.ts：校验整数分与总价，按实付升序排序。试算数据是证据，不作为日常固定报价。

## 工具与业务价值

|工具|价值|当前验证|
|---|---|---|
|query-nearby-stores|锁定用户指定门店|真实调用成功|
|query-meals|确认当前可售编码与展示价条件|真实调用成功|
|query-meal-detail|正确处理套餐组成、规格及加价替换|真实调用成功|
|query-store-coupons|查清实际可用券，避免虚构优惠|成功，返回空；用券试算未验证|
|calculate-price|获得当前真实账单，避免静态价误导|真实调用成功|

## 真实试算节选

地点：深圳金丰城店；到店自取；2026-10-09北京时间。金额为元，包含接口返回的最终price；本次price=productPrice，其他费用字段未单独返回，不把该情况推广到外送。

|方案|试算实付/元|查询时间|
|---|---:|---|
|麦辣鸡腿汉堡+中薯条+锡兰红茶（单点）|47.50|2026-10-09 16:08:54|
|麦辣鸡腿汉堡+中薯条+锡兰红茶（三件套换茶）|35.00|2026-10-09 16:08:54|
|双层吉士汉堡+锡兰红茶（单点）|34.50|2026-10-09 16:08:55|
|双层吉士汉堡+锡兰红茶（人气经典随心配）|14.90|2026-10-09 16:08:55|

两组比较各自保持完全相同餐品与份量：麦辣汉堡三件组合单点47.50元/套餐35.00元；双层吉士与茶单点34.50元/随心配14.90元。不同组份量不同，不跨组宣称相同体验下最便宜。

脱敏数据见 [接口验证摘要](docs/接口验证摘要.json)。未公开token、账户券码、个人地址、traceId或支付链接。

## 状态与限制

上述是Codex环境中的真实MCP验证，不是WorkBuddy开发记录。WorkBuddy导入与实际技能对话未验证；外送、得来速、预约、实际用券、特制和写操作未验证。后续在WorkBuddy完成真实测试后补充workbuddy.md，不能伪造。

官方：[MCP工具与接入](https://github.com/M-China/mcd-mcp-server)、[比赛要求](https://github.com/M-China/mcd-developer-innovation-challenge/blob/main/activityGuidelines.md)。
