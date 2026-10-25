# ForgeCPP · Windows 本地 C++ 评测台

浏览器界面、本机编译运行，无需 WSL、管理员权限、订阅或评测云服务器。Windows 10/11 x64。
支持 C++11 / C++14 / C++17 / C++23（实际特性取决于便携编译器版本）、题库、测试数据 ZIP 导入、加权评分、编译缓存和自定义输入。

## 从已交付的 v2 / v2.1 Windows 便携版升级

1. 导出重要题目和源码，关闭原来的 START.cmd 黑色启动窗口。
2. 下载本次聊天提供的 `ForgeCPP-update-3.0.0.zip` 并完整解压。仓库里 `.zip.b64` 是供更新器自动解码的同一更新包，不是可直接解压的 ZIP。
3. 把解压出的所有文件及 static 文件夹复制到原来包含 START.cmd 的文件夹；同名文件选择替换。
4. **不要删除 data、runtime、payload、toolchain 文件夹。更新包不包含 Python 运行时或 C++ 编译器，不能单独启动。**
5. 双击新的 START.cmd。浏览器自动打开，必要时 Ctrl+F5 强制刷新。
6. 右上角“软件更新”可检查版本。首次应显示“当前版本 3.0.0”及“已是最新版本”。

必须进行这一次手动替换，旧版仅刷新网页不会凭空获得更新功能。

## 今后的更新

固定发布仓库：https://github.com/ZhuyilongNB/forgecpp-updates

发布包以 Base64 文本存储，避免上传存储服务连接限制；更新器自动解码后再校验 ZIP 长度及 SHA-256。手工从仓库下载 `.zip.b64` 后，可在 Windows 命令提示符运行 `certutil -decode ForgeCPP-update-3.0.0.zip.b64 ForgeCPP-update-3.0.0.zip` 还原 ZIP。

固定版本清单：https://raw.githubusercontent.com/ZhuyilongNB/forgecpp-updates/main/latest.json

点击“软件更新 → 检查更新”，有更新时用户确认“安装并重启”。只在用户点击时访问更新源，不上传题目、代码或运行结果。没有网络也能离线评测；只有在线检查/下载更新需要网络。

新版更新过程：下载并校验 SHA-256 → 检查白名单文件/压缩包大小/语法 → 等待无评测任务 → 停服务 → 备份应用及 SQLite 题库 → 替换应用文件 → 原端口重启 → 启动健康检查。失败会恢复上一版应用；中断后下次 START.cmd 会尝试恢复。备份在 `data/updates/backups/`，成功升级后最多保留最近三份。

更新只替换应用白名单文件，不替换 data、runtime、payload、toolchain。数据库备份仅用于人工恢复，不在回滚时覆盖当前个人数据；格式 1 更新不得做破坏性数据库迁移。浏览器源码草稿保存在当前端口对应的浏览器存储，升级前请导出重要代码。

SHA-256 是完整性校验，**不是独立数字签名**；发布仓库的控制者可发布可执行软件。请保护 GitHub 发布账号及开启 2FA。不要安装未知来源更新或把仓库转交给不信任的人。

## 启动和安全边界

完整 Windows 便携包内含 Python 与编译器，双击 START.cmd 即可。首次会在本地解压编译器，请预留约 1.2 GB 空间。请保持启动窗口打开。

`http://127.0.0.1:8765/` 指向你自己的电脑，只有本地服务运行时才能打开；若端口被占用，使用 START.cmd 自动打开的地址。以后升级会保持该次运行的端口，网页在服务重启后自动刷新。

只运行你信任的 C++ 代码。Windows Job Object 提供资源/进程限制，**不是安全沙箱**，程序仍可能访问本机文件及网络；不要向公网开放、不要给陌生人提交代码。内存为 Job 峰值提交内存，不等于 RSS；超内存可能为 RE，不保证准确标为 MLE。

单工作进程，队列最多 8 项，每次运行总预算 60 秒。编译缓存最多 40 个程序 / 128 MiB。无额外数据库服务。

## 测试数据 ZIP

“编辑题目与数据 → 导入 ZIP / 数据文件”，支持同目录内同名前缀 `.in` 与 `.out` 或 `.ans`，子目录可用。例如 `data/01.in` 和 `data/01.out`。

ZIP 最大 12 MiB，匹配数据解压后合计最多 8 MiB，200 组 / 1000 文件，UTF-8 文本（支持 BOM）。支持 Store/Deflate；拒绝路径越界、链接、加密、重复答案和缺失配对；README/隐藏元数据忽略。导入是追加，不会自动保存；点击“保存题目”。

## 开发与验证

源码全部位于更新 ZIP 内：server.py / winrunner.py / updater.py / bootstrap.py / zip_import.py / static。仅 Python 标准库，无付费 API。

`python -m unittest test_updater test_zip_import` 检查更新/ZIP。`python selftest.py` 检查实际编译评测；`CHECK.cmd` 在 Windows 使用便携 Python 运行检查。

本次已在 Linux 兼容环境验证更新下载/安装/原端口重启及健康检查失败回滚，并验证原有评测回归。**当前未在真实 Windows 机器验证本版更新器。** 发布前的 Linux 测试不能代替 Windows 实测。如首次升级出错，请保留原软件文件夹及启动窗口截图，不要删除个人数据。

## 发布新版本（维护者）

先测试、修改 updater.py 的 VERSION（三段数值版本号），打包允许的应用文件为 `ForgeCPP-update-X.Y.Z.zip`（无顶层目录）。不要加入个人 data、编译器、运行时、密钥、测试时产生的数据库。SHA-256、size、version、notes 写入 latest.json，format 保持 1，url 必须指向这个仓库 main 下同名版本 ZIP。

当前发布方式为：把 ZIP 做 Base64 编码为 `ForgeCPP-update-X.Y.Z.zip.b64`，清单加 `encoding: "base64"`，url 指向该 `.zip.b64` 文件；size 和 sha256 仍为解码后 ZIP 的值。先提交不可变版本数据文件，再更新 latest.json；旧版本数据文件不覆盖。发布后验证公开下载、哈希和客户端检查。仅创建仓库或刷新旧版网页不会自动完成这些开发/发布步骤。现有格式不更新便携编译器或 Python；如需更新这些依赖，应单独提供完整包。
