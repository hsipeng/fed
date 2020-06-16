# nodejs 脚本逻辑

## 项目结构

![image-20200503093056835](https://i.loli.net/2020/05/19/GY2T5lNesEJ8BO9.png)

## 技术栈

* typscript
* Nodejs



## package.json 配置

bin 指定运行起始脚本

files 包含需要publish 的包

```json
{
  "name": "ts-git-script",
  "version": "1.0.0",
  "main": "./scripts/index.js",
  "license": "MIT",
  "files": [
    "bin",
    "scripts",
    "lib"
  ],
  "bin": {
    "myGulpCli": "./bin/myGulpCli"
  },
  "types": "./types/index.d.ts",
  "scripts": {
    "watch": "npm run build -- --watch --diagnostics",
    "compile": "tsc --pretty",
    "build": "rimraf scripts types && tsc --pretty"
  },
  
  ....
}
```



## tsconfig.json

"target": "es5",  ts 编译类型
"outDir": "scripts", 编译输出文件夹目录
"module": "commonjs", 编译格式
"moduleResolution": "node", 查找依赖方式

```json
{
	"compilerOptions": {
		"target": "es5",
		"outDir": "scripts",
		"module": "commonjs",
		"moduleResolution": "node",
		"resolveJsonModule": true,
		"esModuleInterop": true,
		"skipDefaultLibCheck": true,
		"sourceMap": false,
		"declaration": true /* Generates corresponding '.d.ts' file. */,
        // "declarationMap": true /* Generates a sourcemap for each corresponding '.d.ts' file. */,
        "declarationDir": "./types" /* Output directory for generated declaration files. */,
		"noUnusedLocals": true,
		"noUnusedParameters": false,
		"noFallthroughCasesInSwitch": true,
		"removeComments": true,
		"suppressImplicitAnyIndexErrors": true,
		"lib": [
			"es2017",
			"es2015",
			"es6"
		],
		"types": [
			"node"
		]
	},
	"include": [
		"src/**/*"
	],
	"exclude": [
		"node_modules",
		"scripts"
	]
}
```



## myGulpCli

cross-spawn 跨平台执行nodejs命令运行脚本

方便拓展js命令

```js
#!/usr/bin/env node
/**
 * Copyright (c) 2015-present, Facebook, Inc.
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
 */

'use strict';

// Makes the script crash on unhandled rejections instead of silently
// ignoring them. In the future, promise rejections that are not handled will
// terminate the Node.js process with a non-zero exit code.
process.on('unhandledRejection', err => {
  throw err;
});

const spawn = require('cross-spawn');
const args = process.argv.slice(2);

const scriptIndex = args.findIndex(
  x => x === 'start'|| x === 'version'|| x === 'init'
);
const script = scriptIndex === -1 ? args[0] : args[scriptIndex];
const nodeArgs = scriptIndex > 0 ? args.slice(0, scriptIndex) : [];

switch (script) {
  case 'version':
  case 'init':
  case 'start': {
    const result = spawn.sync(
      'node',
      nodeArgs
        .concat(require.resolve('../scripts/' + script))
        .concat(args.slice(scriptIndex + 1)),
      { stdio: 'inherit' }
    );
    if (result.signal) {
      if (result.signal === 'SIGKILL') {
        console.log(
          'The build failed because the process exited too early. ' +
            'This probably means the system ran out of memory or someone called ' +
            '`kill -9` on the process.'
        );
      } else if (result.signal === 'SIGTERM') {
        console.log(
          'The build failed because the process exited too early. ' +
            'Someone might have called `kill` or `killall`, or the system could ' +
            'be shutting down.'
        );
      }
      process.exit(1);
    }
    process.exit(result.status);
    break;
  }
  default:
    console.log('Unknown script "' + script + '".');
    console.log('Perhaps you need to update script?');
    console.log(
      'See: 18045835'
    );
    break;
}

```



## npm 包的发布



```bash
npm login


npm publish .
```

