# 反混淆: OLLVM静态还原-1

样例二进制（借用一下OBPO）:

{% embed url="https://github.com/obpo-project/samples/blob/main/arm64/libnative-lib.so" %}

## 分发块

分发块的特点：

* 只对状态变量进行比较，不会进行赋值。
* 只进行比较与分发，没有执行真实的逻辑。



头部分发块的特点：

* 头部分发块一定有很多的入度，这些入度来自于真实块
* 头部分发块支配之后所有的块

## 真实块

真实块的特点：

* 可能含有真实的执行逻辑
* 执行后会对状态变量赋新值（如果不是末尾的返回块）。



## 块分类的功能实现

之前的代码已经能简单的实现状态变量的识别，这一次也一样，我们遍历所有的块+块中的代码，搜索指定的特征进行分类：

* 默认一个块不存在多种角色的可能性
* 如果有状态变量的比较+跳转，那就是分发块
* 如果有状态变量的赋值，那就是真实块
* 如果都没有，那就认为是无关的块，后面再处理

当然，还有另一种识别方法，在MMAT\_PREOPTIMIZED 级别优化后，IDA再每个块提供了的def-undef-use关系，可以直接判断块中的这个关系来进行分类。

关键代码有下面两部分，第一部分是通过任意一个mba可以遍历所有的basic-block

```
current_block = mba.get_mblock(0)
i = 0

# 既然获得了状态变量，通过对于状态变量的操作给块作记号（因为没有精确到block的instruction能力）
all_block_status = {}

while current_block.nextb != None:
    current_block: mblock_t = mba.get_mblock(i)
    # print(f"block-visitor: {hex(current_block.start)} - {hex(current_block.end)}")
    block_kind = check_block(current_block)
    # 通过block的define-use判断block的形态
    all_block_status[current_block] = block_kind
    i += 1
```

check\_block 函数的通过对于状态变量的def-undef-use关系，对block进行分类

注意，为了方便判断，这里写死了在bit-map中作为状态变量伪寄存器的bit号=72，完整的代码参考（todo需要完整的参考代码）

```
def check_block(one_block, reg_bit_value = 72) -> int:
    must_use = one_block.mustbuse
    may_use = one_block.maybuse
    must_def = one_block.mustbdef
    may_def = one_block.maybdef

    use_reg = False
    define_reg = False
    if must_use.reg.has(reg_bit_value) or may_use.reg.has(reg_bit_value):
        use_reg = True

    if must_def.reg.has(reg_bit_value) or may_def.reg.has(reg_bit_value):
        define_reg = True

    if use_reg and (not define_reg):
        # 使用但是不定义，这是分发块
        print(f"dispatch-block: {hex(one_block.start)} - {hex(one_block.end)}")
        return 2
    elif define_reg:
        # 定义了，要不是真实块，要不是头部块
        print(f"real-block: {hex(one_block.start)} - {hex(one_block.end)}")
        return 1
    else:
        print(f"unknown-block: {hex(one_block.start)} - {hex(one_block.end)}")
    return 0
```

当然，有的block既没有使用(must-use/may-use)，也没有定义(define/redefine)状态变量；我们需要仔细的判断OLLVM混淆的具体场景；在这个混淆的技术中，只要区分好分发块和真实块的边界就没问题了。

## 真实块中状态变量的值 - 分发块与真实块的分割点





## 真实块中状态变量的值 - 真实块中状态变量的变化



