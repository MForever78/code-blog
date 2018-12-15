---
layout: post
comments: false
title: React 组件化思想
category: front-end
tag: react front-end facebook flux component
---

> 按：驱动 React.js 高效性能的虚拟 DOM 技术作用的最基础单元是 React 世界中被称为组件（Component）的东西。本文试图用一个具体的例子说明 React 的组件机制、组件间的通信方式，以及衍生出的 Flux 架构模式。

在最近的一个小项目中，我尝试了 React 作为组件化方案，其中遇到的一些问题在国内现有的资料中很少提及，希望大家在看过本文后对 React 的组件机制有一个比较完整的理解，避开一些常见的误区。

官方文档中称 React 倾向于做传统 MVC 架构中的 View 层。不同于 Angular ，你完全可以在大项目中的一个小组件上尝试 React 。事实上，这是学习 React 的一个很好的切入点。这篇文章就将以一个页面上的弹出框这个模型为例，向大家展示我的学习路径。

## 视觉稿

我从设计师那里得到的视觉稿是长这样的：

![](//code.mforever78.com/images/react-interface-draft.png)

没错它确实是一个网页，看起来设计师希望我们模拟出原生 UI 的感觉。这没什么大惊小怪的，毕竟前端工程师就应该满足设计师各种奇怪的需求嘛。

在这张页面上，动态数据大致可以分为这样几项：

1. 用户照片 URL
2. 用户花名（睿西）
3. 标题名（Marketch）
4. 提示文字（上传提示文字）
5. 项目选项
6. 类型选项
7. 链接提示文字以及跳转链接（加入新项目）
8. 按钮提示文字以及对应动作（取消、确认）

## 第一步：划分组件

拿到视觉稿后要做的第一件事就是把这样一个页面的整体划分为许多小组件。React 划分组件的依据是[单一责任原则](https://en.wikipedia.org/wiki/Single_responsibility_principle)。这是从面向对象编程中借来的概念，和我们从学习编程时就被灌输的「不要写过长的函数」思想类似，组件划分得越细，负责的事情越少，维护起来就越简单，逻辑就越清晰。

我划分的结果是长这样的：

![](//code.mforever78.com/images/react-divide-components.png)

组件划分的顺序无外乎自下而上或自顶向下，看个人习惯。我喜欢的方式是后者。

整个页面可视为一个 App（红色标注）；左边是头像和描述作为一个 ImageWrap （蓝色标）；右边橘色框为 BodyWrap ，里面包含一个标题和一个描述；里面绿色的是业务组件 BusinessComponent ；业务组件包含了两个 Dropdown （青色标注）和两个 NavButton （黄色标注）。

用列表描述我们的页面应该是这个样子：

- App
    - ImageWrap
    - BodyWrap
        - BusinessComponent
            - Dropdown
            - NavButton

对于我的划分，你可能有些困惑。比如为什么需要有 BodyWrap 或 BusinessComponent 这种组件集合体，而不是把更小的组件直接放在 App 的下一级。别急，我在后文中将会给出答案。

## 第二步：编写静态版本

有了以上组件划分，我们就可以初步写一个完全没有交互的静态版本了。与组件划分的思路相反，在实现的时候我们按照自下而上的顺序编写代码。这样做的好处是在写完一个组件后可以马上方便地进行测试。

由于代码量较大，这里以一个典型的组件 Dropdown 为例：

```js
var Dropdown = React.createClass({
    propTypes: {
        title: ReactPropTypes.string,
        text: ReactPropTypes.string,
        options: ReactPropTypes.array.isRequired
    },

    render: function() {
        var optionNodes = this.props.options.map(function(option) {
            return (
                <li key={option.id}>{option.text}</li>
            );
        });
        return (
            <div className="select">
                {this.props.text}
                <div className="selector"><i className="iconfont icon-down"></i></div>
                <ul className="dropdown">{optionNodes}</ul>
            </div>
        );
    }
});
```

在编写静态版本时，我们完全不需要用到任何状态信息（state）。我们的 Dropdown 组件接受 `text`，`title` 和 `options` 三个属性（props）。`text` 负责下拉框的占位文字，`title` 是下拉框左边的标题，`options` 就是选项了。从组件的 DOM 结构可以看出来，这里使用了一个 `ul` 来模拟下拉选项。

静态版本实现以后的效果是我们完整地复原了视觉稿的样子，但要想让这个下拉框响应用户的点击动作，我们就要为这个组件增加状态了。

### `props` 和 `state` 的区别

看到这里，有些同学可能会有这样的疑问：下拉框的占位文字应该是随着用户的选择变化的，为什么会是组件的一个属性而不是自身的状态呢？这里就来详细解释一下属性和状态之间的区别。

在 React 世界中的一个很重要的原则就是**让组件尽可能是无状态的**。大多数情况下，我们的组件自身没有状态，只是从父组件那里接受一些属性，然后根据这些属性进行渲染操作。父组件自身维护了一些状态，并且通过 `props` 来传给子组件，从而使 UI 发生变化。对于 Dropdown 组件来说，下拉框内的提示文字是需要父组件在调用时指定的（因为不同情况下初始的提示文字可能不同），如果把它同时作为自己的状态，那么这一处提示文字就可能需要同时通过属性和状态来确定，这样逻辑关系就变得非常复杂了。而像我们选择的那样把这个占位文字作为父组件的状态，Dropdown 组件本身只要把父亲传来的 `props` 原封不动地渲染出来就好，这样让组件本身负责的事情降到最少，维护起来也就更加方便了。

把上面那段话用代码来描述，就是不要写这样的代码：

```js
// 错误的范例！
var Dropdown = React.createClass({
    propTypes: {
        title: ReactPropTypes.string,
        text: ReactPropTypes.string,
        options: ReactPropTypes.array.isRequired
    },

    // 此处 props.text 的作用只是指定了初始值。这是一种 Anti-pattern
    getInitialState: {
        return {text: this.props.text};
    },

    // 改变状态
    _onClick: function(text) {
        this.setState({text: text});
    },
    
    render: function() {
        var optionNodes = this.props.options.map(function(option) {
            return (
                <li key={option.id} onClick={this._onClick.bind(this, option.text)}>{option.text}</li>
            );
        }, this);
        return (
            <div className="select">
                // 此处使用了 state 来管理显示的文字
                {this.state.text}
                <div className="selector"><i className="iconfont icon-down"></i></div>
                <ul className="dropdown">{optionNodes}</ul>
            </div>
        );
    }
});
```

## 第三步：添加状态

那么应该把什么信息作为组件的状态呢？一般而言，状态需要包含那些**仅与自身有关**并且**在组件的回调函数中会发生变化，并且会体现在 UI 上**的信息。仅与自身有关，指的是不需要由父组件决定（就像上面的例子，这种情况应该使用 `props`）；会在回调函数中发生变化，主要指的就是用户的行为导致的变化了。

在 Dropdown 这个组件中，用户点击一下下拉框，我们应该向用户展示出选项，再点击一下，就把选项框收起。这个信息仅与自身有关，并且需要对用户行为作出响应，就是一个状态。加上这个状态以后，我们的组件代码变成了这个样子：

```js
var Dropdown = React.createClass({
    propTypes: {
        text: ReactPropTypes.string.isRequired,
        title: ReactPropTypes.string,
        options: ReactPropTypes.array.isRequired
    },

    getInitialState: function() {
        return {active: false};
    },

    // 响应用户点击事件的回调函数
    toggleFold: function() {
        this.setState({active: !this.state.active});
    },

    render: function() {
        var optionNodes = this.props.options.map(function(option) {
            return (
                <li key={option.id}>{option.text}</li>
            );
        });
        return (
            // 为点击事件的回调函数
            <div className={(this.state.active ? 'active' : '') + " select"}
                 onClick={this.toggleFold}>
                {this.props.text}
                <div className="selector"><i className="iconfont icon-down"></i></div>
                <ul className="dropdown">{optionNodes}</ul>
            </div>
        );
    }
});
```

这样，我们的 Dropdown 组件就可以对用户行为作出响应。

## 第四步：通信

有的同学可能已经发现了，写到这里还有一个遗留的问题没有解决，那就是其他组件如何获取用户的选项。毕竟最终表单的提交需要得到所有用户选择的信息。那么谁来负责整合所有的信息呢？

在上文中提到，我们不是把一些非常小的组件直接组合起来形成一个完整的 App，而是将一些组件又组成了一个组件集合，比如 `BusinessComponent`。这样的做的原因就是我们希望这个组件集合体负责业务逻辑，具体到这里也就是负责整合信息，提交表单。我们把业务逻辑放到这个集合体中去做，避免了 NavButton 或 Dropdown 这些通用组件中 UI 渲染逻辑和业务逻辑的杂糅，提升了它们的通用性，也降低了后期业务逻辑发生变化带来的维护成本。

接下来的问题就是，组件之间应该如何通信呢？

我们认为 Dropdown 组件的选项这些 `<li>` 标签是 Dropdown 的子组件，它们携带了我们需要的用户选项的信息，而需要这些信息的是 Dropdown 的父组件 BusinessComponent 。这样就变成了祖孙之间的通信了。

稍微熟悉 React 的同学应该都清楚，React 完成父子组件通信的方式是很简单的。对于父—子，直接通过 `props` 完成通信就可以了；对于子—父，绑定一个回调函数来完成通信也十分容易。但对这种祖孙关系，甚至兄弟，或者没有明显的关系的情况，应该如何处理呢？

Flux 架构推荐我们使用事件订阅的方式来完成这样的通信。这里以 `Pubsub.js` 这个事件订阅器为例实现：

```js
var React = require('react');
var ReactPropTypes = React.propTypes;
var Pubsub = require('pubsub-js');

var Dropdown = React.createClass({
    propTypes: {
        name: ReactPropTypes.string,
        text: ReactPropTypes.string.isRequired,
        title: ReactPropTypes.string,
        options: ReactPropTypes.array.isRequired
    },

    getInitialState: function() {
        return {active: false};
    },

    toggleFold: function() {
        this.setState({active: !this.state.active});
    },

    // 处理选项点击事件的回调函数
    select: function(id, text) {
        // 发布事件
        Pubsub('select', {
            id: id,
            text: text
        });
    },

    render: function() {
        var optionNodes = this.props.options.map(function(option) {
            return (
                // 为每个选项绑定回调
                <li key={option.id} onClick={this.select.bind(this, option.id, option.text)}>{option.text}</li>
            );
        }, this);
        return (
            <div className={(this.state.active ? 'active' : '') + (this.props.chosen ? ' chosen' : '') + " select"}
                 onClick={this.toggleFold}>
                {this.props.text}
                <div className="selector"><i className="iconfont icon-down"></i></div>
                <ul className="dropdown">{optionNodes}</ul>
            </div>
        );
    }
});
```

然后在业务组件的 `componentWillMount` 事件中订阅这个事件：

```js
var BusinessComponent = React.createClass({
    componentWillMount: function() {
        this.selectSubscribe = Pubsub.subscribe('select', function(msg, data) {
            // 获得选项信息，进行相应处理
        });
    },

    componentWillUnmount: function() {
        Pubsub.unsubscribe(this.selectSubscribe);
    },

    render: function() {
        // ...
    }
});
```

对于这样一个小的项目，我们用上面提到的信息处理的方法完全没有问题，但是对于较大的项目，在所有需要某个信息的地方都去订阅事件就显得非常繁琐了。对于这种情况，Flux 架构给出的解决方案是把信息存储层（Store）放在全局，这样统一管理的方式就方便许多了。想要了解更多信息的同学可以查看[官方文档](https://facebook.github.io/flux/docs/overview.html)。

## 第五步：完成

像这样，把所有组件写好并且组装起来的时候，我们的整个页面就完成了。

如果你有兴趣的话，对于我们的项目，完整的组件方案划分是这样的：

![divide](//code.mforever78.com/images/react-Webview.png)

我们对性能的要求不高，但是使用 React 的组件管理让项目的可维护性和可扩展性都得到了极大的提高。如果你也受够了前端传统 MVC 的痛苦，不如也来试试 React 和 Flux 吧：）
