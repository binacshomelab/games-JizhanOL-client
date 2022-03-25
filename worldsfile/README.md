# Generator

## 0. [必须]使用生成器生成 ==> 因其对其有要求 不能直接修改文本文件

| 参数                            | 数值            | ATTENTION    |
| ------------------------------- | --------------- | ------------ |
| 【设置大区】显示名称 / 大区名称 | 零点机战        |              |
| 【设置小区】登陆IP              | XXX.XXX.XXX.XXX |              |
| 【设置小区】显示名称            | 第一大区        |              |
| 【控制台】                      |                 | **明文生成** |

## 1. 生成文件字符格式为 latin1 ==> 需修改行末字符

`æ&^@^@` 需替换为 `c^@^@`

>   注意: ^@ 其实是一个字符

**必须使用 vim 进入 visual 模式，并修改单字符**

1.   `v` 进入 visual 模式
2.   `y` 复制末尾字符
3.   `p` 粘贴字符
4.   替换 `æ&` 为 `c` 【正常单字符 c 】

## 2. 替换 ini/worlds.bat Bingo!