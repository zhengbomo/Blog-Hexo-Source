---
title: pycharm添加jinja2模板引擎的识别
categories: python
tags: pycharm
date: 2016-09-03 09:09:43
updated: 2016-09-03 09:09:43
---

默认情况下，通过Pycharm新建项目时，有个flask选项，新建完成后，是可以支持jinja模板的识别的，例如跳转，语法高亮等功能，但是如果是自己新建的目录，然后用Pycharm导入的，这个时候html文件里面的模板代码无法被高亮，也无法跳转

<!-- more -->

![](http://7xqzvt.com1.z0.glb.clouddn.com/16-9-3/21068155.jpg)


本身pycharm是支持jinja模板引擎的，只需要在配置文件加上配置即可，打开工程目录`/.idea/xxx.iml`文件，其中xxx为项目名称（即外层目录名），在module节点下添加TemplatesService，如下：
```xml
<component name="TemplatesService">
  <option name="TEMPLATE_CONFIGURATION" value="Jinja2" />
  <option name="TEMPLATE_FOLDERS">
    <list>
      <option value="$MODULE_DIR$/hello/templates" />
    </list>
  </option>
</component>
```
由于我的代码在hello目录下，所以在TemplateFolder路径需要加上，否则会找不到文件，修改完保存即可生效

![](http://7xqzvt.com1.z0.glb.clouddn.com/16-9-3/58402206.jpg)
