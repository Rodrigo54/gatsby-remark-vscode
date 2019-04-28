# gatsby-remark-vscode

A syntax highlighting plugin for [Gatsby](https://www.gatsbyjs.org/) that uses VS Code’s extensions, themes, and highlighting engine. Any language and theme VS Code supports, whether built-in or via a [Marketplace extension](https://marketplace.visualstudio.com/vscode), can be rendered on your Gatsby site.

Includes OS dark mode support 🌙

## Table of contents

- [Getting started](#getting-started)
- [Why gatsby-remark-vscode?](#why-gatsby-remark-vscode)
- [Dark mode support via `prefers-color-scheme`](#dark-mode-support-via-prefers-color-scheme)
- [Built-in languages and themes](#built-in-languages-and-themes)
  - [Languages](#languages)
  - [Themes](#themes)
- [Using languages and themes from an extension](#using-languages-and-themes-from-an-extension)
- [Styles](#styles)
  - [Class names](#class-names)
  - [Variables](#variables)
  - [Tweaking or replacing theme colors](#tweaking-or-replacing-theme-colors)
- [Extra stuff](#extra-stuff)
  - [Line highlighting](#line-highlighting)
  - [Using different themes for different code fences](#using-different-themes-for-different-code-fences)
  - [Arbitrary code fence options](#arbitrary-code-fence-options)

## Why gatsby-remark-vscode?

JavaScript syntax highlighting libraries that were designed to run in the browser, like [Prism](https://www.gatsbyjs.org/packages/gatsby-remark-prismjs/), have to make compromises given the constraints of their intended environment. Since they get downloaded and executed whenever a user visits a page, they have to be ultra-fast and ultra-lightweight. Your Gatsby app, on the other hand, renders to HTML at build-time in Node, so these constraints don’t apply. So why make tradeoffs that don’t buy you anything? There’s no reason why the syntax highlighting on your blog should be any less sophisticated than the syntax highlighting in your code editor. And since VS Code is built with JavaScript and CSS, is open source, and has a rich extension ecosystem, it turns out that it’s pretty easy to hook use its highlighting engine and extensions and get great results. A few examples of where gatsby-remark-vscode excels:

| Scenario                | Others                 | gatsby-remark-vscode |
|-------------------------|------------------------|----------------------|
| Embedded languages      | ![][embedded-others]   | ![][embedded-own]
| Complex TypeScript      | ![][typescript-others] | ![][typescript-own]
| Tricky template strings | ![][templates-others]  | ![][templates-own]
| Uncommon languages      | ![][solidity-others]   | ![][solidity-own]

## Getting started

Install the package:

```bash
npm install --save gatsby-remark-vscode
```

Add to your `gatsby-config.js` (all options are optional; defaults shown here):

```js
{
  // ...
  plugins: [{
    resolve: `gatsby-transformer-remark`,
    options: {
      plugins: [{
        resolve: `gatsby-remark-vscode`,
        // All options are optional. Defaults shown here.
        options: {
          colorTheme: 'Dark+ (default dark)', // Read on for list of included themes. Also accepts object and function forms.
          wrapperClassName: '',  // Additional class put on 'pre' tag
          injectStyles: true,    // Injects (minimal) additional CSS for layout and scrolling
          extensions: [],        // Extensions to download from the marketplace to provide more languages and themes
          languageAliases: {},   // Map of custom/unknown language codes to standard/known language codes
          replaceColor: x => x,  // Function allowing replacement of a theme color with another. Useful for replacing hex colors with CSS variables.
          getLineClassName: ({   // Function allowing dynamic setting of additional class names on individual lines
            content,             //   - the string content of the line
            index,               //   - the zero-based index of the line within the code fence
            language,            //   - the language specified for the code fence
            codeFenceOptions     //   - any options set on the code fence alongside the language (more on this later)
          }) => ''
        }
      }]
```

Write code examples in your markdown file as usual:

    ```js
    this.willBe(highlighted);
    ```

## Dark mode support via `prefers-color-scheme`

Instead of passing a string for `colorTheme`, you can pass an object specifying which theme to use for different values of a user’s operating system color scheme preference.

```js
{
  colorTheme: {
    defaultTheme: 'Solarized Light',    // Required
    prefersDarkTheme: 'Monokai Dimmed', // Optional: used with `prefers-color-scheme: dark`
    prefersLightTheme: 'Quiet Light'    // Optional: used with `prefers-color-scheme: light`
  }
}
```

This places CSS for each theme inside a corresponding `prefers-color-scheme` media query. [See browser support.](#todo-when-not-on-plane)

## Built-in languages and themes

The following can be used without specifying an extension to download from the marketplace:

#### Languages
<details>
  <summary>See all 55 languages</summary>

  - Batch/CMD
  - Clojure
  - CoffeeScript
  - C
  - C++
  - C Platform
  - C#
  - CSS
  - Dockerfile
  - F#
  - Git Commit
  - Git Rebase
  - Diff
  - Ignore
  - Go
  - Groovy
  - Handlebars
  - Hlsl
  - HTML
  - CSHTML
  - PHP HTML
  - INI
  - Java
  - JavaScript
  - JSX
  - JSON
  - JSON with Comments
  - Less
  - Log
  - Lua
  - Makefile
  - Markdown
  - Objective-C
  - Objective-C++
  - Perl
  - Perl 6
  - PHP
  - Powershell
  - Pug
  - Python
  - R
  - Ruby
  - Rust
  - Sass
  - SassDoc
  - ShaderLab
  - Shell
  - SQL
  - Swift
  - TypeScript
  - TSX
  - ASP VB .NET
  - XML
  - XML XSL
  - YAML
</details>

#### Themes
Pro tip: a good way to preview themes is by flipping through them in VS Code. Here’s the list of included ones:

- Abyss
- Dark+ (default dark)
- Light+ (default light)
- Dark (Visual Studio)
- Light (Visual Studio)
- High Contrast
- Kimbie Dark
- Monokai Dimmed
- Monokai
- Quiet Light
- Red
- Solarized Dark
- Solarized Light
- Tomorrow Night Blue

## Using languages and themes from an extension

If you want to use a language or theme not included by default, you can use an extension from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/vscode) that provides that language or theme. `gatsby-remark-vscode` will download it for you; you just need to provide the unique identifier and version of the extension:

![The version and unique identifier can be found in the bottom right corner of the extension’s page on the Visual Studio Marketplace](https://user-images.githubusercontent.com/3277153/56478345-80fb3f00-6463-11e9-84dc-3082d0e6b8f9.png)

Add those strings to the `extensions` option in your plugin configuration in `gatsby-config.js`:

```js
{
  // ...
  plugins: [{
    resolve: `gatsby-transformer-remark`,
    options: {
      plugins: [{
        resolve: `gatsby-remark-vscode`,
        options: {
          extensions: [{
            identifier: 'daltonjorge.scala',
            version: '0.0.5'
          }]
```

Next time you `gatsby develop` or `gatsby build`, the extension will be downloaded and Scala code fences will be highlighted. Extensions are downloaded to `node_modules/gatsby-remark-vscode/lib/extensions`, so they remain cached on disk as long as `gastsby-remark-vscode` does.

## Styles

The CSS for token colors and background colors is generated dynamically from each theme you use and included in the resulting HTML. However, you’ll typically want at least a small amount of additional styling to handle padding and horizontal scrolling. These minimal additional styles are included alongside the dynamically generated token CSS by default, but can be disabled by setting the `injectStyles` option to `false`. If you prefer bundling the styles through your app’s normal asset pipeline, you can simply import the CSS file:

```js
import 'gatsby-remark-vscode/styles.css';
```

### Class names

The generated HTML has ample stable class names, and you can add your own with the `wrapperClassName` and `getLineClassName` option. All (non-token-color) included styles have a single class name’s worth of specificity, so it should be easy to override the built-in styles.

### Variables

The styles also include a few CSS variables you can override. The defaults are:

```css
.vscode-highlight {
  --vscode-highlight-padding-v: 1rem;
  --vscode-highlight-padding-h: 1.5rem;
  --vscode-highlight-padding-top: var(--vscode-highlight-padding-v);
  --vscode-highlight-padding-right: var(--vscode-highlight-padding-h);
  --vscode-highlight-padding-bottom: var(--vscode-highlight-padding-v);
  --vscode-highlight-padding-left: var(--vscode-highlight-padding-h);
  --vscode-highlight-border-radius: 8px;

  /* Line highlighting: see next section */
  --vscode-highlight-line-highlighted-background-color: transparent;
  --vscode-highlight-line-highlighted-border-width: 4px;
  --vscode-highlight-line-highlighted-border-color: transparent;
}
```

The default values are set on `:root`, so you can set them on `.vscode-highlight`, `pre`, your own `wrapperClassName`, the class name matching the theme, or generally any selector more specific than `:root`.

### Tweaking or replacing theme colors

Since the CSS for token colors is auto-generated, it’s fragile and inconvenient to try to override colors by writing more specific CSS. Instead, you can use the `replaceColor` option to replace any value specified by the theme with another valid CSS value. This is especially handy for replacing static colors with variables if you want to support a “dark mode” for your site:

```js
{
  replaceColor: oldColor => ({
    '#ff0000': 'var(--red)',
    '#00ff00': 'var(--green)',
    '#0000ff': 'var(--blue)',
  })[oldColor.toLowerCase()] || oldColor
}
```

## Extra stuff

### Line highlighting

`gatsby-remark-vscode` offers the same line-range-after-language-name strategy of highlighting or emphasizing lines as [gatsby-remark-prismjs](https://github.com/gatsbyjs/gatsby/tree/master/packages/gatsby-remark-prismjs):

    ```js{1,3-5}
    this.isLine(1); // highlighted
    this.isLine(2);
    this.isLine(3); // highlighted
    this.isLine(4); // highlighted
    this.isLine(5); // highlighted
    ```

However, comment directives like `// highlight-line` are not currently supported.

You need to pick your own background color, and optionally a left border width and color, for the highlighted lines. This can be done by setting CSS variables:

```css
.vscode-highlight {
  --vscode-highlight-line-highlighted-background-color: rgba(255, 255, 255, 0.2); /* default: transparent */
  --vscode-highlight-line-highlighted-border-color: rgba(255, 255, 255, 0.5); /* default: transparent */
  --vscode-highlight-line-highlighted-border-width: 2px; /* default: 2px */
}
```

or by setting custom styles on the lines:

```css
.vscode-highlight .vscode-highlight-line-highlighted {
  background-color: rgba(255, 255, 255, 0.2);
  box-shadow: inset 2px 0 0 0 rgba(255, 255, 255, 0.5);
}
```

### Using different themes for different code fences

The `colorTheme` option can take a function instead of a constant value. The function is called once per code fence with information about that code fence, and should return either a string or [an object](#dark-mode-support-via-prefers-color-scheme). See the [following section](#arbitrary-code-fence-options) for an example.

### Arbitrary code fence options

Line numbers and ranges aren’t the only things you can pass as options on your code fence. A JSON-like syntax is supported:

    ```jsx{theme: 'Monokai', someNumbers: {1,2,3}, nested: {objects: 'yep'}}
    <Amazing><Stuff /></Amazing>
    ```

`gatsby-remark-vscode` doesn’t inherently understand these things, but it parses the input and allows you to access it in the `colorTheme` and `getLineClassName` functions:

```js
{
  colorTheme: ({ parsedOptions, language, markdownNode, codeFenceNode }) => {
    // 'language' is 'jsx', in this case
    // 'markdownNode' is the gatsby-transformer-remark GraphQL node
    // 'codeFenceNode' is the Markdown AST node of the current code fence
    // 'parsedOptions' is your parsed object that looks like this:
    // {
    //   theme: 'Monokai',
    //   someNumbers: { '1': true, '2': true, '3': true },
    //   nested: { objects: 'yep' }
    // }
    return parsedOptions.theme || 'Dark+ (default dark)';
  }
}
```

[embedded-others]: https://user-images.githubusercontent.com/3277153/56853797-5debe780-68c8-11e9-91b2-aa651e87a675.png
[embedded-own]: https://user-images.githubusercontent.com/3277153/56853798-5e847e00-68c8-11e9-9eb6-061aa16756ec.png
[typescript-others]: https://user-images.githubusercontent.com/3277153/56853804-5f1d1480-68c8-11e9-965a-bc0adc0e5643.png
[typescript-own]: https://user-images.githubusercontent.com/3277153/56853803-5e847e00-68c8-11e9-9fa6-13a8de51c83d.png
[templates-others]: https://user-images.githubusercontent.com/3277153/56853801-5e847e00-68c8-11e9-9ed6-4a03e187aecd.png
[templates-own]: https://user-images.githubusercontent.com/3277153/56853802-5e847e00-68c8-11e9-8468-dedcd8bcab78.png
[solidity-others]: https://user-images.githubusercontent.com/3277153/56853799-5e847e00-68c8-11e9-8895-535d9e0d555c.png
[solidity-own]: https://user-images.githubusercontent.com/3277153/56853800-5e847e00-68c8-11e9-9c83-5e76146d5e46.png