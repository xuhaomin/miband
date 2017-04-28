# 小米手环
> 最近抓包破解了网易云音乐,keep等app,现在对于具有社交功能/物联网的app后台数据交互十分感兴趣,于是把魔爪伸向了小米运动
- 抓包分析,小米手环与小米运动客户端交互数据后,小米运动客户端会向后台传送健康和运动数据.
## 运动数据格式及请求头
### 包分析
- 运动数据首先会以GET请求向 hm.xiaomi.com/v1/device/active_history.json?r={uuid}&t={time} 发送你的手机及手环信息
- 信息内容是 urlencode 过的 `{'details': ['{"v":"2","id":"{\\"sysimei\\":\\"\\",\\"sysphone\\":\\"\\"}","phone":"{\\"brand\\":\\"Xiaomi\\",\\"model\\":\\"MI 5\\",\\"osversion\\":\\"6.0\\",\\"systemtype\\":\\"MIUI 7 V7.5.4.0.MAACNDE | 稳定版\\",\\"country\\":\\"CN\\",\\"language\\":\\"zh\\",\\"carrier\\":\\"中国移动\\",\\"network \\":\\"wifi\\",\\"resolution\\":\\"1920x1080\\"}","app":"{\\"appversion\\":\\"2.4.1\\",\\"channel\\":\\"Normal\\"}"}'], 'userid': [''], 'imei': [''], 'device_type': ['']}`
- 请求头中有一个 apptoken ,应该是用作认证
- 之后会收到服务器返回的 类似200的信息
- 接着就会向 hm.xiaomi.com/v1/data/band_data.json?r=ffffffff-efef-dfdc-ffff-ffffaf6fffcb&t={time} POST发送运动数据
- 数据是编码过的json数据
- 数据内容是 `{'v': ['2.0'], 'uuid': [''], 'device': ['android_23'], 'data_json': ['[{"data_hr":" ","date":"2017-04-28","data":[{"start":0,"stop":1439,"value":"","tz":32,"did":"","src":8}],"summary":"{\\"v\\":5,\\"slp\\":{\\"st\\":1493308800,\\"ed\\":1493308800,\\"dp\\":0,\\"lt\\":0,\\"wk\\":0,\\"usrSt\\":-1440,\\"usrEd\\":-1440,\\"wc\\":0},\\"stp\\":{\\"ttl\\":178,\\"dis\\":120,\\"cal\\":4,\\"wk\\":4,\\"rn\\":0,\\"runDist\\":11,\\"runCal\\":0},\\"goal\\":8000}","source":8,"type":0}]'], 'timezone': ['Asia/Shanghai'], 'cv': ['3149_2.4.1'], 'device_type': ['0'], 'last_deviceid': ['F7C227FFFE0111AC'], 'channel': ['Normal'], 'lang': ['zh_CN'], 'last_sync_data_time': [''], 'appid': [''], 'userid': [''], 'last_source': ['8'], 'country': [''], 'callid': ['']}`
- data_hr 是形如 `/v7 /v7 /v7 /v7 /v7 /v7 /v7 /v7  ` 的一段代码,中间夹杂着`////` ,总长1920.
- 注意 1920 = 24*4*20,因此猜测小米手环是每三分钟唤醒记录一次数据,如果有运动的痕迹,则记录为'////',如果没有,则默认为'/v7 '
- data_json中 data 中的 value 是形如 `fgAAfgAAfgAAfgAAfgAAfgAAfgAA` 的代码段,总长5760
- 注意 5760 = 24*4*60 `fgAA` 是默认字符串,如果还是三分钟一次记录的,那么就是用长度为4的字符串分别编码记录 心率,步数,距离数据,解码是一个坑,有空再弄
- appid userid 都是和用户有关的id字段
- callid 是 13位的时间戳
- slp应该就是sleep数据 stp 应该就是step数据
- stp中:{"ttl":步数,"dis":距离,"cal":卡路里,"wk":行走卡路里消耗,"rn":0,"runDist":11,"runCal":0},"goal":8000}"

### how to cheat
- 知道数据格式之后,就想如何cheat,首先,数据是由手环传输给手机端,那么监听蓝牙数据接口,分析修改数据,或者修改手环中数据肯定最简单
- 数据传输到手机端后,手机端会对数据进行储存,找到存放地,修改数据,再把数据提交到服务器,肯定是一个方法
- 我试了下直接把修改后的数据发给服务端,app上并没有效果,可能是手机端中数据并没更改,所以没效果

### 暂时告一段落,有空再填坑
