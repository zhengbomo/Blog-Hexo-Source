---
title: React Native Flex布局
date: 2017-05-25 17:12:01
updated: 2017-05-25 17:12:01
categories: iOS
tags: [reactnative]
---


# 简介
React Native通过一个基于FlexBox的布局引擎，在所有移动平台上实现了一致的跨平台样式和布局方案。

## 基础
### 主轴与侧轴
通常元素布局的主轴为水平方向，侧轴为垂直方向

FlexBox布局目前支持的属性有如下6个：
1. `flex`：拉伸比例
2. `flexDirection`：子元素排列方向
4. `alignItems`：子元素沿`flexDirection`垂直方向 的对齐方向
3. `alignSelf`：自身对齐方向
5. `justifyContent`: 子元素沿`flexDirection`方向的对齐位置
6. `flexWrap`：超出可视区域是否换行


## 1. flex
当一个元素定义了`flex`属性时，表示该元素是可伸缩的，flex值表示元素伸缩的比例，通常设置为1，
```js
class App extends Component {
  render() {
    return (
      <View style={{flex: 1}}>
        <View style={{flex: 1, backgroundColor: 'green'}} >
          <View style={{flex: 0.2, backgroundColor: 'red', margin: 10}}>
          <View style={{flex: 0.1, backgroundColor: 'blue', margin: 10}} />
        </View>
        <View style={{ height: 200, backgroundColor: 'gray', margin: 10}} />
      </View>
    );
  }
};
```
* 最外层View，flex为1，表示完全撑开
* 绿色View设置了`flex`，灰色View没有设置，故除去灰色View的剩余空间被绿色View撑满
* 红色View和蓝色View都设置了`flex`，故按比例（0.2：0.1）撑开父容器

## 2. flexDirection
表示子元素在父容器的排列方向，有下面两个取值（移动端不支持`row-reverse`和`column-reverse`）  
* `row`: 主轴正方向，通常是从左到右
* `column`: 侧轴正方向（默认），通常是从上到下
* `row-reverse`: 主轴反方向，通常是从右到左
* `column-reverse`: 侧轴反方向，通常是从下到上

```js
class App extends Component {
  render() {
    return (
      <View style={{flex: 1}}>
        <View style={{flex: 1, flexDirection: 'row-reverse', backgroundColor: 'blue'}} >
          <View style={{width: 50, height: 50, backgroundColor: 'red', margin: 5}}>
            <Text style={{fontSize: 25, color: 'white', alignContent: 'center', textAlign: 'center'}}>1</Text>
          </View>
          <View style={{width: 50, height: 50, backgroundColor: 'red', margin: 5}}>
            <Text style={{fontSize: 25, color: 'white', alignContent: 'center', textAlign: 'center'}}>2</Text>
          </View>
          <View style={{width: 50, height: 50, backgroundColor: 'red', margin: 5}}>
            <Text style={{fontSize: 25, color: 'white', alignContent: 'center', textAlign: 'center'}}>3</Text>
          </View>         
        </View>

         <View style={{flex: 1, flexDirection: 'column', backgroundColor: 'green'}} >
          <View style={{width: 50, height: 50, backgroundColor: 'red', margin: 5}}>
            <Text style={{fontSize: 25, color: 'white', alignContent: 'center', textAlign: 'center'}}>1</Text>
          </View>
          <View style={{width: 50, height: 50, backgroundColor: 'red', margin: 5}}>
            <Text style={{fontSize: 25, color: 'white', alignContent: 'center', textAlign: 'center'}}>2</Text>
          </View>
          <View style={{width: 50, height: 50, backgroundColor: 'red', margin: 5}}>
            <Text style={{fontSize: 25, color: 'white', alignContent: 'center', textAlign: 'center'}}>3</Text>
          </View>          
        </View>
      </View>    
    );
  }
};
```

## 2. 对齐方式枚举

* `flex-start`: 按起始位置方向对齐
* `flex-end` 按结束位置方向对齐
* `center` 居中对齐
* `stretch` 拉伸对齐，拉伸填满空白区域
* `space-between` 每个元素之间间隔相等，并撑开，第一个和最后一个元素对齐首尾
* `space-around`：每个元素撑开，并等分

## 3. alignItems


## 4. alignSelf
与`alignItems`类似，自身的侧轴对其方向

元素的对齐方向，与父容器的`flexDirection`的排列方向有关，如果是横向（`row`或`row-reverse`）则对齐属性则表示纵向对齐，反之亦然
alignSelf的对齐方式主要有四种：

* `flex-start`: 起始排列位置
* `flex-end`: 与`flexDirection`反方向的终点位置
* `center`：居中对齐
* `auto`：
* `stretch`

## 4. justifyContent
子元素的对齐方向
```js
class App extends Component {
  render() {
    return (
      <View style={{flex: 1}}>
        <View style={{flex: 1, flexDirection: 'row', backgroundColor: 'blue', justifyContent: 'flex-end'}} >
          <View style={{width: 50, height: 50, backgroundColor: 'red', margin: 5}}>
            <Text style={{fontSize: 25, color: 'white', alignContent: 'center', textAlign: 'center'}}>1</Text>
          </View>
          <View style={{width: 50, height: 50, backgroundColor: 'red', margin: 5}}>
            <Text style={{fontSize: 25, color: 'white', alignContent: 'center', textAlign: 'center'}}>2</Text>
          </View>
          <View style={{width: 50, height: 50, backgroundColor: 'red', margin: 5}}>
            <Text style={{fontSize: 25, color: 'white', alignContent: 'center', textAlign: 'center'}}>3</Text>
          </View>         
        </View>
      </View>    
    );
  }
};
```









FlexBox

flex: 拉伸属性
flexDirection: 主轴方向
	row:
	column(默认):
	row-reverse:
	column-reverse:

justifyContent: 主轴children对其方式
alignItems: 侧轴children对其方式
alignSelf`: 侧轴元素对其方式，会覆盖父容器的aligItems
flexWrap: 与文本换行类似，子元素超出父元素时是否换行



# 对齐方式
flex-start
flex-end
center
stretch
space-between
space-around











# 八次
