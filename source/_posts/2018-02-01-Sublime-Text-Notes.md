---
title: Sublime Text
date: 2018-02-01 14:54:53
categories: DevTool
tags: [Sublime, 编辑器]
---

> 免责声明：本文绝无引战之意，也不进行对比，只总结 Sublime 开发配置。

<!--more-->

# 插件

## 编辑器自身

- Package Control
- PackageResourceViewer
- auto-save
- All Autocomplete
- AdvancedNewFile
- File Rename

## 主题 + 侧栏

- Material Themes
- colorsublime
- SideBarEnhancements
- SideBarSeparator
- SyncedSideBar
- SyncedSidebarBg
- A File Icon
- Hide Sidebar When Not Focussed：使用 Cmd+B, Cmd+K 切换。

## 补全+智能提示

- SublimeCodeIntel
- sublimelint
- SublimeLinter
- BracketHighlighter
- ColorHighlighter
- Statusbar Path
- JSON Lint
- Pretty JSON
- JSON Comma

## 小工具+MD

- Inc-Dec-Value
- ColorPicker
- Convert case
- MarkdownHighlighting
- Markdown Preview
  - `Build with: markdown` 生成 html
  
## Git 集成

- Git
- [GitGutter](https://github.com/jisaacks/GitGutter)
- [DiffView](https://packagecontrol.io/packages/DiffView)
- GitDiffHelper
- Gitignore
- SublimeGit

## 前端

- Emmet
- HTML-CSS-JS Prettify
- HTML5
- Javascript Completions
- Javascript Console
- jQuery
- SublimeLinter-jshint


## PHP 开发

- SublimeLinter-php
- SublimeLinter-phplint
- PHPIntel
- DocBlockr
- PHP Getters and Setters
- PHP Companion
- PHP Completions Kit
- PHP Code Sniffer(`Phpcs`)

```
    brew search php-code-sniffer
    brew install homebrew/php/php-code-sniffer
    brew search php-cs-fixer
    brew install homebrew/php/php-cs-fixer
    which phpcs
    which php-cs-fixer
```

然后在 `phpcs.sublime-settings` 需要手动设置的选项如下：

```
    "phpcs_show_errors_on_save": false;    // 关闭保存时弹出提示框
    "php_cs_fixer_executable_path": "/usr/local/bin/php-cs-fixer",
    "phpcs_php_path": "/usr/local/bin/phpcs",
```

如果提示找不到文件，则需要手动创建默认配置文件 phpcs.sublime-settings。

- Laravel Blade Highlighter
- Laravel Blade Spacer

## 终端

- VIM Navigation
- Terminal


# 快捷键

有些快捷键是需要先安装相应的插件后才有用的，下面为常用的默认快捷键：

- `cmd + p`：搜索并打开文件，行跳转
- `cmd + shift + p`：打开命令

太 tm 有用了，可以免去记忆很多并不是很快捷的快捷键，安装的插件基本上都可以通过这个命令搜索并打开。

- `cmd + r`：当前文件内查找函数
- `cmd + f`：当前文件内查找
- `cmd+alt+f`：当前文件内查找并替换
- `cmd+shift+f`：指定路径中查找并替换
- `cmd + enter`：无格式换行
- `cmd + option + 1/2/3/…`：设置窗口分组
- `ctrl + 1/2/3/…`：在窗口分组间切换
- `enter／shift+enter`：从搜索结果中向下／上切换
- `cmd + f && alt + enter `: 从搜索结果中多行修改
- `cmd+k && cmd+b `: 隐藏／显示左侧文件树
- `cmd + shift + v `:无格式粘贴

如果在 `sublime text` 中直接使用 `cmd + v` 粘贴一段代码的话，会连同复制源的格式一同复制下来，结果基本上都是很丑的。使用无格式粘贴能确保格式统一。

- `ctrl + '`：打开控制台输出（python）


# 编程字体

- consolas for powerline
- mononoki
- fira-code
- inconsolata
- dejavu-sans-mono-for-powerline
- roboto-mono

安装方式参考：<u>[Mac 流#字体](http://blog.caoxl.com/2018/01/15/Mac-Flow-Dev/#字体)</u>。

# 附录

主要配置文件

```
    {
        "always_show_minimap_viewport": true,
        "show_encoding": true,
        "bold_folder_labels": true,
        "color_scheme": "Packages/Material Theme/schemes/Material-Theme-Palenight.tmTheme",
        "fade_fold_buttons": false,
        "find_selected_text": true,
        "font_face": "dejavu sans mono for powerline",
        // inconsolata
        // mononoki
        // fira code
        // roboto mono
        // consolas for powerline
        "font_options":
        [
            // "bold",
            "gray_antialias",
            "subpixel_antialias"
        ],
        "font_size": 18,
        "highlight_line": true,
        "highlight_modified_tabs": true,
        "ignored_packages":
        [
        ],
        "indent_guide_options":
        [
            "draw_normal",
            "draw_active"
        ],
        "line_padding_bottom": 3,
        "line_padding_top": 3,
        "match_selection": true,
        "material_theme_accent_bright-teal": true,
        "material_theme_accent_lime": true,
        "material_theme_big_fileicons": true,
        "overlay_scroll_bars": "enabled",
        "rulers":
        [
            80, 100, 120
        ],
        "scroll_past_end": true,
        "show_full_path": true,
        "smart_indent": true,
        "tab_completion": false,
        "tab_size": 4,
        "theme": "Material-Theme-Palenight.sublime-theme",
        "translate_tabs_to_spaces": true,
        "trim_trailing_white_space_on_save": true,
        "vintage_start_in_command_mode": true,
    	"word_wrap": true,
    	"wrap_width": 80
    }
```

- `PHP Code Sniffer` 配置示例

```
    {
        // Plugin settings
        // Turn the debug output on/off
        "show_debug": false,
        // Which file types (file extensions), do you want the plugin to
        // execute for
        "extensions_to_execute": ["php"],
        // Do we need to blacklist any sub extensions from extensions_to_execute
        // An example would be ["twig.php"]
        "extensions_to_blacklist": [],
        // Execute the sniffer on file save
        "phpcs_execute_on_save": true,
        // Show the error list after save.
        "phpcs_show_errors_on_save": false,
        // Show the errors in the gutter
        "phpcs_show_gutter_marks": true,
        // Show outline for errors
        "phpcs_outline_for_errors": true,
        // Show the errors in the status bar
        "phpcs_show_errors_in_status": true,
        // Show the errors in the quick panel so you can then goto line
        "phpcs_show_quick_panel": true,
        // The path to the php executable.
        // Needed for windows, or anyone who doesn't/can't make phars
        // executable. Avoid setting this if at all possible
        "phpcs_php_prefix_path": "",
        // Options include:
        // - Sniffer
        // - Fixer
        // - Mess Detector
        //
        // This will prepend the application with the path to php
        // Needed for windows, or anyone who doesn't/can't make phars
        // executable. Avoid setting this if at all possible
        "phpcs_commands_to_php_prefix": [],
        // What color to stylise the icon
        // https://www.sublimetext.com/docs/3/api_reference.html#sublime.View
        // add_regions
        "phpcs_icon_scope_color": "comment",
        // PHP_CodeSniffer settings
        // Do you want to run the phpcs checker?
        "phpcs_sniffer_run": true,
        // Execute the sniffer on file save
        "phpcs_command_on_save": true,
        // It seems python/sublime cannot always find the phpcs application
        // If empty, then use PATH version of phpcs, else use the set value
        "phpcs_executable_path": "/usr/local/bin/phpcs",
        // Additional arguments you can specify into the application
        //
        // Example:
        // {
        //     "--standard": "PEAR",
        //     "-n"
        // }
        "phpcs_additional_args": {
            "--standard": "PSR2",
            "-n": ""
        },
        // PHP-CS-Fixer settings
        // Fix the issues on save
        "php_cs_fixer_on_save": false,
        // Show the quick panel
        "php_cs_fixer_show_quick_panel": false,
        // Path to where you have the php-cs-fixer installed
        "php_cs_fixer_executable_path": "/usr/local/bin/php-cs-fixer",
        // Additional arguments you can specify into the application
        "php_cs_fixer_additional_args": {
        },
        // phpcbf settings
        // Fix the issues on save
        "phpcbf_on_save": false,
        // Show the quick panel
        "phpcbf_show_quick_panel": false,
        // Path to where you have the phpcbf installed
        "phpcbf_executable_path": "",
        // Additional arguments you can specify into the application
        //
        // Example:
        // {
        //     "--level": "all"
        // }
        "phpcbf_additional_args": {
            "--standard": "PSR2",
            "-n": ""
        },
        // PHP Linter settings
        // Are we going to run php -l over the file?
        "phpcs_linter_run": true,
        // Execute the linter on file save
        "phpcs_linter_command_on_save": true,
        // It seems python/sublime cannot always find the php application
        // If empty, then use PATH version of php, else use the set value
        "phpcs_php_path": "/usr/local/bin/phpcs",
        // What is the regex for the linter? Has to provide a named match for 'message' and 'line'
        "phpcs_linter_regex": "(?P<message>.*) on line (?P<line>\\d+)",
        // PHP Mess Detector settings
        // Execute phpmd
        "phpmd_run": false,
        // Execute the phpmd on file save
        "phpmd_command_on_save": true,
        // It seems python/sublime cannot always find the phpmd application
        // If empty, then use PATH version of phpmd, else use the set value
        "phpmd_executable_path": "",
        // Additional arguments you can specify into the application
        //
        // Example:
        // {
        //     "codesize,unusedcode"
        // }
        "phpmd_additional_args": {
            "codesize,unusedcode,naming": ""
        },
        // PHP Scheck settings
        // Execute scheck
        "scheck_run": false,
        // Execute the scheck on file save
        "scheck_command_on_save": false,
        // It seems python/sublime cannot always find the scheck application
        // If empty, then use PATH version of scheck, else use the set value
        "scheck_executable_path": "",
        // Additional arguments you can specify into the application
        //
        //Example:
        //{
        //  "-php_stdlib" : "/path/to/pfff",
        //  "-strict" : ""
        //}
        "scheck_additional_args": {
            "-strict" : ""
        }
    }
```

- 一键清空 PHP 调试代码正则

```
    ^[( )((//)?)]+ *(print_r|var_dump)(.;\s?(//)? *(die;)?$
    
    \/\/ *[(print_r)(var_dump)]+ *(.;$
```

# FAQ

- **Sublime Text 3 插件配置文件 在 Mac 上设置文件保存失败问题？**

需要在 *~/Library/Application\ Support/Sublime\ Text\ 3/Packages/* 路径下手动创建与 Package 同名目录。

- **Sublime Text 不能将 $ 当作 PHP 变量的一部分？**

`Cmd + Shift + P`：Preference:Browse Packages。选择 `User`，新建配置文件 `PHP.sublime-settings`，内容如下：

```
    {
      "word_separators": "./\\()\"'-:,.;<>~!@#%^&*|+=[]{}`~?"
    }
```

其作用就是：当编辑 PHP 文件的时候，覆盖 Sublime 默认配置，使 `$` 成为单词的一部分。


# 参考

- [Sublime Text 3 快捷键汇总](http://jinfang.life/posts/e4aa08c5/)
- [Sublime Text 个人经验](http://kisss.cjli.info/auxiliary/Sublime-Text-Experience.html)
- [80-characters / right margin line in Sublime Text 3](https://stackoverflow.com/questions/25900954/80-characters-right-margin-line-in-sublime-text-3)
- [Fix PHP variable selection in Sublime Text](https://www.youtube.com/watch?v=SVFOOkPA7n4&app=desktop)
- [Sublime Sublime Text](http://blog.cjli.info/2016/10/12/Sublime-Sublime-Text/)
