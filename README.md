# CodeV_Macros
这是一个用于CodeV软件的宏脚本集合，旨在简化和自动化常见任务，提高工作效率。
## 目录
- [CodeV\_Macros](#codev_macros)
  - [目录](#目录)
  - [安装](#安装)
  - [使用说明](#使用说明)
  - [在初始化期间运行的第二个宏程序](#在初始化期间运行的第二个宏程序)
  - [宏函数列表](#宏函数列表)
    - [`@CENZ(^i)` — 定心系数 / Z 因子](#cenzi--定心系数--z-因子)
    - [`@TRA(^xf,^yf,^wave_idx)` — 平均总透过率](#traxfyfwave_idx--平均总透过率)
    - [`@MAX_TELE` — 最大远心角（像面）](#max_tele--最大远心角像面)
    - [`@TELE(^surf_id)` — 最大远心角（指定面）](#telesurf_id--最大远心角指定面)
    - [`@JMRCC(^ra,^fa,^sa,^rb,^fb,^sb,^rc,^fc)` — YZ 面垂直距离](#jmrccrafasarbfbsbrcfc--yz-面垂直距离)
    - [`@JMRCC_W(...)` — 支持独立波长的 JMRCC](#jmrcc_w--支持独立波长的-jmrcc)
  - [示例调用](#示例调用)
## 安装
1. 下载本仓库的代码。
2. 将所需的宏脚本文件复制到CodeV的宏目录中，通常位于`C:\Program Files\CodeV\Macros`。
3. 重启CodeV软件以加载新的宏脚本。
4. 在CodeV中，导航到“工具”菜单，选择“宏”，然后选择所需的宏脚本运行。
## 使用说明
1. 打开CodeV软件。
2. 导航到“工具”菜单，选择“宏”。
3. 选择所需的宏脚本并点击“运行”。
4. 根据宏脚本的提示进行操作。
5. 查看宏脚本的输出结果。

## 在初始化期间运行的第二个宏程序
- 文件：`Startup.seq`：
- 关键点：
  - 设置 `pth seq` 路径并加载宏文件（`IN CENZ/TRA/MAX_TELE/JMRCC/JMRCC_W/TELE`）
  - 初始化 `^PI`、`^RAD` 等全局常量。


## 宏函数列表
### `@CENZ(^i)` — 定心系数 / Z 因子
- 文件：`FCT\CENZ.seq`
- 作用：计算相邻两面 `S^i` 与 `S^(i+1)` 的定心敏感度。
- 公式：`Z = |(y1/R1 - y2/R2)/2|`
- 关键点：
  - 有效半口径 `y = SD - d`（默认 `d=0.2`）
  - `y<=0` 或 `R=0` 时返回 0

### `@TRA(^xf,^yf,^wave_idx)` — 平均总透过率
- 文件：`FCT\TRA.seq`
- 作用：指定视场与波长下的平均总透过率。
- 方法：调用内置 `TRA_1FLD`，读取 `^tra_out(3, ^wave_idx)`
- 失败返回 0

### `@MAX_TELE` — 最大远心角（像面）
- 文件：`FCT\MAX_TELE.seq`
- 作用：遍历所有视场，返回像面 `SI` 上主光线与 Z 轴夹角的最大值（度）。
- 核心：用 `L/M` 方向余弦 + `IND` 折射率归一化求角度。

### `@TELE(^surf_id)` — 最大远心角（指定面）
- 文件：`FCT\TELE.seq`
- 作用：与 `@MAX_TELE` 类似，但计算发生在用户指定表面 `S^surf_id`。
- 输出：最大角度（度）。

### `@JMRCC(^ra,^fa,^sa,^rb,^fb,^sb,^rc,^fc)` — YZ 面垂直距离
- 文件：`FCT\JMRCC.seq`
- 作用：计算基准光线 A 到点 B 的有符号垂直距离（clearance）。
- 点 B 定义：
  - 默认：光线 B 在面 `^sb` 的截点
  - 若 `^rc`、`^fc` 非零：光线 B 与 C 的空间交点
- 输出：`^ypt_wrt_ray`

### `@JMRCC_W(...)` — 支持独立波长的 JMRCC
- 文件：`FCT\JMRCC_W.seq`
- 作用：与 `@JMRCC` 相同，但 A/B/C 可指定不同波长（0 表示参考波长）。
- 输出：`^dist`

## 示例调用
- `@CENZ(8)` / `@CENZ(10)`：定心约束  
- `@MAX_TELE`：远心度约束  
- `@TRA(0,0,4)`：透过率约束

