# Typescript Starter

## 创建文件（夹）

```bash
mkdir typescript-starter && cd typescript-starter
git init && echo "# Typescript Starter" >> README.md
```



## 配置 Yarn

**创建文件（夹）**

```
touch .yarnrc
```

**编辑 `.yarnrc`**

```
registry "https://registry.npm.taobao.org"

sass_binary_site "https://npm.taobao.org/mirrors/node-sass/"
phantomjs_cdnurl "http://cnpmjs.org/downloads"
electron_mirror "https://npm.taobao.org/mirrors/electron/"
sqlite3_binary_host_mirror "https://foxgis.oss-cn-shanghai.aliyuncs.com/"
profiler_binary_host_mirror "https://npm.taobao.org/mirrors/node-inspector/"
chromedriver_cdnurl "https://cdn.npm.taobao.org/dist/chromedriver"
```

**初始化项目**

```bash
yarn init
```

## 配置 Git Hooks

------

> 通过配置 Git Hooks 可以运行一些自定义操作，这里主要通过`commit-msg`钩子对 commit message 进行校验，以规范化提交信息。
>
> 主要用到以下库：
>
> - `husky` 用于实现各种 Git Hooks。
>
> 当然，还存在现场的工具库可用于规范化提交信息，这里为方便高度地自定义提交信息的规范而手写了一个简单 commit message 校验脚本。

**安装依赖库**

```bash
yarn add husky lint-staged --dev
# 或者
npm install husky lint-staged -d
```

**创建建 scripts 文件（夹）**

```bash
mkdir scripts && touch scripts/verify-commit-msg.js
```

**编辑 `scripts/verify-commit-msg.js`**

```js
#!/usr/bin/env node

/**
 * This is a commit-msg sample running in the Node environment,
 *    and will be invoked on the commit-msg git hook.
 * 
 * You can use it by renaming it to `commit-msg` (without path extension),
 *    and then copying the renamed file to your project's directory `.git/hooks/`.
 * 
 * Note: To ensure it can be run, you should grunt the renamed file (`commit-msg`) 
 *    with running command `chmod a+x .git/hooks/commit-msg` in your project's directory.
 */
const chalk = require('chalk')
const message = require('fs')
  .readFileSync(process.argv[2], 'utf-8')
  .trim()

const COMMIT_REG = /^(revert: )?(work|feat|fix|docs|style|refactor|perf|test|workflow|build|ci|chore|release)(\(.+\))?: .{1,50}/

if (!COMMIT_REG.test(message)) {
  console.log()
  console.error(
      `  ${chalk.bgRed.white(' ERROR ')} ${chalk.red(`invalid commit message format.`)}\n\n` 
      + chalk.red(`  Proper commit message format is required for automated changelog generation. Examples:\n\n`) 
      + `    ${chalk.green(`ffeat(pencil): add 'graphiteWidth' option`)}\n` 
      + `    ${chalk.green(`fix(graphite): stop graphite breaking when width < 0.1 (close #28)`)}\n\n` 
      + chalk.red(`  See .github/commit-convention.md for more details.\n`)
  )
  process.exit(1)
}
```

> 上述脚本运行在 node 环境下，主要利用正则表达试对提交信息进行校验。
>
> 上述脚本内容直接拷贝到`.git/hooks/commit-msg`文件中也可成功运行（需要先创建`.git/hooks/commit-msg`文件并利用命令`chmod a+x .git/hooks/commit-msg`赋予可执行权限）。
>
> 更多 commit message 规范细节详见: https://github.com/marktex/commit-convention。

**编辑 `package.json`**

```json
{
    // ...,
    "hooks": {
        "pre-commit": "lint-staged",
        "commit-msg": "node scripts/verify-commit-msg.js ${HUSKY_GIT_PARAMS}"
    },
    "lint-staged": {},
    // ...
}
```

> 注：也可以直接将 `.huskyrc` 文件中的内容写到 `package.json` 文件中的 "husky" 属性下。

**验证测试**

```bash
git add .
git commit -m "test" # 提交信息不符合规范，正常情况下会报错
git commit -m "chore: use husky, lint-staged"  # 提交信息符合规范，正常提交
```

## 配置 EditorConfig

------

> “EditorConfig 帮助开发人员在不同的编辑器和IDE之间定义和维护一致的编码样式。
>
> EditorConfig项目由用于定义编码样式**的文件格式**和一组**文本编辑器插件组成**，这些**插件**使编辑器能够读取文件格式并遵循定义的样式。
>
> EditorConfig文件易于阅读，并且与版本控制系统配合使用。
>
> 对于VS Core，对应的插件名是[EditorConfig for VS Code](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)

**创建文件（夹）**

```bash
touch .editorconfig
```

**编辑`.editorconfig`**

```
    # EditorConfig is awesome: https://EditorConfig.org

    # top-most EditorConfig file
    root = true

    # Unix-style newlines with a newline ending every file
    [*]
    end_of_line = lf
    insert_final_newline = true
    charset = utf-8
    indent_style = space
    indent_size = 4

    [{*.json,*.md,*.yml,*.*rc}]
    indent_style = space
    indent_size = 2
```

## 配置 Prettier

------

> Prettier 是格式化代码工具，用来保持团队的项目风格统一。

**安装依赖库**

```bash
yarn add --dev prettier
# 或者
npm install -d prettier
```

**创建文件（夹）**

```bash
touch .prettierrc.yml .prettierignore
```

**编辑 `.prettierrc.yml`**

```yaml
trailingComma: "all"
tabWidth: 4
semi: false
singleQuote: true
endOfLine: "lf"
printWidth: 120
overrides:
  - files: ["*.md", "*.json", "*.yml", "*.yaml"]
    options:
      tabWidth: 2
```

**编辑 `.prettierignore`**

```
# Ignore artifacts:
build
coverage
```

**编辑 `package.json`**

```json
{
    // ...,
    "husky": {
        "hooks": {
            "pre-commit": "lint-staged"
        }
    },
    "lint-staged": {
        "**/*": "prettier --write --ignore-unknown"
    },
    // ...
}
```

## 配置 ESLint

------

> TypeScirpt 已经全面采用 ESLint 作为代码检查 [The future of TypeScript on ESLint](https://eslint.org/blog/2019/01/future-typescript-eslint)，并且提供了 TypeScript 文件的解析器 和配置选项 [@typescript-eslint/eslint-plugin](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/eslint-plugin)

**安装依赖库**

```bash
yarn add --dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
# 或者
npm install -d eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

**创建文件（夹）**

```bash
yarn eslint --init # 初始化 eslint 配置文件，或者 `npx eslint --init`
touch tsconfig.eslint.json
```

> `tsconfig.eslint.json` 将用于 `eslintrc.parserOptions.project` ，由于该配置要求 include 每个 `*.ts`、`*.js` 文件，而这里仅需要打包 `src` 目录下的代码，所以增加了该配置文件。
>
> 如果 `eslintrc.parserOptions.project` 配置为 `tsconfig.json` ，则 `src` 目录以外的 `*.ts`、`*.js` 文件都会报错：
>
> ```
> Parsing error: "parserOptions.project" has been set for @typescript-eslint/parser.  
> The file does not match your project config: config.ts.  
> The file must be included in at least one of the projects provided.eslint  
> ```
>
> 虽然可以配置 `eslintrc.parserOptions.createDefaultProgram` 但会造成巨大的性能损耗。
>
> [issus: Parsing error: "parserOptions.project"...](https://github.com/typescript-eslint/typescript-eslint/issues/967)

**编辑 `tsconfig.eslint.json`**

```json
{
    "compilerOptions": {
        "baseUrl": "."
    },
    "include": ["**/*.ts?(x)", "**/*.js?(x)"]
}
```

**编辑 `.eslintrc.js`**

```js
module.exports = {
    /* Specify which environments you want to enable. An environment defines global variables that are predefined.*/
    env: {
        browser: true,
        es2021: true,           /* enables ES12 */
        node: true,
    },
    /* Extend the set of enabled rules from base configurations. */
    extends: ["eslint:recommended", "plugin:@typescript-eslint/recommended"],
    /* Specify parser. */
    parser: "@typescript-eslint/parser",
    /* Specify the language options you want to support. */
    parserOptions: {
        project: "./tsconfig.eslint.json",
        ecmaVersion: 2021,                          /* Specify the version of ECMAScript syntax by setting to 2015 (also 6), 2016 (also 7), 2017 (also 8), etc. */
        sourceType: "module",                       /* Set to "script" (default) or "module" if your code is in ECMAScript modules. */
        /* An object indicating which additional language features you'd like to use. */
        ecmaFeatures: {
            // globalReturn: true,                  /* Allow return statements in the global scope */
            // impliedStrict: true,                 /* Enable global strict mode (if ecmaVersion is 5 or greater) */
            // jsx: true,                           /* Enable JSX */
            experimentalObjectRestSpread: true,
        },
    },
    /* Configure plugins. */
    plugins: ["@typescript-eslint"],
    /**
     * Configure rules.
     * To change a rule setting, you must set the rule ID equal to one of these values:
     * "off" or 0 - turn the rule off
     * "warn" or 1 - turn the rule on as a warning (doesn't affect exit code)
     * "error" or 2 - turn the rule on as an error (exit code is 1 when triggered)
     */
    rules: {
        /* Enable additional rules. */
        indent: ["error", 4],
        "linebreak-style": ["error", "unix"],
        quotes: ["error", "double"],
        semi: ["error", "always"],
        
        /* Override default options for rules from base configurations. */
        // "comma-dangle": ["error", "always"],
        // "no-cond-assign": ["error", "always"],

        /* Disable rules from base configurations. */
        // "no-console": "off",
    },
};
```

**编辑 `package.json`**

```json
{
    // ...,
    "scripts": {
        // ...,
        "lint": "eslint",
        // ...
    },
    // ...,
    "lint-staged": {
        // ...,
        "**/*.ts?(x)": [
            "eslint"
        ]
        // ...,
    },
    // ...
}
```

## 配置 [TypeScript](https://www.tslang.cn/docs/home.html)

------

**安装依赖库**

```bash
yarn add --dev typescript 
# 或者
npm install -d typescript
```

**创建文件（夹）**

```bash
mkdir src && touch src/index.ts
yarn tsc --init # 初始化配置文件，或者 `npx tsc --init`
```

**编辑 `tsconfig.json`**

```json
{
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig.json to read more about this file */

    /* Basic Options */
    // "incremental": true,                   /* Enable incremental compilation */
    "target": "esnext",                       /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', or 'ESNEXT'. */
    "module": "esnext",                  /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', 'es2020', or 'ESNext'. */
    "lib": ["dom", "esnext"],                 /* Specify library files to be included in the compilation. */
    // "allowJs": true,                       /* Allow javascript files to be compiled. */
    // "checkJs": true,                       /* Report errors in .js files. */
    // "jsx": "preserve",                     /* Specify JSX code generation: 'preserve', 'react-native', or 'react'. */
    "declaration": true,                      /* Generates corresponding '.d.ts' file. */
    // "declarationMap": true,                /* Generates a sourcemap for each corresponding '.d.ts' file. */
    "sourceMap": true,                        /* Generates corresponding '.map' file. */
    // "outFile": "./",                       /* Concatenate and emit output to single file. */
    "outDir": "./lib",                        /* Redirect output structure to the directory. */
    // "rootDir": "./",                       /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */
    // "composite": true,                     /* Enable project compilation */
    // "tsBuildInfoFile": "./",               /* Specify file to store incremental compilation information */
    // "removeComments": true,                /* Do not emit comments to output. */
    // "noEmit": true,                        /* Do not emit outputs. */
    // "importHelpers": true,                 /* Import emit helpers from 'tslib'. */
    // "downlevelIteration": true,            /* Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'. */
    // "isolatedModules": true,               /* Transpile each file as a separate module (similar to 'ts.transpileModule'). */

    /* Strict Type-Checking Options */
    "strict": true,                           /* Enable all strict type-checking options. */
    // "noImplicitAny": true,                 /* Raise error on expressions and declarations with an implied 'any' type. */
    // "strictNullChecks": true,              /* Enable strict null checks. */
    // "strictFunctionTypes": true,           /* Enable strict checking of function types. */
    // "strictBindCallApply": true,           /* Enable strict 'bind', 'call', and 'apply' methods on functions. */
    // "strictPropertyInitialization": true,  /* Enable strict checking of property initialization in classes. */
    // "noImplicitThis": true,                /* Raise error on 'this' expressions with an implied 'any' type. */
    // "alwaysStrict": true,                  /* Parse in strict mode and emit "use strict" for each source file. */

    /* Additional Checks */
    // "noUnusedLocals": true,                /* Report errors on unused locals. */
    // "noUnusedParameters": true,            /* Report errors on unused parameters. */
    // "noImplicitReturns": true,             /* Report error when not all code paths in function return a value. */
    // "noFallthroughCasesInSwitch": true,    /* Report errors for fallthrough cases in switch statement. */

    /* Module Resolution Options */
    "moduleResolution": "node",              /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */
    // "baseUrl": "./",                       /* Base directory to resolve non-absolute module names. */
    // "paths": {},                           /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */
    // "rootDirs": [],                        /* List of root folders whose combined content represents the structure of the project at runtime. */
    // "typeRoots": [],                       /* List of folders to include type definitions from. */
    // "types": [],                           /* Type declaration files to be included in compilation. */
    // "allowSyntheticDefaultImports": true,  /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
    "esModuleInterop": true,                  /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
    // "preserveSymlinks": true,              /* Do not resolve the real path of symlinks. */
    // "allowUmdGlobalAccess": true,          /* Allow accessing UMD globals from modules. */

    /* Source Map Options */
    // "sourceRoot": "",                      /* Specify the location where debugger should locate TypeScript files instead of source locations. */
    // "mapRoot": "",                         /* Specify the location where debugger should locate map files instead of generated locations. */
    // "inlineSourceMap": true,               /* Emit a single file with source maps instead of having a separate file. */
    // "inlineSources": true,                 /* Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set. */

    /* Experimental Options */
    "experimentalDecorators": true,           /* Enables experimental support for ES7 decorators. */
    // "emitDecoratorMetadata": true,         /* Enables experimental support for emitting type metadata for decorators. */

    /* Advanced Options */
    "skipLibCheck": true,                     /* Skip type checking of declaration files. */
    "forceConsistentCasingInFileNames": true  /* Disallow inconsistently-cased references to the same file. */
  },
  "include": ["src"]
}
```

## 配置 Rollup

------

> Vue、React 等许多流行库都在使用 Rollup 进行，这里就不在过多介绍，详细信息见 [官网](https://www.rollupjs.com/) 。

**安装依赖库**

```bash
# 安装 rollup 以及要用到的插件
yarn add --dev rollup rollup-plugin-babel rollup-plugin-commonjs rollup-plugin-eslint rollup-plugin-node-resolve rollup-plugin-typescript2 rollup-plugin-json
# 安装 babel 相关的库
yarn add --dev @babel/core @babel/preset-env

# 或者
npm install -d rollup rollup-plugin-babel rollup-plugin-commonjs rollup-plugin-eslint rollup-plugin-node-resolve rollup-plugin-typescript2 
npm install -d @babel/core @babel/preset-env 
```

**创建文件（夹）**

```bash
touch rollup.config.js
touch src/index.ts
```

**编辑 `rollup.config.js`**

```js
import path from "path";
import babel from "rollup-plugin-babel";
import rollupTypescript from "rollup-plugin-typescript2";
import commonjs from "rollup-plugin-commonjs";
import resolve from "rollup-plugin-node-resolve";
import json from 'rollup-plugin-json';

import { name } from "./package.json";

const paths = [
    path.join(__dirname, "/src/index.ts") /* input file(s) path */,
    path.join(__dirname, "/lib") /* output file(s) path */,
];

/* Rollup Configures */
export default {
    /**
     * Specify the bundle's entry point(s) .
     * If you provide an array of entry points or an object mapping names to entry points,
     *    they will be bundled to separate output chunks.
     * PS: it is possible when using the object form to put entry points into different sub-folders by adding a / to the name.
     */
    input: paths[0],
    output: [
        /* 输出 CommonJS 规范的代码 */
        {
            file: path.join(paths[1], "index.js") /* Specify where the file write to. */,
            /**
             * Specify the format of the generated bundle. One of the following:
             * amd – Asynchronous Module Definition, used with module loaders like RequireJS
             * cjs – CommonJS, suitable for Node and other bundlers (alias: commonjs)
             * esm – Keep the bundle as an ES module file,
             *      suitable for other bundlers and inclusion as a <script type=module> tag in modern browsers (alias: esm, module)
             * iife – A self-executing function, suitable for inclusion as a <script> tag.
             *      (If you want to create a bundle for your application, you probably want to use this.)
             * umd – Universal Module Definition, works as amd, cjs and iife all in one
             * system – Native format of the SystemJS loader (alias: systemjs)
             */
            format: "cjs",
            name /* Specify the variable name can be used to access the exports of bundle. */,
        },
        /* 输出 ES module 规范的代码 */
        {
            file: path.join(paths[1], "index.esm.js"),
            format: "esm",
            name,
        },
    ],
    // external: ['lodash'], /* Specify which modules should be treated as external. */
    /**
     * For plugins imported from packages, remember to call the imported plugin function (i.e. commonjs(), not just commonjs).
     */
    plugins: [
        /* Convert CommonJS to ES2015 in order to Rollup can process them. */
        commonjs(),

        /* Teaches Rollup how to find external modules. */
        resolve({
            /* Specify custom options to the resolve plugin. */
            customResolveOptions: {
                moduleDirectory: "node_modules",
            },
        }),

        /* Convert Typescript to javascript. */
        rollupTypescript(),

        /* Use Babel in order to use the latest JavaScript features that aren't yet supported by browsers and Node.js. */
        babel({
            babelrc: false /* Specify whether allow Rollup to bypass `.babelrc` file. */,
            presets: [
                [
                    "@babel/preset-env",
                    {
                        /**
                         * Not needed for Babel 7 - it knows automatically that Rollup understands ES modules & that it shouldn't use any module transform with it.
                         * For Babel <6.13, the env preset includes the transform-es2015-modules-commonjs plugin,
                         *    which converts ES6 modules to CommonJS – preventing Rollup from working.
                         * PS: Setting `modules: false` in `.babelrc` which may conflict, to work around this, should specify `babelrc: false` in rollup config
                         */
                        modules: false,
                    },
                ],
            ],
            // externalHelpers: true, /* Specify whether to bundle in the Babel helpers (by default, set to false so babel helpers will be included in your bundle.) */
            runtimeHelpers: true /*  Specify whether use the transform-runtime plugin. */,
            exclude:
                "node_modules/**" /* Specify which files should not by transpiled by Babel (by default, all files are transpiled.) */,
            extensions: [".ts", ".js", ".jsx", ".es6", ".es", ".mjs"], /* Specify file extensions that Babel should transpile (by default, .js, .jsx, .es6, .es, .mjs are used.) */
        }),

        /* Allow Rollup to import data from a JSON file. */
        json(),
    ],
};
```

**编辑 `src/index.ts`**

```js
export function add(a: number, b: number){
    return a + b;
}

export function minus(a: number, b: number){
    return a - b;
}
```

**编辑`package.json`**

```json
{
    // ...,
    "scripts": {
        "build": "rollup -c"
    },
    // ...,
    "babel":{
        "presets": [
            [
                "@babel/preset-env",
                {
                    "targets": {
                        "node": "current"
                    }
                }
            ]
        ]
    }
}
```

**验证测试**

```bash
yarn build # 或者 npx build
# 或者
yarn rollup -c # 或者 npx rollup -c 
```

一切顺利的话会在`lib/`文件夹下生成 `index.js` 和 `index.esm.js` 文件，分别对应着 commonjs 规范和 es 规范的文件。

> Rollup 虽然大力推行 es 规范，然而很多三方库都仍旧使用 commonjs 规范，为了兼容可以两种规范都生成。

## 配置 API Extractor

------

> 当 src 目录下有多个文件时，打包后会生成多个声明文件，使用 `@microsoft/api-extractor` 这个库可以把所有的 `*.d.ts` 文件合成一个，并且还可以根据写的注释自动生成文档。

**安装依赖库**

```bash
yarn add --dev @microsoft/api-extractor
# 或者
npm install -d @microsoft/api-extractor
```

**创建文件（夹）**

```bash
yarn api-extractor init # 初始化配置文件，或者 `npm api-extractor init`
```

**编辑 `api-extractor.json`**

```json
{
    "$schema": "https://developer.microsoft.com/json-schemas/api-extractor/v7/api-extractor.schema.json",
    "mainEntryPointFilePath": "<projectFolder>/lib/index.d.ts",

    "apiReport": {
        "enabled": true,
        "reportFolder": "<projectFolder>/temp/",
    },

    "docModel": {
        "enabled": true
    },
    
    "dtsRollup": {
        "enabled": true,
        "untrimmedFilePath": "<projectFolder>/dist/<unscopedPackageName>.d.ts",
    }
}
```

**编辑 `package.json`**

```json
{
    // ...,
    "scripts":{
        // ...,
        "api": "api-extractor run",
        // ...
    },
    // ...
}
```

**编辑 `src/index.ts`**

> 加入[ts doc](https://github.com/microsoft/tsdoc) 风格注释

```ts
/**
 * 加法运算
 * @param a - 加数
 * @param b - 被加数
 * @returns the result of a plus b
 * @example
 * ```ts
 * minus(1, 2) => 2
 * ```
 * 
 * @beta
 */
export function add(a: number, b: number) {
    return a + b;
}

/**
 * 减法运算
 * @param a - 减数
 * @param b - 被减数
 * @returns the result of a minus b
 * @example
 * ```ts
 * minus(3, 2) => 1
 * ```
 * 
 * @beta
 */
export function minus(a: number, b: number) {
    return a - b;
}
```

**验证测验**

```bash
yarn build # 可以尝试多写几个方法文件，然后打包后会发现有多个 `.d.ts ` 文件
yarn api
```

> 注意： 这里会生成一个 temp 文件夹，需配置一下`.gitignore` 不然它将被提交提交。
>
> ```
> echo "temp/" >> .gitignore
> ```

## 配置 Jest

------

**安装依赖库**

```bash
yarn add --dev @types/jest jest ts-jest
# 或者
npm install -d @types/jest jest ts-jest 
```

**创建文件（夹）**

```bash
touch src/foo.ts
mkdir test && touch test/foo.spec.ts

yarn ts-jest config:init # 初始化配置文件
# 或者
npx ts-jest config:init
```

**编辑 `package.json`**

```json
{
    // ...,
    "jest":{
        "verbose": true, // 层次显示测试套件中每个测试的结果
        "coverage": true, // 输出测试覆盖率
        "preset": "ts-jest",
        "testEnvironment": "node"
    },
    // ...
}
```

**编辑 `src/foo.ts`**

```ts
export const foo = {
    a: {
        b: {
            c: {
                hello: (name: string) => `Hello, ${name}`,
            },
        },
    },
    name: () => "foo",
};
```

**编辑 `test/foo.spec.ts`**

```ts
import { mocked } from "ts-jest/utils";
import { foo } from "../src/foo";
jest.mock("../src/foo");

// here the whole foo var is mocked deeply
const mockedFoo = mocked(foo, true);

test("deep", () => {
    // there will be no TS error here, and you'll have completion in modern IDEs
    mockedFoo.a.b.c.hello("me");
    // same here
    expect(mockedFoo.a.b.c.hello.mock.calls).toHaveLength(1);
});

test("direct", () => {
    foo.name();
    // here only foo.name is mocked (or its methods if it's an object)
    expect(mocked(foo.name).mock.calls).toHaveLength(1);
});
```

**编辑 `package.json`**

```json
{
    // ...,
    "script":{
        // ...,
        "test": "jest -u",
        // ...
    },
    "hooks": {
        "pre-commit": "lint-staged && jest -u",
        // ...
    },
}
```

> - `--coverage` 输出测试覆盖率
> - `--verbose` 层次显示测试套件中每个测试的结果，会看着更加直观啦

**验证测试**

```
yarn test  
```

> 注意： 这里会生成一个 coverage 文件夹，需配置一下`.gitignore` 不然它将被提交提交。
>
> ```
> echo "coverage/" >> .gitignore
> ```

顺利的话可以靠到测试成功的提示。

## 配置 Conventional Changelog

**安装依赖库**

```bash
yarn add --dev conventional-changelog-cli  
# 或者
npm install -d  conventional-changelog-cli  
```

**编辑 `package.json`**

```json
{
	// ...,
    "scripts": {
        // ...,
        "version": "conventional-changelog -p angular -i CHANGELOG.md -s && git add CHANGELOG.md",
        // ...
    },
    // ...
}
```

> 使用 conventional-changelog 需要注意：
>
> - 非常注意 commit 格式。这里格式采用 [angular commit 规范](https://github.com/angular/angular/blob/master/CONTRIBUTING.md)，会识别 feat 和 fix 开头的 commit 然后自动生成 Change Log 到 `CHANGELOG.md` 文件。
> - 每次更改需要先升级 version 再去生成。
>
> You could follow the following workflow:
>
> 1. Make changes
> 2. Commit those changes
> 3. Pull all the tags
> 4. Run the [`npm version [patch|minor|major\]`](https://docs.npmjs.com/cli/version) command
> 5. Push
>
> If `preversion`, `version`, or `postversion` are in the `scripts` property of the package.json, they will be executed as part of running `npm version`.
>
> You could optionally add a `preversion` script to package your project or running a full suit of test. And a `postversion` script to clean your system and push your release and tags.

**编辑 `.npmrc`**

```
tag-version-prefix=""
message="chore(release): v%s"
```

**编辑 `.yarnrc`**

```
version-tag-prefix ""
version-git-message "chore(release): v%s"
```

