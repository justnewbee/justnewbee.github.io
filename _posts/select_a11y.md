# 原生select的键盘访问

[JSBin测试](http://jsbin.com/kucuhe/1)

| 进入 | 调出 | 取消 | 预选 | 确认 |
| -----: | :-----: | ----- | ----- | ----- |
| **Gecko**Firefox | `↹` | <span style="color: #C00;">无法通过键盘将其展开</span>，但可以通过`↑ → ↓ ←`直接选中 | <span style="color: #C00;">不需要</span> | `↑ → ↓ ←`直接选中 | `↑ → ↓ ←`直接选中，<span style="color: #C00;">但必须在失焦后才会触发change事件</span> |
| **Webkit**Chrome/Safari/Opera | `↹` | `␣ ↑ ↓` | `ESC` <span style="color: #C00;">此时无法直接用TAB跳到下一个输入控件</span> | `↑ ↓` | `␣ ↵` |
| **Trident**IE10 | `↹` | <span style="color: #C00;">无法通过键盘将其展开</span>，但可以通过`↑↓`直接选中 | <span style="color: #C00;">不需要</span> | `↑ ↓`直接选中 | `↑ ↓`直接选中，<span style="color: #0C0;">同时触发change事件</a> |

键盘符号：TAB`↹`、SPACE`␣`、方向键`↑ → ↓ ←`、回车`↵`

# UI库的dropdown的键盘访问

## select2

* 测试地址：<http://ivaynberg.github.io/select2/> (使用The Basics处的测试)
* 进出：`↹`
* 调出：`␣ ↑ ↓ ↵`
* 取消：`ESC ↹`
* 预选：`↑ ↓`
* 确认：`↵` <span style="color: #C00;">大概是由于这里的测试案例都是可以内部搜索的缘故，不支持`␣`确认</span>

## jqueryui - selectmenu

* 测试地址：<http://jqueryui.com/selectmenu/>
* 进出：`↹`
* 调出：`␣` <span style="color: #C00;">类似Firefox的select`↑ → ↓ ←`直接选中，故使用`␣`调出后可以预选</span>
* 取消：`ESC ↹`
* 预选：`↑ → ↓ ←`
* 确认：`␣ ↵`