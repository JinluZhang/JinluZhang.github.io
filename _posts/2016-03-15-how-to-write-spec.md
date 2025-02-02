---
title: '肿么写spec文件'
date: 2016-03-15
categories: 技术
author: Beaug
tags: spec
---

# 1. RPM包

RPM（Red Hat Package Manager），几乎所有的 Linux 发行版本都使用这种形式的软件包管理安装、更新和卸载软件。

RPM的发布基于GPL协议。

RPM包里面包含可执行的二进制程序，程序运行时所需要的文件，其它特定版本文件(软件包的依赖关系)等

安装：rpm -ivh

删除：rpm -e

升级：rpm -Uvh

查询：rpm -qpi 描述信息 ; －qpl 文件信息

# 2. spec文件 

若要构建一个标准的 RPM 包，需要创建 .spec 文件，其中包含软件打包的全部信息。然后，对此文件执行 rpmbuild 命令，经过这一步，系统会按照步骤生成最终的 RPM 包。 

 把源代码包(如以 .tar.gz 结尾的文件)放入 ~/rpmbuild/SOURCES 目录。将.spec 文件放入 ~/rpmbuild/SPECS 目录，并命名为 "*软件包名*.spec" 。当然， *软件包名* 就是最终 RPM 包的名字。

 打包命令为：将目录切换至 ~/rpmbuild/SPECS 并执行

$ rpmbuild -ba *NAME*.spec

当执行此命令时，rpmbuild 会自动读取 .spec 文件并按照下表列出的步骤完成构建。

## 2.1. 宏

下表中，以 % 开头的语句为预定义宏，每个宏的作用如下：

| 阶段     | 读取的文件     | 写入的目录     | 具体动作                                                     |
| -------- | -------------- | -------------- | ------------------------------------------------------------ |
| %prep    | %_sourcedir    | %_builddir     | 读取位于 %_sourcedir 目录的源代码和 patch 。之后，解压源代码至 %_builddir 的子目录并应用所有 patch。 |
| %build   | %_builddir     | %_builddir     | 编译位于 %_builddir 构建目录下的文件。通过执行类似 "./configure && make" 的命令实现。 |
| %install | %_builddir     | %_buildrootdir | 读取位于 %_builddir 构建目录下的文件并将其安装至 %_buildrootdir 目录。这些文件就是用户安装 RPM 后，最终得到的文件。注意一个奇怪的地方: *最终安装目录* **不是** *构建目录*。通过执行类似 "make install" 的命令实现。 |
| %check   | %_builddir     | %_builddir     | 检查软件是否正常运行。通过执行类似 "make test" 的命令实现。很多软件包都不需要此步。 |
| bin      | %_buildrootdir | %_rpmdir       | 读取位于 %_buildrootdir 最终安装目录下的文件，以便最终在 %_rpmdir 目录下创建 RPM 包。在该目录下，不同架构的 RPM 包会分别保存至不同子目录， "noarch" 目录保存适用于所有架构的 RPM 包。这些 RPM 文件就是用户最终安装的 RPM 包。 |
| src      | %_sourcedir    | %_srcrpmdir    | 创建源码 RPM 包（简称 SRPM，以.src.rpm 作为后缀名），并保存至 %_srcrpmdir 目录。SRPM 包通常用于审核和升级软件包。 |

在 rpmbuild 中，对上表中的每个宏代码都有对应的目录：

 

| 宏代码         | 名称              | 默认位置             | 用途                                         |
| -------------- | ----------------- | -------------------- | -------------------------------------------- |
| %_specdir      | Spec 文件目录     | ~/rpmbuild/SPECS     | 保存 RPM 包配置（.spec）文件                 |
| %_sourcedir    | 源代码目录        | ~/rpmbuild/SOURCES   | 保存源码包（如 .tar 包）和所有 patch 补丁    |
| %_builddir     | 构建目录          | ~/rpmbuild/BUILD     | 源码包被解压至此，并在该目录的子目录完成编译 |
| %_buildrootdir | 最终安装目录      | ~/rpmbuild/BUILDROOT | 保存 %install 阶段安装的文件                 |
| %_rpmdir       | 标准 RPM 包目录   | ~/rpmbuild/RPMS      | 生成/保存二进制 RPM 包                       |
| %_srcrpmdir    | 源代码 RPM 包目录 | ~/rpmbuild/SRPMS     | 生成/保存源码 RPM 包(SRPM)                   |

| 宏名称             | 典型扩展             | 意义                                                         |
| ------------------ | -------------------- | ------------------------------------------------------------ |
| %{_bindir}         | /usr/bin             | 二进制目录：保存可执行文件                                   |
| %{_builddir}       | ~/rpmbuild/BUILD     | 构建目录：软件在 build 的子目录被编译。参考 %buildsubdir     |
| %{buildroot}       | ~/rpmbuild/BUILDROOT | Build root：%install 阶段中，将 %{_builddir} 子目录下的文件复制到 %{buildroot} 的子目录（之前，%{buildroot} 使用的位置为 "/var/tmp/"） |
| %{buildsubdir}     | %{_builddir}/%{name} | 构建子目录：%build 阶段中，文件会在 %{_builddir} 的子目录中编译。此宏在 %autosetup 之后设置 |
| %{_datadir}        | /usr/share           | 共享数据目录                                                 |
| %{_defaultdocdir}  | /usr/share/doc       | 默认文档目录                                                 |
| %{dist}            | .fc*NUMBER*          | 发行版名称+版本号（例如 ".fc23"）                            |
| %{fedora}          | *NUMBER*             | Fedora 发行版本号（例如 "23"）                               |
| %{_includedir}     | /usr/include         | 程序头文件目录                                               |
| %{_infodir}        | /usr/share/info      | info 手册目录                                                |
| %{_initrddir}      | /etc/rc.d/init.d     | init 脚本目录                                                |
| %{_libdir}         | /usr/lib             | 共享库目录                                                   |
| %{_libexecdir}     | /usr/libexec         | 仅由系统调用执行该目录中的命令                               |
| %{_localstatedir}  | /var                 | 保存缓存/日志/lock等信息的目录                               |
| %{_mandir}         | /usr/share/man       | man 手册目录                                                 |
| %{name}            |                      | 软件包名称，通过 Name: tag 设置                              |
| %{_sbindir}        | /usr/sbin            | 保存管理员可执行命令                                         |
| %{_sharedstatedir} | /var/lib             | 保存程序运行所处理的文件                                     |
| %{_sysconfdir}     | /etc                 | 配置文件目录                                                 |
| %{version}         |                      | 软件包版本，通过 Version: tag 设置                           |

 

## 2.2. 标签

- **Name**: 软件包名，应与 SPEC 文件名一致。命名必须符合 [软件包命名规定](https://fedoraproject.org/wiki/Packaging/NamingGuidelines)。
- **Version**: 上游版本号。请查看 [版本标签规定](https://fedoraproject.org/wiki/Packaging/NamingGuidelines#Version_Tag)。如果包含非数字字符，您可能需要将它们包含在 Release 标签中。如果上游采用日期作为版本号，请考虑以：[yy.mm](http://yy.mm/)[dd] (例如 2008-05-01 可变为8.05) 格式作为版本号。
- **Release**: 发行编号。初始值为 1%{?dist}。每次制作新包时，请递增该数字。当上游发布新版本时，请修改 Version 标签并重置 Release 的数字为 1。具体参考打包规定中的 [Release 标签部分](https://fedoraproject.org/wiki/Packaging/NamingGuidelines#Release_Tag)，以及[Dist tag](https://fedoraproject.org/wiki/Packaging/DistTag)。
- **Summary**: 一行简短的软件包介绍。请使用美式英语。**请勿在结尾添加标点！**
- **Group**: 指定软件包组，例如 "Applications/Engineering"；执行 "less /usr/share/doc/rpm-*/GROUPS" 查看完整的组列表。任何包含文档的子软件包，使用 "Documentation" 组（如kernel-doc）。***注意 Fedora 17+ 后已废除此标签。***[***Spec 文件参考手册***](http://docs.fedoraproject.org/en-US/Fedora_Draft_Documentation/0.1/html/Packagers_Guide/chap-Packagers_Guide-Spec_File_Reference-Preamble.html) ***有介绍***
- **License**: 授权协议，必须是开源许可证。请*不要*使用旧的 Copyright 标签。协议采用标准缩写（如 "GPLv2+"）并且描述明确（如， "GPLv2+" 表示 GPL 2 及后续版本，而不是 "GPL" 或 "GPLv2" 这种不准确的写法)。参考 [Licensing](https://fedoraproject.org/wiki/Licensing) 和 [Licensing Guidelines](https://fedoraproject.org/wiki/Packaging/LicensingGuidelines)。如果一个软件采用多个协议，可以使用 "and" 和 "or"（例如 "GPLv2 and BSD"）来描述。
- **URL**: 该软件包的项目主页。***注意：源码包 URL 请使用 Source0 指定。***
- **Source0**: 软件源码包的 URL 地址。"Source" 与 "Source0" 相同。**强烈建议**提供完整 URL 地址，文件名用于查找 SOURCES 目录。如果可能，建议使用 %{name} 和 %{version} 替换 URL 中的名称/版本，这样更新时就会自动对应。下载源码包时，需要 [保留时间戳](https://fedoraproject.org/wiki/Packaging:Guidelines#Timestamps)。如果有多个源码包，请用 Source1，Source2 等依次列出。如果你需要添加额外文件，请将它们列在后面。更多特殊案例（如 revision control），请参考 [Source URL](https://fedoraproject.org/wiki/Packaging/SourceURL)。
- **Patch0**: 用于源码的补丁名称。如果你需要在源码包解压后对一些代码做修改，你应该修改代码并使用 diff 命令生成 patch 文件，然后放在 ~/rpmbuild/SOURCES 目录下。一个 Patch 应该只做一种修改，所以可能会包含多个 patch 文件。
- **BuildArch**: 如果你要打包的文件不依赖任何架构（例如 shell 脚本，数据文件），请使用 "BuildArch: noarch"。RPM 架构会变成 "noarch"。
- **BuildRoot**: 在 %install 阶段（%build 阶段后）文件需要安装至此位置。Fedora 不需要此标签，只有 EPEL5 还需要它。默认情况下，根目录为 "%{_topdir}/BUILDROOT/"。
- **BuildRequires**: 编译软件包所需的依赖包列表，以逗号分隔。此标签可以多次指定。编译依赖 *不会* 自动判断，所以需要列出编译所需的*所有*依赖包。[常见的软件包可省略](https://fedoraproject.org/wiki/Packaging/Guidelines#Exceptions_2)，例如 gcc。如果有必要，你可以指定需要的最低版本（例："ocaml >= 3.08"）。如果你需要找到包含 /EGGS 文件的软件包，可执行 "rpm -qf /EGGS"。如果你需要找到包含 EGGS 程序的软件包，可执行 "rpm -qf `which EGGS`"。请保持最小依赖（例如，如果你不需要 perl 的功能，可使用 sed 代替），但请注意，如果不包含相关依赖，某些程序会禁用一些功能；此时，你需要添加这些依赖。[auto-buildrequires](https://apps.fedoraproject.org/packages/auto-buildrequires) 软件包可能会有帮助。
- **Requires**: 安装软件包时所需的依赖包列表，以逗号分隔。请注意， BuildRequires 标签是编译所需的依赖，而 Requires 标签是安装/运行程序所需的依赖。大多数情况下，rpmbuild 会自动探测依赖，所以可能不需要 Requires 标签。然而，你也可以明确标明需要哪些软件包，或由于未自动探测所需依赖而需要手动标明。
- **%description**: 程序的详细/多行描述，请使用美式英语。每行必须小于等于 80 个字符。空行表示开始新段落。使用图形安装软件时会重新格式化段落；以空格开头的行被视为已格式化的格式，一般使用等宽字体显示。参考 [RPM Guide](http://docs.fedoraproject.org/drafts/rpm-guide-en/ch09s03.html)。
- **%prep**: 打包准备阶段执行一些命令（如，解压源码包，打补丁等），以便开始编译。一般仅包含 "%autosetup"；如果源码包需要解压并切换至 NAME 目录，则输入 "%autosetup -n NAME"。查看 %prep 部分了解更多信息。
- **%build**: 包含构建阶段执行的命令，构建完成后便开始后续安装。程序应该包含有如何编译的介绍。查看 %build 部分了解更多信息。
- **%install**: 包含安装阶段执行的命令。命令将文件从 %{_builddir} 目录安装至 %{buildroot} 目录。查看 %install 部分了解更多信息。
- **%check**: 包含测试阶段执行的命令。此阶段在 %install 之后执行，通常包含 "make test" 或 "make check" 命令。此阶段要与 %build 分开，以便在需要时忽略测试。
- **%clean**: 清理安装目录的命令。此阶段在 Fedora 中是多余的，仅针对 EPEL。一般只包含：rm -rf %{buildroot}
- **%files**: 需要被打包/安装的文件列表。查看 %files 部分了解更多信息。
- **%changelog**: RPM 包变更日志。请使用示例中的格式。**注意，不是软件本身的变更日志。**
- **ExcludeArch**: 排除某些架构。如果该软件不能在某些架构上正常编译或工作，通过该标签列出。
- **ExclusiveArch**: 列出该软件包独占的架构。
- 你可以加入一些代码片段，以便在真实系统上安装/删除包时执行这些代码（相反，%install 脚本仅将文件虚拟【pseudo】安装至 build root 目录）。这些代码称为 "scriptlets"，通常用于从软件包更新系统信息。查看 "Scriptlets" 部分了解更多信息。



# 3. 参考资料

https://fedoraproject.org/wiki/How_to_create_an_RPM_package/zh-cn

https://fedoraproject.org/wiki/How_to_create_a_GNU_Hello_RPM_package/zh-cn

http://docs.fedoraproject.org/en-US/Fedora_Draft_Documentation/0.1/html/RPM_Guide/ch-creating-rpms.html
