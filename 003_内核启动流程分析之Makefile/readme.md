# 内核Makefile

Linux内核Makefile的参考文档`Documentation/kbuild/makefiles.txt`

[参考文档](makefiles.txt)

Linux内核的`Makefile`可以分成五类：

| 名称 | 描述 |
| - | -|
| 顶层Makefile | 它是所有Makefile的核心，从总体上控制着内核的编译、链接 |
| `.config` | 配置文件，在配置内核时生成。所有Makefile（包括顶层目录及各级子目录）都是根据`.config`来决定使用哪些文件 |
| arch/$(ARCH)/Makefile | 对应体系结构的Makefile，它用来决定哪些体系结构相关的文件参与内核的生成，并提供一些规则来生成特定格式的内核镜像 |
| scripts/Makefile.* | Makefile共用的通用规则、脚本等 |
| kbuild Makefiles | 各级子目录下的Makefile，他们相对简单，被上一层的Makefile调用来编译当前各级目录下的文件 |

其中，`.config`上一节已经分析过了，接下来重点分析`顶层Makefile`，`架构Makefile`和`子目录Makefile`。

## 1. 子目录Makefile

 以`drivers/char/Makefile`为例。可以看到，子目录Makefile就是`obj-y`和`obj-m`的组合。

 ```makefile
obj-y	 += mem.o random.o tty_io.o n_tty.o tty_ioctl.o

obj-m    += s3c24xx_leds.o
obj-m    += s3c24xx_buttons.o
obj-m    += ker_rw.o
obj-$(CONFIG_LEGACY_PTYS)	+= pty.o
obj-$(CONFIG_UNIX98_PTYS)	+= pty.o
obj-y				+= misc.o
obj-$(CONFIG_VT)		+= vt_ioctl.o vc_screen.o consolemap.o \
				   consolemap_deftbl.o selection.o keyboard.o
obj-$(CONFIG_HW_CONSOLE)	+= vt.o defkeymap.o
obj-$(CONFIG_MAGIC_SYSRQ)	+= sysrq.o
obj-$(CONFIG_ESPSERIAL)		+= esp.o
obj-$(CONFIG_MVME147_SCC)	+= generic_serial.o vme_scc.o
obj-$(CONFIG_MVME162_SCC)	+= generic_serial.o vme_scc.o
obj-$(CONFIG_BVME6000_SCC)	+= generic_serial.o vme_scc.o
obj-$(CONFIG_ROCKETPORT)	+= rocket.o
obj-$(CONFIG_SERIAL167)		+= serial167.o
 ```

## 2. ARCH架构Makefile

我们使用的是arm架构，接下来分析架构Makefile。我们分析Makefile，就从他的命令来分析。我们编译内核时，直接输入`make`或`make uImage`。

![uImage](pic/001.jpg)

`uImage`位于`arch/arm`架构路径下，但我们是在顶层目录执行`make uImage`的，合理猜测一下，这个架构Makefile会被包含进顶层Makefile里面去。

可以看到，uImage依赖于vmlinux。

```makefile
zImage Image xipImage bootpImage uImage: vmlinux
	$(Q)$(MAKE) $(build)=$(boot) MACHINE=$(MACHINE) $(boot)/$@
```

## 3. 顶层Makefile

+ `默认目标` 执行`make`或`make all`就是生成`vmlinux`

    ```makefile
    # The all: target is the default when no target is given on the
    # command line.
    # This allow a user to issue only 'make' to build a kernel including modules
    # Defaults vmlinux but it is usually overridden in the arch makefile
    all: vmlinux
    ```

+ `包含auto.conf`

    ```makefile
    # Objects we will link into vmlinux / subdirs we need to visit
    init-y		:= init/
    drivers-y	:= drivers/ sound/
    net-y		:= net/
    libs-y		:= lib/
    core-y		:= usr/
    endif # KBUILD_EXTMOD

    ifeq ($(dot-config),1)
    # Read in config
    -include include/config/auto.conf
    ```

+ `包含架构Makefile`

    ```makefile
    # The all: target is the default when no target is given on the
    # command line.
    # This allow a user to issue only 'make' to build a kernel including modules
    # Defaults vmlinux but it is usually overridden in the arch makefile
    all: vmlinux

    ifdef CONFIG_CC_OPTIMIZE_FOR_SIZE
    COPTIMIZE	= -Os
    else
    COPTIMIZE	= -O2
    endif
    # COPTIMIZE may be overridden on the make command line with
    # 	make ... COPTIMIZE=""
    # The resulting object may be easier to debug with KGDB
    CFLAGS		+= $(COPTIMIZE)

    include $(srctree)/arch/$(ARCH)/Makefile    # 包含了架构Makefile
    ```

## 4. 终极目标`vmlinux`

可以看到，`vmlinux`依赖：`链接脚本` + `vmlinux-init` + `vmlinux-main`。

```makefile
# Build vmlinux
# ---------------------------------------------------------------------------
# vmlinux is built from the objects selected by $(vmlinux-init) and
# $(vmlinux-main). Most are built-in.o files from top-level directories
# in the kernel tree, others are specified in arch/$(ARCH)/Makefile.
# Ordering when linking is important, and $(vmlinux-init) must be first.
#
# vmlinux
#   ^
#   |
#   +-< $(vmlinux-init)
#   |   +--< init/version.o + more
#   |
#   +--< $(vmlinux-main)
#   |    +--< driver/built-in.o mm/built-in.o + more
#   |
#   +-< kallsyms.o (see description in CONFIG_KALLSYMS section)
#
# vmlinux version (uname -v) cannot be updated during normal
# descending-into-subdirs phase since we do not yet know if we need to
# update vmlinux.
# Therefore this step is delayed until just before final link of vmlinux -
# except in the kallsyms case where it is done just before adding the
# symbols to the kernel.
#
# System.map is generated to document addresses of all kernel symbols

vmlinux-init := $(head-y) $(init-y)
vmlinux-main := $(core-y) $(libs-y) $(drivers-y) $(net-y)
vmlinux-all  := $(vmlinux-init) $(vmlinux-main)
vmlinux-lds  := arch/$(ARCH)/kernel/vmlinux.lds

# vmlinux image - including updated kernel symbols
vmlinux: $(vmlinux-lds) $(vmlinux-init) $(vmlinux-main) $(kallsyms.o) FORCE
```

### 4.1 `vmlinux-init`

```makefile
# 顶层目录Makefile
vmlinux-init := $(head-y) $(init-y)

init-y		:= init/
init-y		:= $(patsubst %/, %/built-in.o, $(init-y))  # = init/built-in.o
# 这说明init目录下所有的文件，会被编译成一个built-in.o

# 子目录Makefile
head-y		:= arch/arm/kernel/head$(MMUEXT).o arch/arm/kernel/init_task.o
```

### 4.2 `vmlinux-main`

```makefile
# 顶层目录Makefile
vmlinux-main := $(core-y) $(libs-y) $(drivers-y) $(net-y)

core-y		:= usr/
core-y		+= kernel/ mm/ fs/ ipc/ security/ crypto/ block/
core-y		:= $(patsubst %/, %/built-in.o, $(core-y))
# $(core-y) = usr/built-in.o kernel/built-in.o mm/built-in.o fs/built-in.o ipc/built-in.o security/built-in.o crypto/built-in.o block/built-in.o
# 所有这些目录下面涉及的文件，都会编译成built-in.o

libs-y		:= lib/
libs-y		:= $(libs-y1) $(libs-y2)
libs-y1		:= $(patsubst %/, %/lib.a, $(libs-y))
libs-y2		:= $(patsubst %/, %/built-in.o, $(libs-y))
# $(libs-y) = lib/lib.a lib/built-in.o

drivers-y	:= drivers/ sound/
drivers-y	:= $(patsubst %/, %/built-in.o, $(drivers-y))
# $(drivers-y) = drivers/built-in.o sound/built-in.o

net-y		:= net/
net-y		:= $(patsubst %/, %/built-in.o, $(net-y))
# $(net-y) = net/built-in.o
```

vmlinux-main依赖总结如下：

| 变量 | 值 |
| :-: | - |
| $(head-y) | arch/arm/kernel/head$(MMUEXT).o arch/arm/kernel/init_task.o |
| $(init-y) | init/built-in.o |
| $(core-y) |  usr/built-in.o kernel/built-in.o mm/built-in.o fs/built-in.o ipc/built-in.o security/built-in.o crypto/built-in.o block/built-in.o |
| $(libs-y) | lib/lib.a lib/built-in.o |
| $(drivers-y) | drivers/built-in.o sound/built-in.o |
| $(net-y) | net/built-in.o |

那么这一大堆目标文件，怎么链接到一起的？

### 4.3 `查看编译详细信息`

执行`make V=1`，就能打印编译的详细过程：

```sh
  arm-linux-ld -EL  -p --no-undefined -X -o vmlinux -T arch/arm/kernel/vmlinux.lds arch/arm/kernel/head.o arch/arm/kernel/init_task.o  init/built-in.o --start-group  usr/built-in.o  arch/arm/kernel/built-in.o  arch/arm/mm/built-in.o  arch/arm/common/built-in.o  arch/arm/mach-s3c2410/built-in.o  arch/arm/mach-s3c2400/built-in.o  arch/arm/mach-s3c2412/built-in.o  arch/arm/mach-s3c2440/built-in.o  arch/arm/mach-s3c2442/built-in.o  arch/arm/mach-s3c2443/built-in.o  arch/arm/nwfpe/built-in.o  arch/arm/plat-s3c24xx/built-in.o  kernel/built-in.o  mm/built-in.o  fs/built-in.o  ipc/built-in.o  security/built-in.o  crypto/built-in.o  block/built-in.o  arch/arm/lib/lib.a  lib/lib.a  arch/arm/lib/built-in.o  lib/built-in.o  drivers/built-in.o  sound/built-in.o  net/built-in.o --end-group .tmp_kallsyms2.o
```

可以看到：

+ 链接脚本：`arch/arm/kernel/vmlinux.lds`
+ 第一个文件：arch/arm/kernel/head.S

## 5. 链接脚本

首先是放`text.head`段，然后是`init.text`段，依次排下来，这就是Makefile的全部内容。

```lds
SECTIONS
{
    . = (0xc0000000) + 0x00008000;

    .text.head : {
        _stext = .;
        _sinittext = .;
        *(.text.head)
    }

    .init : { /* Init code and data		*/
        *(.init.text)
        _einittext = .;
        __proc_info_begin = .;
        *(.proc.info.init)
        __proc_info_end = .;
        __arch_info_begin = .;
        *(.arch.info.init)
        __arch_info_end = .;
        __tagtable_begin = .;
        *(.taglist.init)
        __tagtable_end = .;
        . = ALIGN(16);
        __setup_start = .;
        *(.init.setup)
        __setup_end = .;
        __early_begin = .;
        *(.early_param.init)
        __early_end = .;
        __initcall_start = .;
        *(.initcall0.init) *(.initcall0s.init) *(.initcall1.init) *(.initcall1s.init) *(.initcall2.init) *(.initcall2s.init) *(.initcall3.init) *(.initcall3s.init) *(.initcall4.init) *(.initcall4s.init) *(.initcall5.init) *(.initcall5s.init) *(.initcallrootfs.init) *(.initcall6.init) *(.initcall6s.init) *(.initcall7.init) *(.initcall7s.init)
        __initcall_end = .;
        __con_initcall_start = .;
        *(.con_initcall.init)
        __con_initcall_end = .;
        __security_initcall_start = .;
        *(.security_initcall.init)
        __security_initcall_end = .;

        . = ALIGN(32);
        __initramfs_start = .;
        usr/built-in.o(.init.ramfs)
        __initramfs_end = .;

        . = ALIGN(4096);
        __per_cpu_start = .;
        *(.data.percpu)
        __per_cpu_end = .;

        __init_begin = _stext;
        *(.init.data)
        . = ALIGN(4096);
        __init_end = .;
    }

    /DISCARD/ : { /* Exit code and data		*/
        *(.exit.text)
        *(.exit.data)
        *(.exitcall.exit)
    }

    .text : { /* Real text segment		*/
        _text = .; /* Text and read-only data	*/
        __exception_text_start = .;
        *(.exception.text)
        __exception_text_end = .;
        . = ALIGN(8); *(.text) *(.text.init.refok)
        . = ALIGN(8); __sched_text_start = .; *(.sched.text) __sched_text_end = .;
        . = ALIGN(8); __lock_text_start = .; *(.spinlock.text) __lock_text_end = .;

        *(.fixup)

        *(.gnu.warning)
        *(.rodata)
        *(.rodata.*)
        *(.glue_7)
        *(.glue_7t)
        *(.got) /* Global offset table		*/
    }

    . = ALIGN((4096)); .rodata : AT(ADDR(.rodata) - 0) { __start_rodata = .; *(.rodata) *(.rodata.*) *(__vermagic) } .rodata1 : AT(ADDR(.rodata1) - 0) { *(.rodata1) } .pci_fixup : AT(ADDR(.pci_fixup) - 0) { __start_pci_fixups_early = .; *(.pci_fixup_early) __end_pci_fixups_early = .; __start_pci_fixups_header = .; *(.pci_fixup_header) __end_pci_fixups_header = .; __start_pci_fixups_final = .; *(.pci_fixup_final) __end_pci_fixups_final = .; __start_pci_fixups_enable = .; *(.pci_fixup_enable) __end_pci_fixups_enable = .; __start_pci_fixups_resume = .; *(.pci_fixup_resume) __end_pci_fixups_resume = .; } .rio_route : AT(ADDR(.rio_route) - 0) { __start_rio_route_ops = .; *(.rio_route_ops) __end_rio_route_ops = .; } __ksymtab : AT(ADDR(__ksymtab) - 0) { __start___ksymtab = .; *(__ksymtab) __stop___ksymtab = .; } __ksymtab_gpl : AT(ADDR(__ksymtab_gpl) - 0) { __start___ksymtab_gpl = .; *(__ksymtab_gpl) __stop___ksymtab_gpl = .; } __ksymtab_unused : AT(ADDR(__ksymtab_unused) - 0) { __start___ksymtab_unused = .; *(__ksymtab_unused) __stop___ksymtab_unused = .; } __ksymtab_unused_gpl : AT(ADDR(__ksymtab_unused_gpl) - 0) { __start___ksymtab_unused_gpl = .; *(__ksymtab_unused_gpl) __stop___ksymtab_unused_gpl = .; } __ksymtab_gpl_future : AT(ADDR(__ksymtab_gpl_future) - 0) { __start___ksymtab_gpl_future = .; *(__ksymtab_gpl_future) __stop___ksymtab_gpl_future = .; } __kcrctab : AT(ADDR(__kcrctab) - 0) { __start___kcrctab = .; *(__kcrctab) __stop___kcrctab = .; } __kcrctab_gpl : AT(ADDR(__kcrctab_gpl) - 0) { __start___kcrctab_gpl = .; *(__kcrctab_gpl) __stop___kcrctab_gpl = .; } __kcrctab_unused : AT(ADDR(__kcrctab_unused) - 0) { __start___kcrctab_unused = .; *(__kcrctab_unused) __stop___kcrctab_unused = .; } __kcrctab_unused_gpl : AT(ADDR(__kcrctab_unused_gpl) - 0) { __start___kcrctab_unused_gpl = .; *(__kcrctab_unused_gpl) __stop___kcrctab_unused_gpl = .; } __kcrctab_gpl_future : AT(ADDR(__kcrctab_gpl_future) - 0) { __start___kcrctab_gpl_future = .; *(__kcrctab_gpl_future) __stop___kcrctab_gpl_future = .; } __ksymtab_strings : AT(ADDR(__ksymtab_strings) - 0) { *(__ksymtab_strings) } __param : AT(ADDR(__param) - 0) { __start___param = .; *(__param) __stop___param = .; __end_rodata = .; } . = ALIGN((4096));

    _etext = .; /* End of text and rodata section */

    . = ALIGN(8192);
    __data_loc = .;


    .data : AT(__data_loc) {
        __data_start = .; /* address in memory */

        /*
        * first, the init task union, aligned
        * to an 8192 byte boundary.
        */
        *(.data.init_task)
        . = ALIGN(4096);
        __nosave_begin = .;
        *(.data.nosave)
        . = ALIGN(4096);
        __nosave_end = .;

        /*
        * then the cacheline aligned data
        */
        . = ALIGN(32);
        *(.data.cacheline_aligned)

        /*
        * The exception fixup table (might need resorting at runtime)
        */
        . = ALIGN(32);
        __start___ex_table = .;

        *(__ex_table)

        __stop___ex_table = .;

        /*
        * and the usual data section
        */
        *(.data) *(.data.init.refok)
        CONSTRUCTORS

        _edata = .;
    }
    _edata_loc = __data_loc + SIZEOF(.data);

    .bss : {
        __bss_start = .; /* BSS				*/
        *(.bss)
        *(COMMON)
        _end = .;
    }
    /* Stabs debugging sections.	*/
    .stab 0 : { *(.stab) }
    .stabstr 0 : { *(.stabstr) }
    .stab.excl 0 : { *(.stab.excl) }
    .stab.exclstr 0 : { *(.stab.exclstr) }
    .stab.index 0 : { *(.stab.index) }
    .stab.indexstr 0 : { *(.stab.indexstr) }
    .comment 0 : { *(.comment) }
}
```
