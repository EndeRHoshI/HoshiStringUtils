# HoshiResUtils
Android 资源管理工具
## 前言
开发中我们经常遇到一些需要批量处理资源的场景，例如统一整理 drawable 图片，导出导入 string.xml 进行多语言翻译等，这里写一个 Kotlin 小工具类来进行处理

## drawable 图片资源管理工具
### 场景描述
有时候我们需要精简 apk 内容，缩小 apk 包大小，要删掉一些不需要的 drawable 下的图片，经过一些调查，发现当前最省事的一个做法是，只留下 xxhdpi 的图片，详见[文章](https://www.jianshu.com/p/0eb2824d6011)，这样，就可以写一些代码，直接整理 drawable 文件夹，只留下 xxhdpi 的文件了；同样的，如果 xxhdpi 中没有，其它的尺寸下有的话，也只保留最大的尺寸的那一份
### 开发思路
1. 遍历 res 文件夹内的子文件夹，找出 drawable-xxhdpi 之类的文件夹（drawable 文件夹排除在外）
2. 根据定义好的各个尺寸的优先级，从高到低以其作为基准（这里默认最高优先级是 drawable-xxhdpi），将其它 drawable 文件夹中有重复的删除，不重复的保留（举例子，执行到最高的 drawable-xxhdpi 时，其余尺寸文件夹中的同名文件将被删除，然后又轮到 drawable-xhdpi，其余文件夹中同名的文件又被删除，以此类推）
3. 这里为什么引入优先级的概念，是因为处理低优先级的尺寸时可以直接跳过已经处理过的高优先级尺寸，节省一些时间（虽然也没多大变化），如果想再简化一点的话，可以直接用顺序来表示优先级，直接按顺序取基准尺寸来处理，反正以低优先级尺寸作为基准去删除高优先级尺寸的文件时，高优先级尺寸中已经没有同名文件了（处理高优先级时已经去掉其它尺寸的所有同名文件了），结果也是差不多的，不需要额外做处理
### 使用步骤
1. 打开 [DrawableMgr](src/main/kotlin/drawable/DrawableMgr.kt)，填入需要整理的 res 文件夹路径
2. 在 `DensityQualifier` 密封类中配置好各个尺寸的优先级
3. 调整 `getDensityQualifierList()` 方法，配置好需要处理的各个尺寸（有些尺寸有些时候不需要处理，如 nodpi、anydpi 这些）
4. 运行 main 方法，这样所有的 drawable 文件夹内就会得到唯一的一份图片资源
5. 最后可以和 UI 协调，把所有低尺寸文件都出一份 drawable-xxhdpi 的，然后统一放到 drawable-xxhdpi 中，最后把其它低尺寸的文件删掉
## string.xml 多语言字符串管理工具
### 场景描述
在做多语言适配时，我们需要提供应用内的文案给翻译人员进行翻译，你不能直接把整个应用的 string.xml 打包扔给对方，一般来说，将项目内的 string.xml 导出为 Excel 文件再交给翻译人员会比较好
### 开发思路
1. 直接遍历找到整个项目中符合正则匹配的 `*string*.xml`、`*array*.xml` 的文件，string 和 array 前后可以有其它字符，但是一定要以 .xml 为结尾
2. 找到后进行分组，每个组里面以 values 为基础，其余的 values-xxx 则是其它的语种（这里进行分组是因为有时候我们模块化开发会有不同的模块，多渠道发布有不同的渠道，多版本定制化开发会有不同的 app 版本，这样会导致不止一组 string.xml 存在）
3. 分好组后，从所有 string.xml 中读取键值对，以 values 下的作为基准，导出到 Excel 中，values 中没有的，直接忽略掉
### 使用步骤
1. 输入项目路径和参数，控制是全量输出、已翻译完全的还是未翻译完全的
2. 直接遍历找到整个项目中符合正则匹配的 `*string*.xml`、`*array*.xml` 的文件
3. 以 string.xml 和 array.xml 作为基准进行判断已翻译还是未翻译，多出的忽略不理
4. 生成一个 excel，每个文件放一个工作表，每种语言一个列，这样从 xml 到 excel 就完成了
5. 输入 excel 路径和参数，控制是全量输出、已翻译的还是未翻译的
6. 读取 excel，逐个工作表去读取，逐行逐列读取，转换回 xml，然后就可以把 xml 放回到项目中
### 其它事项
#### 翻译前
交给负责翻译的人员前，可以对 Excel 进行一些处理，以免翻译人员进行翻译时出错
1. 标注需要翻译的列，或者把不需要翻译的列删除掉
2. 告知翻译人员，部分文案存在占位符，占位符只用于占位，不需要翻译
#### 翻译后
拿到已经翻译的 Excel，也要注意以下问题：
1. xml -> Excel 时，会把转义字符去掉，Excel -> xml 时，要留意是否要再加上转义字符，特别是要留意意大利语和法语，他们的语言中经常带有 `'` 字符，会导致 xml 报错，需要加上 `\` 转义
2. RTL 语系，要注意以下：
   1. 你要确保你的手机也打开 RTL 布局开关，可以在开发者模式里面进行设置
   2. 处理格式化占位符（%s、%d 等）时，编译可能不会报错，但是到了特定页面无法正确处理占位符（读成了s%）而导致闪退，实践中发现好像如果被夹在希伯来语中间就需要改成 s%，如果不是则不用，还需要再测试一下，最简单直接的处理手段：直接删掉，然后手打 %s，让他自己排版；除了格式化占位符问题，RTL 布局也是个问题，这个要做得很好，应该需要 UI 介入，是题外话了
   3. 在设置 gravity 时，要记得设置水平方向的 gravity（忘记为什么了，后续遇到可以再说明下），使用 left、right 相关属性时，用 start、end 替代
3. Span 的高亮文案，翻译不当会导致不能正确高亮，高亮翻译时，可以在全文中使用不会被翻译到的字符（如阿拉伯数字）代替需要高亮的部分，翻译好高亮部分再套进去即可
4. 文案最好要有相关人员审核，否则会出现各种问题
   1. 机翻不通顺是最常见的
   2. 如果是客户自愿提供翻译的，有些内容错误无法把关（漏翻译、或者错误翻译）
5. 文案的版本管理要注意，举个例子，送出去翻译后，改了一些文案，翻译回来后，又没及时更新，把旧的文案又覆盖上去了