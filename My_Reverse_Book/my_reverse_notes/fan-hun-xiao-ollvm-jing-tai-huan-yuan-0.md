# 反混淆: OLLVM静态还原-0

## 反混淆的主流程

在接下来的好几个章节中，我们将讨论如何静态对抗Flat控制流混淆技术，包括从较为初级的裸状态变量的Flat混淆到一些对于状态变量保护后的flat混淆。

首先，对于Flat混淆技术的对抗核心的问题是攻击状态变量来还原控制流，会包含以下这些流程：

1. 定位状态变量
2. 定位分发块/真实块等
3. 还原状态变量在真实块的值，确认真实的控制流
4. 使用上面的信息，最终还原控制流结构

&#x20;

但是，在当前的对抗环境下，每一步有很多的困难，尤其是各种增强或者变种后的Flat技术，这些技术会想办法来保护混淆状态变量，比如：

1. 状态变量特殊化，不再是一个局部变量，甚至是多重状态变量
2. 控制流插入状态变量的计算后比较逻辑，不直接比较大数
3. 虚假控制流干扰对于分支逻辑的判断
4. 状态变量混合执行流程
5. 混合其它的混淆方案

&#x20;

使用IDA Pro和他的中间语言micorocode作为的还原的工具，选择IDA的原因是这个工作很方便，大家都会用；

虽然他的micorocode相关的资料比较匮乏，但是现有的开源资料已经足够完成后面的工作了。



样例二进制（借用一下OBPO）:

{% embed url="https://github.com/obpo-project/samples/blob/main/arm64/libnative-lib.so" %}

## 熟悉IDA-micorocode

{% embed url="http://www.hex-rays.com/products/ida/support/ppt/recon2018.ppt" %}

安装lucid插件：



首先需要清楚IDA实现F5中的几个优化过程：

<table data-header-hidden><thead><tr><th width="264"></th><th></th></tr></thead><tbody><tr><td>MMAT_ZERO </td><td>microcode does not exist</td></tr><tr><td>MMAT_GENERATED </td><td>generated microcode</td></tr><tr><td>MMAT_PREOPTIMIZED </td><td>preoptimized pass is complete</td></tr><tr><td>MMAT_LOCOPT </td><td><p>local optimization of each basic block is complete.</p><p>control flow graph is ready too.</p></td></tr><tr><td>MMAT_CALLS </td><td>detected call arguments. see also hxe_calls_done</td></tr><tr><td>MMAT_GLBOPT1 </td><td>performed the first pass of global optimization</td></tr><tr><td>MMAT_GLBOPT2 </td><td>most global optimization passes are done</td></tr><tr><td>MMAT_GLBOPT3 </td><td>completed all global optimization. microcode is fixed now.</td></tr><tr><td>MMAT_LVARS </td><td>allocated local variables</td></tr></tbody></table>

在这个系列中，我们尽可能的在更高层的中间语言执行优化，也就是说尽可能在IDA的其它优化之后执行优化。这么做的原因是，借用IDA的常量传播等优化功能，可以省去很多复杂的跨basic-block的分析工作。



### 插件优化层次的选择

越是往后，IDA本身的优化越多。

插件的优化所在的层次和插件本身的功能和需求有关。例如，这一次我实现的是一个以攻击状态变量为主的OLLVM还原插件，那么可以列出以下这些要求：

* 需要有清晰的状态变量+用常量给状态变量赋值+状态变量比较常量
* 状态变量的个数，尽可能收敛

通过对中间语言的查看，上面的需求至少是要在MMAT\_CALLS及以上实现。

说明：比如汇编里面有这样的代码

```
.text:000000000001744C                 MOVK            W11, #0xC0B6,LSL#16
.text:0000000000017450                 MOVK            W12, #0xC265,LSL#16
//... 中间隔了很多的其它代码
.text:0000000000017568                 CMP             W8, W11
.text:000000000001756C                 B.EQ            loc_17758
.text:0000000000017570                 CMP             W8, W12
.text:0000000000017574                 B.EQ            loc_17594
```

在MMAT\_LOCOPT优化层，中间语言长这样，和汇编的意思差不多：

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

但是MMAT\_CALLS 优化层，由于执行了常量传播+替换，w11和w12直接被替换成了常量，因此就可以直接看出来是状态变量的比较操作了。

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

所以这个反混淆优化的层次，最少要在MMAT\_CALLS之上实现，



## 基础样例的反混淆流程

* 识别状态变量：
* 状态变量区分基本块：
* 识别分发块 -> 真是块 状态变量的入度：
* 找到真实块中状态变量的赋值点：
* 在中间语言还原控制流：





## 识别状态变量

使用一种较为朴素的方案：

遍历所有的中间语言，查找最多的用于比较固定值后跳转的变量，代码如下：

```
import ida_hexrays
import ida_kernwin
import idaapi
from ida_hexrays import *


def is_jmp_condition(opcode: ida_hexrays.mop_t) -> bool:
    if opcode in [m_jcnd, m_jnz, m_jz, m_jae, m_jb, m_ja, m_jbe, m_jg, m_jge, m_jl, m_jle, m_jtbl]:
        return True
    return False

class jz_info_t:
    def __init__(self, op=None, nseen=0):
        self.op = op
        self.nseen = nseen
        self.nums = []

class compare_collector(minsn_visitor_t):
    """
    Looks for jz, jg comparisons against constant values. Utilised jz_info_t class.
    """
    def __init__(self):
        minsn_visitor_t.__init__(self)
        self.seen_comparisons = []

    def visit_minsn(self):
        ins: minsn_t = self.curins

        if is_jmp_condition(ins.opcode):
            print(f"jz_collector_x: {ins._print()}")
        else:
            return 0
        if ins.r.t != mop_n: # 这里假定了都是在比较常量，在初始的flat的确是这样，但是变种情况有很多
            return 0

        found = 0
        this_mop = ins.l
        idx_found = 0
        # 这里直接拿了D810的统计方法
        for sc in self.seen_comparisons:
            if sc.op.equal_mops(this_mop, EQ_IGNSIZE): # 如果左侧变量相同，进行合并
                sc.nseen += 1
                sc.nums.append(ins.r)
                found = sc.nseen
                break
            idx_found += 1

        # 如果在已有的内容不存在，加入一个
        if not found:
            jz = jz_info_t()
            jz.op = this_mop
            jz.nseen = 1
            jz.nums.append(ins.r)
            self.seen_comparisons.append(jz)

        return 0


g_Last = ida_hexrays.MMAT_ZERO
class BlockOptimizerManager(optblock_t):
    def __init__(self, *args):
        super().__init__(*args)

    def func(self, blk: mblock_t):
        global g_Last

        mba: mbl_array_t = blk.mba
        mba_maturity = mba.maturity
        # 这个优化方式每个层级只执行一次
        if g_Last == mba_maturity:
            return 0
        g_Last = mba_maturity
        # 这个优化只在MMAT_GLBOPT1层级执行
        if mba_maturity != ida_hexrays.MMAT_GLBOPT1:
            return 0
        self.get_compare_vars(mba)
        return 0

    def get_compare_vars(self, mba: mbl_array_t):
        """
        处理状态变量,通过比较的次数进行判断
        """
        print(f"Run the compare_collector")
        jzc = compare_collector()
        mba.for_all_topinsns(jzc)

        jzc.seen_comparisons.sort(key=lambda x: x.nseen)

        for one in jzc.seen_comparisons:
            op = one.op
            times = one.nseen
            print(f"Compare variable={op.dstr()} times={times}")

if __name__ == '__main__':
    form = ida_kernwin.find_widget("Output window")
    ida_kernwin.activate_widget(form, True)
    idaapi.process_ui_action("msglist:Clear")

    testPlugin = BlockOptimizerManager()
    is_remove = testPlugin.remove()

    if not is_remove:
        print(f"testPlugin.remove {is_remove}")
    testPlugin.install()

```

可以直接在IDA的File -> Script Command 复制黏贴并执行这部分代码，然后在sub\_172BC函数按下F5就能在console中看到输出结果了。

```
Compare variable=%var_91.1 times=1
Compare variable=%var_58.1 times=1
Compare variable=w11.4 times=2
Compare variable=w10.4 times=6
Compare variable=w9.4 times=18
Compare variable=w8.4 times=43
```

因此，w8.4  w9.1 w10.4 这三个变量的使用此时明显超过了正常的情况，很有可能是状态变量。

可以在lucid的输出结果中与汇编进行对比。

