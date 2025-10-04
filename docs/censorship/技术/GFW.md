---
title: "GFW"
description:
published: true
date: "2025-10-04T16:56:32"
特殊标签标记: #无标签
editor: markdown
dateCreated: "2023-02-17T17:44:15"
---

## 简介

Great Firewall 简称 GFW，被用来描述大陆的网络封锁，因为这种审查的本质上是一个巨型网络防火墙，
所以有人将<ruby>长城<rt>Great Wall</rt></ruby>这个词变体为了<ruby>长城防火墙<rt>Great Firewall</rt></ruby>。

## 原理

GFW 有许多手段阻止网络连接，其中包括但不限于：

+   [DNS 污染](/censorship/技术/DNS_污染.md)
+   [TCP 重置攻击](/censorship/技术/TCP_重置攻击.md)
+   深度包检测
+   SNI 干扰机制（网络劣化）
+   中间人攻击

关键词：

+   [SNI](#sni)
+   [依附的自由](/theme/突破网络审查主题.md#依附的自由)

### 网络数据包

网络的基本原理是 [网络数据包][]，服务器与客户端互相通过网络数据包来通讯。网络数据包往往包含协议标识符、IP 地址和内容等。

[网络数据包]: https://en.wikipedia.org/wiki/Network_packet

由于早期网络世界缺乏密码学，各种协议都算是明文，这就给了 GFW 审查网络的机会。

早期的 GFW 能识别 http 协议的 IP 地址，然后将需要封锁的网站的数据包丢弃即可。

> [!note]+ 丢包与黑洞
>
> 使用 DNS 或 hosts 来去广告时，往往会将目标域名引向 0.0.0.0 或 127.0.0.1 这样的保留 IP 地址，
> 当浏览器发现访问到这样的地址时，就会立刻停止访问，算是中断广告的好方法，因为不会影响网站的正常使用。
>
> 这种用法被称作黑洞（路由），如果互联网的基础设施也这么做，对于用户来说，就是一种静默丢包，GFW 从而达成 IP 封锁的效果。

除了直接研究单个数据包，GFW 也会将被分片的数据包重建、重组（拼）成完整的内容，比如重建出完整的 HTTP、TELNET、
FTP 等明文协议，然后进行检测，比如 HTTP 协议会检测 URL 是否是关键词，从而选择性丢弃数据包。

重建网络数据包，算是深度包检测的前置条件，根据 GFW 对数据包的探测情况，有以下称呼：

| OSI 模型 | 协议 | 名称           |
| -------- | ---- | -------------- |
| 网络层   | IP   | （常规包检测） |
| 传输层   | TCP  | 浅度包检测     |
| 应用层   | HTTP | 深度包检测     |

时间来到 HTTPS（HTTP over TLS）的时代，由于 TLS 是加密的算法，所以数据包不再是明文，对于 GFW 而言是一种打击，
比如 Steam 在启用全站 HTTPS 之前，经常会遭受打不开网站的情况。[^11470][^55413]

[^11470]: 店长, 《[一个困难的请愿：请 Steam 启用全站 SSL 加密。](https://web.archive.org/web/20230813184514/https://bitinn.net/11470/)》, 比特客栈的文艺复兴, 2017-07-20. (参照 2023-10-24).

[^55413]: 辰一丶, 《[Steam社区已强制使用https协议 轻松访问再无压力](https://web.archive.org/web/20211212045615/https://www.gamersky.com/news/201806/1055413.shtml)》, 游民星空, 2018-06-03. (参照 2023-10-24).

不过这种情况没有持续太久，因为 GFW 开始大规模实施了 SNI 封锁。

SNI 是 Server Name Indication（服务器名称指示）的缩写，SNI 会找 TLS 握手时明文传输，
目的是让共用 IP 地址和 TCP 端口号的网站能被区别，但也因为 SNI 会明文传输，
所以 Cloudflare 的联合创始人兼首席执行官 Matthew Prince 表示：传统 SNI 绝对是加密装甲中的最后缝隙之一。
{: #sni }

所以 GFW 能通过深度包检测，检测到应用层的 SNI 握手明文，然后丢包或者发送 TCP 重置，从而封禁特定网站。

对此，存在名为<ruby>[域前置](/anti-censorship/域前置.md)<rt>Domain fronting</rt></ruby>的绕过方案。

### 限制 QUIC

QUIC 是 Google 设计的传输层网络协议，基于 UDP，目的是解决 TCP 性能较低的问题。

2017 年，Google 开始在 YouTube 上少量测试 QUIC（HTTP/3）传输视频流，而在当时只要解决 DNS 污染问题，
就可以直连 YouTube 服务器观看视频。[^70189]

[^70189]: alect, 《[谷歌给我们省钱呢……y2b 走 quic 协议不经过代理……直连了](https://web.archive.org/web/20210114065857/https://www.v2ex.com/t/370189)》, V2EX, 2017-06-21. (参照 2023-10-24).

不过很快 QUIC 就被 GFW 限制或封锁了，无法再直连观看 YouTube。以至于需要关闭浏览器的 QUIC 功能，
避免被 QoS（运营商针对 UDP 干扰）影响。[^nh2-3][^12287]

[^nh2-3]: Toyo, 《[Chrome浏览器关闭 QUIC，避免部分地区运营商UDP QOS对速度的影响](https://web.archive.org/web/20220512074451/https://doubibackup.com/zly21nh2-3.html)》, 逗比根据地（备份）, 2017-08-30. (参照 2023-10-24).

[^12287]: 老刘, 《[禁用QUIC协议 保障网页加载速度](https://web.archive.org/web/20230529180508/https://www.livelu.com/201712287.html)》, 生活之路, 2017-12-10. (参照 2023-10-28).

因为 QoS 被认为是 GFW 干扰 UDP/QUIC 的手段，[^30080] 并且早期的 Proxy 工具，比如 Shadowsocks 不会让 UDP Over TCP，
所以出现 QUIC 连接时，SS 客户端会以 UDP 与服务端通讯，[^s1127] 结果 UDP 连接可能会因为 QoS 而非常不稳定，或是直接中断。

[^30080]: Anthr@X, 「[最近发现GFW在UDP无差别丢包上给QQ开了后门，目测丢包手段用的是路由器的QoS功能,大部分端口丢包率在8%-10%左右，ovpn最多能承受2%的丢包率，所以UDP服务要么换端口，要么改进算法加入冗](https://web.archive.org/web/20231024143855/https://twitter.com/anthrax0/status/481318169882030080)」, X（Twitter）, 2014-06-24. (参照 2023-10-24).

[^s1127]: ArchGuyWu, 《[能支持udp over tcp吗？ · Issue #1127 · shadowsocks/shadowsocks-rust](https://web.archive.org/web/20231024143113/https://github.com/shadowsocks/shadowsocks-rust/issues/1127)》, GitHub, 2023-03-03. (参照 2023-10-24).

<!--
    然而，GFW 以能封尽封为哲学基础，罔顾历史发展规律，堂而皇之地做出一系列荒唐行为。
-->

### DNS 污染

详情请阅读 [DNS 污染](/censorship/技术/DNS_污染.md) 条目。

### 限制干净的 DNS

DNS 的问题很多，因为最初设计的时候没有考虑安全性，结果导致网络存在许多安全隐患，加强 DNS 的安全性就成了重要的工作。
但 GFW 也不会坐视不管。

〔待续〕

## 历史

### 电子邮件

2006年10月，一些向境外发出的电子邮件会被退信，而接收方会收到「aaazzzaaazzzaaazzzaaazzzaaazzz」内容的空白邮件。[^41237]
在检查邮件服务器日志后，有人发现是中间人返回了退信通知，并给窜改邮件内容。而这个中间人就是 GFW。[^41029]

[^41237]: 钉子, 《[没错还是它！GFW让邮件内容变成了aaazzzaaazzzaaazzzaaazzzaaazzz](https://web.archive.org/web/20080821155840/http://blog.5dmail.net/user1/1/20061018141237.html)》, 钉子的博客, 2006-10-18. (参照 2024-04-26).

[^41029]: xmbbx, 《[\[原创\] 证实收到’aaazzzaaazzzaaazzzaaazzzaaazzz’邮件的真实原因](https://web.archive.org/web/20120202093648/http://bbs.chinaunix.net/forum.php?mod=viewthread&tid=841029)》, ChinaUnix.net, 2006-10-13. (参照 2024-04-26).

不过此问题持续时间较短，通常几小时，所以后续记录较少。[^70718] 但是 2007年7月，此问题持续多日困扰着人们，
于是 TOM163 邮箱[^70717]、中国万网企业邮局[^2312] 等电子邮箱服务商，都发布相关公告。

[^70718]: Reuters, 《[Chinese Internet censors blamed for email chaos](https://web.archive.org/web/20190224115746/https://www.reuters.com/article/us-china-internet/chinese-internet-censors-blamed-for-email-chaos-idUSPEK9185520070718)》, Reuters, 2007-08-09. 参照: 2024-04-26. [Online].

[^2312]: 中国万网客服中心, 《[万网关于海外邮件通信问题的进展通告](https://web.archive.org/web/20070811062826/http://www.net.cn/service/a/zytz/200707/2312.html)》, 万网, 2007-07-18. (参照 2024-04-26).

[^70717]: TOM163客服中心, 《[国际电邮通信问题的重要通知](https://web.archive.org/web/20070817040525/http://vip.tom.com/popup/070717.html)》, TOM163, 2007-07-17. (参照 2024-04-26).

TOM163 邮箱的说法是「国家主干网与国际线路连接不稳定」，但没有说到关键点。万网同样没有指出关键点，
但至少给出了意味深长的「未知的技术问题」。

GFW 修改内容为 a28z，也许有些多此一举，应该可以无感知的单纯拦截邮件。并且为什么是 a28z，这可能是永远不会有答案的问题吧。

### Steam

2018年8月 月底，一则 Steam 动态开始流传，内容如下：

> [!quote]+ 未知细节的 Steam 动态[^67073]
>
> 还能上动态的伙计们，告诉你们个不幸的好消息，大清研制的「TCP 旁路阻断」技术已脱离 EA 阶段，现已正式发行。[^ea]
>
> 发行受众：全天朝人民<br>
> 发行价格：免费<br>
> 针对范围：当局熟知的「够毙名单」上所有的域名，例如 [Pixiv](/company/pixiv.md)，各种外国社交软件，Steam 社区等<br>
> 效果：DNS 污染的升级版，改 hosts 文件大法已失效
>
> 怎么说呢，这一天来临其实不奇怪，很多人也料到了。。其余的不予置评，毕竟当局也是衣食父母，不要抱怨太多喵。

[^67073]: Eji, 「[今天中国的大话题是 GFW 的加强和 PIXIV 的官方封锁……](https://web.archive.org/web/20231023013117/https://twitter.com/ejiwarp/status/1040270990952067073)」, X（Twitter）, 2018-09-14. (参照 2023-10-23).

[^ea]: EA 是指 Early Access（抢先体验），指代一些游戏在正式版本发售前，发布的评估版本。这里是指之前也有「TCP 旁路阻断」，但属于小规模测试。

这标志着「TCP 旁路阻断」正式成为 GFW 的主要功能，开始大规模使用。

2021年3月，[GitHub 遭到了网络劣化](/website/GitHub.md#网络劣化)。但在 2023年2月17日，被发现相关劣化解除了，
并且 Steam 的网络劣化也消失了（仅指 store.steampowered.com）。[^916909]

[^916909]: XIU2, 《[Steam、Github 域名疑似已解除 SNI 干扰（已无法复现），可以正常链接了？](https://web.archive.org/web/20230217093733/https://www.v2ex.com/t/916909)》, V2EX, 2023-02-17. (参照 2023-02-17).

### Greasy Fork

2017年10月2日，知名用户脚本 ([userscript](https://en.wikipedia.org/wiki/Userscript)) 分享网站 Greasy Fork，
被发现无法在大陆打开。于是有用户认为 Greasy Fork 被封锁，但也有人认为是 `ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js` 被墙，
所以导致网站需要很久才能打开。[^56219]

[^56219]: hacker6, 《[中国的greasyfork被墙了吗，打不开了啊，所有地区都打不开了](https://web.archive.org/web/20250210133851/https://greasyfork.org/en/discussions/greasyfork/56219-中国的greasyfork被墙了吗-打不开了啊-所有地区都打不开了)》, Greasy Fork Discussions, 2017-10-02. (参照 2025-02-10).

> [!note]+ 关于油猴
>
> 更常见的称呼是油猴脚本，之所以使用油猴代之，[可能](https://sspai.com/post/68574) 是因为最初流行的插件，
> 名为 Greasymonkey（油腻猴子），之后即使出现了 Violentmonkey（暴力猴），也没有撼动这个脚本类型的名称。

2025年2月6日，Greasy Fork 再次被发现无法在大陆开启。这次经验丰富的用户发现，网站遭到了 [SNI 封锁](#sni)，
以及 [DNS 污染](/censorship/技术/DNS_污染.md)。[^78992] 被封原因尚不明确，也许是上面有各种自动刷题的脚本、
去广告的脚本，亦或是含有敏感词的脚本。

[^78992]: letsgogogo, 《[greasyfork.org 已被中国封锁！](https://greasyfork.org/zh-CN/discussions/greasyfork/278992-greasyfork-org-已被中国封锁)》, Greasy Fork Discussions, 2025-02-06. (参照 2025-02-10).

## 没有使出全力的 GFW

最初的 GFW 没有打击各种 Proxy（代理）工具并没有用尽全力，可能是考虑到使用 Proxy 的用户，需要从事外贸等。
这算是依附的自由。

之后常见的 Socks、PPTP、L2TP 等烂大街协议，开始被限制，不过 Cisco AnyConnect 被刻意忽略了较长时间。
可能是因为 Cisco AnyConnect 用户的价值更高，所以这段时间，第三方开发者编写的兼容端 OpenConnect 能够获得不错的效果。
这也算是依附的自由。

（从特殊时间〔比如开会〕，大多数工具难以连接来看，GFW 有能力禁止大多工具，只是考虑到影响，没有这么做）

2024年4月11日，有 V2EX 用户表示，自己的 AnyConnect VPN 无法连接到成都移动企业宽带的公司，需要使用有备案的域名，
才不会被 SNI 阻断。评论中有人说，上海电信在 2 月的月底，也出现了类似的情况。[^31577]

[^31577]: hikarikongou, 《[成都移动企宽开始封未备案域名 SNI 了](https://web.archive.org/web/20240411042641/https://v2ex.com/t/1031577)》, V2EX, 2024-04-11. (参照 2024-04-11).

## 另一种分裂网风险

「网络无国界」曾经是与「信息高速路」经常提及的互联网概念，表示通讯的全球化、无障碍、快捷等含义。不过由于各种原因，
网络上出现了各种分裂网。

比如政府各式各样的审查，带来了 Tor 网络等暗网；商业公司出于各种考虑，带来了无法被外部检索的深网；GFW 将网络半切割；
朝鲜直接是大型局域网。

这样的互联网分离状态，被相关研究者称为 [分裂互联网](https://en.wikipedia.org/wiki/Splinternet) 或者网络巴尔干化。

不过还存在另一种分裂网风险，就是基础协议的分裂，比如 [W3Techs 统计](https://w3techs.com/technologies/details/ce-http3)，
使用 HTTP/3（QUIC）的网站有 27 %。QUIC 会用到 UDP，而 UDP、GFW 与运营商之间存在难以调和的矛盾。结果就是像 IPv6 一样，
拖慢了部署，等到实际部署，也是在 GFW 准备好后。

防篡改、加密相关的，比如 DNSSEC、ESNI、ECH、DNSCrypt、DoT、DoH 等技术，均因为 GFW，在境内得不到推广。
这么看来 TLS 能被允许，也是因为 TLS 存在 SNI 漏洞吧。

时间来到 20 ~ 30 年后，也许 GFW 外已经禁用了不安全的协议，而在 GFW 内，依然使用着不安全的协议，这就是另一种分裂网风险。

## 其他时间线

### 2021年5月 屏蔽 vercel

2021年5月16日，有用户发现 GFW 墙了 `vercel.com`、`vercel.app` 域名。[^gh566]

[^gh566]: xinjiajuan, 《[【关于vercel被墙】vercel在国内被墙 · Issue #566 · travellings-link/travellings](https://github.com/travellings-link/travellings/issues/566)》, GitHub, 2021-05-16. (参照 2025-10-04, [Internet Archive](https://web.archive.org/web/20240715085252/https://github.com/travellings-link/travellings/issues/566)).

### 2023年8月5日 河南审查加强

2023年8月5日，河南被洛阳联通电信发现加强了网络审查，[^62578] 最初有人以为是白名单，
后来进行实验后发现是某种根据侵略性的黑名单。并且如果设备上开启了 RFC 1323 时间戳这个功能，
就不会感受到被强化后的 GFW 的效果。[^i2229][^i2426]

[^62578]: benbeu, 《[河南洛阳联通电信好像都是域名白名单了，有了解的朋友嘛？](https://web.archive.org/web/20230812184100/https://www.v2ex.com/t/962578)》, V2EX, 2023-08-05. (参照 2023-08-13).

[^i2229]: toyo2333, 《[reality里client-fingerprint可以不同端用一套么？ · Issue #2229 · XTLS/Xray-core](https://web.archive.org/web/20230812225350/https://github.com/XTLS/Xray-core/issues/2229)》, GitHub, 2023-06-20. (参照 2023-08-13).

[^i2426]: 5e2t, 《[河南新上的SNI/HOST黑名单墙 · Issue #2426 · XTLS/Xray-core](https://web.archive.org/web/20230813032548/https://github.com/XTLS/Xray-core/issues/2426)》, GitHub, 2023-08-10. (参照 2023-08-13).

2023年8月14日，有博主表示自己没有备案的博客，被河南的新型 GFW 审查模式给阻断了。[^asbt]

[^asbt]: MisakaNo, 《[启用 TCP timestamps 以解决 SNI 阻断问题](https://web.archive.org/web/20230828102030/https://blog.misaka.rest/2023/08/14/anti-sni-block-timestamps/)》, MisakaNo の 小破站, 2023-08-14. (参照 2023-09-04).

2023年9月2日，V2EX 上有讨论贴讨论了家宽自建非常规端口的 HTTP 服务器遇到的问题，
大致表示河南的新型 GFW 审查模式会阻断没有备案的域名。[^70368]

[^70368]: lanwairen123, 《[在河南郑州除了 vpn 还有访问自建 http/https 服务的途径吗，杀疯了](https://web.archive.org/web/20230904025320/https://www.v2ex.com/t/970368)》, V2EX, 2023-09-02. (参照 2023-09-04).

### 2023年8月 直接限制 VPN 使用者

> [!note]+ 备注
>
> 本文与 GFW 关系不大，待拆分到其他合适的条目。

2023年8月 ~ 9月，有手机用户遭到停机，咨询后得知：「因为安装了 Telegram 之类的，诈骗高风险软件」。如要恢复，
需要去营业厅检查和签承诺书。[^72773][^mx1568]

[^72773]: leyun2017, 《[手机因为安装了 telegram 导致号码被停机，移动要求带着手机去营业厅检查和签承诺书怎么办](https://web.archive.org/web/20230911094253/https://v2ex.com/t/972773)》, V2EX, 2023-09-11. (参照 2023-09-17).

[^mx1568]: 萌欣的小窝, 「[详情：昨天手机号被移动公司冻结了……](https://web.archive.org/web/20230809141331/https://t.me/s/mengxin10824_chan/1568)」, Telegram, 2023-06-24. (参照 2023-10-09).

> [!note]+ 原因猜测
>
> Telegram 即使设置了代理，也会在部分时候直连。或者在代理软件未启动时，Telegram 后台自启并进行网络连接。
> 因此暴露手机安装了 Telegram。
>
> 解决方法可能需要使用 AFWall+ 等防火墙，禁止 Telegram 等软件的直连，尽量避免泄漏任何 Telegram 的痕迹。
>
> 另一个较低的可能性是系统或者某些软件，读取了手机上已安装的软件清单，并上传。随后分享给了运营商，
> 所以运营商知晓了用户安装了 Telegram。

2023年9月15日 或 16 日，多个内蒙古自治区乌海市的手机用户，收到了公安局的群发短信：[^76760][^68249]

[^76760]: 李老师不是你老师, 「[9月16日，内蒙古乌海市的市民收到了公安局的群发短信，称“您的手机疑似安装了VPN等‘翻墙’软件，请及时卸载”。](https://web.archive.org/web/20230917065206/https://twitter.com/whyyoutouzhele/status/1702992057596276760)」, X（Twitter）, 2023-09-16. (参照 2023-09-17).

[^68249]: 吴文行wenxingwu, 「[嗅觉这么灵敏？……](http://archive.today/2023.09.17-074248/https://twitter.com/wuwenhang/status/1702951041279668249 "http://archive.today/pXv44")」, X（Twitter）, 2023-09-16. (参照 2023-09-17).

> [!quote]+ 短信
>
> 【乌海市公安局】【乌海市公安局】：您的手机疑似安装了 VPN 等“翻墙”软件，请及时检查并卸载。否则，公安机关将依据《
> [中华人民共和国计算机信息网络国际联网管理暂行规定](/rule/国务院/中华人民共和国计算机信息网络国际联网管理暂行规定.md)》
> 第六条、第十四条之规定给予警告，并处 15000 元一下罚款。

2023年9月16日，江西某高校发布通知：[^19432]

[^19432]: 李老师不是你老师, 「[9月16日，江西某高校发布通知 将开展网络“翻墙”问题专项排查整治活动……](http://archive.today/2023.09.17-074253/https://twitter.com/whyyoutouzhele/status/1702983498489819432 "http://archive.today/2pvG6")」, X（Twitter）, 2023-09-16. (参照 2023-09-17).

> [!quote]+ 【关于开展网络“翻墙”问题专项排查政治活动的通知】
>
> “翻墙”属于违法行为，根据《
> [中华人民共和国计算机信息网络国际联网管理暂行规定](/rule/国务院/中华人民共和国计算机信息网络国际联网管理暂行规定.md)
> 》相关条款明确，“计算机信息网络直接进行国际联网，必须使用邮电部国家共用电信网提供的国际出入口信道。
> 任何单位和个人不得自行建立或者擅自使用其他信道进行国际联网，违反上述规定的，由公安机关责令停止联网，给予警告，
> 可以并处 15000 元一下的罚款；有违法所得的，没收违法所得”。
>
> 请班上同学一定强化遵纪守法观念和安全防范意识，做到科学规范上网。根据学校安排，现请各班周末 2 天内完成自查，
> 做好统计摸排，并填写各班《自查表》；
>
> 下周开始，智慧校园管理中心技术排查阶段（9月18日 - 25日），将安排技术人员才用网络流量勘察和现场检测方式开展整治活动，
> 请大家不要以身试法。
>
> 《班级-网络“翻墙”自查
> 表》【报送时间】9月17日（周
> 〇）〇〇〇〇〇【报送邮箱】：
> 〇〇〇〇〇〇〇〇〇〇
>
> 谢谢大家！

备注：「〇」表示被遮挡的文字。

### 2023年10月1日 多域名被封锁

2023年10月1日，多个域名被封锁，包括 Minecraft、Visual Studio Code，原理是 DNS 污染，访问这些域名会被重定向到「
[国家反诈中心](/censorship/国家反诈中心.md)」页面。[^08371]

[^08371]: 林小槐, 《[部分地区（主要是移动）把 mojang 的正版验证 API 给屏蔽掉了 xs](https://web.archive.org/web/20231002015758/https://twitter.com/Stapx_Happy/status/1708305027293708371)》, X（Twitter）, 2023-10-01. (参照 2023-10-02).

[Cloudflare](/serviceprovider/Cloudflare.md) 的 `https://1.1.1.1` 也被发现遭到干扰。

> [!abstract]+ 找到影响的域名清单
>
> +   [https://api.minecraftservices.com](https://web.archive.org/web/20231001024856/https://www.17ce.com/site/http/20231001_ba8ef210600411eeac17e316c626d952:1.html), 17CE.
> +   [https://api.mojang.com](https://web.archive.org/web/20231001025239/https://www.17ce.com/site/http/20231001_142c61e0600511ee9594d1b290c5208d:1.html), 17CE.
> +   [https://code.visualstudio.com/](https://web.archive.org/web/20231001064755/https://www.17ce.com/site/http/20231001_0b1fc800602611eeac17e316c626d952:1.html), 17CE.
> +   [https://vercel.com/](https://web.archive.org/web/20231001064730/https://www.17ce.com/site/http/20231001_d1a7b6f0602511ee9594d1b290c5208d:1.html), 17CE.
> +   [https://www.minecraft.net](https://web.archive.org/web/20231001025634/https://www.17ce.com/site/http/20231001_e4567b80600511ee9594d1b290c5208d:1.html#s304437), 17CE.
> +   [https://1.1.1.1](https://web.archive.org/web/20231002015338/https://17ce.com/site/http/20231002_a5ed427060b911ee9594d1b290c5208d:1.html)》, 17CE.

### 2023年11月9日 波动

2023年11月9日，许多 BandwagonHost VPS（搬瓦工）用户发现，自己购买的搬瓦工 VPS，被 GFW 封禁了。[^29188][^90329]
之后逐渐恢复，同时被封禁的还有新加坡腾讯云。[^17312] 可能是 GFW 在调试某些功能。

[^29188]: Ronin, 《[震惊！瓦工大规模被墙](https://web.archive.org/web/20231109073619/https://hostloc.com/thread-1229188-1-1.html)》, 全球主机交流论坛, 2023-11-09. (参照 2023-11-10).

[^90329]: jcfkccp, 《[大墙又发威了？](https://web.archive.org/web/20231109105753/https://www.v2ex.com/t/990329)》, V2EX, 2023-11-09. (参照 2023-11-10).

[^17312]: https://t.me/c/1362299370/7312

### 2023年11月 封禁原神亚服

2023年11月，有《原神》用户发现 `osasiadispatch.yuanshen.com` 域名被 SNI 阻断，导致无法直连国际版的《原神》亚服，
可能在一年前就已经有类似情况，有人猜测是亚服涌入的人太多，地理位置近，还是 cloudflare 的 IP，
所以触发了 GFW 的风控。[^93059]

[^93059]: SunsetShimmer, 《[原神 osasiadispatch 域名被墙了？](https://web.archive.org/web/20231119085828/https://www.v2ex.com/t/993059)》, V2EX, 2023-11-18. (参照 2023-11-19).

### 2025年 屏蔽 Let's Encrypt 的 CRL 域名

2025年7月11日，有用户在得知 Let's Encrypt 将从 OCSP 切换到 CRL 时，对 CRL 在大陆的可访问性好奇，于是进行了测试。
测试结果显示 GFW 会屏蔽 Let's Encrypt 的 CRL 域名 `e5.c.lencr.org`，方式是 TCP RST。[^44666]

[^44666]: MiKing233, 《[关于 Let’s Encrypt 签发的证书 Ending OCSP Support in 2025 的后续问题](https://web.archive.org/web/20250822144257/https://www.v2ex.com/t/1144666)》, V2EX, 2025-07-11. (参照 2025-09-06).

截至 2025年8月20日，此屏蔽依然存在。[^53589]

[^53589]: villivateur, 《[Let’s Encrypt 颁发的证书所包含的 CRL 链接完全被墙，可能导致所有用 LE 证书的网站在国内无法打开或者弹出警告。](https://web.archive.org/web/20250820065243/https://www.v2ex.com/t/1153589)》, V2EX, 2025-08-20. (参照 2025-09-06).

### 2025年8月20日 屏蔽境外 443 端口

2025年8月20日 约 00:34 至 01:48 间，GFW 对所有指向 TCP 443 端口的连接无条件注入伪造的 TCP `RST+ACK` 包，
导致连接中断。[^50820]

[^50820]: Mingshi Wu, 《[2025年8月20日中国防火长城GFW对443端口实施无条件封禁的分析](https://web.archive.org/web/20250902021856/https://gfw.report/blog/gfw_unconditional_rst_20250820/zh/)》, GFW Report, 2025-08-19. (参照 2025-09-06).

期间，苹果、特斯拉和必应等网站均受到干扰或阻断，无法正常使用。[^53869]

[^53869]: 中國警告☮️ Alert Cina, 「[2025年8月20日0点34分,中国防火长城GFW（Great Firewall）突然开始屏蔽所有的境外443端口……](https://x.com/AlertCina/status/1957981000107253869)」, X (formerly Twitter), 2025-08-20. (参照 2025-09-06).

## 相似软件或服务

### 默安科技

2019年3月13日，社群上出现了一张来源不明的照片。主要内容是一个宣传产品的介绍，标题是「翻墙行为态势监测」，
文字有「领先的第三方云计算安全服务商 云下到云上 开发到运营」，以及产品名称「幻阵」，功能「加密通道态势感知」
「社工采集系统」。「加密通道态势感知」的内容中能看到 VPN 三个字母的图标，以及自由门的标志。[^07420]

[^07420]: 好五倍, 《[【图说天朝】“翻墙行为态势监测”](https://web.archive.org/web/20220704000142/https://chinadigitaltimes.net/chinese/607420.html)》, 中国数字时代, 2019-03-14. (参照 2024-09-21).

中国数字时代的编辑根据图中的信息，发现这是默安科技公司的服务。曾保障「两会」，以及互联网大会乌镇峰会的信息安全。

### 安博通

2024年9月4日，可视化网络安全专用核心系统产品与安全服务提供商：安博通，在微信公众号发布了文章：
《嘿！我看到你「翻墙」了！》。[^DdZ4S]

[^DdZ4S]: 安博通, 《[嘿！我看到你“翻墙”了！](http://archive.today/2024.09.14-182902/https://mp.weixin.qq.com/s/ywlOrOoMqWjB7TKMaNFSFw)》, 微信公众号, 2024-09-14. (参照 2024-09-21).

文章介绍了翻墙概念、翻墙违法、高校治理翻墙和案例。关于治理细节有提到：利用助机器学习与 JA4 指纹，
能识别代理软件流量和代理协议，从而精准封堵。示例图片中出现了以下软件或协议（近 10 天共检测到的翻墙行为）：

| 名称            | 占比 |     次数 |
| --------------- | ---: | -------: |
| 蓝灯（Lantern） | 16 % | 31468 次 |
| 自由门          | 17 % | 33748 次 |
| SOCKS5          |  6 % | 12000 次 |
| WireGuard代理   |  6 % | 12429 次 |
| ShadowRocket    | 15 % | 28433 次 |
| ShellVPN        |  6 % | 11674 次 |
| Clash           | 10 % | 18786 次 |
| Proxy_NATAPP    |  4 % |  8032 次 |
| 无界浏览        | 14 % | 26677 次 |
| 其他代理        |  6 % | 11909 次 |

比较奇怪的是 Shadowrocket 与 Clash 属于客户端软件，并非流量协议，理应无法识别。然后是蓝灯，现在蓝灯太容易被 GFW 干扰，
理应没有多少人使用。最后是 Proxy_NATAPP，NATAPP 是做内网穿透用的服务，类似花生壳，注册还需实名认证，
应该无法直接用来翻墙。

## 相关研究

许多人对 GFW 有更深入的研究：

> [!abstract]+ 相关研究
>
> +   gfwrev, 《[深入理解GFW：内部结构](https://web.archive.org/web/20230918212230/http://gfwrev.blogspot.com/2010/02/gfw.html)》, GFW技术评论, 2010-02-18. (参照 2023-10-28).
> +   DDoSolitary, 《[GFW的DNS包伪造的简单研究](https://web.archive.org/web/20230518081344/https://blog.ddosolitary.org/posts/research-on-dns-packet-forgery-of-gfw/)》, DDoSolitary’s Blog, 2017-09-26. (参照 2023-10-28).
> +   清雨影, 《[中国网络防火长城简史](https://web.archive.org/web/20230518085534/https://blog.tsingjyujing.com/spam/gfw-history)》, 清雨影的Blog, 2023-01-09. (参照 2023-10-28).
