---
title: Storybook-typescript重构
date: 2020-12-28 15:23:32
tags:
- 组件
categories: 组件
author: 杨红梅
keywords: storybook  typescript react 组件库 UI组件
description: storybook组件库引入typescript（基于react）
cover: https://img10.360buyimg.com/imagetools/jfs/t1/156730/39/2480/35419/5fe93f39E95a59010/12c3a050227901de.jpg
top_img: https://img10.360buyimg.com/imagetools/jfs/t1/156730/39/2480/35419/5fe93f39E95a59010/12c3a050227901de.jpg
---


TypeScript流行开来，大型项目的开发目前大多都已经加入了这一强类型校验语言，那么我们之前搭建的组件库引入TypeScript就显得势在必行。怎样在已有组件库项目中配置相关内容并且很好兼容之前组件。本文就是我的践行之路，写出来自己的经历，供大家借鉴，指正。


## 背景 React + StoryBook

1.	StoryBook是什么
2.	StoryBook预览结构
3.	StoryBook文件结构
4.	引入TypeScript依赖
5.	修改配置文件config
6.	预览组件
7.	发布组件



## 	StoryBook是什么

1.	Storybook是一个UI工具，组件库，可以让我们的项目开发，更高效，更独立。他可以让你只写组件，而不用开始一个大型项目，开发组件的同时，组件开发完成，只要在项目中引用，便可以快速搭建起一个页面。
2.	Storybook的开发，不依赖项目，就可以本地预览，查看效果，自己调试。独立完成组件的开发。



## 	Storybook预览结构

1.	编辑好的组件，都放在左侧侧边栏上面，当你点击任何一个组件实现预览的时候，Storybook都会在右侧画布里面插入一个iframe：

![](img3.png)

你可以在这个iframe中看到组件的基本Dom结构，包括它生成后的class及内容。
2.	右侧的预览窗口同时提供了工具栏，可以切换工具以查看不同状态下的组件预览格式。![](img1.png)
3.	调整方式包括：放大、缩小、背景色、位置等。
4.	Docs选项显示的是自动生成的组件文档（基于源代码），使用文档在与团队
开发并且共享组件的时候很有用处。
5.	Storybook还提供了可以自定义的工具栏，但是这需要自己手动安装Storybook的插件。
6.	画布下方的controls可以动态的与组件进行数据的交互，相当于我们平常的控制台样式管理打开的模式下，手动的调试一些边缘情况，以查看它的展示效果。
7.	画布下方的Actions，顾名思义，是检查动作的按钮，可以在点击等时间中，查看输出状态。就是控制台的功能了。

![](img2.png)


## Storybook文件结构
``` js
   .storybook/main.js   //storybook入口文件
   .storybook/prwview.js  // 本地预览入口文件
   src/component/mycomponent.js   // 组件js文件
   src/component/mycomponent.css  // 组件样式文件
   src/component/mycomponent.stories.js  // 组件预览文件
   src/index.js// 组件开发入口文件
   index.js  // 入口文件
   package.json  // 配置文件
   babel.config.json   // 配置文件

```


## 引入Storybook依赖

运行的预览效果是在init的时候，会在src/stories下面生成3个demo组件，Button、Page、Header
运行时可能会报错的消息：

命令窗口执行

``` js
    npm install typescript –s
```

安装typescript

``` js
    Npm install @types/react-css-modules
```

安装支持TS的css作用域，支持styleName

``` js
    Npm install @types/react
```


## 修改配置文件config

1.	在babel.config.js中增加typescript配置

```js

return {
    presets: [
      ['@babel/env', { useBuiltIns: 'usage', corejs: 3 }],
      '@babel/react',
      '@babel/typescript'
    ],
    plugins: [
      '@babel/plugin-transform-runtime',
      '@babel/proposal-object-rest-spread',
      '@babel/transform-react-jsx',
      // ["@babel/plugin-proposal-decorators", { "legacy": true }],
      '@babel/proposal-class-properties',
      [
        'babel-plugin-react-css-modules',
        {
          generateScopedName: '[local]___[hash:base64:5]',
          exclude: 'node_modules',
          filetypes: {
            '.scss': {
              syntax: 'postcss-scss',
              plugins: ['postcss-nested']
            }
          }
        }
      ]
    ]
  };

```

2.	增加tsconfig.json文件

```js
{
  "compilerOptions": {
      "outDir": "dist/build/",
      "sourceMap": true,
      "noImplicitAny": true,
      "module": "esnext",
      "target": "esnext",
      "jsx": "react",
      "typeRoots": ["./@types", "./node_modules/@types"], 
      "allowJs": true,
      "allowSyntheticDefaultImports": true,
      "esModuleInterop": true,
      "forceConsistentCasingInFileNames": true,
      "suppressImplicitAnyIndexErrors": true
  },
  "include": ["src"],
  "exclude": [
    "node_modules",
    "dist",
    "config",
    "public"
  ]
}
```


3.	新增.storybook/webpack.common.config.js文件

```js
const path = require('path')
const fs = require('fs')
const appDirectory = fs.realpathSync(process.cwd())
module.exports = {
  plugins: [
    // your custom plugins
  ],
  resolve: {
    extensions: ['', '.js', '.jsx', '.ts', '.tsx']
  },
  module: {
    rules: [
      {
        test: /\.(mjs|jsx?)$/,
        use: [
          {
            loader: require.resolve('babel-loader'),
            options: {
              cacheDirectory: path.join(
                appDirectory,
                '/node_modules/.cache/storybook'
              ),
              babelrc: false,
              plugins: [
                [
                  require.resolve('babel-plugin-react-docgen'),
                  {
                    DOC_GEN_COLLECTION_NAME: 'STORYBOOK_REACT_CLASSES'
                  }
                ]
              ]
            }
          }
        ],
        include: [appDirectory],
        exclude: [path.join(appDirectory, '/node_modules')]
      },
      {
        test: /\.(ts|tsx)$/,
        loader: require.resolve('ts-loader')
      },
      {
        test: /\.md$/,
        use: [
          {
            loader: require.resolve('raw-loader')
          }
        ]
      },
      {
        test: /\.(s*)css$/,
        sideEffects: true,
        use: [
          require.resolve('style-loader'),
          {
            loader: require.resolve('css-loader'),
            options: {
              importLoaders: 1,
              modules: {
                localIdentName: '[local]___[hash:base64:5]'
              }
            }
          },
          // {
          //   loader: 'typings-for-css-modules-loader',
          //   options: {
          //     importLoaders: 1,
          //     modules: true,
          //     namedExport: true,
          //     sass: true,
          //     localIdentName: '[local]___[hash:base64:5]'
          //   }
          // },
          {
            loader: require.resolve('postcss-loader'),
            options: {
              ident: 'postcss',
              postcss: {},
              syntax: 'postcss-scss',
              plugins: () => [
                require('postcss-nested'),
                require('postcss-flexbugs-fixes'),
                require('postcss-preset-env')({
                  autoprefixer: {
                    flexbox: 'no-2009'
                  },
                  stage: 3
                }),
                require('postcss-aspect-ratio-mini'),
                require('postcss-write-svg')({ utf8: false }),
                require('postcss-px-to-viewport')({
                  viewportWidth: 750,
                  viewportHeight: 1334,
                  unitPrecision: 3,
                  viewportUnit: 'vw',
                  selectorBlackList: ['.ignore', '.hairlines'],
                  minPixelValue: 1,
                  mediaQuery: false
                }),
                require('postcss-viewport-units'),
                require('cssnano')({
                  preset: [
                    'advanced',
                    {
                      reduceIdents: false,
                      zindex: false
                    }
                  ],
                  autoprefixer: false,
                  'postcss-zindex': false
                })
              ]
            }
          }
        ]
      },
      {
        test: /\.(svg|ico|jpg|jpeg|png|gif|eot|otf|webp|ttf|woff|woff2|cur|ani)(\?.*)?$/,
        loader: require.resolve('file-loader'),
        query: { name: 'static/media/[name].[hash:8].[ext]' }
      },
      {
        test: /\.(mp4|webm|wav|mp3|m4a|aac|oga)(\?.*)?$/,
        loader: require.resolve('url-loader'),
        query: { limit: 10000, name: 'static/media/[name].[hash:8].[ext]' }
      },
      {
        test: /\.stories\.[j|t]sx?$/,
        loaders: [require.resolve('@storybook/source-loader')],
        enforce: 'pre'
      }
    ]
  }
}
```

4.	修改.storybook/main.js文件

```js
const custom = require('./webpack.common.config.js')
module.exports = {
  // stories: ['../src/**/*.stories.tsx'],
  addons: [
    '@storybook/addon-actions/register',
    '@storybook/addon-knobs/register',
    '@storybook/addon-notes/register-panel',
    '@storybook/addon-events/register',
    '@storybook/addon-cssresources/register',
    '@storybook/addon-storysource',
    '@storybook/addon-links/register',
    '@storybook/addon-backgrounds/register',
    '@storybook/addon-options/register',
    '@storybook/addon-viewport/register',
    '@storybook/addon-a11y/register'
  ],
  webpackFinal: async config => {
    return {
      ...config,
      module: { ...config.module, rules: custom.module.rules }
    }
  }
}
```

5.	修改.storybook/preview.js文件

```js

// automatically import all files ending in *.stories.js
addDecorator(
  withInfo({
    inline: false
  })
);
addDecorator(withCssResources);
addDecorator(centered);
addDecorator(withA11y);

addParameters({
  options: {
    isFullscreen: false,
    showAddonsPanel: true,
    showSearchBox: false,
    panelPosition: 'right',
    theme: create({
      base: 'light',
      brandTitle: 'PAIMAI UI',
      brandUrl: 'https://zpsy.jd.com/',
      gridCellSize: 12
    }),
    hierarchySeparator: /\/|\./,
    hierarchyRootSeparator: /\|/,
    enableShortcuts: true
  },
  viewport: {
    defaultViewport: 'iphonex',
    viewports: INITIAL_VIEWPORTS
  },
  backgrounds: [
    { name: 'white', value: '#fff', default: true },
    { name: 'black', value: '#000' }
  ],
  cssresources: [
    {
      id: `bluetheme`,
      code: `<style>body { background-color: lightblue; }</style>`,
      picked: false
    }
  ]
});

addDecorator((storyFn, context) => withConsole()(storyFn)(context));
setConsoleOptions({
  panelExclude: []
});

function loadStories() {
  const req = require.context('../src/components', true, /.stories.[j|t]sx?/);
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);
```

6.	新增.storybook/tsconfig.json文件

```js
{
  "extends": "../tsconfig",
  "compilerOptions": {
    "jsx": "react",
    "isolatedModules": false,
    "noEmit": false
  }
}
```

7.	修改index.js为index.tsx

```js
declare module 'Button';
declare module 'Header';
declare module 'GoTop';
```

注意：正确按照TS格式书写组件，并且应用style.class实现样式作用域问题，如果要使用ES6语言 import styles from ../index.scss 需要增加.index.css.d.ts文件，以声明变量并导出。
如果使用了   插件，可以自动生成.css.d.ts但是该插件使用要求babel要降级版本，所以我的项目没有使用这个功能，需要手动添加.css.d.ts文件。如果你有好的方法，请留言


## 预览组件

- 执行

```js
npm run storybook 
```

就可以看到组件运行的样子。

## 发布

- 发布到npm之前需要先完善一下README.md文档。同时，根目录下创建一个.npmignore文件。登录自己的npm账号，最后执行npm publish ,发布即可






