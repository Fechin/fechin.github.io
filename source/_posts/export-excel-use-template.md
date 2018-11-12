---
title: 用模板的思路实现 Excel 导出
date: 2018-03-03 03:07:23
tags:
- Excel
- 方案
- 导出Excel
mp3: /static/mp3/%E8%B0%AD%E8%8C%9C%20-%20%E5%85%89%E6%98%8E.mp3
cover:  /static/images/index/Softonic-image-1.png
---
![](/static/images/icourt/wechat/1531520045411_.pic.jpg)

------------------
Alpha 系统中的任务导出功能，第一版引用了经典的 Excel 操作工具 Apache POI。在第二版需求中，产品对样式进行了调整，内容更加直观，POI 提供了灵活多样的方法来处理 Excel，但是，每次样式的改变带来的都是代码修改。

POI 把 Excel 封装成了 Java 对象，对 Excel 的修改不得不调用内部方法一一设置，尽管方法库比较丰富，基本满足业务需求，但是，性能、扩展以及易用性方面都有待考证，是否还有更好的办法实现 Excel 导出功能？

对于样式的设置和修改，最好的方法当然是不修改。如果用模板加自定义语法来实现，达到使用者不需要关心样式设置，并且可以不懂 POI，只需要了解基本的语法即可完成一个 Excel 导出，相比 POI 性能是否有所提升？下文，我们一起来实施这个方案。

### 文件结构

在此之前，需要了解 Excel xlsx 格式的文件结构，Excel 文件使用 OOXML（Open Office XML）文件格式，这是微软为 Office 2007 产品开发的技术规范，一个开放文档格式标准。Excel 以熟悉的 XML 为存储语言，了解内部结构有助于对 xlsx 进行深度定制，下图是一个较为全面的 Excel 元素集合。

![xlsx 文件组成](/static/images/icourt/wechat/006tKfTcly1fktlxunqd0j30dw0fnta4.jpg)

------------------
xlsx 文件实际上是一个压缩的 XML 文件集合，包含元数据、媒体和图片等文件，开发者可以方便地查看和编辑 Excel 工作簿的结构内容，我们把它看做 zip 压缩包，通过 `unzip` 命令解压得到如下内容：
```
Archive:  template.xlsx
 ----------------------------------------------
├── [Content_Types].xml -------- 内容类型
├── _rels
│   └── .rels ----------------- 关系配置
├── docProps
│   ├── app.xml --------------- 扩展属性
│   └── core.xml -------------- 核心属性
└── xl ------------------------- Excel 实际内容
    ├── _rels
    │   └── workbook.xml.rels
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
xl/worksheets/sheet{n}.xml 文件包含了工作表的行、列、合并的单元格以及单元格其它特征。每个工作表对应一个 sheet.xml，文件名序号`n`从 1 开始，它是实现方案的重要文件。
```xml
<worksheet ... >
  <cols><!-- 列集合 -->
    <col min="1" max="1" width="5" ... />
    ...
  </cols>
  <sheetData>
    <row r="1" spans="1:10" ... /><!-- 行数据 -->
      <c r="B1" s="21" t="s"><!-- cell：一个单元格 -->
        <v>11</v><!-- value：内容下标，见 sharedStrings.xml -->
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
xl/sharedStrings.xml 存储工作表中出现的字符串。Excel 为了节省空间，每一个相同的字符只会保存一次。一个`<si>`（SharedStringItem）标签代表一个内容，从 0 开始， 被 sheet.xml 中的`<v>`（value）标签引用，程序中可以把 sharedStrings.xml 解析成一个 List 集合，集合下标也是引用下标。
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
  <si><!-- 在根节点 sst 中的顺序作为引用下标 -->
    <t>任务附件</t>
   ...
  </si>
  ...
</sst>
```


### 实现思路
sheet.xml 和 sharedStrings.xml 是存储内容的关键文件，是我们需要修改的地方。设置单元格的值有两种方法，一种是官方默认用下标从 sharedStrings 文件中获取，支持压缩；另一种是采取内联字符串的方式将文本放在`t`标签。
1. 默认：`<c r="B1" s="21" t="s"><v>11</v></c>`
2. 内联：`<c r="B1" s="21" t="inlineStr"><is><t>任务名称</t></is></c>`

我们采取内联方式，内容直接存储在单元格标签里，也就是说可以抛弃 sharedStrings.xml 文件，视它不存在。把 sheet.xml 文件人工转换成 sheet.ftl 模板文件，渲染 sheet.ftl 并打包到目标 xlsx 即可导出一个完整的 Excel。

以上，美中不足的是需要解压 xlsx 模板文件，提取并修改 sheet.xml 为模板文件，稍有不慎可能会格式错乱，牵一发而动全身，使用过程复杂，违背了方案的初衷。

以用户的角度出发，在 Excel 文档中设置标记语法，可视化编辑，通过代码逻辑实现模板转换、生成和替换，完成内容渲染和导出。一切，交给程序，如图右方案。
![方案对比](/static/images/icourt/wechat/excel-export-scheme-contrast.png)


### 公式分解
![转化成公式](/static/images/icourt/wechat/Jietu20180315-175600.jpg)

如图，`#`号作为公式的标识，可能在公式的前后以及内容中间出现，目前只实现了 Freemarker 模板语法对接，后期可以支持多种模板引擎，如常用的 Velocity，Thymeleaf。

`#`号中间的公式参考 Freemarker 语法

| 公式                    | 实例                                         | 说明                                                                                  |
| --------                | --------                                     | --------                                                                              |
| #xxx#                   | #matterName!#                                | 字符串、数字等值                                                                      |
| #xxx.xx#                | #tasks.attendeesStr!#                        | 对象属性值                                                                            |
| #xxx.xxx?string('','')# | #taskGroups.state?string('已完成','未完成')# | 取值并判断，参考 Freemarker 语法                                                        |
| #loopTo#n#              | #loopTo#8#taskGroups.tasks.name!#            | 1. 循环取值<br>2.loopTo 表示开始循环<br>3. 数字 8 表示到第几行停止循环<br>4. 支持嵌套循环 |



### 导出效果
![导出后效果](/static/images/icourt/wechat/Jietu20180302-164141.jpg)

