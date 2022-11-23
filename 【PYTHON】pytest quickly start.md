# 一、环境安装
`pip install pytest`

# 二 、用例编写

##  默认的用例识别规则

- 用例文件：以`test_`开头 或者`_test` 开头的文件会被识别为用例文件
- 用例类：测试文件中每个`Test`开头的类就是一个测试用例类
- 测试用例：`测试类中每个test开头的方法`  或 `测试文件中每个test开头的函数` 就是一个测试用例

# 三、执行pytest 测试用例

## 1. 执行pytest

命令行执行
`1. python -m pytest [...]`

除了命令行运行方式外，pytest 还支持在程序中运行，在程序中运行的命令如下：
`pytest.main([...])`

不管是使用命令行运行或者使用 pytest.main() 的方式运行，它们支持的参数都是一样的。 需要注意的是：pytest 的参数必须放在一个 list 或者 tuple 里。

## 2. pytest 常用参数

`pytest -help` : 查看 具体参数

-   -m: 用表达式指定多个标记名。 pytest 提供了一个装饰器 @pytest.mark.xxx，用于标记测试并分组（xxx是你定义的分组名），以便你快速选中并运行，各个分组直接用 and、or 来分割。
-   -v: 运行时输出更详细的用例执行信息 不使用 -v 参数，运行时不会显示运行的具体测试用例名称；使用 -v 参数，会在 console 里打印出具体哪条测试用例被运行。  
-   -q: 类似 unittest 里的 verbosity，用来简化运行输出信息。 使用 -q 运行测试用例，仅仅显示很简单的运行信息， 例如：
	 `.s..  [100%] 
	 `3 passed, 1 skipped in 9.60s`
-   -k: 可以通过表达式运行指定的测试用例。 它是一种模糊匹配，用 and 或 or 区分各个关键字，匹配范围有文件名、类名、函数名。   
-   -x: 出现一条测试用例失败就退出测试。 在调试时，这个功能非常有用。当出现测试失败时，停止运行后续的测试。
-   -s: 显示print内容 在运行测试脚本时，为了调试或打印一些内容，我们会在代码中加一些print内容，但是在运行pytest时，这些内容不会显示出来。如果带上-s，就可以显示了。

## 3. 选择测试用例执行

pytest 里选择测试用例执行有很多方法，可以按照测试文件夹、测试文件、测试类和测试方法四种。

-   按照测试文件夹执行
    `# 执行所有当前文件夹及子文件夹下的所有测试用例 pytest . # 执行跟当前文件夹同级的tests文件夹及子文件夹下的所有测试用例 pytest ../tests`
-   按照测试文件执行
    `# 运行test_se.py下的所有的测试用例 pytest test_se.py`
-   按照测试类执行
    `按照测试类执行，必须以如下格式： pytest 文件名 .py:: 测试类，其中“::”是分隔符，用于分割测试module和测试类。 # 运行test_se.py文件下的，类名是TestSE下的所有测试用例 pytest test_se.py::TestSE`
-   按照测试方法执行
    `同样的测试方法执行，必须以如下格式： pytest 文件名 .py:: 测试类 :: 测试方法，其中 “::” 是分隔符，用于分割测试module、测试类，以及测试方法。 # 运行test_se.py文件下的，类名是TestSE下的，名字为test_get_new_message的测试用例  pytest test_se.py::TestSE::test_get_new_message`

    以上选择测试用例执行的方法，可以不在命令行，而直接在测试程序里执行，其语法为
    `pytest.main([模块.py::类::方法])`

## 4.动态挑选测试用例执行 - 按Tag

首先给测试用例打标签（mark），在 Class、method 上加上如下装饰器：
`@pytest.mark.xxx`
在运行时，命令行动态指定标签运行：
`# 同时选中带有这两个标签的所有测试用例运行 pytest -m "mark1 and mark2" # 选中带有mark1的测试用例，不运行mark2的测试用例 pytest -m "mark1 and not mark2" # 选中带有mark1或 mark2标签的所有测试用例 pytest -m "mark1 or mark2"`

## 5. 动态匹配测试用例执行 - 按名称

pytest中，动态挑选测试用例，除了打标签（mark）外，还有另外一种方式：按照文件名、类名、方法名来模糊匹配的
`# -k 参数是按照文件名、类名、方法名来模糊匹配的 pytest -k xxxPattern`
-   按照文件名称全匹配：
    `# 运行test_se.py下的所有的测试 pytest -k "test_se.py"`
-   按照文件名字部分匹配：
    `# 因为se能匹配上test_se.py,故运行test_se.py下所有的测试 pytest -k "se"`
-   按照类名匹配：
    `# 因为Baidu能匹配上test_baidu.py里定义的测试类Baidu,故运行Baidu测试类下所有的测试,你也可以写成Bai. pytest -k "Baidu"`
-   按照方法名匹配：
    `# message只能匹配test_se.py中定义的测试类TestSE下的测试方法test_get_new_message,故仅有test_get_new_message这个方法会执行 pytest -k "message"`

## 6. 忽略测试用例执行

-   直接忽略测试执行
    `直接忽略可以使用 @pytest.mark.skip 装饰器来实现。 @pytest.mark.skip(reason='skip此测试用例') def test_get_new_message:`
-   按条件忽略测试执行—使用’skipif’忽略
    `按skipif条件，当条件符合时便会忽略某条测试用例执行。 # 定义一个flag，用来指示是否要skip一个测试用例 flag = 1 # 此处判断flag的值，为1则忽略，0则不忽略  @pytest.mark.skipif(flag == 1, reason='by condition') def test_get_new_message:`
-   按条件忽略测试执行—使用’-m’或者’-k’忽略
    `# 忽略方法名包含method的测试用例 pytest -k "not method" # 忽略被装饰器@pytest.mark.slow装饰的所有测试用例 pytest -m "not slow"`

## 7. 多进程运行case

`pytest test_se.py -n NUM # 其中NUM填写并发的进程数 # 需要安装pytest-xdist`

## 8. 重试运行case

`pytest test_se.py --reruns NUM # 其中NUM填写重试的次数 # 需要安装pytest-rerunfailures`