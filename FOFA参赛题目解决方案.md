# FOFA 参赛题目解决方案

参赛包 `pkg-08e82c8d`，发放时间 `2026-09-22 10:56:20 +08:00`。本文按原题顺序覆盖 100 道题，提供查询候选、关键判断和待复核项。查询只做了离线检查，评分结果以比赛环境为准。

## 解题方法

1. **先区分字段。** 主域用 `domain`，访问主机名用 `host`，网页标题、响应头、正文分别用 `title`、`header`、`body`。服务协议用 `protocol`，TCP 和 UDP 用 `base_protocol`，非 HTTP 服务的响应原文用 `banner`。查询字段与 API 返回列可能不同。参见 [FOFA 搜索入口与语法](https://en.fofa.info/) 和 [FOFA API 查询接口](https://fofa.info/api/batches_pages)。
2. **再确定逻辑和边界。** “任一”用 `||`，“同时”用 `&&`。将任一组明确括起来。完整相等用支持 `==` 的字段，按字符形态用 `*=` 与通配符 `?`、`*`。空字符串查询需核对是否包含未采集字段。参见 [FOFA 模糊搜索说明](https://github.com/FofaInfo/Awesome-FOFA/blob/main/Basic%20scenario/The%20correct%20way%20to%20use%20Fuzzy%20Search%20in%20FOFA.md)。
3. **验证候选。** 从短查询开始，逐步加入条件。对分组、非空、通配符和时间区间各选一条预期命中记录和一条预期排除记录。先确认检索权限和枚举，再解释零结果。表中的 **复核** 表示存在标签、格式或索引语义需要在比赛环境确认。标为 **样例** 的行用保留域名或文档网段演示同类逻辑。

证书题区分三个概念。`cert.is_valid` 表示受信，`cert.is_expired` 判断是否过期，`cert.is_match` 判断证书是否匹配资产域名。比较证书颁发者和持有者使用 `cert.is_equal`。参见 [FOFA 官方证书说明](https://github.com/FofaInfo/Awesome-FOFA/blob/main/Basic%20scenario/How%20to%20effectively%20eliminate%20interference%20through%20certificates%20in%20the%20asset%20discovery.md)。

## M001 至 M025 地址、地域、协议和产品

| 题号 | 查询候选或解法 | 要点 |
|---|---|---|
| M001-S009 | `ip="20.247.40.92"` | 单个 IPv4 地址。 |
| M002-S004 | `ip="124.255.254.0/24"` | CIDR 覆盖整个网段。 |
| M003-S002 | `domain="zendesk.com"` | 按资产主域匹配。 |
| M004-S003 | `host=="www.amazon.com"` | 指定完整主机名。**复核** 主机字段精确匹配对协议和端口的规范化。 |
| M005-S004 | `is_domain=true && host*="*vpn*"` | 在访问主机名中做任意位置的子串匹配。 |
| M006-S002 | `country="GB"` | 英国使用 GB。 |
| M007-S007 | `country="KR" && region="Gangwon-do"` | **复核** 数据库可能以 Gangwon 等名称存储江原道。 |
| M008-S009 | `country="DE" && region="Bavaria" && city="Munich"` | **复核** Bavaria/Bayern、Munich/München 的实际索引名。 |
| M009-S008 | `asn="4134"` | ASN 只填数字。 |
| M010-S009 | `country="NL" && (cloud_name="Amazon EC2" \|\| cloud_name="Amazon S3" \|\| cloud_name="Amazon Elastic Beanstalk")` | **复核** 三类服务的实际标签。若索引只提供 AWS 厂商级标签，厂商查询只能作为近似。 |
| M011-S005 | `country="CN" && region="Hebei" && cloud_name*="*Huawei*"` | **复核** 河北、华为云在比赛索引中的规范名。 |
| M012-S009 | `port="3000"` | 没有额外协议限制。 |
| M013-S003 | `(title="Jenkins" \|\| body="jenkins") && country!="CN" && after="2026-08-01"` | 国家和更新时间约束两个分支。 |
| M014-S002 | `country="CN" && protocol="dns"` | 按识别的 DNS 服务筛选，不猜测端口。 |
| M015-S001 | `protocol="socks5"` | **复核** 先确认协议索引是否将 v5 单列为 socks5。 |
| M016-S006 | `protocol="ssh" && port="22022"` | SSH 可出现在非默认端口。 |
| M017-S001 | `base_protocol="tcp"` | TCP 是传输层协议。 |
| M018-S006 | `protocol*="snmp*"` | 用协议前缀涵盖多个 SNMP 版本。 |
| M019-S010 | `protocol="memcached" && country="US"` | 服务类型与美国地域取交集。 |
| M020-S005 | `protocol="ftp" && base_protocol="tcp"` | 识别协议和传输层分别限定。 |
| M021-S010 | `protocol="ssh" && banner="OpenSSH_10.4"` | 同时验证协议标签和响应标记。 |
| M022-S007 | `country="JP" && app="HIKVISION-网络摄像头"` | **复核** 组件库中摄像头专用规则名。厂商整类标签可能混入其他设备。 |
| M023-S002 | `domain="163.com" && host*="?.?.163.com"` | 每个问号匹配一个字符。用裸域、三级子域和两字符标签作反例。 |
| M024-S005 | `product.version="10.0.0"` | **复核** 独立版本过滤是否支持，以及版本是否按完整值匹配。不要补产品。 |
| M025-S010 | `category="中间件"` | **复核** 选择产品分类菜单中的实际枚举。 |

## M026 至 M050 响应、证书和指纹

| 题号 | 查询候选或解法 | 要点 |
|---|---|---|
| M026-S002 | `body="NoSuchUpload"` | 标记出现在 HTML 正文。 |
| M027-S008 | `header="Content-Security-Policy-Report-Only:"` | 用冒号保留头字段名边界。 |
| M028-S011 | `type="subdomain" && status_code="308"` | 限定网站类记录。 |
| M029-S010 | `ip="192.0.2.0/24" && banner="SSH-2.0-OpenSSH_X.Y"` | **样例** 在文档网段用虚构版本匹配服务响应。 |
| M030-S003 | `header="Set-Cookie: PHPSESSID"` | **复核** 标准头部通常冒号后有空格，比较比赛索引对该空格的序列化。 |
| M031-S009 | `js_name="jquery.min.js"` | 使用脚本引用名称字段。 |
| M032-S010 | `country="NL" && city="Amsterdam" && port="554" && banner="200 OK"` | 200 OK 来自服务原文。 |
| M033-S003 | `cert.issuer.org="GlobalSign" && cert.is_expired=false` | 签发组织与过期状态分别判断。 |
| M034-S007 | `cert.subject.cn=="apple.com" && is_domain=true && domain!="apple.com"` | **复核** CN 完全相等，以及 domain 按主域排除的语义。 |
| M035-S010 | `cert="TrustAsia"` | 搜索证书全文。 |
| M036-S007 | `cert.subject.org="Alibaba"` | 按证书主体组织中的文字匹配。 |
| M037-S008 | `domain="sohu.com" && banner!="" && banner!="apache"` | **复核** 非空条件是否排除缺失字段。 |
| M038-S009 | `cert.domain="adobe.com"` | 证书中的根域使用专用字段。 |
| M039-S001 | `cert.subject.cn=="qq.com" \|\| cert.subject.cn*="*.qq.com"` | 以 CN 的裸域和点边界后缀表达其根域。 |
| M040-S011 | `cert="godaddy" && domain!="godaddy.com"` | 证书全文关键词与资产主域分别筛选。 |
| M041-S001 | `cert.domain="google.com" && is_honeypot=false && is_fraud=false` | 两个 FOFA 标记均为假。 |
| M042-S003 | `cert.subject.cn=="example.com" \|\| cert.subject.cn*="*.example.com" \|\| cert.domain="example.com"` | CN 裸域、CN 子域、证书根域任一命中。 |
| M043-S004 | `tls.version="TLS 1.0" && cert.is_expired=false` | **复核** TLS 版本枚举实际写法。 |
| M044-S007 | `jarm="21d21d00021d21d00021d21d21d21dce319294781f0e12867fd06896a000ea" && status_code="307" && header="Content-Length: 0"` | JARM、HTTP 状态码及头部取交集。复核头部值边界，避免把文本前缀当作数值结构。 |
| M045-S001 | `os="linux"` | 按操作系统标签筛选。 |
| M046-S013 | `country="CN" && country!="HK" && country!="MO" && country!="TW" && type="subdomain" && is_domain=false` | 无绑定域名也可归入网站类。 |
| M047-S011 | `cert.is_equal=false` | 比较签发者与持有者。 |
| M048-S001 | `country="CA" && cert.is_match=false` | 比较证书和域名，地域限定加拿大。 |
| M049-S010 | `country="TR" && tls.ja3s="f4febc55ea12b31ae17cfb7e614afda8"` | JA3S 与 JARM 是不同字段。 |
| M050-S004 | `org*="*Inc*"` | 匹配 ASN 组织名中任意位置的 Inc。 |

## M051 至 M080 组合查询与转义

| 题号 | 查询候选或解法 | 要点 |
|---|---|---|
| M051-S001 | `ip="192.0.2.0/24" && org="Example Hosting" && port="2087" && title=="示例管理面板"` | **样例** 网络组织、端口、完整标题取交集。 |
| M052-S002 | `domain*="*.io" && app="WordPress"` | 用主域后缀限定范围。组件名见 [FOFA 产品规则示例](https://en.fofa.info/php-hotspot)。 |
| M053-S009 | `country="CN" && region="Fujian" && city="Xiamen" && port="80" && status_code="200"` | **复核** 厦门与福建的地理标签。 |
| M054-S009 | `domain="example.com" && body="/api/example"` | **样例** 正文路径是接口线索。区域和网络组织可作为同层范围条件。 |
| M055-S003 | `domain="example.com" && header="Set-Cookie:" && body="/example-admin"` | **样例** 响应头与正文路径是同时满足的条件。 |
| M056-S008 | `domain="example.com" && (body="Login" \|\| body="admin" \|\| body="Dashboard" \|\| body="管理" \|\| header="Set-Cookie:")` | **样例** 主域约束全部五个分支。 |
| M057-S012 | `ip="192.0.2.0/24" && status_code="200" && (body="Welcome" \|\| body="Sign in" \|\| body="Dashboard")` | **样例** 网段和状态码必须同时成立，正文关键词三选一。 |
| M058-S012 | `country="DE" && (host="amazonaws.com" \|\| domain="amazonaws.com" \|\| cname_domain="amazonaws.com" \|\| cert="amazonaws.com" \|\| cert.domain="amazonaws.com")` | 德国地域约束五条线索。访问主机名的匹配边界需抽样核对。 |
| M059-S006 | `port="143" && cert.sn="1" && (cert="TAK" \|\| cert="ATAK" \|\| cert="WinTAK" \|\| cert="Squad" \|\| cert="CoT")` | 序列号保留为字符串，证书文字分支取并集。**复核** 若要求严格词边界，检查全文检索的分词。 |
| M060-S010 | `ip="113.247.228.0/24" && (port="80" \|\| port="443" \|\| port="8080" \|\| port="8443")` | 任一端口记录命中即可。 |
| M061-S005 | `(title="Sign in" \|\| title="Login") && port="80" && is_ipv6=false` | 两个标题分支共用端口及地址类型条件。 |
| M062-S010 | `country="NZ" && (port="80" \|\| port="443") && is_cloud=true` | 使用云资产布尔标签。 |
| M063-S001 | `(banner="IIS" && banner="1.") \|\| header="IIS"` | Banner 分支需要同时满足两项，响应头分支可独立命中。 |
| M064-S004 | `country="JP" && is_domain=true && port="443" && status_code="200" && cert.is_valid=true && (title="大学" \|\| title="学部" \|\| title="研究" \|\| body="入学案内" \|\| body="学生")` | 五个词任一匹配，前面的范围和状态条件始终成立。 |
| M065-S004 | `domain="example.com" && (title="系统" \|\| title="平台" \|\| title="登录" \|\| title="管理" \|\| title="API" \|\| host*="*api*.example.com" \|\| host*="*admin*.example.com" \|\| host*="*dev*.example.com" \|\| host*="*test*.example.com" \|\| host*="*uat*.example.com" \|\| host*="*pre*.example.com")` | **样例** 标题和子域名分支都受主域约束，子域分支只改变主域之前的部分。 |
| M066-S006 | `category="邮件系统" && title="Mail" && country!="US"` | **复核** 采用平台的邮件系统分类标签。 |
| M067-S008 | `country="CN" && country!="HK" && country!="MO" && country!="TW" && port="8888"` | 同一资产记录上明确排除三个地区代码。 |
| M068-S005 | `body="Jenkins" && title!="Jenkins"` | 题目未要求标题非空，无需添加空值条件。 |
| M069-S001 | `country="TW" && asn="20940" && port="443" && tls.version="TLS 1.3" && cert.issuer.org!="Let's Encrypt" && cert.issuer.org!="ZeroSSL"` | 两个签发组织都要排除。**复核** TLS 枚举和签发组织的索引值。 |
| M070-S003 | `country!="JP" && app="HFS" && status_code="200" && title=="HFS /" && cloud_name!="Amazon"` | **复核** HFS 组件和 Amazon 云标签。若 Amazon 按多项服务登记，对各实际枚举分别排除。 |
| M071-S006 | `country="CN" && country!="HK" && country!="MO" && country!="TW" && app="Grafana"` | **复核** Grafana 规则名。 |
| M072-S008 | `domain*="*.edu.ar" && (port="80" \|\| port="443") && server!="nginx" && server!="WAF"` | **复核** 若 WAF 仅由产品或类别识别，使用平台认可的排除字段，单用 Server 可能漏排。 |
| M073-S003 | `ip="1.1.1.1" && is_domain=true` | 指定 IP 和绑定域名的资产标记。 |
| M074-S008 | `cert.domain="aliyun.com" && cert.is_valid=true && is_domain=false` | 证书包含某个根域，与资产是否绑定域名是两个独立维度。 |
| M075-S006 | `country="CN" && is_domain=true && header="x-qiniu-zone:" && domain!="qiniudn.com"` | 域名存在、头字段存在与主域排除同时生效。 |
| M076-S004 | 见下方合成样例 | 标题为空、200、两类指纹均存在、主机名含关键词，最后再判断主域形态。 |
| M077-S009 | `body="class=\"container\""` | FOFA 字符串中的内部双引号使用反斜杠转义。 |
| M078-S009 | `body="\"exception\": \"Symfony\\Component\\ErrorHandler\\Error\\FatalError\""` | 先解码原题 JSON。查询值保留原题的引号和单个反斜杠。 |
| M079-S006 | `ip="40.88.0.0/16"` | 按题目的 B 段惯用含义保留前两个八位组，不使用历史有类寻址。 |
| M080-S006 | `(ip="47.96.0.1" \|\| ip="47.96.0.2/31" \|\| ip="47.96.0.4/31") && (port="80" \|\| port="443")` | 覆盖 .1 至 .5，排除 .0 和 .6。 |

## M081 至 M100 综合题与不可转换题

| 题号 | 查询候选或解法 | 要点 |
|---|---|---|
| M081-S011 | 见下方完整查询 | 至少两项有值可枚举四项中的六个二元组合。 |
| M082-S001 | 见下方完整查询 | 绑定主域或证书根域任一命中，全局同时限制证书匹配、受信和时间窗。 |
| M083-S008 | `banner="Server: nginx" && title="Welcome"` | **复核** Shodan 裸词搜服务响应，FOFA 的网页头与 Banner 分开索引。若题意只认 HTTP 头，使用 header 字段。 |
| M084-S004 | `title="Grafana" && body="JavaScript"` | **复核** Google intext 和 FOFA HTML 正文的索引方式不同，属于意图映射。 |
| M085-S001 | `body="login" && is_domain=true && (host*="*bet*" \|\| host*="*casino*" \|\| host*="*kasino*")` | HTML 与主机名任一关键词取交集。 |
| M086-S006 | `host*="*google*" && domain!="google.com" && is_domain=true && title==""` | Hunter 的 domain 表示完整域名，suffix 排除由 FOFA 主域表达。**复核** 标题字段必须存在且为空时的行为。 |
| M087-S009 | `header="Server: Tengine" \|\| header="X-Cache: MISS"` | **复核** 保持每组键和值在同一段文本，确认索引空格、大小写和完整值边界。 |
| M088-S004 | `region="HK" && server=="Apache" && asn!="4134"` | 修复弯引号，保留精确匹配。**复核** 比赛数据用 region=HK 还是 country=HK 划分香港。 |
| M089-S010 | `app="HFS" && (port="80" \|\| (port="8080" && status_code="200"))` | 修复混合竖线与弯引号。状态码仅约束 8080 分支。 |
| M090-S008 | 见下方完整查询 | 只保留指定的两个头部值，其余仅匹配字段名。 |
| M091-S009 | 该需求不能直接转换为FOFA搜索语句 | 1000000 超过端口取值上限 65535。 |
| M092-S007 | 该需求不能直接转换为FOFA搜索语句 | 1.2.3.4.5 既非合法 IP 也非合法 CIDR。 |
| M093-S004 | 该需求不能直接转换为FOFA搜索语句 | 同一记录的国家字段不能同时等于和不等于 US。 |
| M094-S004 | 该需求不能直接转换为FOFA搜索语句 | 缺少可检索的安全定义和排序指标。 |
| M095-S010 | 该需求不能直接转换为FOFA搜索语句 | 容器实时内存利用率需要运行时监控数据。 |
| M096-S001 | 见下方字段提取与合成样例 | 产品名和 Remoting 开头的字段各自匹配网页正文或服务响应，去除示例值。 |
| M097-S001 | 见下方图标复核方法 | 需要题目指定站点在评分索引中的图标哈希。 |
| M098-S001 | `is_domain=true && host*="*.gov.cn.*" && domain!="gov.cn"` | 结构候选，用于识别可疑嵌入式域名，见下方复核说明。 |
| M099-S001 | `app="APACHE-HTTP_Server"` | 使用 FOFA 发布的 Apache HTTP Server 组件规则。见下方产品归属说明。 |
| M100-S001 | `cert.issuer.cn=="ZeroSSL ECC DV SSL CA 2" && cert.sn="132577581667764526475870473477991557894"` | 签发者通用名称和证书序列号取交集。序列号转为精确十进制字符串。 |

## 需要展开的查询

### M076 空标题与主域形态

以下用合成记录演示。测试时让主域分别为 abc.test、abcd.test、abcde.test 和 example.invalid。abcd.test 和 example.invalid 应通过主域形态检查，且仍需满足所有其他条件。

```text
title=="" && status_code="200" && fid!="" && icon_hash!="" &&
host*="*review*" && (domain*="*.invalid" || domain*="????.test")
```

额外加入一条未采集标题的记录和一条未采集指纹的记录，分别检验空字符串和缺失字段的差异。

### M081 四个字段中至少两项存在

```text
app="Grafana" && (
  (header_hash!="" && body_hash!="") ||
  (header_hash!="" && icon_hash!="") ||
  (header_hash!="" && fid!="") ||
  (body_hash!="" && icon_hash!="") ||
  (body_hash!="" && fid!="") ||
  (icon_hash!="" && fid!="")
)
```

复核时用记录分别提供 0、1、2、3、4 项字段。后三种应命中。两项字段有值的情况应覆盖六种组合。

### M082 六个主域与证书到期时间窗

```text
(
  domain="kuaihuoyun.com" || cert.domain="kuaihuoyun.com" ||
  domain="wlhyos.cn" || cert.domain="wlhyos.cn" ||
  domain="wlhyos.com" || cert.domain="wlhyos.com" ||
  domain="56ctms.com" || cert.domain="56ctms.com" ||
  domain="xiaokuaikeji.com" || cert.domain="xiaokuaikeji.com" ||
  domain="zhihuiwuliu.org.cn" || cert.domain="zhihuiwuliu.org.cn"
) && cert.is_match=true && cert.is_valid=true &&
cert.not_after.after="2024-01-01 00:00:00" &&
cert.not_after.before="2026-08-30 18:17:49"
```

**复核** `cert.not_after.after` 和 `cert.not_after.before` 是否接受秒级时间、是否按严格边界比较，以及评分索引的时区。分别检查位于边界前一秒、恰好等于边界、后一秒的记录。若平台只支持日级精度，截断会改变结果集。保留原始时间值供核对。

### M090 从响应样本提取头部条件

```text
header="Content-Length: 1171" && header="Etag: \"6aa522a3-493\"" &&
header="Accept-Ranges:" && header="Cache-Control:" && header="Content-Type:" &&
header="Date:" && header="Last-Modified:" && header="Server:" && header="Vary:"
```

同名 Cache-Control 出现两次，但题目只要求该字段存在。状态行不生成条件。**复核** 索引如何保留双引号与头部空格，并用不同 Content-Length 值检验 FOFA 的文本匹配边界。

### M096 从 Jenkins 文档中提取字段

官方样例中符合前缀要求的字段是 `Jenkins-Agent-Protocols`、`Jenkins-Version`、`Jenkins-Session` 和 `Remoting-Minimum-Version`。示例版本号和会话值不进入检索。以下用文档网段演示正文与服务响应共八个分支。参见 [Jenkins TCP Agent Listener Port](https://www.jenkins.io/doc/book/security/services/#tcp-agent-listener-port)。

```text
ip="192.0.2.0/24" && (
  body="Jenkins-Agent-Protocols" || banner="Jenkins-Agent-Protocols" ||
  body="Jenkins-Version" || banner="Jenkins-Version" ||
  body="Jenkins-Session" || banner="Jenkins-Session" ||
  body="Remoting-Minimum-Version" || banner="Remoting-Minimum-Version"
)
```

### M097 核实 Octodex 图标哈希

打开 [GitHub Octodex](https://octodex.github.com/) 检查页面声明的站点 favicon。页面里的 Octocat 插画是内容资源。用 [FOFA 官方图标搜索](https://en.fofa.info/) 的上传入口读取实际图标对应的查找语句，或在比赛索引中读取已知 Octodex 记录的 `icon_hash`。查到后用该值匹配图标，并用精确主机名条件排除原站。以比赛快照存储的图标版本为准。本次未取得可核实的哈希值。

### M098 检查嵌入式仿冒域名的边界

题目给出的字符串把类似政府域名的部分放在另一个注册域下。候选只保留这一命名特征。用合成的 `agency.gov.cn.example.test` 作为正例，`agency.gov.cn` 和 `agency.gov.cn-example.test` 作为反例。命名模式只能筛出待复核结果。仅凭题目中的域名无法判断页面文案、图标或托管关联。

### M099 核实 CVE 的产品归属

Apache 的公告将该编号归于 **Apache HTTP Server**。题目问对应产品，因此直接采用 FOFA 发布的组件标识。组件标签识别的是产品，不能用来判定单个实例是否受该 CVE 影响。参见 [Apache 官方安全公告](https://httpd.apache.org/security/vulnerabilities_24.html#CVE-2021-41773) 和 [FOFA 产品规则示例](https://en.fofa.info/php-hotspot)。

### M100 证书序列号的进制转换

去掉冒号后，按十六进制大整数解析 `63BD83716214836EF80075356161EB06`，得到十进制 `132577581667764526475870473477991557894`。序列号在查询中仍作为字符串。`cert.issuer.cn` 对应签发者通用名称，可用不同签发者的合成记录检验同一序列号是否被正确排除。

## 后续整理答卷时的检查

- 按本包模板保留 `选手名称`、`参赛包编号` 和 `答案`。编号用 `pkg-08e82c8d`。输入、输出各有 100 个唯一题号且集合一致，最好沿用原顺序。
- 每个 `查询语句` 只保留一条已确认查询，或按 README 使用固定句 `该需求不能直接转换为FOFA搜索语句`。区分平台确实无法表达、暂时缺少输入和仍待确认的候选。
- 交付 JSON 时使用 JSON 序列化器处理第二层转义。把序列化后的文件重新解析，并检查 M077、M078、M090 得到的查询与原查询逐字符一致。文件使用 UTF-8，大小不超过 README 标注的 8 MB。
