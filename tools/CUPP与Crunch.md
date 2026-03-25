# 一、引言

在密码爆破（如使用 [[Hydra]]）中，字典的质量直接影响成功率。CUPP 和 Crunch 是两款经典的字典生成工具：**CUPP 擅长根据个人信息“定制”密码**，**Crunch 擅长按规则“穷举”所有可能组合**。两者结合使用，可极大提升爆破效率。

# 二、CUPP（Common User Passwords Profiler）

## 2.1 概念
CUPP 是一款基于**社会工程学**的密码字典生成工具。它通过收集目标的人物信息（**姓名**、**生日**、**宠物名**等），根据常见密码构造模式生成针对性极强的密码列表。
## 2.2 工作原理
CUPP 执行交互式问卷，依次询问目标的姓名、昵称、生日、伴侣姓名、子女姓名、宠物名、公司名、键盘习惯等信息，然后将这些信息组合、变形（大小写、加年份、加特殊符号等），最终生成一个按概率排序的字典文件。
## 2.3 环境要求与安装
- **Kali Linux**：默认已安装，直接使用 `cupp` 命令。
- **手动安装**：
```bash
git clone https://github.com/Mebus/cupp.git
cd cupp
chmod +x cupp.py
```
## 2.4 基本使用方法（语法格式）

CUPP 的使用方式简单，主要分为交互模式、下载常用字典模式、结合已有字典模式。
```bash
cupp [选项]
```
**示例**：
```bash
# 交互模式（推荐）
cupp -i
```
**说明**：`-i` 启动交互式问答，根据输入生成定制字典。
## 2.5 常用选项

|选项|说明|示例|
|---|---|---|
|`-i`|交互式生成字典|`cupp -i`|
|`-w`|使用已有的字典文件作为基础，生成变体|`cupp -w passlist.txt`|
|`-l`|从互联网下载常用弱口令字典（如rockyou）|`cupp -l`|
|`-a`|仅用默认规则生成，不交互|`cupp -a`|

## 2.6 进阶用法

**1. 结合 [[Hydra]] 使用**
```bash
# 先生成针对某用户的字典
cupp -i
# 输出文件通常为 username.txt，可直接用于 Hydra
hydra -l target_user -P username.txt ssh://192.168.1.100
```
**2. 与现有字典合并去重**
```bash
# 将 CUPP 生成的字典与其他字典合并并去重
cat cupp_dict.txt rockyou.txt | sort -u > final_dict.txt
```
**3. 自定义模板**  
CUPP 的配置文件 `cupp.cfg` 可修改密码生成规则（如最小长度、是否包含年份等）。

# 三、Crunch
## 3.1 概念
Crunch 是一款基于字符集和长度范围的密码字典生成工具。它可以按指定规则生成所有可能的排列组合，适合已知密码格式的场景（如“密码是8位数字”或“以admin开头后跟3位数字”）。
## 3.2 工作原理
Crunch 根据用户给出的最小长度、最大长度、字符集，通过递归或迭代方式生成所有可能的字符串组合，并输出到文件或标准输出。它支持指定模式（如 `@@@@` 表示4个任意字符）和多种输出格式。

## 3.3 环境要求与安装

- **Kali Linux**：默认已安装，直接使用 `crunch` 命令。
    
- **手动安装**：
    
    bash
    
    复制
    
    下载
    
    sudo apt install crunch   # Debian/Ubuntu
    

## 3.4 基本使用方法（语法格式）

Crunch 的基本语法如下：

bash

复制

下载

crunch <min-len> <max-len> <charset> -o <输出文件>

**示例**：

bash

复制

下载

# 生成所有4位小写字母组合，输出到 words.txt
crunch 4 4 abcdefghijklmnopqrstuvwxyz -o words.txt

**说明**：`<min-len>` 和 `<max-len>` 是密码长度范围，`<charset>` 是字符集。

### 3.5 常用选项

|选项|说明|示例|
|---|---|---|
|`-o`|指定输出文件|`-o dict.txt`|
|`-t`|指定密码模式（@代表小写字母，%代表数字，^代表特殊符号）|`crunch 8 8 -t admin%%%`|
|`-p`|生成指定字符串的所有排列（不指定长度）|`crunch 0 0 -p 123 abc`|
|`-z`|输出时使用 gzip 压缩|`-z gzip`|
|`-b`|限制输出文件大小（MB），配合 `-o START` 分割|`-b 20mb -o START`|
|`-l`|指定模式中的占位符映射（当 -t 中包含特殊字符时使用）|`-l @%^`|
|`-s`|从指定字符串开始生成|`-s admin`|
|`-e`|在指定字符串处结束|`-e admin999`|

### 3.6 进阶用法

**1. 模式生成（最常用）**

bash

复制

下载

# 生成 8 位密码，前四位固定 "pass"，后四位为数字
crunch 8 8 -t pass%%%% -o pass_digits.txt
# 输出：pass0000, pass0001, ..., pass9999

**2. 字符集简写**

bash

复制

下载

# 使用内置字符集：lower（小写），upper（大写），numeric（数字），symbols（符号）
crunch 4 4 lower -o lower.txt          # 所有4位小写字母
crunch 6 6 numeric -o digits.txt       # 所有6位数字

**3. 排列生成（不重复排列）**

bash

复制

下载

# 生成 "123", "abc", "ABC" 的所有排列（每个排列长度固定为3）
crunch 0 0 -p 123 abc ABC > perm.txt

**4. 配合 `[[Hydra]]` 使用**

bash

复制

下载

# 生成动态字典并通过管道直接喂给 Hydra（避免中间文件）
crunch 8 8 numeric | hydra -l admin -P - ssh://192.168.1.100
# 注意：-P - 表示从标准输入读取密码

**5. 限制输出大小并分割**

bash

复制

下载

# 每 20MB 生成一个文件，文件名为 chunk_xxxx.txt
crunch 8 8 numeric -b 20mb -o chunk_%%.txt