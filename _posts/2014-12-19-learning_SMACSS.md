---
layout: post
comments: false
title: SMACSS 学习笔记
category: Notes
tag: front-end css
---

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

一般来讲，有三种状态可能出现变化：

1. 类名
    - 一般通过 JavaScript 根据用户行为来变化
2. 伪类
    - 注：可通过伪类的变化来改变元素的兄弟（siblings）和后代。要想改变其他，还得使用 JavaScript
3. 媒体查询（media query）


