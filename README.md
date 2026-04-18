# CodeV_Macros
这是一个用于CodeV软件的宏函数集合，方便在优化宏中调用。
## 目录
- [CodeV\_Macros](#codev_macros)
  - [目录](#目录)
  - [安装](#安装)
  - [使用说明](#使用说明)
  - [优化宏调用前声明](#优化宏调用前声明)
  - [启动宏说明](#启动宏说明)
  - [宏函数列表](#宏函数列表)
    - [CENZ](#cenz)
    - [ETCT](#etct)
    - [GETSAG](#getsag)
    - [INFLECT](#inflect)
    - [TRA](#tra)
    - [MAX\_TELE](#max_tele)
    - [TELE](#tele)
    - [JMRCC](#jmrcc)
    - [JMRCC\_W](#jmrcc_w)
  - [示例调用](#示例调用)
  - [备注](#备注)

## 安装
1. 下载本仓库代码。
2. 将需要的 `.seq` 文件复制到 CodeV 宏目录（通常为 `C:\Program Files\CodeV\Macros`）。
3. 若使用自定义路径，请在 `Startup.seq` 中配置 `pth seq` 路径。

   
## 使用说明
1. 打开 CodeV。
2. “工具”菜单 → “宏”。
3. 选择宏文件并运行。
4. 按宏提示输入参数。
5. 查看输出结果。

## 优化宏调用前声明
1. 如果在优化宏中调用这些函数，建议先在宏开头声明所需函数，避免未加载导致出错。
2. 另外也应避免在 `AUT...GO` 块内声明，防止重复加载报错。

示例：
- IN CENZ
- IN ETCT
- IN GETSAG
- IN INFLECT
- IN TRA
- IN MAX_TELE
- IN JMRCC
- IN JMRCC_W
- IN TELE

说明：
- 若 `Startup.seq` 已加载这些函数，可省略。
- 若使用自定义路径，确保已执行对应 `pth seq` 设置。

## 启动宏说明
- 文件：`Startup.seq`  
- 作用：在 CodeV 启动时自动运行，配置全局变量和宏路径。  
- 关键点：  
  - `pth seq` 设定宏路径，并通过 `IN` 加载宏文件：`CENZ/ETCT/GETSAG/INFLECT/TRA/MAX_TELE/JMRCC/JMRCC_W/TELE`  
  - 初始化全局常量 `^PI`、`^RAD` 等。

## 宏函数列表

### CENZ
- 函数：`@CENZ(^i)`  
- 文件：`FCT\CENZ.seq`  
- 作用：计算相邻两面 `S^i` 与 `S^(i+1)` 的定心敏感度。  
- 公式：`Z = |(y1/R1 - y2/R2)/2|`  
- 输入：`^i` 起始面号  
- 输出：`^Z`  
- 说明：有效半口径 `y = SD - d`（默认 `d=0.2`）；`y<=0` 或 `R=0` 返回 0。

### ETCT
- 函数：`@ETCT(^s,^z)`  
- 文件：`FCT\ETCT.seq`  
- 作用：计算指定表面在指定组态下的边缘厚度与中心厚度比值（`ET/CT`）。  
- 输入：`^s` 表面编号；`^z` 组态编号。  
- 输出：`^res`（厚度比值）。  
- 说明：先读取 `CT S^s Z^z` 与 `ET S^s Z^z`；当 `|CT| < 0.0001` 时使用 `0.0001` 作为分母进行保护，避免除零错误。

### GETSAG
- 函数：`@GETSAG(^s,^z,^y)`  
- 文件：`FCT\GETSAG.seq`  
- 作用：获取指定表面在指定组态与指定 Y 高度处的绝对矢高。  
- 输入：`^s` 表面编号；`^z` 组态编号；`^y` 为 Y 坐标绝对高度。  
- 输出：`^sag`（绝对矢高值）。  
- 说明：通过 `SAGF(^s,^z,0,^y)` 采样并取绝对值；固定坐标采样方式更适合在 `AUT` 中约束，减少优化震荡。

### INFLECT
- 函数：`@INFLECT(^s,^z,^y)`  
- 文件：`FCT\INFLECT.seq`  
- 作用：提取指定表面在指定组态、指定 Y 高度处的子午二阶导数 `d2z/dy2`。  
- 输入：`^s` 表面编号；`^z` 组态编号；`^y` 为 Y 坐标绝对高度。  
- 输出：`^d2zdy2`。  
- 说明：调用 `SDERIVF(^s,^z,0,^y,^deriv_data)` 后读取 `^deriv_data(5)`；若状态非 0（如探针越界）则返回 0 以保证优化流程稳定。

### TRA
- 函数：`@TRA(^xf,^yf,^wave_idx)`  
- 文件：`FCT\TRA.seq`  
- 作用：指定视场与波长下的平均总透过率。  
- 输入：`^xf`、`^yf` 视场坐标；`^wave_idx` 波长索引。  
- 输出：`^result`  
- 说明：调用内置 `TRA_1FLD` 读取 `^tra_out(3, ^wave_idx)`；失败返回 0。数组设计上支持最多 21 个波长加 1 个平均值（具体索引请按当前系统设置确认）。

### MAX_TELE
- 函数：`@MAX_TELE`  
- 文件：`FCT\MAX_TELE.seq`  
- 作用：遍历所有视场，返回像面 `SI` 上主光线与 Z 轴夹角的最大值。  
- 输出：最大角度（度）。  
- 说明：默认使用变焦位 `Z1` 和参考波长。
### TELE
- 函数：`@TELE(^surf_id)`  
- 文件：`FCT\TELE.seq`  
- 作用：与 `@MAX_TELE` 类似，但计算发生在指定表面 `S^surf_id`。  
- 输入：`^surf_id` 表面编号  
- 输出：最大角度（度）。  
- 说明：默认使用变焦位 `Z1` 和参考波长。

### JMRCC
- 函数：`@JMRCC(^ra,^fa,^sa,^rb,^fb,^sb,^rc,^fc)`  
- 文件：`FCT\JMRCC.seq`  
- 作用：计算基准光线 A 到点 B 的有符号垂直距离（clearance）。  
- 输入：`^ra,^fa,^sa` 定义光线 A；`^rb,^fb,^sb` 定义光线 B；`^rc,^fc` 可选定义光线 C。  
- 输出：`^ypt_wrt_ray`  
- 说明：默认点 B 为光线 B 在面 `^sb` 的截点；若 `^rc`、`^fc` 非零，则点 B 为光线 B 与 C 的空间交点。

### JMRCC_W
- 函数：`@JMRCC_W(^ra,^fa,^sa,^wa,^rb,^fb,^sb,^wb,^rc,^fc,^wc)`  
- 文件：`FCT\JMRCC_W.seq`  
- 作用：与 `@JMRCC` 相同，但 A/B/C 可指定不同波长。  
- 输入：`^wa/^wb/^wc` 为波长索引，传入 `0` 表示参考波长。  
- 输出：`^dist`


## 示例调用
- `@CENZ(8)` 或 `@CENZ(10)`：定心约束  
- `@ETCT(5,1)`：第 5 面在第 1 组态下的 ET/CT 约束  
- `@GETSAG(4,1,3.0)`：第 4 面在 `Y=3.0` 处的绝对矢高约束  
- `@INFLECT(4,1,2.5)`：第 4 面在 `Y=2.5` 处的曲率符号约束  
- `@MAX_TELE`：远心度约束  
- `@TRA(0,0,4)`：透过率约束  

## 备注
- 角度单位为度。  
- 距离单位随系统设置（通常为 mm）。  