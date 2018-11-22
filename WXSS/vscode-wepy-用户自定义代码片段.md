# VSCode wepy 自定义代码片段

```json
{
  "wepy-page": {
    "prefix": "wepy",
    "body": [
      "<template>",
      "\t<view></view>",
      "</template>\n",
      "<script>",
      "import wepy from 'wepy'\n",
      "export default class ${1:${TM_FILENAME_BASE/([a-zA-Z])(.*)/${1:/upcase}$2/g}} extends wepy.${2|page,component,app|} {",
      "\t$3",
      "}",
      "</script>\n",
      "<style lang='scss'>\n",
      "</style>\n"
    ]
  },
  "Single Element": {
    "prefix": "singletag",
    "body": [
      "<$1 />\n"
    ]
  },
  "wepy-config": {
    "prefix": "configwepy",
    "body": [
      "config = {",
      "\tnavigationBarTitleText: '$1'",
      "}",
    ]
  }
}
```

