---
title: 用模板的思路实现Excel导出
date: 2018-03-03 03:07:23
tags:
mp3: http://oybheyjxt.bkt.clouddn.com/%E8%B0%AD%E8%8C%9C%20-%20%E5%85%89%E6%98%8E.mp3
cover: http://odwjyz4z6.bkt.clouddn.com/index/Softonic-image-1.png
---
Alpha系统中的任务导出功能，第一版引用了经典的Excel操作工具Apache POI。在第二版需求中，产品对样式进行了调整，POI给我们提供了设置样式的方法，但是每次样式的改变带来的都是代码修改，有没有更好的实现方案呢？
 
POI把Excel封装成了Java对象，所以对Excel的修改不得不调用对象方法进行设置，不管是性能、扩展或是易用性方面都有待考证，是否还有更好的办法来处理Excel？
 
对于样式的设置和修改，最好的方法当然是不修改。于是，我考虑用模板加自定义语法来实现，达到使用者不需要关心样式设置，也可以不懂POI，只需要了解基本的语法即可完成一个Excel导出，并且相比POI性能也有不少提升。

------------------
### 文件结构

介绍方案之前，需要了解Excel xlsx格式的文件结构，Excel文件使用OOXML（Open Office XML）文件格式。 这是微软为Office 2007产品开发的技术规范，一个开放文档格式标准；以熟悉的XML为存储语言，了解内部结构使得我们有机会对它进行深度定制。

![xlsx文件组成](http://odwjyz4z6.bkt.clouddn.com/icourt/wechat/006tKfTcly1fktlxunqd0j30dw0fnta4.jpg)

------------------
xlsx文件实际上是一个压缩的XML文件集合，包含元数据、媒体和图片等文件，开发者可以方便地查看和编辑Excel工作簿的结构内容，把它看做zip压缩包，通过 `unzip` 命令解压得到如下内容：
```
Archive:  template.xlsx
 --------------------------------------------------
├── [Content_Types].xml -------- 内容类型
├── _rels
│   └── .rels ----------------- 关系配置
├── docProps
│   ├── app.xml --------------- 扩展属性
│   └── core.xml -------------- 核心属性
└── xl ------------------------- Excel实际内容
    ├── _rels
    │   └── workbook.xml.relsX
    ├── sharedStrings.xml ------ 内容数据库
    ├── styles.xml ------------- 单元格样式
    ├── theme
    │   └── theme1.xml -------- 主题样式
    ├── workbook.xml ----------- 工作表
    └── worksheets
        ├── sheet1.xml --------- 第一个工作表
        └── sheet2.xml --------- 第二个工作表

6 directories, 12 files
```
------------------
`sheet{n}.xml` 文件包含了工作表的数据、公式和特征。每个工作表对应一个sheet.xml，序号`n`从1开始。
```xml
<worksheet ... >
  <cols><!-- 列集合 -->
    <col min="1" max="1" width="5" ... />
    ...
  </cols>
  <sheetData>
    <row r="1" spans="1:10" ... /><!-- 行数据 -->
      <c r="B1" s="21" t="s"><!-- cell：一个单元格 -->
        <v>11</v><!-- value：内容下标，见sharedStrings.xml -->
      </c>
      <c r="C1" s="21"/>
    </row>
  </sheetData>
  <mergeCells count="6"><!-- 合并单元格集合 -->
    <mergeCell ref="B6:J6"/>
  </mergeCells>
  ...
</worksheet>
```
------------------
`sharedStrings.xml` 存储工作表中出现的每个唯一字符串。Excel为了节省空间，每一个相同的字符只会保存一次。一个`<si>`（SharedStringItem）标签代表一个内容，从0开始，被sheet.xml中的`<v>`（value）标签引用。
```xml
<sst ...>
  <si><!-- SharedStringItem：字符内容条目 -->
    <t>状态</t><!-- Text：具体内容 -->
    ...
  </si>
  <si>
    <t>任务详情</t>
    ...
  </si>
  <si><!-- 在根节点sst中的顺序作为引用下标 -->
    <t>任务附件</t>
   ...
  </si>
  ...
</sst>
```

------------------

### 实现思路
sheet和sharedStrings是存储内容的关键文件，也是需要修改的地方。设置单元格的值有两种方法，一种是官方默认用下标从sharedStrings文件中获取，支持压缩；

另一种是采取内联字符串的方式将文本放在`t`标签，我们先采取内联方式，后期如果有必要可以进行优化。
默认：`<c r="B1" s="21" t="s"><v>11</v></c>`
内联：`<c r="B1" s="21" t="inlineStr"><is><t>任务名称</t></is></c>`

选择内联方式，也就是说可以抛弃sharedStrings文件，视它不存在。为了实现导出功能我们完全可以把sheet.xml文件做成模板，替换xlsx中的sheet.xml为模板渲染后的文件。

美中不足的是需要解压xlsx模板文件，提取并修改sheet.xml为模板文件，稍有不慎可能会格式错乱，牵一发而动全身，使用过程复杂，违背了这个方案的初衷。

以用户的角度出发，我考虑直接在Excel设置标记语法，通过程序代码来实现模板转换、生成和替换，完成内容渲染和导出。
![方案对比](http://odwjyz4z6.bkt.clouddn.com/icourt/wechat/excel-export-scheme-contrast.png)

------------------

### 公式分解
`#`号作为公式的标识，可能在公式的前后以及内容中间出现，目前只实现了Freemarker模板语法对接，后期可以支持多种模板引擎，如常用的Velocity，Thymeleaf。


| 公式      |    实例    |   说明  |
| -------- | --------| -------- |
| `#xxx#`  | `#matterName!#` |  字符串、数字等值   |
| `#xxx.xx#`  | `#tasks.attendeesStr!#` |  对象属性值   |
| `#xxx.xxx?string('','')#`  | `#taskGroups.state?string('已完成','未完成')#` |  取值并判断，参考Freemarker语法  |
| `#loopTo#n#`  | `#loopTo#8#taskGroups.tasks.name!#` |  1.循环取值<br>2.`loopTo`表示开始循环<br>3.数字8表示到第几行停止循环<br>4.支持嵌套循环   |

`#`号中间的公式参考Freemarker语法
![转化成公式](http://odwjyz4z6.bkt.clouddn.com/icourt/wechat/Jietu20180302-163914.jpg)

------------------

### 导出效果
![导出后效果](http://odwjyz4z6.bkt.clouddn.com/icourt/wechat/Jietu20180302-164141.jpg)

