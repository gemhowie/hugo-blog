+++
title = 'VScode Configuration'
draft = false

+++

## 插件

```
Chinese (Simplified) (简体中文) Language Pack for Visual Studio Code
vim
matchit

GitLens
Git Graph

Brackets Light Pro 或者 Dracula Official
Material Icon Theme

C/C++

python
Pylance
Ruff
Black Formatter
Mypy （不是官方的Mypy Type Checker）
原因Mypy插件介绍中有解释:
Runs on your entire workspace. (This is different from Microsoft's Python extension's mypy functionality which only lints each file separately, leading to incomplete type checking.)
在整个工作区运行。(这与微软 Python 扩展的 mypy 功能不同，后者只对每个文件进行单独衬垫，导致类型检查不完整）
```



## 快捷键

ctrl + shift + p 关键字 open keyboard shortcuts (JSON)

复制keybindings.json内容

```json
// 将键绑定放在此文件中以覆盖默认值
// Place your key bindings in this file to override the defaults
[
  {
    "key": "F12",
    "command": "workbench.action.togglePanel", // 切换面板可见性
  },
  {
    "key": "Ctrl+]",
    // "command": "editor.action.goToTypeDefinition"
    "command": "editor.action.revealDefinition", // 跳转到定义
  },
  {
    "key": "Ctrl+T",
    "command": "workbench.action.navigateBack", // 返回上一步
  },
  {
    "key": "Ctrl+F12",
    "command": "workbench.action.toggleMaximizedPanel",
  },
  {
    "key": "Ctrl+Shift+\\",
    "command": "extension.matchitSelectItems",
    "when": "editorTextFocus"
  },
  {
    "key": "Ctrl+Shift+/",
    "command": "extension.matchitDeleteItems",
    "when": "editorTextFocus"
  },
  {
    "key": "Shift+H",
    "command": "workbench.action.previousEditorInGroup",
    // 已知问题，在normal模式下进行替换，或者移动操作时，r/f/t/F/T，如果替换的字符或搜索的字符刚好是H，则会触发命令，而不是输入H，这种场景比较少，暂时可以忽略
    // 已有人提issue，未解决，https://github.com/VSCodeVim/Vim/issues/8476
    "when": "editorTextFocus && vim.active && vim.mode == 'Normal'",
  },
  {
    "key": "Shift+L",
    "command": "workbench.action.nextEditorInGroup",
    // "when": "editorTextFocus && vim.mode != 'Insert' && vim.mode != 'Replace' && !(vim.mode =~ /Visual/)",
    "when": "editorTextFocus && vim.active && vim.mode == 'Normal'",
  },
]
```



## 设置

ctrl + shift + p 关键字 Open User Settings (JSON)

复制settings.json内容

```json
{
  // 主题
  "workbench.colorTheme": "Brackets Light Pro", // 主题
  "workbench.iconTheme": "material-icon-theme", // 图标主题
  // 布局
  "window.titleBarStyle": "custom", // 标题栏外观
  "window.zoomLevel": 0, // 窗口缩放级别
  "workbench.statusBar.visible": true, // 显示状态栏
  "editor.minimap.enabled": false, // 关闭代码小地图
  "zenMode.centerLayout": false, // zen模式关闭居中布局，采用全屏显示
  "zenMode.hideStatusBar": false, // zen模式显示状态栏
  // 搜索
  "search.exclude": { // 搜索排除的文件和目录
    "**/.git": true,
    "**/*.bundle.js": true,
    "**/bin-packages": true,
    "**/frontend-dist": true,
    "**/npm-packages-offline-cache": true
  },
  "search.useGlobalIgnoreFiles": true, // 搜索时使用全局的gitignore文件
  // 编辑器
  "files.autoSave": "off", // 关闭自动保存
  "editor.quickSuggestions": { // 各种输入场景下的自动显示建议
    "other": "on",
    "comments": "off",
    "strings": "off"
  },
  "editor.cursorSurroundingLines": 7, // 光标周围可见的前置行
  "editor.rulers": [ // 等宽字符垂直标尺
    120
  ],
  // 扩展
  "extensions.ignoreRecommendations": true, // 不显示扩展建议的通知
  // 终端
  "terminal.integrated.defaultProfile.windows": "Command Prompt", // 部分版本windows上，PowerShell无法自动进入python虚拟环境，改成cmd才可以
  // 远程ssh
  "remote.SSH.showLoginTerminal": true,
  "remote.SSH.remotePlatform": {
    "Ubuntu": "linux",
  },
  // vim配置
  "vim.handleKeys": { // ctrl-a 保留全选的功能
    "<C-a>": false,
  },
  "vim.enableNeovim": false, // 使用neovim来执行命令，比较明显的区别是插件的vim使用JavaScript的语法来写正则表达式，neovim用的是原生的语法
  "remote.extensionKind": {
    "vscodevim.vim": [ // 使用远程计算机上的vim扩展
      "workspace"
    ]
  },
  "vim.easymotion": true, // 开启easymotion功能，默认按两次leader键为执行前缀easymotion-prefix，然后可以跟各种vim的移动操作，如h, j, k, l, w, s, 再输入mark进行跳转
  // "vim.sneak": true,  // 类似f/F，支持多行范围的双字符搜索（只想搜一个字符时，第二个字符按回车即可），绑定s/S，会覆盖s的原始功能（清除选中内容并进入插入模式）
  // 不想s功能被覆盖，可以使能vim.sneakReplacesF，直接替换f的功能，但为了和默认的f行为一致，替换f之后，也只支持单字符搜索，唯一区别就是增加了多行搜索的能力
  "vim.sneakReplacesF": true, // 使用单字符版本的sneak替换f/F
  "vim.visualstar": true, // 开启可视模式下按*或#搜索选择的内容
  "vim.ignorecase": false, // 关闭忽略大小写搜索
  "vim.useSystemClipboard": true, // 使用系统剪切板
  "vim.useCtrlKeys": true, // 启用vim ctrl快捷键功能，比如翻页ctrl+f, ctrl+b, ctrl+d, ctrl+u
  "vim.hlsearch": true, // 开启高亮搜索
  "vim.leader": " ", // leader键设置
  "vim.insertModeKeyBindings": [
    {
      "before": [
        "k",
        "j"
      ],
      "after": [
        "<Esc>"
      ]
    },
    {
      "before": [
        "j",
        "k"
      ],
      "after": [
        "<Esc>"
      ]
    }
  ],
  "vim.visualModeKeyBindingsNonRecursive": [
    {
      "before": [
        "p"
      ],
      "after": [ // 粘贴选中的内容后，寄存器会被选中的内容替换，这里再复制一次，恢复之前寄存器的内容
        "p",
        "g",
        "v",
        "y"
      ]
    },
    {
      "before": [
        "<leader>",
        "s", // search
        "g" // grep
      ],
      "commands": [
        "workbench.action.findInFiles"
      ]
    },
    {
      "before": [
        "<leader>",
        "s", // search
        "r" // replace
      ],
      "commands": [
        "actions.find"
      ]
    },
    {
      "before": [
        "<leader>",
        "s", // search
        "w" // word
      ],
      "commands": [
        "actions.find"
      ]
    },
    {
      "before": [
        "v"
      ],
      "commands": [
        "editor.action.smartSelect.expand"
      ]
    },
    {
      "before": [
        "V" // 使用V，会占用从visual模式切换到visual line模式的功能，可以先按<ESC>再按V进入visual line模式
      ],
      "commands": [
        "editor.action.smartSelect.shrink"
      ]
    },
    // // 注释原因：visualMode下 matchit 的跳转无法让vim选中内容，算是一个bug，已提issue，未解决
    // // 但是vim原生的%跳转是可以正常跳转和选中内容的，只是原生的%支持的功能较少，matchit支持HTML/CSS/C/C++/Java/Perl/Javascript中html标签和括号之间的跳转
    // {
    //   "before": ["%"],
    //   "commands": ["extension.matchitJumpItems"]
    // },
  ],
  "vim.normalModeKeyBindingsNonRecursive": [
    {
      "before": [
        "<leader>",
        "f", // find
        "f" // file (project root dir)
      ],
      "commands": [
        "workbench.action.quickOpen"
      ]
    },
    {
      "before": [
        "<leader>",
        "f", // find
        "F" // file (cwd)，弹出对话框的方式选择文件打开，全键盘操作不方便
      ],
      "commands": [
        "workbench.action.files.openFile"
      ]
    },
    {
      "before": [
        "<leader>",
        "f", // find
        "r" // recent，只能记录openFile对话框方式打开的文件，用处不大
      ],
      "commands": [
        "workbench.action.openRecent"
      ]
    },
    {
      "before": [
        "<leader>",
        "f", // file
        "p" // absolute path
      ],
      "commands": [
        "workbench.action.files.copyPathOfActiveFile"
      ]
    },
    {
      "before": [
        "<leader>",
        "f", // file
        "P" // relative path
      ],
      "commands": [
        "copyRelativeFilePath"
      ]
    },
    {
      "before": [
        "<leader>",
        "s", // search
        "g" // grep
      ],
      "commands": [
        "workbench.action.findInFiles"
      ]
    },
    {
      "before": [
        "<leader>",
        "s", // search
        "r" // replace
      ],
      "commands": [
        "actions.find"
      ]
    },
    {
      "before": [
        "<leader>",
        "s", // search
        "w" // word
      ],
      "commands": [
        "actions.find"
      ]
    },
    {
      "before": [
        "<leader>",
        "s", // search
        "s" // symbol
      ],
      "commands": [
        "workbench.action.gotoSymbol"
      ]
    },
    {
      "before": [
        "<leader>",
        "b", // buffer
        "d" // delete
      ],
      "commands": [
        "workbench.action.closeActiveEditor"
      ]
    },
    {
      "before": [
        "<leader>",
        "b", // buffer
        "o" // delete other editors
      ],
      "commands": [
        "workbench.action.closeOtherEditors" // 关闭当前Group中的其他Editors
      ]
    },
    {
      "before": [
        "<leader>",
        "-"
      ],
      "commands": [
        "workbench.action.splitEditorDown" // 向下分屏，分屏会创建一个新的Group
      ]
    },
    {
      "before": [
        "<leader>",
        "|"
      ],
      "commands": [
        "workbench.action.splitEditorRight" // 向右分屏，分屏会创建一个新的Group
      ]
    },
    {
      "before": [
        "<leader>",
        "b", // buffer
        "q" // quit
      ],
      "commands": [
        "workbench.action.closeGroup"  // 关闭当前Group，只剩一个Group时无效
      ]
    },
    {
      "before": [
        "<leader>",
        "b", // buffer
        "O" // delete other groups
      ],
      "commands": [
        "workbench.action.editorLayoutSingle", // 关闭其他Group，只保留当前Group
        "workbench.action.focusFirstEditorGroup" // 但有时候关闭其他Group后，焦点不在仅剩的Group上，可以聚焦一次来规避
      ]
    },
    {
      "before": [
        "<leader>",
        "b",
        "4"
      ],
      "commands": [
        "workbench.action.editorLayoutTwoByTwoGrid" // 创建2x2的Group布局
      ]
    },
    {
      "before": [
        "<leader>",
        "c", // code
        "r" // rename
      ],
      "commands": [
        "editor.action.rename"
      ]
    },
    {
      "before": [
        "%"
      ],
      "commands": [
        "extension.matchitJumpItems"
      ]
    },
    {
      "before": [
        "<leader>",
        "a",
        "z" // zenMode
      ],
      "commands": [
        "workbench.action.toggleZenMode"
      ]
    },
    {
      "before": [
        "<C-h>",
      ],
      "after": [
        "<C-w>",
        "h"
      ]
    },
    {
      "before": [
        "<C-j>",
      ],
      "after": [
        "<C-w>",
        "j"
      ]
    },
    {
      "before": [
        "<C-k>",
      ],
      "after": [
        "<C-w>",
        "k"
      ]
    },
    {
      "before": [
        "<C-l>",
      ],
      "after": [
        "<C-w>",
        "l"
      ]
    },
    {
      "before": [
        "<leader>",
        "w",
        "w"
      ],
      "after": [
        "<C-w>",
        "w" // other window，切换到其他Group
      ]
    },
    {
      "before": [
        "<leader>",
        "w",
        "o"
      ],
      // "after": [ // <C-w>o 同"workbench.action.editorLayoutSingle"
      //   "<C-w>",
      //   "o"
      // ],
      "commands": [
        "workbench.action.editorLayoutSingle", // 关闭其他Group，只保留当前Group
        "workbench.action.focusFirstEditorGroup" // 但有时候关闭其他Group后，焦点不在仅剩的Group上，可以聚焦一次来规避
      ]
    },
    {
      "before": [
        "<leader>",
        "CR"
      ],
      "commands": [
        ":noh"  // 关闭高亮搜索
      ],
    },
  ],
  // 插件配置
  "mypy.runUsingActiveInterpreter": true,
  "python.languageServer": "Pylance",
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnSave": true,
  },
  "ruff.nativeServer": true,
  "black-formatter.args": [
    "--skip-string-normalization", // 跳过单引号转双引号
    "--preview", // 启用预览版的特性，（如字符串换行，但最新版似乎已经移除该特性）
  ],
}
```



## Python相关

### 修改pip源

升级 pip 到最新的版本 (>=10.0.0) 后进行配置：

```
python -m pip install --upgrade pip

pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip config set install.trusted-host pypi.tuna.tsinghua.edu.cn
pip config set global.timeout 600
```

如果您到 pip 默认源的网络连接较差，临时使用本镜像站来升级 pip：

```
python -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip
```



### 安装虚拟环境管理器

#### Windows

```
pip install virtualenvwrapper-win
```

设置系统环境变量WORKON_HOME（虚拟环境的存放位置）

#### Linux

```
pip install virtualenvwrapper
```

#### 常用命令

```
# 列出虚拟环境列表
workon
或
lsvirtualenv

# 新建虚拟环境
mkvirtualenv [envname]
# 指定版本
mkvirtualenv -p python3 [envname]

# 进入虚拟环境
workon [envname]

# 删除虚拟环境，或者直接删除目录
rmvirtualenv [envname]

# 退出虚拟环境
deactivate
```



### python lsp设置

```
  "python.languageServer": "Pylance",
```



### mypy设置

mypy扩展不自带mypy，需要自己安装 `pip install mypy`



使用守护进程(dmypy)的方式可以在内存中进行检查，当文件改变时，只会重新检查该文件，提升检查速度

mypy扩展默认依赖PATH路径包含dmypy这个可执行文件

如果想用不同版本的dmypy，也可以在设置中指定Mypy: Dmypy Executable为dmypy的绝对路径

对应设置为

```
  "mypy.dmypyExecutable": "E:\\virtualenvs\\nvim_venv\\Scripts\\dmypy.exe",
```

如果不想用全局的dmypy，而想要用虚拟环境中安装的dmypy版本，可以勾选Mypy: Run Using Active Interpreter，该选项会覆盖mypy.dmypyExecutable

对应设置为

```
  "mypy.runUsingActiveInterpreter": true,
```



### black设置

black扩展自带了一个black版本，可以不用额外安装

```
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnSave": true,
  },
  "black-formatter.args": [
    "--skip-string-normalization",  // 跳过单引号转双引号
    "--preview",  // 启用预览版的特性，（如字符串换行，但似乎最新版已经被移除）
  ],
```



### ruff设置

ruff扩展自带了一个ruff版本，可以不用额外安装

自 Ruff v0.4.5 起，Ruff 内置了一个用 Rust 编写的语言服务器： ⚡ ruff server ⚡ 。

要选择使用 Ruff 服务器支持（现已推出 Beta 版），请使用 “ruff.nativeServer”: true 启用 “本地服务器 ”扩展设置。

```
  "ruff.nativeServer": true,
```



