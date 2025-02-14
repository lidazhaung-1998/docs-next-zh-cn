---
badges:
  - breaking
---

# 自定义元素交互 <MigrationBadges :badges="$frontmatter.badges" />

## 概览

- **非兼容**：检测并确定哪些标签应该被视为自定义元素的过程，现在会在模板编译期间执行，且应该通过编译器选项而不是运行时配置来配置。
- **非兼容**：特定 `is` prop 用法仅限于保留的 `<component>` 标记。
- **新增**：为了支持 2.x 在原生元素上使用 `is` 的用例来处理原生 HTML 解析限制，我们用 `vue:` 前缀来解析一个 Vue 组件。

## 自主定制元素

如果我们想在 Vue 外部定义添加自定义元素 (例如使用 Web 组件 API)，我们需要“指示”Vue 将其视为自定义元素。让我们以下面的模板为例。

```html
<plastic-button></plastic-button>
```

### 2.x 语法

在 Vue 2.x 中，通过 `Vue.config.ignoredElements` 配置自定义元素：

```js
// 这将使Vue忽略在Vue外部定义的自定义元素
// (例如：使用 Web Components API)

Vue.config.ignoredElements = ['plastic-button']
```

### 3.x 语法

**在 Vue 3.0 中，此检查在模板编译期间执行**指示编译器将 `<plastic-button>` 视为自定义元素：

- 如果使用生成步骤：将 `isCustomElement` 传递给 Vue 模板编译器，如果使用 `vue-loader`，则应通过 `vue-loader` 的 `compilerOptions` 选项传递：

  ```js
  // webpack 中的配置
  rules: [
    {
      test: /\.vue$/,
      use: 'vue-loader',
      options: {
        compilerOptions: {
          isCustomElement: tag => tag === 'plastic-button'
        }
      }
    }
    // ...
  ]
  ```

- 如果使用动态模板编译，请通过 `app.config.isCustomElement` 传递：

  ```js
  const app = Vue.createApp({})
  app.config.isCustomElement = tag => tag === 'plastic-button'
  ```

  需要注意的是，运行时配置只会影响运行时模板编译——它不会影响预编译的模板。

## 定制内置元素

自定义元素规范提供了一种将自定义元素作为[自定义内置模板](https://html.spec.whatwg.org/multipage/custom-elements.html#custom-elements-customized-builtin-example)的方法，方法是向内置元素添加 `is` 属性：

```html
<button is="plastic-button">点击我!</button>
```

Vue 对 `is` 特殊 prop 的使用是在模拟 native attribute 在浏览器中普遍可用之前的作用。但是，在 2.x 中，它被解释为渲染一个名为 `plastic-button` 的 Vue 组件，这将阻止上面提到的自定义内置元素的原生使用。

在 3.0 中，我们仅将 Vue 对 `is` 属性的特殊处理限制到 `<component>` tag。

- 在保留的 `<component>` tag 上使用时，它的行为将与 2.x 中完全相同；
- 在普通组件上使用时，它的行为将类似于普通 prop：

  ```html
  <foo is="bar" />
  ```

  - 2.x 行为：渲染 `bar` 组件。
  - 3.x 行为：通过 `is` prop 渲染 `foo` 组件。

- 在普通元素上使用时，它将作为 `is` prop 传递给 `createElement` 调用，并作为原生 attribute 渲染，这支持使用自定义的内置元素。

  ```html
  <button is="plastic-button">点击我！</button>
  ```

  - 2.x 行为：渲染 `plastic-button` 组件。
  - 3.x 行为：通过回调渲染原生的 button。

    ```js
    document.createElement('button', { is: 'plastic-button' })
    ```

[迁移构建开关：`COMPILER_IS_ON_ELEMENT`](migration-build.html#兼容性配置)

## `vue:` 用于 DOM 内模板解析解决方案

> 提示：本节仅影响直接在页面的 HTML 中写入 Vue 模板的情况。
> 在 DOM 模板中使用时，模板受原生 HTML 解析规则的约束。一些 HTML 元素，例如 `<ul>`，`<ol>`，`<table>` 和 `<select>` 对它们内部可以出现的元素有限制，和一些像 `<li>`，`<tr>`，和 `<option>` 只能出现在某些其他元素中。

### 2.x 语法

在 Vue 2 中，我们建议在原生 tag 上使用 `is` prop 来解决这些限制：

```html
<table>
  <tr is="blog-post-row"></tr>
</table>
```

### 3.x 语法

随着 `is` 的行为变化，现在将元素解析为一个 Vue 组件需要 `vue:` 前缀：

```html
<table>
  <tr is="vue:blog-post-row"></tr>
</table>
```

## 迁移策略

- 替换 `config.ignoredElements` 与 `vue-loader` 的 `compilerOptions` (使用 build 步骤) 或 `app.config.isCustomElement` (使用动态模板编译)

- 将所有非 `<component>` tags 与 `is` 用法更改为 `<component is="...">` (对于 SFC 模板) 或 `vue:` 前缀 (对于 DOM 模板)。
