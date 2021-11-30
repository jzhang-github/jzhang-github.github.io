# POSCAR 和 arc 文件相互转换

## 1、打印帮助文档
```
python  PosAndArc.py
```
### 输出：
```
Traceback (most recent call last):
  File "PosAndArc.py", line 209, in <module>
    assert len(argv) > 1, "\nPlease specify a file name. \n\nUsage: Command + file_name + lasp_in_file (optional)\nfile_name: a file you want to convert. The file format will be determined automatically.\nlasp_in_file: an input file of lasp. By default: lasp.in.\n\nYou can also find a mannual at: \n\nFor now, this script can only convert a cuboid box."
AssertionError:
Please specify a file name.

Usage: Command + file_name + lasp_in_file (optional)
file_name: a file you want to convert. The file format will be determined automatically.
lasp_in_file: an input file of lasp. By default: lasp.in.

You can also find a mannual at: https://github.com/jzhang-github/jzhang-github.github.io/blob/gh-pages/for_phu/PosAndArc.md

For now, this script can only convert a cuboid box.
```
### 参数说明：
```file_name```：POSCAR或arc输入文件名称（必填）。

```lasp_in_file```：Lasp in文件名称（选填）。若不填，默认读取```lasp.in```文件。

## 2、POSCAR转化为arc文件
```
python  PosAndArc.py POSCAR
```
### 屏幕输出：
```
Converting vasp file to arc file...
```
### 文件输出：
```lasp.in```：若工作文件夹内包含同名文件，则在新文件名后追加```_new```，以免覆盖。
## 3、arc转化为POSCAR文件
```
python  PosAndArc.py input.arc
```
### 屏幕输出：
```
Converting arc file to vasp file...
```
### 文件输出：
```POSCAR```：若工作文件夹内包含同名文件，则在新文件名后追加```_new```，以免覆盖。

## 其他：
脚本默认文件名称：
* VASP格式：```POSCAR```
* Material Studio 格式：```input.arc```
* LASP格式：```lasp.in```
