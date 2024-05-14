# 内核Makefile

Linux内核的`Makefile`可以分成五类：

| 名称 | 描述 |
| - | -|
| 顶层Makefile | 它是所有Makefile的核心，从总体上控制着内核的编译、链接 |
| `.config` | 配置文件，在配置内核时生成。所有Makefile（包括顶层目录及各级子目录）都是根据`.config`来决定使用哪些文件 |
| arch/$(ARCH)/Makefile | 对应体系结构的Makefile，它用来决定哪些体系结构相关的文件参与内核的生成，并提供一些规则来生成特定格式的内核镜像 |
| scripts/Makefile.* | Makefile共用的通用规则、脚本等 |
| kbuild Makefiles | 各级子目录下的Makefile，他们相对简单，被上一层的Makefile调用来编译当前各级目录下的文件 |


