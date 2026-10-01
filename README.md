微信定时发送 · wechat-auto-sender
一个用脚本模拟人手去操作电脑版微信、定时给指定好友发消息、提醒他「续火花」的小项目。
写在最前面
这是一个个人向的自娱自乐脚本。起因很朴素：跟朋友互续火花得每天惦记着点一下，手动提醒太累，
于是干脆写了个东西让电脑在固定时刻替我把话说出去。
- 它不是外挂，也不是协议 hook。 不破解微信、不读内存、不调私有接口——就是老老实实
驱动鼠标和键盘：点开搜索框、粘贴备注名、点进候选里的聊天窗口、把消息发进去。
所有动作都发生在你自己的电脑上，用的就是你自己登录的那个微信。
- 目标是「低频、少量、熟人之间」：默认每天 00:15 给配置好的联系人发一条，每次触发还会
随机延后 0~3 分钟，每步操作之间都有随机间隔，鼠标走曲线轨迹，长文本分段粘贴。
- 发之前会先「认人」。 进聊天窗口后截一张标题栏的图，跟之前存好的参考图做图像比对，
对不上就不发（verify_mode: strict）。宁可漏发，也不能发错人。
- 首次给某联系人发消息时会自动截一张「参考图」存进 assets/refs/，
之后每次都拿它核对——所以第一次跑完记得自己看一眼那张图，确认进的确实是对的人。
- 还有两道人工闸门：dry_run 演练模式走完整流程但不点发送；每个联系人还要单独 approved
确认。两个都开才真的发，想拦随时能拦。
⚠️ 三条 important
1. 本项目只面向 本人账号 + 低频 + 少量联系人 的个人提醒场景。
禁止用于营销、群发、引流、骚扰或任何违法违规用途。
2. 它会真实驱动你的鼠标键盘，不能保证不会触发微信风控导致封号。
作者的用法是低频自用，效果是"好像没事"，但风险自负，不建议拿去商用。
3. 微信 4.x 不暴露控件树，定位全靠「窗口边缘偏移量 + 图像比对」，所以
窗口尺寸、系统缩放比例、显示器变了就要重校准（python tools/calibrate.py）。
如果你是大神
看到觉得有意思、能把它做得更完整——更好的联系人定位、跨平台、多账号、更稳的校验、
别的可玩的点——欢迎开 issue 或提 PR，原作者会很开心 😄
一、它能做什么
能力
说明
定时发送
全局时间点（如 00:15），支持星期过滤；每个联系人也可单独指定时间
拟人化操作
每步随机间隔、鼠标走曲线轨迹、长文本分段粘贴，规避「一次性大段粘贴」的机器特征
双重安全闸门
dry_run 演练 + 每个联系人 approved 确认，两者都开才真的发
身份复核
发送前用「会话标题参考图」做图像比对（verify_mode: strict），防止发错人
随时刹车
检测到用户键鼠活动立即中止；根目录放一个 STOP 文件即停止所有任务
双界面
管理员工作台（终端控制台）+ 网页控制台 http://127.0.0.1:8788（仅本机可访问）
执行留证
每次执行前后截图存 logs/snapshots/，日志滚动保留 30 天
悬浮窗
右上角置顶倒计时，任务结束自动关闭（用了 WS_EX_NOACTIVATE，不抢微信焦点）
运行时采集
首次给某联系人发送时会自动截取「参考图」存到 assets/refs/，用于后续身份核对
二、环境要求
- Windows 10/11（依赖 Win32 API 与 pywin32，必须是 Windows）
- Python 3.11+
- 电脑版微信 4.x 或 3.x（已登录并保持在线，可以关窗口但不要退出登录）
- 微信窗口尺寸 / 系统缩放比例已知（默认按 1645×1006、缩放 150% 校准；换显示器需重校准）
三、安装
pip install -r requirements.txt
依赖（见 requirements.txt）：pywin32 / pillow / numpy / opencv-python-headless /
PyYAML / uiautomation / pyperclip / apscheduler / pystray / pywebview。
四、快速开始（首次使用四步）
第 1 步 · 登录并保持微信在线（关掉窗口可以，别退出登录）。
第 2 步 · 位置校准（换显示器 / 改了缩放比例后必做）：
python tools/calibrate.py
打开生成的 logs/calibrate.png，看红点 / 蓝框是否落在搜索框、候选列表、标题条、消息区、输入框的正确位置。
第 3 步 · 填联系人 + 跑一次演练：
编辑 config/config.example.yaml（复制为 config/config.yaml 后改），把 targets 里的
name 填成微信里的备注名（必须完全一致），然后保持 approved: false 跑：
python app.py --run-now
演练会走完整流程、截图，但不点发送：
点击搜索框 → 粘贴名字 → 点候选行 → 用标题参考图核对身份 → 并把参考图存进 assets/refs/<联系人>.png。
打开这张参考图确认确实是目标联系人本人，再进入第 4 步。
第 4 步 · 确认无误后放开：把该联系人的 approved: true，并把 safety.dry_run: false。
日常使用直接双击 start.bat（或桌面快捷方式），会同时打开网页控制台和管理员工作台，
菜单 6 / 7 可以查看任务与暂停。
想先只验证链路通不通、不动正式配置：用自检配置发给自己 ——
python app.py --config config/config.test.yaml --run-now --real
五、命令行参数（python app.py ...）
参数
作用
--run-now
立即执行一次后退出
--real
配合 --run-now 强制真实发送（忽略 dry_run）
--config <路径>
指定配置文件（默认 config/config.yaml）
--no-web
不启动网页控制台
--no-browser
启动后不自动打开浏览器
--web-port <端口>
网页控制台端口，默认 8788
--no-tray
常驻运行但不显示托盘图标
--probe
只跑控件树探测后退出
python client.py 是桌面客户端入口（独立窗口 / 无 CMD 黑窗），--startup 用于开机自启，
--no-window 只跑后端 + 托盘。
六、配置速览（config/config.yaml）
schedule:
  times: ["00:15"]        # 全局发送时刻（24 小时制）
  jitter_minutes: 3       # 每次触发随机延后 0~N 分钟，避免整点这种机器特征
  weekdays: [1,2,3,4,5,6,7]   # 1=周一 … 7=周日

targets:
  - name: "示例联系人"     # 微信备注名，必须完全一致
    enabled: true
    approved: false       # 首次必须 false，人工核对截图后再改 true
    times: []             # 留空 = 用上面的全局时间
    messages:
      - "早上好{name}，今天是{date}，{weekday}。{greeting}"
消息支持变量：{name} {date} {weekday} {time} {greeting}。
safety 段常用开关：dry_run（演练）、max_retry（失败重试上限，绝不循环重试）、
abort_on_user_input（检测到用户操作立即中止）、snapshot（执行前后截图）、
verify_mode（strict=必须校验通过才发）、kill_switch_file（STOP 文件急停）。
外观相关环境变量（不用改代码）：WECHAT_THEME / WECHAT_COLOR / WECHAT_TUI /
WECHAT_ASCII / WECHAT_CONSOLE_LOG，详见 docs/界面设计说明.md。
七、目录结构
wechat-auto-sender/
├─ app.py                 # 主入口：调度 + 系统托盘 + 工作台
├─ client.py              # 桌面客户端入口（pywebview 窗口，无黑窗）
├─ start.bat              # 双击即启动（自动探测 python）
├─ requirements.txt       # 依赖声明
├─ .gitignore             # 忽略 data/ logs/ dist/ assets/refs/ 等
├─ wechat_auto/           # 核心自动化包
│  ├─ config_loader.py    # 配置解析
│  ├─ sender.py           # 发送主流程（搜索→选人→核对→发送→复核）
│  ├─ vision.py           # 截图 + 模板匹配（身份/消息复核）
│  ├─ safety.py           # 单实例锁、急停
│  ├─ overlay.py          # 右上角倒计时悬浮窗
│  ├─ webapp.py           # 网页控制台（http.server）
│  ├─ input/              # DPI 感知、鼠标轨迹、键鼠输入
│  └─ wechat/             # 窗口定位、控件解析、联系人识别
├─ workbench/             # 管理员工作台（控制台视觉系统 + 联系人数据 + UI）
├─ web/index.html         # 网页控制台前端
├─ tools/                 # 校准 / 打包 / 诊断 / 测试脚本
├─ config/                # 配置（config.yaml 不入库，见 .gitignore）
├─ docs/                  # 文档与界面说明
└─ assets/                # 图标与运行时参考图（refs/ 含真实界面信息，不入库）
八、常用工具脚本
脚本
用途
python tools/calibrate.py
生成位置校准图，核对坐标是否还准
python tools/build_dist.py
PyInstaller 打包成 dist/微信定时发送/
node tools/make-shortcut.js
生成桌面 .lnk 快捷方式（手工构造 MS-SHLLINK，不依赖 COM）
python tools/test_*.py
各模块自检（test_client / test_web / test_workbench / test_contact_link / test_overlay / test_row_filter / test_resolve / test_search / test_single_instance / test_layout / test_overlay …）
python tools/diag_window.py
窗口 / 坐标诊断
python tools/probe*.py
若干维度的探测小工具
九、踩坑提示（都是实测结论）
1. 别让别的全屏窗口盖住微信。 程序截图用的是整屏抓取，一旦被最大化窗口（浏览器、
另一个 IDE）盖住，会抓到遮挡窗口的画面 → 图像比对必然失配 → 判定「搜索未生效」而中止。
2. 微信 4.x 不暴露控件树，所有定位靠「窗口边缘偏移量 + 图像比对」。
Ctrl+F 在 4.x 实测无效，所以搜索框用坐标点击（search_click: true）。
3. 搜索面板第一行是「搜索网络结果」，按回车会打开微信内置浏览器而不是聊天窗口，
所以程序不按回车，改为点击带彩色头像的候选行，并逐个用标题参考图确认。
4. 候选面板会保留上次的滚动位置，搜索后程序先把它滚到顶部再找行。
5. 参考图（模板）失效时重跑 tools/calibrate.py；头像过筛（像素阈值）可在
wechat_auto/wechat/locator.py 里调 s 阈值。
十、已经有的文档
- docs/如何真正发出消息.md —— 从「跑得起来」到「发得出去」的完整排障路径
- docs/双击无反应排查手册.md —— 双击没反应 / 闪退的系统性排查手册
- docs/界面设计说明.md —— 控制台视觉系统与环境变量
- docs/网页控制台说明.md —— 网页控制台功能说明
- docs/audit-2026-09-16.md —— 一次系统性深度复查报告（问题清单与修复结论）
十一、许可
仅供个人学习与使用；二次分发或商用前请自行评估合规性。
