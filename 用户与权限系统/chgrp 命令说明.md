# 前言

不像 `chown` 命令可以同时修改文件的所属者以及所属组，`chgrp` 仅仅只能修改文件的所属组。所以在使用时我们可以将 `chown` 和 `chgrp` 命令进行区分开来使用，仅使用 `chown` 修改文件的所属者，使用 `chgrp` 修改文件的所属组。

`chgrp` 语法如下：

```bash
chgrp [-Rh] [GROUP] FILE...
```

可选参数说明：

```
-R,-recursive         递归修改所属组, 主要用于文件夹
-h,--no-dereference   修改符号链接本身, 而不是所链接的具体文件
```

另外 `[GROUP]` 指定具体组，可以是组的名称也可以是 GID，但要特别注意，如果是 GID 时需要在 GID 的前面加个 `+` 以做区分，否则会当做组的名称处理。示例：

```bash
chgrp +100 examplefile
```

# 修改普通文件所属组

这个特别简单，不需要加任何参数。示例命令如下：

```bash
$ chgrp www-data filename
```

另外，也可以同时指定修改多个文件：

```bash
$ chgrp www-data file1 file2 file3
```

但是有一点需要注意的是，如果你指定的组是一个具体的 GID，一定要在 G1D 前面加上 `+`，否则会当做组的名来处理。比如组 $GID_{www-data}=1000$，使用GID 的命令就为：

```bash
$ sudo chgrp +1000 filename
```

# 修改符号链接文件所属组

当一个文件是一个符号链接时，`chgrp` 命令的默认行为是修改符号链接指向的目标文件，而不是符号链接本身。比如在有如下两个文件，其中 rd 是文件 readme 的符号链接：

```bash
$ ls -l
lrwxrwxrwx  1 root root    6 12月 19 20:09 rd -> readme
-rw-r--r--  1 root root    0 12月 19 20:09 readme
```

当我们修改 rd 的文件所属组时其实修改是目标文件 readme 的所属组，示例：

```bash
$ sudo chgrp www-data rd

$ ls -l
lrwxrwxrwx  1 root root        6 12月 19 20:09 rd -> readme
-rw-r--r--  1 root www-data    0 12月 19 20:09 readme
```

显而易见，真正改编的是文件 readme 而不是符号链接 rd。

|**注意**|
|:------|
|在某些 Linux 发行版上，当你 `chgrp` 修改符号链接的所属组时可能会得到一个错误： `“cannot dereference ‘symlink1’: Permission denied” error`。发生该错误是因为默认情况下在大多数 Linux 发行版上，符号链接受到保护，并且你无法对目标文件进行操作。 该行为定义在文件 `/proc/sys/fs/protected_symlinks` 中，`1` 表示开启保护， `0` 表示关闭保护。当然了，并不推荐关闭该保护行为。|

对于符号链接 `chgrp` 命令的默认行为是修改目标文件这个问题该怎么解决呢？如我实际上就是想修改符号链接的所属组咋整？

使用 `-h` 参数即可！

示例：

```bash
$ sudo chgrp -h www-data rd

$ ls -l
lrwxrwxrwx  1 root www-data    6 12月 19 20:09 rd -> readme
-rw-r--r--  1 root www-data    0 12月 19 20:09 readme
```

现在修改的就是符号链接本身了~


# 递归修改文件所属组

如果修改的文件是一个目录，在修改目录的所属组时想将该目录内的文件的所属组也一并修改就可以使用递归参数 `-R` 来实现。比如有一个目录 `/var/www`，想将该目录以及目录内的所有文件的所属组都修改为 `www-data`，就可以直接使用下面的命令来实现：

```bash
$ sudo chgrp -R www-data /var/www
```

另外，如果你修改的是一个符号链接，该符号链接指向的是一个目录时，还需要加上 `-h` 参数。示例：

```bash
$ ls
lrwxrwxrwx  1 root root    6 12月 19 20:09 dr -> /var/www
```

递归修改符号链接所属组权限：

```bash
$ sudo chgrp -Rh www-data dr
```

--

完结，撒花🎉🎉🎉~