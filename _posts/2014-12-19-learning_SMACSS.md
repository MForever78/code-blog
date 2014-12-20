---
layout: post
comments: false
title: SMACSS 学习笔记
category: Notes
tag: front-end css
---

写出可用的 CSS 并不难，但是写出可维护的 CSS 一直是我在考虑的一个问题。如何让一个团队写出来的 CSS 像是同一个人写出来的，在这点上，Google CSS style guide 看起来规定得太少。这时候 CSS 模式便被提出，本文就是我在学习 SMACSS( Scalable and Modular Architecture for CSS ) 时候的笔记。

## CSS 的目录划分

1. Base
    - 基本概念：使用元素、伪类、子元素、后代选择器指定默认样式，不规定 ID 和 class 样式
    - 主要功能：设置标题大小，默认链接样式、默认字体样式、页面背景
    - 注意：不应在 base 里使用 `!important`；应当始终设置
2. Layout
    - 基本概念：划分页面的几个部分（如 Header, Sidebar, Content, Footer）
    - 注意：一般使用单一选择器（ID 或 class 均可），尽可能使用 ID （因为会给 JavaScript 提供便利）
    - 若想要设置多种不同风格的 layout 供用户选择，可在 layout 的父层级上指定类型（如 body），然后再用后代选择器指定样式，如 `.l-fixed #article`
3. Module
    - 基本概念：可重用的组件
    - 注意：避免使用 ID 选择器
    - 使用语义化的类名，如 `.fld-name`、`fld-items`；避免使用元素选择器
    - 同一 module 在不同情景下的表现区分，可用 sub-class 的方式来实现。如：

    ```css
    .pod {
        width: 100%;
    }
    .pod input[type=text] {
        width: 50%;
    }
    .pod-constrained input[type=text] {
        width: 100%;
    }
    ```
    - 需要考虑 specificity 的时候，可以将类名叠加起来，如 `.pod.pod-callout`
4. State
    - 基本概念：规定指定 module 和 layout 在特定状态下的样式
    - State 样式具有最高优先级，可覆盖其他任何样式
    - State 和 sub-class 区别于两点：
        1. State 可以应用在 layout 和 module 上
        2. State 通常意味着 JavaScript 依赖
    - State 可（应）使用 `!important`，因为不会出现同时有两个相反的状态出现在同一元素上的情形
    - 当某种状态和某个 module 高度相关时，应考虑在命名上加以区分，如 `.is-tab-active`
5. Theme
    - 基本概念：规定 layout 和 module 的样式

## 命名规则

1. layout 和 state 使用前缀
    - `.layout-` 或 `.l-`
    - `.is-hidden` 、 `.is-collapsed`
2. module 使用语义，模块内元素使用前缀
    - `.example`、`.callout`
    - `.example-caption`

## 状态变化

一般来讲，元素样式变化可通过三种状态变化进行：

1. 类
    - 一般通过 JavaScript 根据用户行为来变化
    - 可通过给指定元素加减 State 类使其变化
    - 对于子菜单，可用兄弟选择器来实现，会使动作更易扩展，对其他的元素影响更小，如：

    ```html
    <div id="content">
        <div class="toolbar">
            <button id="btn-new" class="btn is-active" data-action="menu">New</button>
            <div id="menu-new" class="menu">
                <ul> ... </ul>
            </div>
        </div>
    </div>
    ```

    ```css
    /* CSS for styling */
    .btn.is-active { color: #000; }
    .btn.is-active + .menu { display: block; }
    ```
    - 用属性选择器实现状态变化，如：

    ```css
    .btn[data-state=default] { color: #333; }
    .btn[data-state=pressed] { color: #000; }
    .btn[data-state=disabled] { opacity: .5; pointer-events: none; }
    ```

    ```html
    <button class="btn" data-state="disabled">Disabled</button>
    ```

    ```javascript
    // bind a click handler to each button
    $(".btn").bind("click", function(){
        // change the state to pressed
        $(this).attr('data-state', 'pressed');
    });
    ```
    - JavaScript 负责行为，可以描述状态变化，不应该用来添加内联样式（inline styles）；CSS 负责表现。两者配合实现动画：

    ```css
    @-webkit-keyframes fade {
        0% { opacity:0;  }
      100% { opacity:1; display:block; }
    }
    .is-visible {
        opacity: 1;
        animation: fade 2s;
    }
    .is-hidden {
        opacity: 0;
        animation: fade 2s reverse;
    }
    .is-removed {
        display: none;
    }
    ```

    ```javascript
    function showMessage (s) {
        var el = document.getElementById('message');
        el.innerHTML = s;
        /* set state */
        el.className = 'is-visible';
        setTimeout(function(){
            /* set state back */
            el.className = 'is-hidden';
            setTimeout(function(){
                el.className = 'is-removed';
            }, 2000);
    }, 3000); }
    ```
2. 伪类
    - 注：可通过伪类的变化来改变元素的兄弟（siblings）和后代。要想改变其他，还得使用 JavaScript
    - 使用了 sub-class 以后也要加上相应的伪类
3. 媒体查询（media query）
    - 使用响应式设计的时候，在每个的 module 下立刻写出对应的 media query，保证 module 的集中，如：

    ```css
    /* default state for nav items */
    .nav > li {
        float: left;
    }
    /* alternate state for nav items on small screens */
    @media screen and (max-width: 400px) {
        .nav > li {
            float: none;
        }
    }
    ... elsewhere for layout ...
    /* default layout */
    .content {
        float: left;
        width: 75%;
    }
    .sidebar {
        float: right;
        width: 25%;
    }
    /* alternate state for layout on small screens */
    @media screen and (max-width: 400px) {
        .content, .sidebar {
            float: none;
            width: auto;
        }
    }
    ```

## 选择器的深度（Depth）

- CSS 不应依赖与 HTML 的结构；被选择的 HTML 元素位置不应太深。一个反例：`body.article > #main > #content > #intro > p > b `

