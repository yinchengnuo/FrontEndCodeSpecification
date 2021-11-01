## 前言

随着前端技术的发展，前端项目正在变得越来越复杂。JavaScript 作为一门弱类型解释性编程语言，其灵活多变的语法极大的提高了前端开发效率，但同时也存在一系列问题。

好在随着 NodeJs 出现和前端工程化发展，前端社区涌现了很多工具和方案来解决前端开发过程中的各个痛点。代码格式化就是其中之一！

在大型项目开发过程中，代码维护所占的时间比重往往大于新功能的开发。因此编写符合团队编码规范的代码是至关重要的。这样做不仅可以很大程度地避免基本语法错误，也保证了代码的可读性。

对于代码版本管理系统（Git），代码格式不一致带来的问题是严重的，在代码一致的情况下，因为格式不同，触发了版本管理系统标记为 diff，导致无法检查代码和校验。

同时因为 JavaScript 语言本身的问题，我们不希望开发者在前端开发中使用一些 JavaScript 的糟粕特性，尽管这些特性有时候能够出其不意的解决问题。但是对于一个多人协作的大型前端项目，我们需要的是一个统一的可维护的开发环境。这就是我们为什么需要 ESLint、Prettier 的原因。这也是目前前端最主流最全面的代码规范方案。

前端开发的小伙伴一定听说过 ESLint，甚至可以说是天天用。但是几乎都是使用的被各种 CLI 集成好的，于是你可能会有一下问题：

- 什么是 ESLint？
- ESLint 原理是什么？
- 如何配置 ESLint？
- 如何自定义规则？
- 代码编辑器为什么需要 ESLint 插件？
- 项目中为什么需要安装 ESLint Node 包？
- ESLint 在各种 CLI 中是如何集成的？
- 如何配置 GIT Hook 阻止不合格的代码 Commit？
- ESLint 和 Prettier 如何配合？
- 如何保证团队各成员开发环境配置一致？
- 前端代码规范的最佳实践是什么？
- ...

希望可以通过这篇文章，让大家基本弄懂以上问题！

## ESLint

### 什么是 ESLint

ESLint 是一个可以识别代码并按照规则给出报告代码的检测工具。

这种代码检查是一种静态的分析，常用于寻找有问题的模式或者代码，并且不依赖于具体的编码风格。

对大多数编程语言来说都会有代码检查，一般来说编译程序会内置检查工具。但是 JavaScript 是一个动态的弱类型语言，在开发中比较容易出错。同时因为没有编译过程，为了验证 JavaScript 代码错误通常需要在执行过程中不断调试。ESLint 这样的工具可以让程序员在编码的过程中发现问题，而不是在执行的过程中发现问题。

在许多方面，ESLint 和 JSLint、JSHint 相似，不过以下几点除外：

- ESLint 使用 Espree 解析 JavaScript
- ESLint 使用 AST 去分析 JavaScript 代码
- ESLint 是完全插件化的，每一个规则都是一个插件并且你可以在运行时添加更多的规则

因为 ESLint 使用 AST 去分析 JavaScript 代码，因此 ESLint 可以检查出指定规则的 ES 语法问题。和 Babel 类似，ESLint 也是做的语法方面的分析。这就可以让我们实现禁止在 JavaScript 代码中使用某个语法的功能。

### Node 包 ESLint VS 编辑器 ESLint 插件 VS CLI 集成 ESLint

简单的来说：**Node ESLint 及各种集成 ESLint 的 CLI 可以实现在开发运行终端打印出代码规范错误甚至中断运行。而代码编辑器 ESLint 做的只是在编辑器中给不合规范的代码下面展示一条红色的波浪线同时给出提示。两者都通过项目根目录中的 .eslintrc(.js/json...) 或 package.josn 中的 eslintConfig 中的配置规则来规范代码。**

因此，这里强烈推荐在代码编辑器中使用 ESLint 插件作为代码提示，可以极大的提高 ESLint 工作效率。如果没有编辑器中的 ESLint 插件，ESLint 作为项目开发时依赖的 Node 包同样可以工作。例子：

```sh
mkdir eslint  && cd eslint && npm init -y  && yarn add eslint -D
```

进入 eslint 文件夹，新建 index.js 。写入：

```js
const a = 123;
```

终端执行：

```sh
npx eslint index.js
```

会输出：

> /Users/yinchengnuo/Desktop/eslint/index.js
>
> 1:1 error Parsing error: The keyword 'const' is reserved
>
> ✖ 1 problem (1 error, 0 warnings)

是的，报错了。提示 const 这个语法不支持。ESLint 默认支持的 ES 语法为 ES5（ESLint 内置了一套 [lint 规则](https://eslint.bootcss.com/docs/rules/)，无需配置默认生效）。因此不识别 ES6 语法，所以需要配置 ESLint。

在项目根目录新建 .eslintrc。

输入：

```json
{
  "env": {
    "es6": true
  }
}
```

再执行：

```sh
npx eslint index.js
```

此时终端没有任何输出，表示代码通过 ESLint 校验。

如果此时，你的代码编辑器安装了 ESLint 插件。可以清晰的看到在 创建 .eslintrc 之前 const 下面会出现一条红色的波浪线，如果鼠标移动到波浪线之上，可以看到：

> Parsing error: The keyword 'const' is reserved eslint

因此，代码编辑器的 ESLint 插件可以让我们在 ESLint 执行之前就发现代码中的不规范。

那么需要要配置好 .eslintrc 完全依赖代码编辑器的 ESLint 插件，不用在终端执行 npx eslint xxx.js 不是也挺好的？还更方便，这样不好嘛？

**不好**。编辑器的 ESLint 插件仅仅提供了提示功能，即在不规范代码下生成红色下划波浪线。除此之外，什么都做不了。这就意味着有不负责任或水平不够的开发者会忽视这些警告，继续发布代码，造成因代码规范引起的问题。因此我们需要一种方式，在代码中出现不规范时：**不能编译！不能提交！不能构建！**

这也就是为什么 CLI 如 vue-cli 会将 ESLint 集成于其中的原因。vue-cli 会在源代码编译前先执行 npx eslint src/xxx 。如果代码有规范问题，vue-cli 会终止代码编译，首先将不规范信息打印到终端，其次会获取到 ESLint 报错信息通过 devServer HRM 时时更新到浏览器页面上，让开发者看到这些信息。从一定程度上算是强制用户遵循 ESLint 规则，保证代码规范统一。

关于强制要求前端开发者遵循代码规范的实现，我们下面展开，这里主要说的是： Node ESLint & 代码编辑器 ESLint 插件 & cli 中集成 ESLint 的区别。总结来说就是：**编辑器的 ESLint 插件用于从编辑器层面提示代码不规范信息，Node ESLint 包用于配合构建工具在开发环境运行时以及生产构建时抛出错误或提示，同时二者共享 .eslintrc(.js/json...) 配置以达到编辑器中的提示和终端中抛出的错误信息一致。**

### 如何配置 ESLint

上面通过对 .eslintrc 的简单配置实现了简单代码的校验。然而实际上 ESLint 的配置远非这么简单。我们可以通过 eslint --init 指令来窥见端倪：

在终端输入：

```sh
npx eslint --init
```

会看到：

> ? How would you like to use ESLint? …
>
> To check syntax only
>
> To check syntax and find problems
>
> To check syntax, find problems, and enforce code style

选择：**To check syntax, find problems, and enforce code style** => **JavaScript modules (import/export)** => **Vue.js** => **Yes** => **Browser** => **Use a popular style guide** => **Standard: https://github.com/standard/standard** => **JSON** > **Yes**

后项目目录生成了 .eslintrc.json，内容如下：

```json
{
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": ["plugin:vue/essential", "standard"],
  "parserOptions": {
    "ecmaVersion": 13,
    "parser": "@typescript-eslint/parser",
    "sourceType": "module"
  },
  "plugins": ["vue", "@typescript-eslint"],
  "rules": {}
}
```

以上配置就是 Eslint 最常用配置，主要介绍下这几个：

#### env

env 规定了项目运行环境或所依赖的工具库。配置后使用了这些环境或工具的特有 API 不会提示异常。如仅配置 browser 和 es5 为 true。项目中使用 require 和 es6 语法，ESLint 便会校验不通过。全部 env 点 [这里](https://eslint.bootcss.com/docs/user-guide/configuring#specifying-environments)。

#### extends

extends 中包含了 ESLint 的共享机制，让我们已非常低的成本就能用上最好的社区实践。上面的配置中使用了 plugin:vue/essential 和 standard。 standard 是 eslint-config-standard 的简写，extends 属性值可以省略 eslint-config-包名称的前缀。同时因为 eslint --init 初始化使用了 vue 作为开发框架，因此 ESLint 安装了 eslint-plugin-vue 作为 ESLint 规则扩展，plugin:vue/essential 是 plugin:eslint-plugin-vue/essential 的简写。

extends 可以是一个字符串，也可以是一个数组。其中可以包含以下内容：

以 eslint: 开头的字符串，如 eslint:recommended，这样写意味着使用 ESLint 的推荐配置。在 [这里](https://eslint.bootcss.com/docs/rules/) 可以查看其具体有哪些规则。

以 plugin: 开头的字符串，如 "plugin:vue/essential"，这些写意味着应用第三方插件，eslint-plugin-vue 的所有推荐规则，关于 plugin 后文中我们还会讨论。

以 eslint-config-开头的包，这其实是第三方规则的集合，由于 eslint 中添加了额外的处理，我们也可以省略 eslint-config-，如上面的 eslint-config- standard 也可以写作 standard。

最后 extends 可以是一个本地路径，指向本地的 ESLint 配置，如 ./rules/react

extents 中的每一项内容最终都指向了一个和 ESLint 本身配置规则相同的对象。
如果我们在 npm 中搜索 eslint-config- 可以发现大量的 ESLint 拓展配置模块，我们可以直接通过这些模块在 ESLint 中使用上流行的风格，也可以把自己的配置结果封装为一个模块，供之后复用。

#### parserOptions

ESLint 默认使用 Espree 作为 JavaScript 解析器。parserOptions 就是在 Espree 解析 JavaScript 代码时传递给 Espree 的配置项。Espree 默认只能解析 JavaScript 代码，如果要解析 TypeScript 代码，就需要向上面一样设置 parserOptions 的 parser 为 @typescript-eslint/parser。

#### plugins

上面说过，我们可以通过 extends 使用一些第三方配置好的规则，而这些规则都是 ESLint 内置的，同时这些规则仅能校验 JavaScript/TypeScript。对于 j/tsx 和 .vue 文件就无能为力了。因此需要拓展，这就是 [plugin](https://eslint.org/docs/user-guide/configuring/plugins#plugins)。plugin 在运行时要做的事情很多，首先是实现规则，其次是指定解析器解析响应代码（parser、parserOptions），最后是定义规则（rules）。你也可以认为一个 plugin 就是一个 .eslintrc。插件运行时插件 .eslintrc 会被继承到项目 .eslintrc。这就是 plugins 和 extends 的区别。

#### rules

在 rules 中可以配置 extends 规则和 plugins 注入并运行的规则。rules 参数指定的规则可按如下几种方式进行扩展：

规则结果：

- "off"或 0 -关闭规则
- "warn" 或 1 - 开启规则, 使用警告 程序不会退出
- "error"或 2 - 开启规则, 使用错误 程序退出

规则启用方式：

- 启用基础配置中没有规则
- 继承基础配置中的规则，改变其错误级别，但不改变其附加选项：
  - 基础配置: "eqeqeq": ["error", "allow-null"]
  - 扩展配置: "eqeqeq": "warn"
  - 最终有效配置："eqeqeq": ["warn", "allow-null"]
- 覆盖基础配置中的规则:
  - 基础配置："quotes": ["error", "single", "avoid-escape"]
  - 扩展配置："quotes": ["error", "single"]
  - 最终有效配置: "quotes": ["error", "single"]

#### globals

使用未在当前文件中定义的全局变量时，会命中 no-undef 规则，通过 globals 配置指定的全局变量无视 no-undef 规则，[详细](https://eslint.org/docs/user-guide/configuring/language-options#specifying-globals)。

#### 其他配置

ESLint 整体配置较多，这里不一一列举，完整配置见 [这里](https://eslint.org/docs/user-guide/configuring/)。

### 如何使用 ESLint

首先要配置好 .eslintrc。接着安装编辑器插件 ESLint，然后在需要的时候执行 npx eslint "src/" 即可。当环境配置成功（.eslintrc 按照上文 如何配置 ESLint 中 通过 npx eslint --init 生成的配置），如有以下代码：

```js
const a = 123;
```

不用做任何操作，你会在编辑器中看到：

```js
const a = 123;   'a' is assigned a value but never used.

    Too many blank lines at the end of file. Max of 0 allowed
```

执行 npx eslint "src" 就会在终端看到：

> 1:7 error 'a' is assigned a value but never used no-unused-vars
>
> 1:14 error Extra semicolon semi
>
> 2:1 error Too many blank lines at the end of file. Max of 0 allowed no-multiple-empty-line

编辑器的提示，自然是编辑器 ESLint 插件的作用，而终端输出则是 Node ESLint 的作用。我们可以在写代码时利用编辑器插件迅速看到不规范的代码，当保存或提交代码时执行 npx eslint。

各类框架集成 CLI 就是这么做的，如 VUE-CLI。可以在创建项目时选择 ESLint 后选择 Lint on save 还是 Lint and fix on commit 就是设置 VUE-CLI 执行 ESLint 的时机。

VUE-CLI 的配置项目也包含了对 lint 更详细的[配置](https://cli.vuejs.org/zh/config/#lintonsave)。

### ESLint 的问题

ESLint 虽然强大，但其主要作用还是做代码检查。ESLint 的代码检查分为两部分：

- 语法层面，如禁止使用 var、eval 等
- 风格层面，如 tab size、单引号还是双引号、加不加分号 等

默认情况下 ESLint 的语法检查能力仅限于 js 文件，即使是通过 plugin 加持后可以检查 .vue、.j/tsx，ESLint 对 .json、.css 等文件的格式检查就无能为力了。

同时对于有处理能力的文件，通过 eslint --fix 参数可以对不符合 ESLint 规则的代码做有限的格式化，对 .json、.css 等没有检查能力的文件的格式化同样无能为力。

因为开发效率的需要，我们需要一个格式化工具。这个工具要不仅简答、高效，还要足够强大，能处理多种文件。

这个工具就是 Prettier。

## Prettier

### 什么是 Prettier

Prettier 是一个代码格式化工具。

说到代码格式化，就说说到代码风格了。代码风格是每个程序员都要面对的问题。想要提高代码质量和开发效率，首先就要到从代码风格入手。

但现实中却很少看到代码风格管理很好的团队。因为在大多数时候，代码风格起于讨论，也止于讨论，虎头蛇尾有始无终。无法确定一个让所有人都满意的方案，就很难执行下去。就算勉强少数人服从了大多数，但在实际开发过程中还会遇到各种各样的问题。例如不同开发人员用不同的 IDE，用相同 IDE 的又因为设置不同默认的缩进也不同。自己又懒得去设置，或者不会设置，最后就乱了。好不容易实现了功能，每次代码审查前还要手动去修改这些细节的东西，实在头疼。那如何解决呢？

这时候 Prettier 郑重提出：大家不要吵！用这种风格还是那种风格是半斤八两的关系，但是最后用没用上却是 0 和 1 的关系。咱们先提高代码的可读性和可维护性再说，具体什么风格我给你们定。大家都遵循 Prettier 给出的方案就好了，保证一切顺利进行下去。这就是 Prettier 的 opinionated！

Prettier 在自己官网首页列出这么几点：

- An opinionated code formatter
- Supports many languages
- Integrates with most editors
- Has few options

官方首先告诉你，Prettier 是一个 Opinionated 的代码格式化工具。所以要掌握 Prettier 的精髓就是要理解这个单词。

什么是 Opinionated？这个词对于理解框架或技术非常重要。很多框架在文档开始就会告诉你它是 Opinionated 还是 Unopinionated。

例如看 Angular 官方的 Style Guide：

> Looking for an opinionated guide to Angular syntax, conventions, and application structure? Step right in! This style guide presents preferred conventions and, as importantly, explains why.

熟悉前端三大框架的同学应该很容易就能理解：Angular 和 React/Vue 在开发体验上有什么区别？Angular 的 Opinionated，规定了你的代码结构，让你最好按照它给你指定的方式组织代码。React/Vue 则没管这么多，相对比较自由。

Prettier 说自己是一个 Opinionated code formatter，就是说：你必须认同我的观点，按照我说的做。否则你就别用我，硬着头皮用就会处处不爽！

因此，官方说 Perttier Has few options 就很能理解了。

最后 Prettier，有着强大的编辑器和多语言支持，是前端开发代码格式化工具的不二之选。

结合上文 ESLint 的不能很好进行格式化和代码风格统一的问题。因此，主流的前端代码规范方案为：

> #### ESLint 用于语法检查。
>
> #### Prettier 用于代码格式化和风格统一。

### 如何使用 Prettier

和 ESLint 一样，Prettier 也分编辑器插件和 Node 包，同时接受项目目录下的 .prettierrc 作为配置项。例子：

```sh
mkdir prettier  && cd prettier && npm init -y  && yarn add prettier -D
```

进入 prettier 文件夹，新建 index.css 。写入：

```css
.index {
  width:      300px;
      height: 100px      ;
}
```

终端执行：

```sh
npx prettier --write index.css
```

会输出：

> index.css 27ms

此时的代码就会变为：

```css
.index {
  width: 300px;
  height: 100px;
}
```

当然使用编辑器 prettier 插件更简单并能达到同样效果，选择 prettier 格式代码即可。

### 如何配置 Prettier

和 ESLint 不同，.prettierrc 的配置项少的可怜。截止目前 .prettierrc 只有二十来个配置项目，其中也只有约一半是和代码风格相关的。下面简单列举：

|       配置项        |  默认值  |                                   描述                                    |
| :-----------------: | :------: | :-----------------------------------------------------------------------: |
|     printWidth      |    80    |                     单行输出（不折行）的（最大）长度                      |
|      tabWidth       |    2     |                          每一个水平缩进的空格数                           |
|       useTabs       |  false   |                      使用 tab（制表位）缩进而非空格                       |
|        semi         |   true   |                            在语句末尾添加分号                             |
|     singleQuote     |  false   |                    使用单引号而非双引号（jsx 不适用）                     |
|   trailingCommas    |   none   |                       在任何可能的多行中输入尾逗号                        |
|   bracketSpacing    |   true   |         在对象字面量声明所使用的的花括号后（{）和前（}）输出空格          |
| jsxBracketSameLinte |  false   | 在多行 JSX 元素最后一行的末尾添加 > 而使 > 单独一行（不适用于自闭和元素） |
|    alwaysParens     |  avoid   |                      为单行箭头函数的参数添加圆括号                       |
|     rangeStart      |    0     |  指定起止偏移字符(单独指定开始或结束、两者同时指定、分别指定)格式化代码   |
|      rangeEnd       | Infinity |  指定起止偏移字符(单独指定开始或结束、两者同时指定、分别指定)格式化代码   |
|       parser        | babylon  |                           指定使用哪一种解析器                            |
|      filePath       |   none   |                 指定文件的输入路径，这将被用于解析器参照                  |
|         ...         |    -     |                                     -                                     |

## ESLint VS Prettier

通过对 ESLint 和 Prettier 的深入学习。我们得出了前端代码格式化的最佳实践，即：

**使用 ESLint 用于语法检查，使用 Prettier 用于代码格式化和风格统一。**

但是同时又因为 ESLint 语法检查中包含了代码风格相关规则，这就会导致 ESLint 规则和 Prettier 格式化的结果冲突。

在编辑器都安装 ESLint 和 Prettier 的情况下，ESLint 配置为 standard + typescript + vue，Prettier 无配置的情况下。以 Mac 电脑为例，按下 command + shift + F 启动 Prettier 格式化（需设置 Prettier 为编辑器默认格式化工具）后，编辑器会出现大量 ESLint 红色波浪线提示。

以以下代码为例：

```vue
<script lang="ts">
import { defineComponent } from "vue"; // Strings must use singlequote

export default defineComponent({
  setup() {
    // Missing space before function parentheses
    console.log(1); // Extra semicolon
    const a = 1; // Extra semicolon
    const b = 123; // Extra semicolon
    const str = "123"; // Strings must use singlequote
    interface A {
      name: string;
      age: number;
    }
    const test: A = {
      name: "ycn", // Strings must use singlequote
      age: 18, // Unexpected trailing comma
    }; // Extra semicolon
    console.log(a, b, str, test); // Extra semicolon
    return {
      //
    }; // Extra semicolon
  }, // Unexpected trailing comma
}); // Extra semicolon
</script>
```

在终端执行：

```sh
npx eslint "src/views/index/index.vue"
```

输出：

> 36:33 error Strings must use singlequote quotes
>
> 36:38 error Extra semicolon semi
>
> 39:8 error Missing space before function parentheses space-before-function-paren
>
> 40:19 error Extra semicolon semi
>
> 41:16 error Extra semicolon semi
>
> 42:18 error Extra semicolon semi
>
> 43:17 error Strings must use singlequote quotes
>
> 43:22 error Extra semicolon semi
>
> 49:13 error Strings must use singlequote quotes
>
> 50:14 error Unexpected trailing comma comma-dangle
>
> 51:6 error Extra semicolon semi
>
> 52:33 error Extra semicolon semi
>
> 55:6 error Extra semicolon semi
>
> 56:4 error Unexpected trailing comma comma-dangle
>
> 57:3 error Extra semicolon semi

原因很简单，格式化规则冲突。冲突项：

|           冲突项            |                   信息                    |      描述      | ESLint | Prettier |
| :-------------------------: | :---------------------------------------: | :------------: | :----: | :------: |
|            semi             |       Strings must use singlequote        |   字符串引号   | 单引号 |  双引号  |
| space-before-function-paren | Missing space before function parentheses |  函数名后空格  |  必须  |    无    |
|           quotes            |              Extra semicolon              |  语句末尾分号  |  禁止  |   添加   |
|        comma-dangle         |         Unexpected trailing comma         | 数组对象尾逗号 |  禁止  |   添加   |

既然有冲突，那我们就解决冲突。

上面说过：使用 ESLint 用于语法检查，使用 Prettier 用于代码格式化和风格统一。

在此基础上，解决冲突就有了三种方式：

> - 修改 .eslintrc
> - 修改 .prettierrc
> - 同时修改 .eslintrc 和 .prettierrc

#### 解决方案一：只修改 .eslintrc

*第一种*：最直接的方式就是修改 .eslintrc rules。添加：

```json
"space-before-function-paren": ["error", "never"],
"semi": ["error", "always"],
"quotes": ["error", "double"],
"comma-dangle": ["error", "always-multiline"]
```

至 .eslintrc rules 中。编辑器中的报错提示会立即消失，同时执行：

```sh
npx eslint "src/views/index/index.vue"
```

不会有任何报错。

*第二种*：通过 eslint-config-prettier 屏蔽 eslint 报错，而无须添加 rules：

```sh
yarn add eslint-config-prettier -D
```

在 .eslintrc extend 数组中最后一项加入 "prettier" 即可让编辑器中的报错提示立即消失，终端执行 eslint 检查也不会有任何报错。

*解决方案一的问题：*

1. 不同于 ESLint 或 standard 等工具会在文档中明确标注出所用规则，Prettier 没有在官方文档中标注出 prettier 过程中所按照的格式化规则，只有在 Prettier [源码](https://github.com/prettier/eslint-config-prettier/blob/main/index.js)中才能窥知一二。这样就会带来一个问题：如果仅用修改 .eslintrc rules 规则来使 ESLint 放弃对 Prettier 格式化校验，我们会永远弄不清楚 Prettier 规则和 ESLint 规则有多少冲突，从而导致随着开发进行，.eslintrc rules 不断被修改。这不符合我们希望拥有一个统一且稳定系统的初衷。
2. 使用 eslint-config-prettier 可以不用往 .eslintrc rules 中添加任何规则而达到和往 .eslintrc rules 添加规则一样的效果。但是这样同样会有问题：Prettier 只是一个格式化工具，不具备在编辑器中显示错误的功能。eslint-config-prettier 使得 ESLint 放弃了对 Prettier 所有格式化规则的校验，这就使得一些不符合 Prettier 格式化规则的写法不会被代码编辑器提示，这同样不完美。

#### 解决方案二：只修改 .prettierrc

还是以上的代码示例，我们只需在 .prettierrc 写入：

```js
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "none"
}
```

即配置 Prettier 格式化代码的方式按照 ESLint 的规则来。

**解决方案二的问题：**

通过上文得知，Prettier 是一个 Opinionated 的代码格式化工具。自身配置项本来就少，跟代码格式风格相关的更是寥寥无几。因此无法配置 .prettierrc 使 Prettier 完全按照 ESLint 规则来格式化。如上面的例子：space-before-function-paren 这个规则就无法通过 .prettierrc 设置从而遵循 ESLint 规则，导致格式化后的代码中仍有部分代码在编辑器中爆红，无法通过 终端 Lint。

#### 解决方案三：同时修改 .eslintrc 和 .prettierrc

那我们能不能把两者结合起来，风格相关的且 Prettier 支持配置的，通过配置 .prettierrc 设置编辑器插件 Prettier 和 Node 包 Prettier。Prettier 设置之外的规则，通过设置 .eslintrc 来解决。不偏不倚，取中庸之道。如何？

**解决方案三的问题：**

不好。只要涉及到解决方案一。就会有 rules 过多或 eslint-config-prettier 屏蔽编辑器报错的情况发生！

难道真的没有完美的解决方案了嘛？

答案是：**有**。

#### 终极解决方案

对于方案一中 eslint-config-prettier 的问题，作为作者的 Prettier 官方自然也注意到了。如果你细心的话，eslint-config-prettier GitHub README 的第一段就是：

> Runs Prettier as an ESLint rule and reports differences as individual ESLint issues.
>
> If your desired formatting does not match Prettier’s output, you should use a different tool such as prettier-eslint instead.

简单的说，Prettier 官方建议：如果 Prettier 和 eslint-config-prettier 无法完美满足你的格式化需求，建议使用 prettier-eslint。

我们想要的效果是：**无论是代码语法还是代码风格，都只有一种写法。任何第二种写法都会报错，除了编辑器爆红，终端 Lint 也不能通过。**

我们上文深入的学习了 ESLint 和 Prettier 之后发现。Prettier 只是作为一个格式化工具，除此之外不会有任何编辑时和终端运行提示。但是 ESLint 不仅能够在编辑时提示，还能在终端运行时提示，同时**新版的 ESLint 支持的 --fix 参数使得 ESLint 具有了 Prettier 能力**。

**这就是 prettier-eslint 的原理。**

**注意：这里的 prettier-eslint 指的是 编辑器 prettier-eslint 插件，不是 Node 包，同时我们需要卸载编辑器 Prettier 插件，改用 Prettier ESLint 插件，除此之外不需要任何变动。**

还记得这个代码示例嘛：

```vue
<script lang="ts">
import { defineComponent } from "vue"; // Strings must use singlequote

export default defineComponent({
  setup() {
    // Missing space before function parentheses
    console.log(1); // Extra semicolon
    const a = 1; // Extra semicolon
    const b = 123; // Extra semicolon
    const str = "123"; // Strings must use singlequote
    interface A {
      name: string;
      age: number;
    }
    const test: A = {
      name: "ycn", // Strings must use singlequote
      age: 18, // Unexpected trailing comma
    }; // Extra semicolon
    console.log(a, b, str, test); // Extra semicolon
    return {
      //
    }; // Extra semicolon
  }, // Unexpected trailing comma
}); // Extra semicolon
</script>
```

上面代码中的注释就是使用编辑器 Prettier 插件格式化代码后的 ESLint 爆红。

使用 Prettier ESLint 代替 Prettier 格式化后，你会发现上面的爆红全没了。

如果不使用 Prettier ESLint，在终端执行：

```sh
npx prettier --write "src" && npx eslint --fix --ext .js,.ts,.vue "src"
```

可以达到 Prettier ESLint 一样的效果。因此可以认为 Prettier ESLint 就是以上终端执行的封装。

同时 Prettier ESLint 插件依赖项目中安装的 devDependencies 中的 Node 包 ESLint 和 Prettier。如果项目中没有安装 Node 包 ESLint 和 Prettier，Prettier ESLint 将无法工作。

以上，就是 ESLint 配合 Prettier 的最佳实践。

Prettier ESLint 做的就是先使用 prettier 对所有源码中的文件（.vue,.js,.ts,.css,.scss,.json,.md等）用 prettier 的规则进行格式化，此时的代码必然会与 ESLint 有冲突。再调用 ESLint --fix 对 .js,.ts,.vue 等涉及到 ESLint 语法检查的地方用 ESLint 进行格式化。简单的说，**就是凡是 ESLint 严格限制的部分，都用 ESLint 格式化，反之用 Prettier。**

这样，实现了 ESLint 对代码语法和风格的严格控制，同时 ESLint + Prettier 格式化提升开发效率。

于是，.prettierrc 的多数配置项都将变得无用。以为多数 .prettierrc 的配置项格式化的结果都可以通过 eslint --fix 实现。目前已知除了：**printWidth**。

ESLint 主要用于做强约束，而规定单行代码最大长度的 printWidth 并不能被按照 ESLint 的 "max-len" 规则格式化。

因此，我们仍然需要 .prettierrc 及其中的唯一属性 printWidth（默认值：80），如果使用默认值，则无需使用 .prettierrc。

同时为了对此规则做强约束，我们需要在 .eslintrc rules 添加 "max-len": ["error", { "code": printWidth }] 用于约束单行代码最大长度。

## ESLint + Prettier(Prettier ESLint)

通过以上深入学习，我们终于得到了 ESLint + Prettier 配合使用的最佳方案：

> - 编辑器安装插件 ESLint 和 Prettier ESLint，并保持插件 0 设置
> - 设置编辑器 editor.defaultFormatter 为 rvest.vs-code-prettier-eslint（此处为 vscode 配置方式）
> - 项目中 devDependencies 安装 ESLint 及其插件和 Prettier
> - .eslintrc 作为代码语法和风格强约束配置
> - .prettierrc 只需设置 printWidth，同时 printWidth 的值需要设置给 .eslintrc rules "max-len"

不知道你是否有一个问题，就是我们已经通过编辑器插件实现了对代码的语法检查和快速格式化。为什么项目中还需要这两个工具的 Node 包呢？好问题，接着往下看！

## GitHook Lint Code

还记得最开始的问题嘛？

> 那么需要要配置好 .eslintrc 完全依赖代码编辑器的 ESLint 插件，不用在终端执行 npx eslint xxx.js 不是也挺好的？还更方便，这样不好嘛？

**编辑器插件只是保证了及时的代码错误提示和快速的代码格式化，不能保证开发者不把这些不规范的代码提交至代码仓库。**

Node 包 ESLint 和 Prettier 可以实现和编辑器插件同样的效果，只是需要在终端输入命令行。我们平时开发一定不会用这些功能。但是我们可以在开发者 git commit 之前帮助开发者 prettier 一下代码，同时 eslint --fix 一下代码，如果 eslint 不通过，那么就认为代码不合规范，禁止提交。

目前主流的校验方式是使用 Node 包 **Husky + LintStaged。**

### Husky

Husky，没错，就是哈士奇。husky 是一个 Git Hook 工具。作用就是在 git 执行一些操作的时候触发一些钩子，在钩子执行时执行一些自己需要的命令，比如代码的 eslint 代码规范校验、commit message 规范校验、单元测试及覆盖率校验等。这里我们只展开 eslint 代码规范校验。

安装：

```sh
yarn add husky -D
```

初始化 husky：

```sh
npx husky install
```

项目内会出现一个 .husky 文件夹，用于存放 git hook 触发时想要执行的命令。

注册 [pageage.json script Hook](https://www.ruanyifeng.com/blog/2016/10/npm_scripts.html) 用于在项目成员新打开项目安装 Node 包后初始化 husky：

```sh
npm set-script prepare "husky install"
```

设置 prepare 后，package.json 中的 script 中会多出 "prepare": "husky install"。

意为 npm install 之行结束后执行 husky install。

husky install 后生成的 .husky 文件夹建立了 .git 文件夹中 git hook 和我们设置给 husky 的 husky 之间的联系。

初始化之后，我们需要添加一个 husky hook：

```sh
npx husky add .husky/pre-commit "xxx"
```

添加完成后 "xxx" 命令就会在 git commit 之前执行。

那我们直接在添加 husky hook 时候，设置 "xxx" 为：

```sh
npx prettier --write "src" && npx eslint --fix --ext .js,.ts,.vue "src"
```

不久大功告成了。只要上述命令执行报错，那么 git commit 会执行失败。

但是实际上我们需要设置 "xxx" 指令为 "npx lint-staged"

### LintStaged

如果你一次 git commit 只是想提交部分代码，而其余编写中的代码都有格式问题。那么在提交部分代码时，eslint 校验所有代码必然会导致校验失败从而无法 commit。

LintStaged 的作用就是帮 eslint 和 prettier 拿到将要 commit 的文件，而不是所有文件。

安装：

```sh
yarn add lint-staged -D
```

在项目根目录下新建 .lintstagedrc 并写入

```json
{
  "src/**": [
    "npx prettier --write"
  ],
  "src/**/*.{js,ts,vue}": [
    "npx eslint --fix"
  ]
}
```

大功告成。

## 规范之道

当我们在讨论前端代码规范时，我们在讨论什么？

加不加分号？单引号还是双引号？用不用 var ？

诚然，上面说的都属于前端代码规范。但是实际上前端代码规范的本质是：**统一**。

统一的编辑器（设置）、统一的插件、统一的 Node 版本、统一的包管理工具、统一的语法、统一的代码风格...

加分号也好，不加分号也罢。只有在统一规范的基础上，才能高效提升团队效率和水平。

而规范也不仅仅是始于讨论，终于讨论，或是出一份文档草草了之。

需要在制度的基础上，用合适的工具让开发人员强制遵守，同时因为效率的提升而乐于遵守。

## 最佳实践

通过对以上工具链的深入学习和使用，我们对前端代码规范有了初步认识。下面我们通过从零开始配置一个Vite项目的代码格式化，来将以上学习串联起来。

开发环境：

> Node v16.0.0
>
> Yarn v1.22.4
>
> vscode v1.61

打开终端，输入:

```sh
yarn create @vitejs/app
```

操作： 输入项目名 => 输入包名 => vue => vue-ts 即可

选择使用 Vite 是因为 Vite 这个脚手架完全没有对代码格式化做任何配置，完美契合我们的需求。

接着使用 vscode 打开项目。

安装 vscode 插件 ESLint、Prettier ESLint、Vue Language Features (Volar)（vue 3 项目专用编辑器插件）

在项目根目录新建 .vscode => settings.json，并写入

```json
{
  "editor.tabSize": 2,
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "rvest.vs-code-prettier-eslint"
}
```

安装 ESLint，项目根目录执行

```sh
yarn add eslint -D
```

初始化 ESLint：

```sh
npx eslint --init
```

选择：

> **To check syntax, find problems, and enforce code style**
>
>**JavaScript modules (import/export)**
>
>**Vue.js**
>
>**Yes**
>
>**Browser**
>
>**Use a popular style guide**
>
>**Standard: https://github.com/standard/standard**
>
>**JSON**
>
>**Yes**
>

在 .eslintrc.json rules 中写入：

```json
"max-len": ["error", { "code": 120 }]
```

同时在根目录创建 .prettierrc 文件，并写入：

```json
{
  "printWidth": 120
}
```

顺带把 Prettier 也装了：

```sh
yarn add prettier -D
```

下面开始配置 git hook：

```sh
yarn add husky lint-staged -D
```

初始化 husky（须在 .git 文件夹创建后）：

```sh
npx husky install
```

配置 npm script prepare husky 初始化钩子

```sh
npm set-script prepare "husky install"
```

添加 git pre-commit 钩子：

```sh
npx husky add .husky/pre-commit "npx lint-staged"
```

配置 lint-staged，在项目根目录创建 .lintstagedrc，写入：

```json
{
  "src/**/*.{js,ts,vue,scss,json,md}": [
    "npx prettier --write"
  ],
  "src/**/*.{js,ts,vue}": [
    "npx eslint --fix"
  ]
}
```

[代码地址](https://github.com/yinchengnuo/FrontEndCodeSpecification)

完！