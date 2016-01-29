---
title: 对 React 生态圈进行单元测试的探索
date: 2016-01-29 15:00:38
tags: 
  - Tech Salon
  - test
  - react
  - react-redux
  - karma
  - jest
---

<p align="center">
  <img style="border-radius: 4px; box-shadow: 1px 2px 4px #aaa;" src="http://images.dtcj.com/DTCJ/1454055712615/59d88e80-c661-11e5-a8af-23b8bf415c38.jpeg">
</p>

第一次 TapasFE Tech Salon 于北京时间 2016-01-29 11:00 如期举行。会议上 [leeching](https://github.com/leeching) 针对目前 `React` 生态圈进行单元测试的方法及存在的问题进行了分享，内容如下。

### 前端测试难点

1. 代码耦合度高（业务代码和业务代码，业务代码和外部库）
2. 异步逻辑（事件回调，异步请求，promise）
3. 对于react组件，获取父组件下的子组件的`ref`是难点

### jest-jasmine

#### jest的问题

1. jest的配置麻烦，至少要配置三处文件scripts/babel-jest.js，package.json，__mocks__
2. jest测试运行中的报错信息难以解读
3. jest的单元测试中的console无法在terminal中显示，不方便调试
4. jest会自动mock所有包，实际上，只有极个别的包才需要mock

#### jest的优点

1. mock机制方便测试

*从实用性上考虑，karma+karma-webpack+jasmine优于jest*

### 测试步骤

1. 判断当前组件是否能够被测试（耦合度，模块化），是否有测试的价值（逻辑过于简单和逻辑过于复杂的组件都没有测试的价值，前者可以在测试其父组件的同时顺便测一下，后者应该在模块内部划分成更小的组件以提高测试效率）
2. 将组件以及相关的测试所需的辅助模块引入测试文件
3. 准备mock数据
4. 渲染组件并获取组件的实例以及真实dom，使用`renderComponent`辅助函数
5. 以五大类测试内容为基础编写测试用例
6. 每次跑完一个用例都需要通过`afterEach`做清理工作

### 测试内容

1. 各阶段的数据渲染是否准确
2. 各阶段的组件状态是否准确
3. 回调执行后的数据和状态变动是否符合预期
4. api请求的地址，类型和发送数据是否符合预期
5. helper函数的逻辑是否符合预期

### 不需要测试的部分

1. 和组件状态无关的样式不测
2. 如果组件内部使用了antd提供的组件，只需要测试向antd提供的props数据是否准确

### 测试mock

1. 函数: `spyOn(mock, 'clickHandler').and.callThrough();`
2. fetch: require('testHelper').mockFetch
3. 外部库提供的`props`，比如`react-router`会在container Component的`props`上新增`route`, `routeParams`（组件尽可能避免对这些`props`的依赖）
4. react-redux: 使用`renderProviderComponent`函数测试

### react-redux测试

1. `connect`组件的时候要传入第四个参数`{withRef: true}`
2. 在`beforeEach`的回调中声明一个完整的`Provider`组件，并用`renderConnectComponent`辅助函数render，获得组件实例和真实dom

### 其他

1. 约定在`this._refs`上添加对需要测试的内部组件的引用
2. 不要编写过多的smart组件，因为在测试smart组件的时候需要mock很多数据，测试效率低
3. 涉及到状态相关的动作，比如`setState`，`dispatch`，应当谨慎处理，不要为了方便开发随意执行这些动作
4. 尽可能编写同步代码，谨慎对待异步代码
