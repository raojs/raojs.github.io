---
layout: post
title:  'Utility-first CSS 以及 TailwindCSS'
date:   2021-09-28
updated-date: 2021-10-14
categories: css
---

从第一个内联 CSS 开始，CSS 写法及组织方式在不断演进。本文将主要介绍工具优先(Utility-first)的 CSS 组织概念以及一款 Utility-first CSS 框架 [TailwindCSS](https://tailwindcss.com/)。

首先我们先了解一下什么是 "Semantic" CSS 以及 Utility-first CSS。

**`"Semantic" CSS`**

常见的 HTML 与样式分离的写法，通常类名会有语义化的含义。

```html
<style>
  .title {
    font-size: 2em;
  }
</style>

<div class="title">
  Hello
</div>
```

[BEM(Block Element Modifer)](http://getbem.com/introduction/) 也是语义化的，提供了样式与结构解耦的方案，为复杂项目的构建提供了更多的选择。

**`Utility first CSS`**

通过抽取通用工具类以满足页面可视化需要，以减少 CSS 编写为目标，同时实现样式的高复用，通常类的命名即可以反映出相关样式定义。

```html
<style>
  .align-right {
    text-align: right;
  }
</style>

<div class="align-right">
  Hello
</div>
```

在语义化 CSS 写法中如果需要新增组件或业务内容，很有可能需要定义字体大小、颜色等样式。然而通常一个项目的大体风格是固定的，在 Utility first 理念下可以直接使用诸如 .text-sm、.color-red 等工具类而不会增加 CSS 体积。

如果再极端一点，所有定义的类都只有一行样式定义对应唯一一个 CSS 样式(One line of CSS for One Style)，就是所谓的原子化 CSS(Atomic CSS)。Facebook 首页通过 Atomic CSS 将首页 CSS 减少了80%，具体可以看[这里](https://engineering.fb.com/2020/05/08/web/facebook-redesign/)。

## 关于 TailwindCSS

### 基本概念及用法

1. 提供了一套默认的命名约定，比如以下代码中类名 p0 包含 `padding: 0px;`、w-1/2 包含 `width: 50%` 等等。

```html
<div class="p0 w-1/2 text-sm text-white bg-green-500 border">
  Hello
</div>
```

伪类及状态添加相关前缀即可。

```html
<div class="bg-white hover:bg-red-100 first:rotate-45">
  Hello
</div>
```

2. 支持响应式设计及深色模式，添加相应 class 即可。

```html
<!-- md: @media (min-width: 768px) { ... } -->
<!-- lg: @media (min-width: 1024px) { ... } -->
<img class="w-16 md:w-32 lg:w-48" src="...">
```

深色模式生效需要配置 `darkMode`，当值为 `media` 时会自动使用 [prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme) 媒体查询特性，当值为 `class` 时会根据 html 标签上的 `class="dark"` 来区分，通常用于手动控制主题的场景。

```js
// tailwind.config.js
module.exports = {
  darkMode: 'media',// 'media' or 'class'
  // ...
}
```

```html
<div class="bg-white dark:bg-gray-100">
  Hello
</div>
```

3. TailwindCSS 提供的是一套工具类的构建系统，工具类的生成规则都是可配置的。可以覆盖或继承默认配置。

```js
// tailwind.config.js
const colors = require('tailwindcss/colors');

module.exports = {
  theme: {
    screens: {
      sm: '480px',
    },
    colors: {
      gray: colors.coolGray,
    },
    extend: {
      spacing: {
        '128': '32rem',
      },
      borderRadius: {
        '4xl': '2rem',
      }
    }
  }
}
```

### 生产环境优化

tailiwind 提供了数千个类名，所有 CSS 在未压缩的情况下有 3M 之多，通过 [`PurgeCSS`](https://purgecss.com/) 可以删除未用到的类名样式，达到类似 tree-shaking 的效果。

需要注意清理过程是基于静态分析，动态生成的类名 PurgeCSS 无法识别从而导致需要的样式被清理掉。例如：

```html
<div class="text-{{  error  ?  'red'  :  'green'  }}-600"></div>
```

需要改写为：

```html
<div class="{{  error  ?  'text-red-600'  :  'text-green-600'  }}"></div>
```

如果一定要动态生成或者类名为随机生成，则可以把相关类名加入 safelist 防止被清理。

```js
// tailwind.config.js
const safecolors = ['pink', 'blue', 'green', 'red', 'purple', 'yellow', 'indigo', 'gray']; // 列表中的颜色在使用时随机生成
let safelist = [];
safecolors.forEach(function(color) {
  safelist.push(`bg-${color}-800`);
  safelist.push(`bg-${color}-900`);
  safelist.push(`hover:bg-${color}-900`);
});

module.exports = {
  purge: {
    content: ['./index.html', './src/**/*.{vue,js,ts,jsx,tsx}'],
    options: {
      safelist,
    },
  },
  // ...
}
```

### 开发编译性能问题

在 v2.1 之前，开发环境采用 AOT(Ahead-of-Time) 模式编译，会在初始化阶段编译所有内容，出于文件大小考虑，一些变体(Variants)也被默认禁用。甚至因为性能问题产生了一个新库 [WindiCSS](https://windicss.org/)，WindiCSS 作者说到：

> 使用 Tailwind 编码非常愉悦，但是随着项目越来越大，初始编译时间达到了 3 秒，热更新时间超过 1 s。

为此 tailwind 在 v2.1 推出了 [JIT(just-in-time)](https://tailwindcss.com/docs/just-in-time-mode) 模式，会根据模版内容按需即时编译。因为是按需编译，可以写一些复杂的变体组合而无需考虑大小问题，比如 `sm:hover:active:disabled:opacity-75`。

可以通过配置 `tailwind.config.js` 文件开启 JIT 模式

```js
// tailwind.config.js
module.exports = {
  mode: 'jit',
  purge: [
    // ...
  ],
  theme: {
    // ...
  }
  // ...
}
```

### 编译过程

_以下代码摘自 [tailwind css v2.2.17 源码](https://github.com/tailwindlabs/tailwindcss/tree/v2.2.17)部分内容。_

TailwindCSS 是基于 PostCSS 开发的。编译首先获取配置信息（用户自定义配置结合 defaultConfig.stub.js 中的默认配置），根据配置决定采用 aot 还是 jit 模式。

```js
// stubs/defaultConfig.stub.js
const colors = require('../colors')

module.exports = {
  purge: [],
  presets: [],
  darkMode: false, // or 'media' or 'class'
  theme: {
    screens: {
      sm: '640px',
      md: '768px',
      lg: '1024px',
      xl: '1280px',
  // ...
```

```js
// src/index.js
module.exports = function tailwindcss(config) {
  const resolvedConfigPath = resolveConfigPath(config)
  const getConfig = getConfigFunction(resolvedConfigPath || config)
  const mode = _.get(getConfig(), 'mode', 'aot')

  if (mode === 'jit') {
    return {
      postcssPlugin: 'tailwindcss',
      plugins: jitPlugins(config),
    }
  }

  // ...

  return {
    postcssPlugin: 'tailwindcss',
    plugins: [...plugins, processTailwindFeatures(getConfig), formatCSS],
  }
}
```

先来看 aot 模式，`processTailwindFeatures` 中定义了装饰符处理、函数处理、工具类生成以及无用代码清理。

```js
// src/processTailwindFeatures.js
export default function (getConfig) {
  return function (css, result) {
    const config = getConfig()

    // ...

    return postcss([
      substituteTailwindAtRules(config, getProcessedPlugins()), // 处理 @tailwind base; @tailwind components; ...
      evaluateTailwindFunctions({ tailwindConfig: config }), // 处理 calc(100vh - theme('spacing.12')); ...
      substituteVariantsAtRules(config, getProcessedPlugins()), // 处理 hove focus ...
      substituteResponsiveAtRules(config), // @responsive ...
      convertLayerAtRulesToControlComments(config), // @layer ...
      substituteScreenAtRules({ tailwindConfig: config }), // @screen
      substituteClassApplyAtRules(config, getProcessedPlugins, configChanged), // @apply
      applyImportantConfiguration(config), // important config
      purgeUnusedStyles(config, configChanged, registerDependency),
    ]).process(css, { from: _.get(css, 'source.input.file') })
  }
}
```

其中工具类的生成方式都定义在 `src/plugins` 目录下，比如 `src/plugins/textAlign.js` 内容如下：

```js
// src/plugins/textAlign.js
export default function () {
  return function ({ addUtilities, variants }) {
    addUtilities(
      {
        '.text-left': { 'text-align': 'left' },
        '.text-center': { 'text-align': 'center' },
        '.text-right': { 'text-align': 'right' },
        '.text-justify': { 'text-align': 'justify' },
      },
      variants('textAlign')
    )
  }
}
```

`purgeUnusedStyles` 的过程主要是正则匹配得到相应字符串数组后交由 PurgeCSS 处理。

```js
// src/lib/purgeUnusedStyles.js
export function tailwindExtractor(content) {
  // Capture as liberally as possible, including things like `h-(screen-1.5)`
  const broadMatches = content.match(/[^<>"'`\s]*[^<>"'`\s:]/g) || []
  
  // ...
}
```

整个大致编译过程就完成了。jit 模式也类似，不同之处在于会添加上下文监听以及去除了 `purgeUnusedStyles` 环节。

### 总结

- Utility-first CSS 的理念是工具类优先，在新增工具类的时候需要仔细斟酌命名保证尽量通用，命名的工具类他人需要熟悉使用有一定成本。
- TailwindCSS 是一种 Utility-first 实现，通过一定规则定义工具类的命名来覆盖 90% 到 95% 的使用场景。
- TailwindCSS 项目随着项目业务的增加，其维护成本和样式体积都是初步放缓的，并且可以实现无用代码消除。
- TailwindCSS 本质上是一款 PostCSS 插件，所以它可以很容易的和其他 PostCSS 插件结合使用比如 [Autoprefixer](https://github.com/postcss/autoprefixer)、[postcss-nesting](https://github.com/csstools/postcss-nesting)

### 参考资料

- [CSS Utility Classes and "Separation of Concerns"](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/)
- [Atomic CSS-in-JS](https://sebastienlorber.com/atomic-css-in-js)
- [Atomic CSS 理念分析，及相关框架 TailWindCSS 学习](https://juejin.cn/post/6970737559429185549)
