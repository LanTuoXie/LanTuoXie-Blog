# VS Code 用户自定义代码片段(React)

`.jsx`React组件模板:`javascriptreact.json`

```json
{
  "Import React": {
    "prefix": "importreact",
    "body": [
      "import React from 'react'\n",
      "class ${1:${TM_FILENAME/(.*).(?:jsx|js)/$1/i}} extends React.Component {",
      "  render () {",
      "    return (",
      "      ${2|null|}",
      "    )",
      "  }",
      "}\n",
      "export default ${1:${TM_FILENAME/(.*).(?:jsx|js)/$1/i}}\n"
    ],
    "description": "引入React基本组件"
  },
  "Import PropTypes": {
    "prefix": "importproptypes",
    "body": [
      "import PropTypes from 'prop-types'"
    ],
    "description": "引入prop-types"
  },
  "Define defaultProps": {
    "prefix": "defaultProps",
    "body": [
      "static defaultProps = {",
      "  $1",
      "}\n"
    ]
  }
}

```

`.js`redux模板：`javascript.json`

```json
{
  "Import Redux Action": {
    "prefix": "importaction",
    "body": [
      "import { ${1|handleActions,createAction|} } from 'redux-actions'"
    ],
    "description": "导入redux-actions"
  },
  "Create Reducer": {
    "prefix": "reducer",
    "body": [
      "import { handleActions } from 'reduce-actions'",
      "import * as ${1:types} from '@/constants/${2:${TM_FILENAME/(.*)State.(?:jsx|js)/$1Const/i}}'\n",
      "const ${3:initState} = {",
      "\t$4",
      "}\n",
      "export default handleActions({",
      "\t$5",
      "}, ${3:initState})\n"
    ],
    "description": "导入一个reducer模板"
  },
  "Create Action": {
    "prefix": "action",
    "body": [
      "import { ${1|createAction, createActions|} } from 'redux-actions'",
      "import * as ${2:types} from '@/constans/${3:${TM_FILENAME/(.*)Action.(?:jsx|js)/$1Const/i}}'\n",
      "$0"
    ]
  }
}

```