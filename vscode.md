# vscode 用户自定义代码片段


### 变量

- `TM_SELECTED_TEXT`：当前选定的文本或空字符串
- `TM_CURRENT_LINE`：当前行的内容
- `TM_CURRENT_WORD`：光标所处单词或空字符串
- `TM_LINE_INDEX`：行号
- `TM_LINE_NUMBER`：行号
- `CLIPBOARD`：剪贴板中的内容

和文件名和路径有关：

- `TM_FILENAME`：当前文件的文件名
- `TM_FILENAME_BASE`：当前文件的文件名（不含后缀名）
- `TM_DIRECTORY`：当前文件的文件夹
- `TM_FILEPATH`：当前文件的绝对路径

和时间日期有关的：

- `CURRENT_YEAR`：当前年份
- `CURRENT_YEAR_SHORT`：当前年份的后两位
- `CURRENT_MONTH`：格式化为两位数字的当前月份，如 02
- `CURRENT_MONTH_NAME`：月份
- `CURRENT_MONTH_NAME_SHORT`：月份简称
- `CURRENT_DATE`：当天月份第几天
- `CURRENT_DAY_NAME`：星期几
- `CURRENT_DAY_NAME_SHORT`：星期几简称
- `CURRENT_HOUR`：当前小时
- `CURRENT_MINUTE`：当前分钟
- `CURRENT_SECOND`：当前秒数

### format_string

- `${$1:/upcase}`：大写字母
- `${$1:/downcase}`：小写字母
- `${$1:/capitalize}`：单词首个字母大写
- `${$1:+word}`：如果匹配成功，拼上`word`
- `${$1:?word1:word2}`：如果匹配成功，拼上`word1`否则拼上`word2`
- `${$1:-word}`：如果匹配不成功，拼上`word`
- `${$1:word}`：和`${$1:-word}`一样
