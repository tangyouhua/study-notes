# Mini Rollup（7）

> 前端进阶训练营笔记-2月打卡-Day12，2023-2-17

手写 RollUp，了解其原理。

## 目标

- v0.7
  - 增加 bundle，实现单模块与多模块导入

## 多模块打包 bundle

### 实现思路

build 打包

1. fetchModule 读取模块 fs.readFileSync，返回模块对象
2. module.expandAllStatements 组装输出 ast，递归依赖模块 fetchModule
3. generate 根据语法树生成代码
4. fs.writeFileSync 写入模块

单元测试需要考虑文件读写不受影响环境，用 mock 实现。

### 实现 fetchModule

#### fetchModule 单元测试

```JavaScript
// lib/__tests__/bundle.spec.js
jest.mock('fs') // fs采用mock

describe('测试Bundle', () => {
    it('fetchModule', () => {
        const bundle = new Bundle({ entry: './a.js' })
        fs.readFileSync.mockReturnValueOnce(`const a = 1`)
        const module = bundle.fetchModule('index.js')
        const { calls } = fs.readFileSync.mock
        expect(calls[0][0]).toBe('index.js')
        expect(module.code.toString()).toBe(`const a = 1`)
    });
});
```

#### fetchModule 功能实现

```JavaScript
// lib/bundle.js
/*
 * main.js -> import foo
 * importer is main.js, importee is foo
 */
fetchModule(importee, importer) {
    let route
    if (!importer) {
        route = importee // main module
    } else {
        // calculate path of importer
        if (path.isAbsolute(importee)) {
            route = importee
        } else {
            route = path.resolve(
                path.dirname(importer),
                importee.replace(/\.js$/, '') + '.js'
            )
        }
    }

    if (route) {
        const code = fs.readFileSync(route, 'utf-8').toString()
        const module = new Module({
            code,
            path: route,
            bundle: this
        })
    }
}
```

项目根目录执行 jest bundle，1个单元测试运行通过。

### 生成单条语句与多条语句

#### 单条语句与多条语句单元测试

- 单条语句 build 结果，写入该条语句到 bundle.js
- 多条语句 build 结果，按照指定分隔符合并输出 tree shaking 后结果到 bundle.js

```JavaScript
// lib/__tests__/bundle.spec.js
describe('build', () => {
    it('单条语句', () => {
        const bundle = new Bundle({ entry: 'index.js' })
        fs.readFileSync.mockReturnValueOnce(`console.log(1)`)
        const module = bundle.build('bundle.js')
        const { calls } = fs.writeFileSync.mock
        expect(calls[0][0]).toBe('bundle.js')
        expect(calls[0][1]).toBe(`console.log(1)`)
    });

    it('多条语句', () => {
        const bundle = new Bundle({ entry: 'index.js' })
        fs.readFileSync.mockReturnValueOnce(`
            const a = () => 1;
            const b = () => 2;
            a()
        `)
        fs.writeFileSync.mock.calls = []
        const module = bundle.build('bundle.js')
        const { calls } = fs.writeFileSync.mock
        expect(calls[0][0]).toBe('bundle.js')
        expect(calls[0][1]).toBe(`const a = () => 1;\na()`)
    });
});
```

#### build 基本功能实现

首先，在 analyse 中加入 _source 属性，记录原始代码供 generate 时使用：

```JavaScript
// lib/analyse.js
function analyse(ast, magicString, module) {
    Object.defineProperties(statement, {
        _defines: { value: {} },
        _dependsOn: { value: {} },
        _included: { value: false, writable: true },
        _source: { value: magicString.snip(statement.start, statement.end) }
    })
}
```

然后，在 bundle 中实现 generate 函数：

```JavaScript
// lib/bundle.js
generate() {
    const magicString = new MagicString.Bundle()
    this.statements.forEach(statement => {
        const source = statement._source.clone()
        /*
         * At this time, tree shaking is almost done
         * safe to remove export statements
         * export const a = 1 => const a = 1
         */
        if (statement.type === 'ExportNamedDeclaration') {
            source.remove(statement.start, statement.declaration.start)
        }
        magicString.addSource({
            content: source,
            separator: '\n'
        })
    })
    return { code: magicString.toString() }
}
```

项目根目录执行 jest bundle，3个单元测试运行通过。

### build 支持多模块

#### 增加多模块单元测试

通过 jest mock 后的 fs 对象，模拟模块加载：

- 第一次先返回 main 模块，导入模块 a，调用函数
- 第二次返回 a 模块，导出函数 a

经过 build 执行后，应当返回模块 a 中的定义。

```JavaScript
// lib/__tests__/bundle.spec.js
describe('build', () => {
    it('多模块', () => {
        const bundle = new Bundle({ entry: 'index.js' })
        fs.readFileSync
            .mockReturnValueOnce(`import { a } from './a';\na()`) // load main
            .mockReturnValueOnce(`export const a = () => 1;`) // load module a
        fs.writeFileSync.mock.calls = []
        bundle.build('bundle.js')
        const { calls } = fs.writeFileSync.mock
        expect(calls[0][0]).toBe('bundle.js')
        expect(calls[0][1]).toBe(`const a = () => 1;\na()`)
    });
});
```

#### 多模块实现 - module

补全 module 中多模块 todo 部分的代码：

- 根据 name 查找 import 声明
- 通过 bundle 的 fetchModule 方法解析模块
- 从被导入的模块 importee 中获取 exports
- 递归执行上述步骤直到全部完成

```JavaScript
// lib/module.js
define(name) {
    if (has(this.imports, name)) { // from import
        const importDeclaration = this.imports[name]
        const source = importDeclaration.source
        const module = this.bundle.fetchModule(source, this.path)
        const exportData = module.exports[importDeclaration.name]
        return module.define(exportData.localName)
    } else {...}
}
```

项目根目录执行 jest bundle，4个单元测试运行通过。

完整代码可查看 [v0.7 版本](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.7)。

## 效果

- [v0.7](https://github.com/tangyouhua/lab-mini-rollup/releases/tag/v0.7)
  - 执行 `jest module` 可以看到除 4 个单元测试通过

## 总结

- 增加 bundle 并实现 fetchModule，完善 module 对多模块导入，实现完整的模块导入功能

此文章为2月Day13学习笔记，内容来源于极客时间《前端进阶训练营》
