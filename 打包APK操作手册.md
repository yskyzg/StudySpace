# StudeSpace · APK 打包操作手册（零基础版）

> 这份手册假设你完全没学过编程。照着做就行，遇到看不懂的词不用纠结。  
> 我们已经把复杂的部分全部配好了，你要做的只是「传文件」和「点按钮」。



---

## 一、先弄明白这件事是怎么运作的（30 秒）

```
你电脑上的文件  →  上传到 GitHub（一个存代码的网站，免费、公开）
        →  上传后 GitHub 会自动帮你"编译"成 APK（这叫 GitHub Actions）
        →  你去网页上把 APK 下载下来，传到手机安装
```

**关键点**：编译在云端（GitHub 的服务器上）进行，你的电脑**什么都不用装**——不用装 Android Studio，不用装编程环境，不用敲命令。

**费用**：项目公开（开源）的话，GitHub Actions 完全免费。这就是"开源公开"的一个实际好处。

---

## 二、第一次打包：从零到装上手机

### 第 1 步：准备一个 GitHub 账号

你说你已经有账号了，那登录 <https://github.com> 即可。如果需要在手机上操作，浏览器打开也能用。

### 第 2 步：建一个「仓库」（仓库 = 放你项目文件的网盘文件夹）

> ⚠️ 这一步很关键：要给 **APK 项目单独建一个新仓库**，不要传到你原来那个网站项目的仓库里。  
> 因为自动打包的配置必须放在仓库的"最外层"，混在原项目里会失效，也可能影响你原来的项目。

1. 登录后，点右上角 **「+」** 号 → **「New repository」**（新建仓库）
2. 填几个东西：
   - **Repository name**（仓库名）：填 `study-wr-apk`（或任何你喜欢英文名，不能有中文和空格）
   - **Description**（描述，可填）：`番茄学习助手 APK 版`
   - ✅ 选择 **Public**（公开）——这就是"开源"
   - ❌ **不要**勾选 "Add a README file" 等任何初始化选项（我们本地已经有文件了）
3. 点 **「Create repository」**（创建仓库）

### 第 3 步：把 `www` 文件夹里的东西传上去

打开电脑上的 `E:\09 code\Review\www` 文件夹，把**里面的内容**（不是 www 文件夹本身）全部上传到刚建的仓库：

必须上传的（`assets` 和 `.github` 若拖不进去，用下面的「新建文件」办法）：

- `index.html` ← 最重要，这就是你的 App 内容
- `package.json`
- `capacitor.config.json`
- `assets` 文件夹（里面只有 `logo.svg`，桌面图标和启动图靠它生成）
- `android.keystore.b64` ← **固定签名密钥，千万别删**（保证新 APK 能覆盖安装、不丢数据）
- `.github` 文件夹（自动打包的"说明书"）← 别忘了这个隐藏文件夹
- `.gitignore`

> ⚠️ **每次改完功能重新上传时，`index.html`、`package.json`、`capacitor.config.json` 三个都要覆盖。**  
> `capacitor.config.json` 决定 App 名字（StudySpace），`package.json` 决定打包时装哪些插件。只传 `index.html` 会出现「名字还是旧的」「导出/分享、通知、返回键不生效」这类问题。  
> `assets/logo.svg` 决定桌面图标和启动图。网页上不能直接传文件夹，就用「Create new file」新建一个叫 `assets/logo.svg` 的文件，把 `www/assets/logo.svg` 的内容复制进去。  
> `android.keystore.b64` 是一段文本，用「Create new file」新建同名文件、把内容粘进去即可。删了它，新 APK 签名就会变、只能卸载重装（数据丢失）。

建议一起上传的：

- `PRD_番茄学习助手_APK重构.md`（产品文档）
- `打包APK操作手册.md`（本手册）

> ⚠️ **找不到 `.github` 和 `.gitignore`？** 它们以点开头，是隐藏文件。  
> Windows 文件管理器：点顶部「查看」→ 勾选「隐藏的项目」就能看到。  
> 如果 `.github` 没传上去，自动打包就不会运行。

> 🆘 **`.github` 文件夹拖不进去 / 传不了？** 这是 GitHub 网页版的限制（选不了文件夹）。  
> 别用上传，改用**新建文件**的方式，方法见下面「第 3 步的备用方案」。

> 💡 传到仓库后，仓库**第一层**就应该能看到 `index.html`、`package.json` 这些文件——不要在仓库里再套一层 `www` 文件夹。

拖进去后，页面底部会出现 **「Commit changes」**（提交更改）按钮，随便写一句说明（比如 `initial`），点它。

### 第 3 步的备用方案：`.github` 文件夹传不了怎么办

GitHub 网页版选不了文件夹，所以 `.github` 文件夹经常拖不上去。用这个办法绕过：

1. 回到仓库的 **Code 页**，点 **「Add file」→「Create new file」**（创建新文件）
2. 在**文件名输入框**里，完整输入这一串（直接复制，别换行）：
   ```
   .github/workflows/build-apk.yml
   ```
   只要文件名里带 `/`，GitHub 会**自动帮你建好文件夹**，不用手动建。
3. 在下面的大文本框里，把本手册「附录：build-apk.yml 全文」里的内容**整段复制粘贴**进去
4. 拉到最下面点 **「Commit changes」**

这样 `.github/workflows/build-apk.yml` 就建好了，效果和上传文件夹完全一样。

> 同理，`.gitignore` 如果也传不了，就用「Create new file」，文件名填 `.gitignore`，内容从本地 `www/.gitignore` 复制进去。  
> 其他文件（index.html、package.json、capacitor.config.json）是普通文件，正常拖拽上传即可。

### 第 4 步：等它自动打包

1. 在仓库页面顶部，点 **「Actions」** 标签
2. 第一次进去，左边会出现一个叫 **「构建安卓 APK」** 的流程，点它
3. 点右边 **「Run workflow」** → 再点绿色的 **「Run workflow」** 按钮（手动触发一次）
4. 就会看到一次运行记录，点进去，能看到实时进度
5. **首次构建要 5–15 分钟**（要在云端下载一堆东西），以后会快很多
6. 出现绿色对勾 ✅ 就是成功了；红色叉 ❌ 就是失败了（把报错截图发我）

### 第 5 步：下载 APK

1. 在里面那个绿色成功的页面，拉到最底部，找到 **「Artifacts」**（产物）区域
2. 会看到一个叫 **「番茄学习助手-APK」** 的东西，点它下载（得到一个 zip 压缩包）
3. 解压这个 zip，里面有个 `app-debug.apk` 文件——这就是你的 App 安装包

### 第 6 步：装到手机

1. 把 `app-debug.apk` 传到手机（微信文件传输助手、QQ、数据线都行）
2. 在手机上点开它
3. 手机会提示「禁止安装未知来源应用」——去设置里允许一下（各品牌位置不同，一般搜"未知来源"或"安装未知应用"）
4. 装完就能用了 🎉

> 说明：这种叫 **Debug APK**（调试版），个人使用完全没问题。缺点是没经过官方应用商店签名，不能上架商店——但现在先跑通流程，上架是后面的事。

> 提示音规则：开始、结束、15 分钟、最后 5 分钟各自独立开关。「最后 5 分钟」只在**大于 1 小时的倒计时**且该提示音开启时，于剩 5 分钟主动提醒。正向计时、倒计时未到点主动点「结束并记录」都会响结束提示音；倒计时到点已响过一次后，是否再响一次由专注页「计时设置 → 结束后再响一次」控制（默认关闭）。

---

## 三、以后想改功能 / 加界面：怎么出新版本

流程变成「改文件 → 传文件 → 下载新 APK」，前面建仓库那套只做一次。

### 情况 A：你自己改了文件（比如改个颜色、改段文字）

1. 在 `www` 文件夹里改好 `index.html`（App 本体就在这里）
2. 打开 GitHub 仓库，点「上传」→ 把改好的文件传上去覆盖旧的
3. 提交（Commit changes）
4. **它会自动开始打包**——不用再手动点 Run workflow（因为我们配了 push 自动触发）
5. 等绿色对勾，去 Artifacts 下载新的 APK，装到手机**直接覆盖安装**（数据会保留，见下方常见问题）

> ✅ **覆盖安装不会丢数据**：打包流程已改为把 `android.keystore.b64` 显式写进生成的 Android 工程，并在 `build.gradle` 里指定这份 PKCS12 签名；每次构建签名完全一致，`versionCode` 还会自动递增，避免「同一版本冲突」。下载后直接点在旧版上安装即可，学习计划、专注记录、错题、设置全都还在。  
> ⚠️ 如果你手机里装的是**固定签名方案之前**的旧包（随机签名），第一次升级时签名对不上，系统会报「软件包冲突」并要求先卸载——这一次会丢数据。请先「导出备份 JSON」，卸载旧包，装新版后再导入；从这一版开始后续都能无损覆盖。

### 情况 B：让我帮你改（推荐，你不用懂代码）

1. 你直接告诉我："把学习目标页加一个周趋势图" 或 "把底部导航改成 5 个"
2. 我改好文件
3. 你按上面的步骤把改好的文件上传到 GitHub
4. 等打包 → 下载 → 安装

### 一个省事的小技巧

如果你用的是 **GitHub Desktop**（一个图形化软件，不用敲命令），或者 GitHub 网页的上传功能，传文件就像发邮件附件一样简单。你不用学 Git 命令。

---

## 四、常见问题

**Q：新 APK 能直接覆盖旧的吗？数据会不会丢？**  
A：能，而且**数据会保留**。打包时已显式把仓库里的 `android.keystore.b64` 配成 debug 签名（`storeType=PKCS12`、别名 `androiddebugkey`），并把 `versionCode` 设为 GitHub Actions 构建序号，所以每次新包都是同签名的升级，学习数据原封不动。只要你不删这个密钥文件、不改 `capacitor.config.json` 里的 appId（`com.studywr.pomodoro`），就能一直无损升级。

> 唯一例外：从"随机签名"的旧版本升级到这个"固定签名"版本时，签名对不上，会提示「同一个版本 / 软件包冲突」，需要先卸载一次（数据会清掉）。这是最后一次，用「导出备份 JSON」过渡即可。  
> 千万别删 `android.keystore.b64`——删了构建就会换回随机签名，又变成只能卸载重装。它是一段文本，可以用「Create new file」粘贴上传。  
> 每次构建日志会打印 APK 的 `SHA256` 签名指纹，连续两次构建指纹一致就说明签名没变。

**Q：改了文件，但 Actions 没自动打包？**  
A：检查是不是传到了 `main` 或 `master` 分支（默认分支一般就是这俩）。或者干脆去 Actions 页面手动点 Run workflow。

**Q：构建失败（红色叉）怎么办？**  
A：点进那次运行，拉到底看红色报错，截图发我。90% 是配置文件的小问题。

**Q：APK 下载下来打不开 / 安装失败？**  
A：① 确认解压后是 `.apk` 文件不是 zip；② 手机允许"未知来源安装"；③ 安卓版本太低（本方案需要安卓 8.0 以上，现在几乎所有手机都满足）。

**Q：会不会产生费用？**  
A：公开仓库的 GitHub Actions 免费。GitHub 本身也免费。

**Q：我的学习数据会传到网上吗？**  
A：不会。App 的数据只存在你手机本地。上传的只是"代码文件"，不包含你的学习记录。API Key 也只存在你手机本地（但注意：仓库是公开的，如果你把真实 API Key 写进代码文件传上去，别人会看到——所以 Key 只在 App 里填，别写进代码）。

**Q：这个 App 的数据以后换手机怎么办？**  
A：设置 → 数据 →「导出备份（JSON）」，换机后用「导入恢复（JSON）」读回来。

**Q：导出的备份、报告文件存到哪了？能自己选位置吗？**  
A：点导出后**不会跳到新页面**，App 内会弹出「导出完成」结果页，写明文件名、保存位置和完整路径，还能一键复制路径或复制文件内容。  
手机上默认写到「文件管理 → Documents」文件夹，文件名类似 `StudySpace备份_2026-11-22.json`。  
想换地方就点结果页的「**选择保存位置 / 分享**」，系统会弹出面板，可以挑文件夹、存网盘，或发给别人。  
学习数据导出的是 `.html` 报告文件，用浏览器打开后点「打印 / 另存为 PDF」即可得到 PDF；错题集导出同样是 `.html`。

> 注：文件导出/分享依赖 `@capacitor/filesystem` 和 `@capacitor/share` 两个插件，所以 `package.json` 必须一并上传，漏传就打不出带这两个能力的包。

**Q：专注时有后台通知吗？息屏了也会响吗？**  
A：有。点「开始专注」时，正向计时和倒计时都会挂一条「已开始专注 · 科目 · 剩余时间」常驻通知（划不掉）。倒计时会另外预排一条独立的「专注完成！· 科目 · 本次时长」系统通知，到点时直接弹出，**不需要 App 在前台**；息屏、切后台也会按时响。回到前台时，App 会撤掉常驻条并确认完成通知。暂停、主动点「结束并记录」、重置、切模式都会同时撤掉常驻通知和到点预排。

**Q：通知没弹出来 / 通知栏空的，怎么办？**  
A：九成是系统通知权限没给。去「设置 → 通用 → 通知诊断」：

1. 点「**重新检查权限**」——会显示权限是否已授予；
2. 如果是拒绝，点「**打开系统设置**」（或手动到 手机设置 → 应用 → StudySpace → 通知），允许通知后回来重试；
3. 点「**发一条测试通知**」看通知栏能不能出现；
4. 还是不行，把面板里那段「诊断日志」截图发我，里面写明了桥接、插件、权限的真实状态。

> 首次开始专注时会弹一次通知权限请求，**请点「允许」**（Android 13 及以上必需）。

**Q：按手机返回键会直接退出吗？**  
A：不会。返回键按「关下拉 → 关弹窗 → 退回上一层 → 回到首页」的顺序处理；已经在首页时，需要**连续按两次**才会退出，第一次只提示「再按一次退出 StudySpace」。

**Q：AI 助手怎么用？要花钱吗？**  
A：不配置接口也能用，走本地模拟回应，完全免费。想要真实回答，去「设置 → AI 接口」填供应商、API Key、Base URL 和模型名（OpenAI 兼容格式，如 DeepSeek）。App 内用原生网络通道请求，不受浏览器跨域限制；调用失败会自动降级为模拟回应并提示原因。可保存多套预设，一键切换。

**Q：提示词预设和会话里的提示词是什么关系？**  
A：「设置 → AI 接口 → 提示词预设」是全局预设库，可增删改，点名称设为**新会话默认**。每个会话在 AI 助手顶部单独选「提示词」，只影响当前会话，互不干扰。会话列表里会显示各自选的提示词。查看上下文时能看到本会话实际使用的提示词；消息超过 4 条可一键压缩。

**Q：从错题本点「问 AI 这道题」会怎么处理？**  
A：会先弹「选择对话」：可以发到已有会话接着问（显示每个会话当前用的提示词），也可以「＋ 新建对话」。新建时先选这条对话要用的**提示词预设**，再把题目、错因、知识点和图片作为一条消息发出；建好后仍可在 AI 助手顶部单独切换提示词，不影响其他会话。

**Q：AI 助手能接着上面的问题继续问吗？压缩后会不会断？**  
A：能。同一会话里每次提问会把该会话的提示词 + 最近 20 条消息一起发给接口，所以可以直接追问「那第二步呢？」。不同会话的上下文互相隔离。超过 20 条的更早内容会掉出窗口，点「压缩」可以把它们收成一份摘要。压缩时会生成一份**可编辑的接力摘要**：保留会话主题、已问过的问题、已讲到的结论，尤其是最后一条用户问题原文（标为「待继续」）。你可以直接在弹窗里改，确认后下一问会带着这份摘要继续，所以没问完的题不会丢。注意：压缩前如果 AI 还在回复，等回复完成再压缩。

**Q：首页「智能提醒」是根据什么生成的？**  
A：全部来自本机真实数据，不用联网，也没有演示数据。会按优先级生成：今天该复习的错题（1/3/7 天）、未完成的计划、距每日目标的缺口、连续打卡里程碑；没有科目或没有专注记录时给引导。展开后每条都能直接跳到对应页面处理。

**Q：只想清掉某一类数据，怎么办？**  
A：去「设置 → 通用 → 清理数据（可选择清理项）」，勾选要清的项目后确认。可单独清理计时、学习目标、计划、科目、错题、复盘、周报、AI 聊天记录，也可以选「清空所有数据」一次性恢复初始状态。清 AI 聊天记录只删对话，不影响提示词预设和接口配置。

> 清理**计时数据**后，首页学习趋势、学习目标打卡月历、学习统计、每周周报会同步变空；清理**错题数据**后错题趋势和待复习提醒同步更新。清理**科目**只删科目定义与分科目标，已有记录会显示为「未分类」。所有清理都不可撤销，重要数据请先「导出备份（JSON）」。

**Q：「连续打卡」按什么判定？为什么日历里有些天不算？**  
A：当天**有专注记录（时长 > 0）**就算打卡，计划是否完成、错题是否复习都不参与判定；达到全局日目标会额外标红为「达标」，但未达标也算打卡。今天还没计时不算断，从昨天继续往前数。判定只依赖计时数据，所以清掉「计时数据」后连续天数与打卡月历会一起归零。

**Q：怎么看哪几天打了卡？**  
A：「学习目标 → 连续打卡」卡片是和「计划月历」同款的**打卡月历**，可左右翻月，蓝色点=已打卡、红点=达标、浅色=未打卡，今天深色高亮；点任意日期能看到当天专注时长、番茄数、科目分布。首页「连续打卡」小卡也可以点开同一个日历。

---

## 五、文件夹结构：你的原项目 和 APK 项目是完全分开的

```
E:\09 code\Review\              ← 你的原项目（已上传 GitHub，我们一个字都没动）
├── index.html                    你原来的网站
├── README.md / CLAUDE.md         你原来的文档
├── .gitignore                    你原来的（保持原样）
└── www\                        ← APK 项目（独立的新仓库，我新建的都在这）
    ├── index.html                App 本体（改功能就是改这里）
    ├── package.json              打包工具清单
    ├── capacitor.config.json     App 名字、配置
    ├── assets\logo.svg           应用图标与启动图源图（文本，可网页上传）
    ├── .gitignore               新建的（不是改你原来那个）
    ├── .github\workflows\
    │   └── build-apk.yml         自动打包说明书
    ├── PRD_番茄学习助手_APK重构.md
    └── 打包APK操作手册.md         本手册
```

**要点**：

- 根目录那些文件全是你原来的，我**没有改动任何一个**（包括 `.gitignore`）。
- `www` 是独立的 APK 项目，以后单独上传到新仓库。
- `www/index.html` 是**完全重写的 App 本体**（单文件，含全部页面和功能），和根目录那个旧网站已经不是同一份内容；根目录那份原样留着当历史备份。
- 导出/分享由 `@capacitor/filesystem`、`@capacitor/share` 提供，硬件返回键由 `@capacitor/app` 提供，专注结束提醒由 `@capacitor/local-notifications` 提供；这些都在 `package.json` 里，`cap sync` 时自动装进安卓工程，不用手动改安卓代码。
- 桌面图标和启动图不放在安卓工程里，而是由 `assets/logo.svg` 在 GitHub Actions 构建时用 `@capacitor/assets` 自动生成，所以换图标只改这一个 SVG 即可。

---

## 附录：`build-apk.yml` 全文（备用方案复制用）

把下面从 `name:` 开始到最后的**全部内容**复制，粘贴到新建文件的文本框里：

```yaml
name: 构建安卓 APK

on:
  push:
    branches: [main, master]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: 拉取代码
        uses: actions/checkout@v4

      - name: 安装 Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: 安装 Java 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: 缓存 Gradle（加快后续构建）
        uses: actions/cache@v4
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: gradle-${{ runner.os }}-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
          restore-keys: gradle-${{ runner.os }}-

      - name: 安装依赖
        run: npm install

      # 网页文件（index.html）在仓库根目录，这里自动建一个 www 目录并放进去，
      # 这样 Capacitor 就能按配置里的 webDir: "www" 找到它。
      - name: 准备网页资源目录
        run: |
          mkdir -p www
          cp index.html www/index.html

      - name: 重新生成 Android 工程
        run: rm -rf android && npx cap add android

      # 用 assets/logo.svg 生成各密度桌面图标与启动图，
      # 覆盖 cap add android 自带的 Capacitor 默认机器人图标。
      - name: 生成 StudySpace 图标与启动图
        run: |
          if [ -f assets/logo.svg ]; then
            npx @capacitor/assets generate --android --assetPath assets \
              --iconBackgroundColor '#F6F1E5' \
              --iconBackgroundColorDark '#F6F1E5' \
              --splashBackgroundColor '#F6F1E5' \
              --splashBackgroundColorDark '#F6F1E5'
          else
            echo "::warning::未找到 assets/logo.svg，本次使用默认图标"
          fi

      - name: 同步网页资源
        run: npx cap sync android

      - name: 赋予 gradlew 执行权限
        run: chmod +x android/gradlew

      # 固定调试签名：每次构建用同一份密钥，同名覆盖安装时数据才不会丢。
      - name: 固定调试签名（保证覆盖安装保留数据）
        run: |
          mkdir -p ~/.android
          base64 -d android.keystore.b64 > /tmp/studyspace.p12
          keytool -importkeystore \
            -srckeystore /tmp/studyspace.p12 -srcstoretype PKCS12 -srcstorepass android \
            -destkeystore ~/.android/debug.keystore -deststoretype JKS \
            -deststorepass android -destkeypass android -noprompt \
            && echo "已用固定密钥签名" || { cp /tmp/studyspace.p12 ~/.android/debug.keystore && echo "已用 PKCS12 固定密钥签名"; }

      - name: 构建 Debug APK
        run: cd android && ./gradlew assembleDebug --no-daemon

      - name: 上传 APK
        uses: actions/upload-artifact@v4
        with:
          name: StudySpace-APK
          path: android/app/build/outputs/apk/debug/app-debug.apk
          retention-days: 90
```


---

有任何一步卡住了，直接把报错或截图发我，我帮你看。
