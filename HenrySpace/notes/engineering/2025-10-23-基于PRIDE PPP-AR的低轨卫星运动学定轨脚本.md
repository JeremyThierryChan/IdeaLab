# 基于PRIDE PPP-AR的低轨卫星运动学定轨脚本

> 来源：[CSDN 原文](https://blog.csdn.net/m0_49684834/article/details/146405848)  
> 发布时间：2025-10-23  
> 标签：无

---

这份脚本把PPP/PPPAR 的繁琐准备工作前置自动化：从时间解析、表格与产品选择/合并，到 ANTEX/PCO 策略与先验坐标、再到多轮清洗与 AR 固定。理解每一步的输入输出与相互依赖关系，能显著提升你在 GNSS 数据处理中定位结果的可靠性与可重复性。结果图如图所示：运动学定轨的三个方向的精度都在1cm左右。

---

资源下载入口 
1) 背景与目标 
目标：对单站（或 LEO）RINEX 观测进行 PPP/PPPAR 处理，自动化完成表格与产品准备（SP3/ERP/FCB/ATT/LEO Quaternion/ANTEX）、时间解析、SPP 初定位、观测清洗、模糊度解算及最终解算。 
主要依赖：PRIDE-PPPAR 工具链中的 sp3orb / spp / tedit / lsq / redig / arsig 等。 
2) 运行前提与目录约定 
脚本默认的路径约定（可在开头变量修改）： 
 控制文件与目录 
   ctrl_dir="/home/henryyang/work/ppp/config"  ctrl_file="/home/henryyang/work/ppp/config/config"   产品目录 
   product_dir="/home/henryyang/work/ppp/product"  公共产品：product_cmn_dir="$product_dir/common"  LEO 产品：product_leo_dir="$product_dir/leo"   观测与星历 
   rinex_dir="/home/henryyang/work/ppp/obs"  rinexobs="/home/henryyang/work/ppp/obs/grad0060.23o"  rinexnav="/home/henryyang/work/ppp/nav/brdm0060.23p"   
 
 建议：将 site="grad"、AR="A"（模糊度模式）与日期相关的 tmpfobs 调整为你当前要处理的测站与观测日期。 
 
3) 脚本总体流程图 
启动 → 基本变量/函数 → 解析 RINEX 起止时间 → MJD/YDOY 计算    → PrepareTables（gpt3、oceanload、sat_parameters、leap.sec ...）    → 准备产品：SP3 / ERP / OBX(姿态) / OSB/FCB / LEO quaternions / Clock    → ANTEX 选择与下载链接处理、abs_igs.atx 软链    → 产品重命名（sck_ydoy / igserp / fcb_ydoy 等）    → sp3orb 建索引（或预处理）    → spp 初定位（kin_ydoy_site）    → tedit + lsq + redig 清洗迭代（多轮跳变检测）    → 若可 AR：arsig + lsq 复算    → 结束（输出日志/结果） 
4) 核心工具函数逐段讲解 
 Execute / ExecuteWithoutOutput：包装命令执行与日志输出（支持重定向）。  sedi：兼容 macOS 与 Linux 的 sed -i。  ymd2mjd / mjd2ydoy：日期换算（年积日、MJD）。  MergeFiles dir infile-pattern outfile：按通配模式合并多个产品文件。  WgetDownload url：统一的 wget 下载器（断点续传、超时、尝试 3 次）。  PrepareTables mjd_s mjd_e table_dir：检查/链接 PRIDE-PPPAR 所需表格（gpt3_1.grd、oceanload、orography、leap.sec、sat_parameters），必要时自动下载并校验是否过期。  
 
 提示：脚本中用到的彩色输出变量（如 GREEN/RED/CYAN/NC）和消息前缀（MSGERR/MSGINF/MSGSTA）需要在环境或公共脚本中预定义。 
 
5) 数据准备（重头戏） 
5.1 解析观测起止时间 
 从 RINEX 头文件中抓首帧/尾帧时间，生成 ymd_s / hms_s / ymd_e / hms_e。  将起止时间转换到 mjd_s/mjd_e 与 ydoy_s/ydoy_e（年+积日）。  
5.2 表格（Tables） 
 软链或下载 gpt3_1.grd、oceanload、orography_ell(含1x1)、leap.sec、sat_parameters。  对 leap.sec、sat_parameters 做有效性与时效性检查，必要时 FTP 更新。  
5.3 轨道（SP3） 
 优先读取配置文件项 Satellite orbit；否则在本地公共目录按优先级寻找（WUM/IGS/COD …）。  多文件时合并：产出 mersp3_YYYYDOY。  最终复制到当前目录，并更新配置。  
5.4 ERP 
 同 SP3 的策略，优先自定义，其次本地匹配。  多文件合并到 mererp_YYYYDOY。  
5.5 姿态 / 码相位偏差 / LEO 
 OBX（四元数姿态）、FCB/OSB（码/相位偏差）、LEO quaternions（低轨姿态）均支持多文件合并（meratt_* / merfcb_* / merlat_*）。  AR="Y" 时缺少 FCB 会直接报错；AR="A" 仅警告。  
5.6 时钟 & ANTEX 
 Satellite clock 指定的时钟产品复制并（必要时）合并到 mersck_ydoy，随后重命名为 sck_ydoy。  从时钟头部识别使用的 ANTEX（igs14/igsR3/M14.ATX/M20.ATX 等），若本地表目录无对应文件，自动从 IGS/CODE/IGN 等源下载，创建软链 abs_igs.atx。  根据 FCB 的 APC_MODEL 自动调整控制文件中的 “PCO on wide-lane” 开关。  
5.7 统一重命名 
 clk → sck_ydoy，erp → igserp，fcb → fcb_ydoy，att → att_ydoy，lat → lat_ydoy…  
6) 先验坐标：SPP 初定位 
 调用 spp 在全时段（按起止时间、间隔）生成单点定位轨迹：kin_YYYYDOY_site。  若 product_dir/leo/pso_YYYYDOY_site 存在，则用 pso2kin.py 将 PSO 结果转换覆盖 SPP 作为更优先验。  校验站心高程是否在合理范围（-4 km ~ +20 km），异常则报错。  
 
7) 观测预处理：tedit、lsq、redig 清洗循环 
 tedit 
   以 kin_… 先验、rinexobs/nav、freq_cmb（从控制文件读 “Frequency combination”）等为输入，输出短弧与质量控制后的观测。  旧历元（mjd_s <= 51666）会额外启用 -pc_check 0 兼容选项。   lsq（初始最小二乘） 
   运行 lsq ../config/config "rinexobs"，准备 res_ydoy_s 残差文件。   redig（多级跳变与异常点剔除） 
   循序渐进的阈值：400 → 200 → 100 → …（或 80/60/40 for L 模式），每级输出新增剔除条数与新增模糊度数。  如仍有新增，进入 while 循环以 jump_end（100 或 40）继续迭代，直到稳定或达到上限（100 次）。   
 
 经验：Strict editing=YES 更严格，短弧阈值 short 也更紧（对 LEO 更敏感）。 
 
 
8) 模糊度解算（AR）与最终求解 
 若 AR≠N 且存在 ?(mer)fcb_ydoy，且 amb_ydoy 显示可解卫星数 > 0： 
   arsig ../config/config 进行模糊度固定；  再次 lsq 复算输出最终结果（是否打印由 “Verbose output” 控制）。   否则： 
   AR=Y 直接报错提示检查 OSB/观测完整性；  AR=A 仅警告，继续输出浮点解。   
 
9) 常见错误与排查清单 
 “start/end time not found” 
   RINEX 头部无标准时间行；或 tmpfobs/rinexobs 指向错误。   表格文件缺失（gpt3_1.grd / oceanload / leap.sec / sat_parameters） 
   检查 table_dir，确认网络可达，或手动下载。   SP3/ERP/CLK/FCB 文件未找到 
   检查 product_cmn_dir 实际文件名；如为多个文件，确认合并逻辑与输出名。   ANTEX 下载失败 
   备选源尝试：files.igs.org / pcv_archive / CODE_MGEX / IGSR3。   tedit/lsq/redig 中途失败 
   优先查看生成的 rhd/log 输出；检查 Frequency combination 与可用系统是否与 SP3/CLK 覆盖一致。   初定位 NaN 
   spp 输出有 NaN，通常是星历/观测时段不匹配或电离层模型/频率组合设置异常。   
 
10) 可改进点与工程化建议 
 健壮性： 
   统一定义 GREEN/RED/CYAN/NC 和 MSGERR/MSGINF/MSGSTA；  为 get_ctrl 缺失时提供兜底（未给出函数体，建议加入存在性检查）。   可移植性： 
   Darwin 之外系统的判定更细；bc/awk/wget 的可用性预检查。   可配置化： 
   增加 OFFLINE=YES 模式时严格禁网并给出明确操作提示；  将下载源做成列表与优先级表，便于镜像切换。   日志与产出： 
   将关键结果（坐标、RMS、AR 率）汇总到一个 summary_YYYYDOY_site.txt；  对 redig 的每轮统计汇总为 CSV，方便后续画图分析。   性能： 
   对大时段处理，tedit 与 lsq 分段并行；  产品文件缓存与校验（MD5/mtime）。   
11) 附：最小可运行示例与关键命令 
前置：确保 config 中的以下关键项正确： 
 Table directory、Product directory  Frequency combination（示例：-freq G12 R12 E15 C26 J12）  站点行包含 site 名称与截取高度等参数  
典型命令序列（脚本内部已自动化，这里仅给出手动要点）： 
# 1) 建立 SP3 索引或预读（可选）
sp3orb $product_cmn_dir/$sp3 -cfg $ctrl_dir/config

# 2) 先验 SPP
spp -elev 0 -trop non \
    -ts 2023/01/06 00:00:00.000 \
    -te 2023/01/06 23:59:50.000 \
    -ti 10 -o kin_2023006_grad \
    ../obs/grad0060.23o ../nav/brdm0060.23p

# 3) 预处理（tedit）+ LSQ + REDIG 清洗（多轮）
tedit "rinexobs" -time YYYY MM DD hh mm ss.sss -len SESSION -int INTERVAL \
      -xyz kin_YYYYDOY_site -short 120 -lc_check lm -elev CUT \
      -rhd log_YYYYDOY_site -rnxn "rinexnav" -freq FREQS -trunc_dbd OPT -tighter_thre no

lsq ../config/config "rinexobs"
redig res_YYYYDOY -jmp 400 -sht SHORT
# ... 200 / 100（/ 80/60/40 for LEO）

# 4) 模糊度固定 + 复算
arsig ../config/config
lsq ../config/config "rinexobs" 
结语 
这份脚本把 PPP/PPPAR 的繁琐准备工作前置自动化：从时间解析、表格与产品选择/合并，到 ANTEX/PCO 策略与先验坐标、再到多轮清洗与 AR 固定。理解每一步的输入输出与相互依赖关系，能显著提升你在 GNSS 数据处理中定位结果的可靠性与可重复性。 
结果图如图所示： 
 
运动学定轨的三个方向的精度都在1cm左右-
