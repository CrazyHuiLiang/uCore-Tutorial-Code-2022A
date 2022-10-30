# uCore-Tutorial-Code
test-20:34
Course project for THU-OS.

对标 [rCore-Tutorial-v3](https://github.com/rcore-os/rCore-Tutorial-v3/) 的 C 版本代码。

主要参考 [xv6-riscv](https://github.com/mit-pdos/xv6-riscv), [uCore-SMP](https://github.com/TianhuaTao/uCore-SMP)。

实验 lab1-lab5 基准代码分别位于 ch3-ch8　分支下。

注：为了兼容清华 Git 的需求、避免同学在主分支写代码、明确主分支的功能性，特意单独建了仅包含 README 与 LICENSE 的 master 分支，完成课程实验时请在 clone 仓库后先 push master 分支到清华 Git，然后切到自己开发所需的分支进行后续操作。

## 本地开发测试

在本地开发并测试时，需要拉取 uCore-Tutorial-Test-2022A 到 `user` 文件夹。你可以根据网络情况和个人偏好选择下列一项执行：

```bash
# 清华 git 使用 https
git clone https://git.tsinghua.edu.cn/os-lab/public/ucore-tutorial-test-2022a.git user
# 清华 git 使用 ssh
git clone git@git.tsinghua.edu.cn:os-lab/public/ucore-tutorial-test-2022a.git user
# GitHub 使用 https
git clone https://github.com/LearningOS/uCore-Tutorial-Test-2022A.git user
# GitHub 使用 ssh
git clone git@github.com:LearningOS/uCore-Tutorial-Test-2022A.git user
```

注意：`user` 已添加至 `.gitignore`，你无需将其提交，ci 也不会使用它


目录结构

```
bootloader 内核依赖的运行在M特权级的SBI实现
os 内核实现存放的目录
    console 将打印字符的 SBI 接口进一步封装实现更加强大的格式化输出
    entry   设置内核执行环境的一段汇编代码
    main    内核主函数
    sbi     调用底层SBI实现提供的SBI接口
```

## 总结理解

项目使用了汇编和c语言两种编程语言共同编写，使用makefile进行管理编译和链接过程，使用 riscv64-unknown-elf-gcc 进行交叉编译（在x86指令集架构之上编译riscv指令集架构的目标文件），riscv64-unknown-elf-ld进行链接。

对于部分特权指令的使用以及对sbi的调用只能使用汇编来写，这部分内容以内联汇编的形式写在riskv.h 和 sbi.c 文件中。

console.c 对 sbi.c 提供的字符输出能力进行一层间接的封装，当前并没有能力上的新增

printf.c 使用 console.c 的能力进一步提供了格式化输出函数 printf

log.c 基于printf.c 进一步提供了不同颜色/不同日志等级的输出函数

main.c 利用 printf,以及log.c中提供的输出函数进行字符串打印

entry.S 定义了 stack 空间，以及定义了 程序入口标签_entry，在_entry 中跳转main函数

kernel.ld 为链接项目的脚本，决定了elf程序的内存空间布局，定义了链接时的起始地址（0x80200000，此地址有rustsbi），设置入口地址为 entry.S 中定义的 _entry

另外，因为是在写操作系统内核而不是应用层软件，输出字符无法使用c标准库中的printf，需要使用我们自己基于sbi的实现，因此在编译源代码时需要设置 -nostdlib 来移除c标准库的依赖。

使用rustsbi作为 bios，链接生成的可执行文件作为内核，使用qemu-system-riscv64 来加载和执行。

