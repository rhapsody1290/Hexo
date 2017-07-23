---
title: IDEA快捷键

date: 2016-07-08 00:00:00

categories:
- IDEA

tags:
- JavaEE
- IDEA

---

参考http://blog.csdn.net/dc_726/article/details/42784275

## 智能提示
- Alt + /基本代码提示 <font color='gray'>CTRL+空格(和系统输入法冲突，请在Settings->Keymap->mainmenu -> code -Completion->basic，右键添加自己的快捷键)，可以设置为Alt + /</font>
- Ctrl + Alt + /更智能地按类型信息提示 <font color='gray'>默认为Ctrl+Shift+Space</font>
- **ALT + Enter 快速修复，类似Eclipse中的Quick Fix功能**
- Ctrl+Shift+Enter 自动补全末尾的字符，例如敲完if/for时也可以自动补上{}花括号
- **F2/ Shift+F2 移动到有错误的代码**

## 重构

- Ctrl+Shift+Alt+T 重构功能大汇总快捷键，叫做Refactor This
- Shift+F6 直接就是改名
- Ctrl+Alt+V 提取变量
	
## 代码生成

- Ctrl+J 可以查看所有模板
- Alt+Insert，在编辑窗口中点击可以生成构造函数、toString、getter/setter、重写父类方法等


## 编辑

- **Ctrl + Shift + 上下 ： 上下移动当前行**
- Ctrl + Shift + 左右 ：代替鼠标选中代码
- Ctrl + Alt + 上下 ： 复制一行
- Alt + 上下 ： 光标函数间跳转
- **Alt + 左右 ： tab切换**
- **Ctrl + D ： 删除一行**
- Ctrl + / ： 注释
- Ctrl + / Ctrl + Shift + / ： 折叠代码
- Ctrl + '+' Ctrl + Shift + '+'： 展开代码
- ***Ctrl+N / Ctrl+Shift+N 可以打开类或资源***
- Shift+Shift 在一个弹出框中搜索任何东西，包括类、资源、配置项、方法等等
- Ctrl+F/Ctrl+Shift+F 在当前窗口或全工程中查找
- F3/Shift+F3前后移动到下一匹配处
- **格式化代码  **
　　格式化import列表：Ctrl+Alt+O  
　　格式化代码：Ctrl+Alt+L


## 自动代码

- ***CTRL+ALT+L  格式化代码***    
- CTRL+E或者ALT+SHIFT+C 最近更改的代码  
- CTRL+SHIFT+SPACE 自动补全代码  
- CTRL+ALT+SPACE  类名或接口名提示  
- ***CTRL+P   方法参数提示***  
- ***CTRL+ALT+T  代码模版***
- **Ctrl+Shift+T 自动生成测试类**


## 其他

- CIRL+U   大小写切换
- CTRL+Z   倒退
- CTRL+SHIFT+Z  向前
- ***CTRL+ALT+F12  资源管理器打开文件夹在WINDOW窗口快速定位到文件或者文件夹的位置***
- ALT+F1   查找文件所在目录位置
- SHIFT+ALT+INSERT 竖编辑模式
- CTRL+/   注释//  
- CTRL+SHIFT+/  注释
- CTRL+B   快速打开光标处的类或方法
- ***page up 和 page down***
- ***ALT+ ←/→  切换代码视图***
- ***ALT+ ↑/↓  在方法间快速移动定位***
- CTRL+ALT ←/→  返回上次编辑的位置
- ***SHIFT+F6  重构-重命名***
- CTRL+H   显示类结构图
- ***CTRL+Q   显示注释文档***
- ***ALT+1   快速打开或隐藏工程面板***
- ***ALT + 4 Run Console***
- ***ALT + 5 Debug Console***
- ***在任何工具窗口里使用Escape键都可以把焦点移到编辑器上,Shift-Escape不仅可以把焦点移到编辑器上而且还可以隐藏当前（或最后活动的）工具窗口***
- CTRL+W   选中代码，连续按会有其他效果（我去掉了）
- ***CTRL+F4   关闭当前打开文件（我修改为CTRL + W）***
- **Ctrl + Shift + F12 关闭所有工具栏**
- ***Ctrl+H 查看类的继承关系，打开类层次窗口***
- ***Ctrl+F12 查看当前类的所有方法***
- ***Ctrl+B/Ctrl+Alt+B 在继承层次上跳转，分别对应父类或父方法定义和子类或子方法实现***
- ***Ctrl + tab 切来切去***
- ***Ctrl + Shift + A 发号施令***
- ***CTRL+G 定位行***

## JDK doc绑定

CTRL + Q 查看注释文档

![](http://i.imgur.com/vHEExEB.png)
