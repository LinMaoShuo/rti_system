工程目录结构
ToB_Auto_Test
├── Library                   # 自定义 AW 库
│   ├── client/               # API 客户端（约 27 个），继承自 ApiClient
│   │   ├── client.py         # 基类（HTTP/SSH/Telnet/Redfish 等）
│   │   ├── client_factory.py # 客户端工厂方法
│   │   ├── CloudNet/         # CloudNet 专属客户端（tcsm_oc_web, tcsm_sc_openapi_*.py 等）
│   │   ├── access_openapi_client.py
│   │   ├── cpe_client.py
│   │   ├── mml_openapi_client.py
│   │   ├── meao_openapi_client.py
│   │   ├── mepm_client.py
│   │   ├── ssh_client.py
│   │   └── ...               # 其他：redfish_client / pgw_mml / ueg_api / trace_api 等
│   └── common/               # 公共工具
│       ├── Common.py         # YAML 读写、IP 工具等
│       ├── Config.py         # 全局配置加载（ENV1/ENV2 等）
│       ├── Dict.py           # 字典转对象
│       ├── Time.py           # 时间工具
│       ├── alarm/            # 告警信息（vm_alarm.yaml 等）
│       └── alarm_info/       # 告警信息数据
├── Logic                     # 自定义 Logic 库（~120 个 logic_* 函数）
│   ├── __init__.py           # 统一导出所有 logic_ 函数
│   ├── atm_fun.py            # ATP 测试逻辑
│   ├── common_fun.py         # 公共逻辑
│   ├── fm_fun.py             # 故障管理逻辑
│   ├── meao_fun.py           # MEAO 逻辑
│   ├── mepm.py               # MEPM 逻辑
│   ├── mml_fun.py            # MML 指令逻辑（SMF/UPF/VPG 等配置）
│   ├── nte_fun.py            # NTE 信令分析工具逻辑
│   ├── pgw_fun.py            # PGW 逻辑
│   ├── rel.py                # 可靠性测试逻辑（故障注入、告警等待等）
│   ├── ssh_fun.py            # SSH/Telnet 远程操作逻辑
│   ├── trace.py              # 抓包追踪逻辑
│   ├── ue_fun.py             # UE 模拟逻辑
│   ├── ueg_ngvn_fun.py       # UEG/VPG 逻辑
│   ├── tcsm_meo.py           # CloudNet TCSM/MEO 逻辑
│   ├── cpe_fun.py            # CPE 逻辑
│   └── CloudNet/             # CloudNet 专属逻辑（ecs/tcsm/elb/meo 等）
├── Testcase                  # 测试用例脚本（一个文件对应一个用例，以用例 ID 命名）
│   ├── 5GtoB/                # 5G ToB 业务测试
│   │   ├── ATP测试/          # ATP 基础测试
│   │   ├── DFX可靠性/        # 可靠性测试（分层云/计算硬件/网络硬件故障注入等）
│   │   ├── MEC能力开放/      # MEC 业务使能
│   │   ├── 5GLAN组网场景化交换基线/  # VxLAN 功能/可靠性
│   │   ├── ULCL分流/         # ULCL PSCP 分流
│   │   ├── 02_APP集成/       # AppD 集成测试
│   │   ├── 组网专项/         # 4U 分层云组网（服务器重启/DCGW/TOR/FW 重启等）
│   │   ├── 移动MEC运维专项/
│   │   ├── 移动VPN/
│   │   ├── 运维专项/
│   │   └── 风扇L本地运维管理能力增强/
│   └── 边缘云/               # 边缘云产品测试
│       ├── 01_云服务/
│       ├── 02_存储服务/
│       ├── 03_网络服务/
│       ├── 04_网关/
│       └── 07_运营/          # 集采北向 API 用例
├── Resource                  # 环境配置
│   ├── resource_const.yaml   # 共享配置
│   ├── resource_dc{1~40}.yaml      # 按 DC 编号区分的资源配置
│   ├── resource_5GC.yaml     # 
│   ├── resource_manage_plane.yaml  # 管理面资源
│   ├── resource_mepm.yaml    # MEPM 资源
│   ├── resource_ue.yaml      # UE 资源
│   ├── CloudNet/             # CloudNet TCSM 环境资源
│   └── ToB_Auto_Test_env.ini
├── Tool                      # 辅助工具
│   ├── ping_tool/            # 多线程 ping 统计工具
│   ├── PingTool/             # 远程 ping 工具
│   ├── CloudNet/             # CloudNet 资源清理与预置脚本
│   ├── t2.py                 # 根级辅助脚本
│   └── verify_stats.py       # 统计数据验证脚本
├── testsuite.py              # CloudSpider 测试套入口
├── definitions.py            # 项目根目录常量（CONFIG_PATH）
├── init_logic.py             # Logic 层初始化（init_logic('logic')）
├── t2.py                     # 根级辅助脚本
├── requirememt.txt           # Python 依赖（文件名含拼写错误）
└── README.md

依赖



简单描述一下clientfactory.py的设计架构或思路及其优点
ClientFactory 设计架构
核心思路：名称→实例化函数的映射表
用例代码: ClientFactory.getclientenv1(['UPFMML', 'FM', 'NTEV9'])
→ 按 env1 配置，逐一查找 clientmap，调用各自的私有函数
→ 返回 (MmlOpenApi, NewFMClient, TelnetClient) 实例
三层结构
1. 入口层（getclientenv1~6, getclientenv_const 等）
● 绑定不同环境配置对象（ENV1/ENV2/ENVCONST/MGMTPLANE/TCS 等）
● 调用核心方法 get_client(names, env)
2. 调度层（getclient 中的 clientmap 字典）
● 约 30+ 个 key 映射到具体的工厂函数
● 支持单字符串或列表输入，单元素时自动解包
3. 实例化层（getclient* 私有函数）
● 每个函数从 env.xxx 提取对应参数，创建具体 client 实例
● 处理各类初始化和登录（如 SSHClient.login()、sshsuroot()）
优点
● 配置与代码解耦：用例只需写 ['UPFMML', 'FM']，连接哪个 IP、用什么协议、需要哪些登录凭据，全部由 Resource/resourcedc1.yaml 配置决定，换环境不改代码
● 统一入口：30+ 种 client 类型一个方法调用，用例不用关心 import SSHClient/RedfishClient/MmlOpenApi...
● 类型语义清晰：getclientenv1 vs getclientenvmgmtplane，直接表达当前取的是哪个环境，避免把 DC 管理面的 IP 错配到业务面
● 灵活扩展：新增一种网元只需在 clientmap 加一行，再加一个 getclient_* 方法，用例侧零改动
需要注意的点
● client_map 本身就是一个"隐式的注册表"，新增 client 类型容易忘记加映射导致 ModuleNotFoundError
● 部分函数有 None 返回逻辑（如 MPF/SMF IP 可能为 None），调用方需要判空

