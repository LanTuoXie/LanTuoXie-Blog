# Vue源码分析(3)

下面主要研究`./src/core/global-api`这个文件里面的文件。   
所有自定义数据类型都在`./flow`文件夹，且下面内容用到`./flow/global-api.js`    
这个文件夹对应[Vue API文档](https://cn.vuejs.org/v2/api/#Vue-component)的`全局API`，最好参考api文档来分析。

## `./flow/global-api.js`

```js
declare interface GlobalAPI {
  cid: number;
  options: Object;
  config: Config;
  util: Object;

  extend: (options: Object) => Function;
  set: <T>(target: Object | Array<T>, key: string | number, value: T) => T;
  delete: <T>(target: Object| Array<T>, key: string | number) => void;
  nextTick: (fn: Function, context?: Object) => void | Promise<*>;
  use: (plugin: Function | Object) => void;
  mixin: (mixin: Object) => void;
  compile: (template: string) => { render: Function, staticRenderFns: Array<Function> };

  directive: (id: string, def?: Function | Object) => Function | Object | void;
  component: (id: string, def?: Class<Component> | Object) => Class<Component>;
  filter: (id: string, def?: Function) => Function | void;

  // allow dynamic method registration
  [key: string]: any
};
```

- 其实就是Vue

## `./core/global-api/assets.js`

对应Vue API文档的

- `Vue.component`
- `Vue.filter`
- `Vue.directive`
 
```js
/* @flow */

import { ASSET_TYPES } from 'shared/constants'
import { isPlainObject, validateComponentName } from '../util/index'

export function initAssetRegisters (Vue: GlobalAPI) {
  /**
   * Create asset registration methods.
   */
  ASSET_TYPES.forEach(type => {
    Vue[type] = function (
      id: string,
      definition: Function | Object
    ): Function | Object | void {
      if (!definition) {
        return this.options[type + 's'][id]
      } else {
        /* istanbul ignore if */
        if (process.env.NODE_ENV !== 'production' && type === 'component') {
          validateComponentName(id)
        }
        if (type === 'component' && isPlainObject(definition)) {
          definition.name = definition.name || id
          definition = this.options._base.extend(definition)
        }
        if (type === 'directive' && typeof definition === 'function') {
          definition = { bind: definition, update: definition }
        }
        this.options[type + 's'][id] = definition
        return definition
      }
    }
  })
}
```

- `ASSET_TYPES`就是`['component', 'filter', 'directive']`
- `validateComponentName`是验证注册的组件名是否符合规范（组件名只能是字母和'-'），以及不能是HTML的标签名以及Vue预留的标签名<component>
- `this.options[type + 's'][id]`：是`components`、`filters`、`directives`都是按`id`为索引的字典数据存储结构
- 不传`definition`那就是在`this.options[type + 's']`中根据`id`查找相应类型的，传就是注册新的`component || filter || directive`
- `definition.name = definition.name || id`：组件名必须要有，如果没有定义组件名，默认使用组件的id
- `definition = this.options._base.extend(definition)`：组件有个默认设置`_base`，通过合并的方式继承新组件
- 指令，如果传入的是一个函数而不是对象，那个对象就是`definition = { bind: definition, update: definition }`，bind和update的函数
- 组件和指令的`definition`可以传对象或者函数，但是filter是只能是函数，这里面没有检测和处理，而是直接添加进入`filters`
- 这个函数就是全局注册`component、filter、directive`

## `./src/core/global-api/extend.js`

这个文件时全局API的`Vue.extend`
用到的一些辅助函数

- `extend`：就是单纯地将另一个对象的第一层赋值给一个新的对象`to[key] = _from[key]`
