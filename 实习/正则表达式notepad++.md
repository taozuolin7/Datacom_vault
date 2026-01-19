核心说明：使用前需打开「替换」([Ctrl+H](https://zhida.zhihu.com/search?content_id=268548355&content_type=Article&match_order=1&q=Ctrl%2BH&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3Njg5ODY4MjIsInEiOiJDdHJsK0giLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNjg1NDgzNTUsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9._qyyNTUkPg21YKA2BKYrFcaqVcWCvH4Gj_5-HfFx54E&zhida_source=entity)) 或「标记」([Ctrl+M](https://zhida.zhihu.com/search?content_id=268548355&content_type=Article&match_order=1&q=Ctrl%2BM&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3Njg5ODY4MjIsInEiOiJDdHJsK00iLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNjg1NDgzNTUsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.mGHfLGkfMtWNiRcuZ-PO90YuV0svhID2tJDxtmSuQPo&zhida_source=entity))，勾选「[正则表达式](https://zhida.zhihu.com/search?content_id=268548355&content_type=Article&match_order=1&q=%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3Njg5ODY4MjIsInEiOiLmraPliJnooajovr7lvI8iLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNjg1NDgzNTUsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.tw7mlRsAj7xWAUQzMMZkpUyaGikna8e3sZYLweAXllI&zhida_source=entity)」，取消勾选「. matches newline」（除非需跨行匹配）

**一、基础正则语法（必记）**

|        |                 |                              |
| ------ | --------------- | ---------------------------- |
| 语法     | 含义              | 示例                           |
| ^      | 匹配行首            | ^abc：匹配以abc开头的行              |
| $      | 匹配行尾            | abc$：匹配以abc结尾的行              |
| \d     | 匹配任意数字（0-9）     | \d{3}：匹配3位连续数字               |
| \D     | 匹配非数字           | \D+：匹配1个及以上非数字               |
| \w     | 匹配字母/数字/下划线     | \w+：匹配连续字符（含数字）              |
| \s     | 匹配空格/制表符        | \s*：匹配0个及以上空白字符              |
| \R     | 匹配任意换行符（适配所有格式） | (\R\|$)：匹配换行或文件末尾            |
| .      | 匹配任意单个字符（除换行）   | a.b：匹配a+任意字符+b（如acb、adb）     |
| *      | 匹配前一个字符0次及以上    | ab*：匹配a后接0个及以上b（a、ab、abb...） |
| +      | 匹配前一个字符1次及以上    | ab+：匹配a后接1个及以上b（ab、abb...）   |
| {n}    | 匹配前一个字符n次       | \d{4}：匹配4位数字（如年份）            |
| [abc]  | 匹配括号内任意一个字符     | [#//]：匹配#、/、/任意一个            |
| [^abc] | 匹配非括号内的字符       | [^0-9]：等价于\D（非数字）            |
| (...)  | 分组捕获，可通过\1、\2调用 | (a)(b)：\1=\(a\)，\2=\(b\)     |
| (?i)   | 忽略大小写           | (?i)test：匹配test、Test、TEST    |
|        |                 |                              |

**二、高频场景操作（直接套用）**

**1. 行处理（删除/提取）**

|   |   |   |   |
|---|---|---|---|
|需求|查找内容（正则）|替换为|备注|
|删除以数字开头的行|^\d.*(\R\|$)|留空|彻底删除整行|
|删除以字母开头的行|^[a-zA-Z].*(\R\|$)|留空|区分大小写，加(?i)可忽略|
|删除注释行（#/\/\/开头）|^[#//].*(\R\|$)|留空|适用于代码/配置文件|
|删除空行/仅含空格的行|^\s*(\R\|$)|留空|清理无效空白行|
|删除只含3个数字的行|^\d{3}(\R\|$)|留空|仅匹配3位数字，无其他字符|
|提取以[LOG]开头的行|^(?![LOG]).*(\R\|$)|留空|反向删除非目标行，保留LOG行|
|删除以特定字符结尾的行（如;）|.*;(\R\|$)|留空|替换;为目标结尾字符即可|

**2. 字符匹配与替换**

|   |   |   |   |
|---|---|---|---|
|需求|查找内容（正则）|替换为|备注|
|删除所有数字|\d|留空|保留非数字内容|
|删除所有特殊符号（留数字字母）|[^a-zA-Z0-9\s]|留空|\s保留空格，可删除|
|多空格转单空格|\s+|单个空格|清理文本多余空格|
|数字加单位（如px）|(\d+)|\1px|给所有数字后加px|
|提取所有邮箱地址|\w+@\w+\.\w+|-|用Ctrl+M标记行，再提取|
|替换key:value为key=value|(\w+):(\w+)|\1=\2|分组替换分隔符|

**3. 格式调整**

|   |   |   |   |
|---|---|---|---|
|需求|查找内容（正则）|替换为|备注|
|给每一行加前缀（如[ ]）|^(.*)$|[\1]|\1表示整行内容|
|给每一行加后缀（如;）|^(.*)$|\1;|适配代码行尾补全|
|删除行首所有空格|^\s+|留空|清除行首缩进|
|删除行尾所有空格|\s+$|留空|清理行尾多余空白|

**三、关键注意事项**

• 换行符适配：优先用\R，避免直接写\r\n（仅适配Windows），防止换行格式异常

• 大小写：默认区分，加(?i)前缀可忽略（如(?i)^test匹配所有大小写开头的test行）

• 备份优先：批量操作前按[Ctrl+Shift+S](https://zhida.zhihu.com/search?content_id=268548355&content_type=Article&match_order=1&q=Ctrl%2BShift%2BS&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3Njg5ODY4MjIsInEiOiJDdHJsK1NoaWZ0K1MiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNjg1NDgzNTUsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.tOkxdm34y62mVBQA85mQna8hSKQ3a5-NA21da28CV-E&zhida_source=entity)另存文件，避免误操作丢失内容

• 跨行匹配：需勾选「. matches newline」，否则.不匹配换行符

• 特殊字符转义：匹配.、*、+等特殊符号时，需加\转义（如\.匹配实际的.）

**四、快速操作流程**

1. 打开目标文件 → 按Ctrl+H调出替换对话框

2. 勾选「正则表达式」→ 取消「. matches newline」

3. 复制对应场景的正则到「查找内容」

4. 设置「替换为」内容（留空或对应替换值）

5. 点击「全部替换」→ 按Ctrl+S保存