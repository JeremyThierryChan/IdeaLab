# 【python】读取卫星星历（RENIX 3.04）进行卫星位置的计算（北斗卫星专题）

> 来源：[CSDN 原文](https://blog.csdn.net/m0_49684834/article/details/128116573)  
> 发布时间：2022-11-30  
> 标签：无

---

最近的卫星导航数据处理，老师让我们进行卫星位置的计算，从而使用绘图工具进行对卫星星下点的轨迹进行绘图，这里首先的步骤是读取卫星星历数据，计算卫星位置。这次的课程目标主要是针对北斗卫星，进行对卫星位置的定位。

---

最近的卫星导航数据处理，老师让我们进行卫星位置的计算，从而使用绘图工具进行对卫星星下点的轨迹进行绘图，这里首先的步骤是读取卫星星历数据，计算卫星位置。 
  这次的课程目标主要是针对北斗卫星，进行对卫星位置的定位。 
目录 
代码更新： 
2023.5.1更新 
2025.5.13 更新 
 
  
 
 PS:2025.5.13日最新更新 
 
 
 
将GEO卫星，IGSO卫星和MEO卫星进行分类，下列链接提供了相应北斗卫星的PRN号，方便对北斗卫星进行分类。 
中国卫星导航系统管理办公室测试评估研究中心 
根据其含有的卫星PRN号进行分类处理，其中GEO卫星要进行5°偏差改正。 
 
 其中给出了改正矩阵。 
 
 
 计算公式已经给出，接下来是准备文件。 
  
 
 
导航星历文件，这里我截取了只有北斗卫星的卫星导航星历电文，文件格式如下： 
 
 其中的END OF HEADER是模拟读取卫星导航星历文件的格式进行编写的。 
 
 该代码只进行了GEO和其他卫星的卫星位置计算，还未进行对IGSO、MEO卫星的分类工作，大家可以对代码进行功能添加，可以将数据导入到excel进行筛选，完成对IGSO、MEO卫星的分类。 
 
代码如下： 
import math as m
import numpy as np
import csv

infile=open("all.rnx","r")
#读取原星历文件
content=infile.readlines()
infile.close()
start_num=0
all_C=[]#存储北斗星历数据块
for i in range(len(content)):
    if content[i].find("                                                            END OF HEADER") != -1:
        start_num = i + 1
for i in range(len(content)-start_num):
    line=content[start_num+i]
    if line[0]=='C':
        for j in range(0,8):
            all_C.append(content[start_num+i+j])
outFile=open("all_C.txt",'w')#存储为新的文件（只含有北斗卫星）
outFile.writelines(all_C)
outFile.close()

#接下来是计算卫星位置的程序
with open(
          'all_C.txt', 'r') as f:
    if f == 0:
        print("不能打开文件！")
    else:
        print("导航文件打开成功！")
    nfile_lines = f.readlines()  # 按行读取N文件
    print(len(nfile_lines))
    f.close()



def start_num():  # 定义数据记录的起始行
    start_num = 0
    for i in range(len(nfile_lines)):
        if nfile_lines[i].find('END OF HEADER') != -1:
            start_num = i + 1
    return start_num


def rx(fai):
    result = np.mat([[1, 0, 0], [0, m.cos(fai), m.sin(fai)], [0, -1 * m.sin(fai), m.cos(fai)]])
    return result


def rz(fai):
    result = np.mat([[m.cos(fai), m.sin(fai), 0], [-1 * m.sin(fai), m.cos(fai), 0], [0, 0, 1]])
    return result


n_dic_list = []

n_data_lines_nums = int((len(nfile_lines) - start_num()) / 8)
print("一共%d组数据" % (n_data_lines_nums))

# 第j组，第i行
for j in range(n_data_lines_nums):
    n_dic = {}
    for i in range(8):
        data_content = nfile_lines[start_num() + 8 * j + i]
        n_dic['数据组数'] = j + 1
        if i == 0:
            n_dic['卫星PRN号'] = str(data_content[0:3])

            n_dic['历元'] = data_content[4:23]

            n_dic['卫星钟偏差(s)'] = float(
                (data_content.strip('\n')[23:42]))  # 利用字符串切片功能来进行字符串的修改

            n_dic['卫星钟漂移(s/s)'] = float(
                (data_content.strip('\n')[42:61]))

            n_dic['卫星钟漂移速度(s/s*s)'] = float(
                (data_content.strip('\n')[61:80]))

        if i == 1:
            n_dic['IODE'] = float(
                (data_content.strip('\n')[4:23]))
            n_dic['C_rs'] = float(
                (data_content.strip('\n')[23:42]))
            n_dic['n'] = float(
                (data_content.strip('\n')[42:61]))
            n_dic['M0'] = float(
                (data_content.strip('\n')[61:80]))
        if i == 2:
            n_dic['C_uc'] = float(
                (data_content.strip('\n')[4:23]))
            n_dic['e'] = float(
                (data_content.strip('\n')[23:42]))
            n_dic['C_us'] = float(
                (data_content.strip('\n')[42:61]))
            n_dic['sqrt_A'] = float(
                (data_content.strip('\n')[61:80]))
        if i == 3:
            n_dic['TEO'] = float(
                (data_content.strip('\n')[4:23]))
            n_dic['C_ic'] = float(
                (data_content.strip('\n')[23:42]))
            n_dic['OMEGA'] = float(
                (data_content.strip('\n')[42:61]))
            n_dic['C_is'] = float(
                (data_content.strip('\n')[61:80]))

        if i == 4:
            n_dic['I_0'] = float(
                (data_content.strip('\n')[4:23]))
            n_dic['C_rc'] = float(
                (data_content.strip('\n')[23:42]))
            n_dic['w'] = float(
                (data_content.strip('\n')[42:61]))
            n_dic['OMEGA_DOT'] = float(
                (data_content.strip('\n')[61:80]))
        if i == 5:
            n_dic['IDOT'] = float(
                (data_content.strip('\n')[4:23]))
            n_dic['L2_code'] = float(
                (data_content.strip('\n')[23:42]))
            n_dic['PS_week_num'] = float(
                (data_content.strip('\n')[42:61]))

        if i == 6:
            n_dic['卫星精度(m)'] = float(
                (data_content.strip('\n')[4:23]))
            n_dic['卫星健康状态'] = float(
                (data_content.strip('\n')[23:42]))
            n_dic['TGD'] = float(
                (data_content.strip('\n')[42:61]))
            n_dic['IODC'] = float(
                (data_content.strip('\n')[61:80]))

    n_dic_list.append(n_dic)
with open('北斗卫星.csv', 'w', newline='') as f:
    header = ['数据组数', '卫星PRN号', '历元', '卫星钟偏差(s)', '卫星钟漂移(s/s)', '卫星钟漂移速度(s/s*s)', 'IODE',
              'C_rs', 'n', 'M0', 'C_uc', 'e', 'C_us', 'sqrt_A', 'TEO', 'C_ic', 'OMEGA', 'C_is', 'I_0', 'C_rc', 'w',
              'OMEGA_DOT', 'IDOT', 'L2_code', 'PS_week_num', 'L2_P_code', '卫星精度(m)', '卫星健康状态', 'TGD', 'IODC',
              'X', 'Y', 'Z']
    writer = csv.DictWriter(f, fieldnames=header)
    writer.writeheader()
    writer.writerows(n_dic_list)
f.close()
prn_x_y_z = []
with open('北斗卫星.csv', 'rt') as csvfile:
    reader = csv.DictReader(csvfile)
    for row in reader:

        PRN = str(row["卫星PRN号"])
        TIME = row["历元"]

        year = int(TIME.strip('\n')[2:4])

        month = int(TIME.strip('\n')[5:7])

        day = int(TIME.strip('\n')[8:10])

        hour = int(TIME.strip('\n')[11:13])
        minute = int(TIME.strip('\n')[14:16])
        second = float(TIME.strip('\n')[17:19])
        a_0 = float(row["卫星钟偏差(s)"])
        a_1 = float(row["卫星钟漂移(s/s)"])
        a_2 = float(row["卫星钟漂移速度(s/s*s)"])
        IODE = float(row["IODE"])
        C_rs = float(row["C_rs"])
        δn = float(row["n"])
        M0 = float(row["M0"])
        C_uc = float(row["C_uc"])
        e = float(row["e"])
        C_us = float(row["C_us"])
        sqrt_A = float(row["sqrt_A"])
        TEO = float(row["TEO"])
        C_ic = float(row["C_ic"])
        OMEGA = float(row["OMEGA"])
        C_is = float(row["C_is"])
        I_0 = float(row["I_0"])
        C_rc = float(row["C_rc"])
        w = float(row["w"])
        OMEGA_DOT = float(row["OMEGA_DOT"])
        IDOT = float(row["IDOT"])
        L2_code = float(row["L2_code"])
        PS_week_num = float(row["PS_week_num"])


        TGD = float(row["TGD"])
        IODC = float(row["IODC"])
        t1 = None
        # 1.计算卫星运行平均角速度 GM:WGS84下的引力常数 =3.986005e14，a:长半径
        GM = 398600500000000
        n_0 = m.sqrt(GM) / m.pow(sqrt_A, 3)
        n = n_0 + δn

        # 2.计算归化时间t_k 计算t时刻的卫星位置  UT：世界时 此处以小时为单位
        UT = hour + (minute / 60.0) + (second / 3600)
        # GPS时起始时刻1980年1月6日0点   year是两位数 需要转换到四位数
        if year >= 80:
            if year == 80 and month == 1 and day < 6:
                year = year + 2000
            else:
                year = year + 1900
        else:
            year = year + 2000
        if month <= 2:
            year = year - 1
            month = month + 12  # 1，2月视为前一年13，14月

        # 需要将当前需计算的时刻先转换到儒略日再转换到GPS时间
        JD = (365.25 * year) + int(30.6001 * (month + 1)) + day + UT / 24 + 1720981.5

        WN = int((JD - 2444244.5) / 7)  # WN:GPS_week number 目标时刻的GPS周

        t_oc = ((JD - 2444244.5) - (7.0 * WN)) * 24 * 3600.0 - 14  # t_GPS:目标时刻的GPS秒 减去14秒为BDT

        # 对观测时刻t1进行钟差改正,注意：t1应是由接收机接收到的时间

        t_k = -14


        # 3.平近点角计算M_k = M_0+n*t_k
        M_k = M0 + n * t_k  # 实际应该是乘t_k，但是没有接收机的观测时间，所以为了练手设t_k=0

        # 4.偏近点角计算 E_k  (迭代计算) E_k = M_k + e*sin(E_k)
        E = 0;
        E1 = 1;
        count = 0;
        while abs(E1 - E) > 1e-10:
            count = count + 1
            E1 = E
            E = M_k + e * m.sin(E)
            if count > 1e8:
                print("计算偏近点角时未收敛！")
                break

                # 5.计算卫星的真近点角
        V_k = m.atan2((m.sqrt(1 - e * e) * m.sin(E)) , (m.cos(E) - e));

        # 6.计算升交距角 u_0(φ_k), ω:卫星电文给出的近地点角距
        u_0 = V_k + w

        # 7.摄动改正项 δu、δr、δi :升交距角u、卫星矢径r和轨道倾角i的摄动量
        δu = C_uc * m.cos(2 * u_0) + C_us * m.sin(2 * u_0)
        δr = C_rc * m.cos(2 * u_0) + C_rs * m.sin(2 * u_0)
        δi = C_ic * m.cos(2 * u_0) + C_is * m.sin(2 * u_0)

        # 8.计算经过摄动改正的升交距角u_k、卫星矢径r_k和轨道倾角 i_k
        u = u_0 + δu
        r = m.pow(sqrt_A, 2) * (1 - e * m.cos(E)) + δr
        i = I_0 + δi + IDOT * (t_k);  # 实际乘t_k=t-t_oe

        # 9.计算卫星在轨道平面坐标系的坐标,卫星在轨道平面直角坐标系（X轴指向升交点）中的坐标为：
        x_k = r * m.cos(u)
        y_k = r * m.sin(u)

        # 10.观测时刻升交点经度Ω_k的计算，升交点经度Ω_k等于观测时刻升交点赤经Ω与格林尼治恒星时GAST之差  Ω_k=Ω_0+(ω_DOT-omega_e)*t_k-omega_e*t_oe
        omega_e = 7.292115e-5  # 地球自转角速度
        OMEGA_k = OMEGA + (OMEGA_DOT - omega_e) * t_k - omega_e * TEO;  # 星历中给出的Omega即为Omega_o=Omega_t_oe-GAST_w

        # 11.计算卫星在地固系中的直角坐标l
        X_k = x_k * m.cos(OMEGA_k) - y_k * m.cos(i) * m.sin(OMEGA_k)
        Y_k = x_k * m.sin(OMEGA_k) + y_k * m.cos(i) * m.cos(OMEGA_k)
        Z_k = y_k * m.sin(i)

        # 12.判断卫星是否为GEO卫星，否则不进行极移改正。
        if PRN in ['C01', 'C02', 'C03', 'C04', 'C05', 'C59', 'C60', 'C61']:

            fi = omega_e * t_k
            five = ( m.pi/180 * 5)*(-1)
            print(fi)
            a = np.mat([X_k, Y_k, Z_k])


            a = rz(fi) * rx(five) * a.T

            X_k = str(a[0, 0])
            Y_k = str(a[1, 0])
            Z_k = str(a[2, 0])

            if month > 12:  # 恢复历元
                year = year + 1
                month = month - 12

            print("历元：", year, "年", month, "月", day, "日", hour, "时", minute, "分", second, "秒", "卫星PRN号:",
                  PRN,
                  "平均角速度:", n, "卫星平近点角:", M_k, "偏近点角:", E, "真近点角:", V_k, "升交距角:", u_0,
                  "摄动改正项:", δu,
                  δr, δi, "经摄动改正后的升交距角、卫星矢径和轨道倾角:", u, r, i, "轨道平面坐标X,Y:", x_k, y_k,
                  "观测时刻升交点经度:", OMEGA_k, "地固直角坐标系(极移改正)X:", X_k, "地固直角坐标系Y（极移改正）:", Y_k,
                  "地固直角坐标系Z（极移改正）:",
                  Z_k)
            prn_x_y_z.append(PRN + ",")
            prn_x_y_z.append(str(X_k) + ",")
            prn_x_y_z.append(str(Y_k) + ",")
            prn_x_y_z.append(str(Z_k))
            prn_x_y_z.append("\n")


        else:
          prn_x_y_z.append(PRN + ",")
          prn_x_y_z.append(str(X_k) + ",")
          prn_x_y_z.append(str(Y_k) + ",")
          prn_x_y_z.append(str(Z_k))
          prn_x_y_z.append("\n")

print("卫星坐标数据计算完成！")
f = open("北斗卫星位置.txt", 'w')
f.writelines(prn_x_y_z)
print("文件写入成功！")
f.close()













 
 
 问题解决日志： 
 1.在运行结束后，与卫星精密星历进行比对，发现有些卫星的坐标的正负号出现问题，但都是整体性出现的问题，有的卫星坐标XYZ三者都为正确坐标的相反数，不知是何问题，还请广大读者予以指正，该代码仅供参考，还请广大读者进行批评指正。（问题已解决） 
 2.在转换坐标系时，旋转矩阵发生错误以至于和精密星历完全对不上，差异较大，在修改过后误差约为数十米，仍未得到解决方案，请各位读者指正。（问题未完全解决） 
 （2023.4.29改正） 
 
结果图如下： 
 
2023.5.1更新 
选取程序运行目录下的.rnx文件，生成多个计算出的卫星： 
代码如下： 
import math as m
import numpy as np
import csv
import os

def rx(fai):
    result = np.mat([[1, 0, 0], [0, m.cos(fai), m.sin(fai)], [0, -1 * m.sin(fai), m.cos(fai)]])
    return result


def rz(fai):
    result = np.mat([[m.cos(fai), m.sin(fai), 0], [-1 * m.sin(fai), m.cos(fai), 0], [0, 0, 1]])
    return result


def reader_GNSS(filename):
    infile = open("all.rnx", "r")
    # 读取原星历文件
    content = infile.readlines()
    infile.close()
    start_num = 0
    all_C = []  # 存储北斗星历数据块
    for i in range(len(content)):
        if content[i].find("                                                            END OF HEADER") != -1:
            start_num = i + 1
    for i in range(len(content) - start_num):
        line = content[start_num + i]
        if line[0] == 'C':
            for j in range(0, 8):
                all_C.append(content[start_num + i + j])
    outFile = open("all_C.txt", 'w')  # 存储为新的文件（只含有北斗卫星）
    outFile.writelines(all_C)
    outFile.close()

    # 接下来是计算卫星位置的程序
    with open(
            'all_C.txt', 'r') as f:
        if f == 0:
            print("不能打开文件！")
        else:
            print("导航文件打开成功！")
        nfile_lines = f.readlines()  # 按行读取N文件
        print(len(nfile_lines))
        f.close()

    start_num = 0
    for i in range(len(nfile_lines)):
        if nfile_lines[i].find('END OF HEADER') != -1:
            start_num = i + 1

    n_dic_list = []

    n_data_lines_nums = int((len(nfile_lines) - start_num) / 8)
    print("一共%d组数据" % (n_data_lines_nums))

    # 第j组，第i行
    for j in range(n_data_lines_nums):
        n_dic = {}
        for i in range(8):
            data_content = nfile_lines[start_num + 8 * j + i]
            n_dic['数据组数'] = j + 1
            if i == 0:
                n_dic['卫星PRN号'] = str(data_content[0:3])

                n_dic['历元'] = data_content[4:23]

                n_dic['卫星钟偏差(s)'] = float(
                    (data_content.strip('\n')[23:42]))  # 利用字符串切片功能来进行字符串的修改

                n_dic['卫星钟漂移(s/s)'] = float(
                    (data_content.strip('\n')[42:61]))

                n_dic['卫星钟漂移速度(s/s*s)'] = float(
                    (data_content.strip('\n')[61:80]))

            if i == 1:
                n_dic['IODE'] = float(
                    (data_content.strip('\n')[4:23]))
                n_dic['C_rs'] = float(
                    (data_content.strip('\n')[23:42]))
                n_dic['n'] = float(
                    (data_content.strip('\n')[42:61]))
                n_dic['M0'] = float(
                    (data_content.strip('\n')[61:80]))
            if i == 2:
                n_dic['C_uc'] = float(
                    (data_content.strip('\n')[4:23]))
                n_dic['e'] = float(
                    (data_content.strip('\n')[23:42]))
                n_dic['C_us'] = float(
                    (data_content.strip('\n')[42:61]))
                n_dic['sqrt_A'] = float(
                    (data_content.strip('\n')[61:80]))
            if i == 3:
                n_dic['TEO'] = float(
                    (data_content.strip('\n')[4:23]))
                n_dic['C_ic'] = float(
                    (data_content.strip('\n')[23:42]))
                n_dic['OMEGA'] = float(
                    (data_content.strip('\n')[42:61]))
                n_dic['C_is'] = float(
                    (data_content.strip('\n')[61:80]))

            if i == 4:
                n_dic['I_0'] = float(
                    (data_content.strip('\n')[4:23]))
                n_dic['C_rc'] = float(
                    (data_content.strip('\n')[23:42]))
                n_dic['w'] = float(
                    (data_content.strip('\n')[42:61]))
                n_dic['OMEGA_DOT'] = float(
                    (data_content.strip('\n')[61:80]))
            if i == 5:
                n_dic['IDOT'] = float(
                    (data_content.strip('\n')[4:23]))
                n_dic['L2_code'] = float(
                    (data_content.strip('\n')[23:42]))
                n_dic['PS_week_num'] = float(
                    (data_content.strip('\n')[42:61]))

            if i == 6:
                n_dic['卫星精度(m)'] = float(
                    (data_content.strip('\n')[4:23]))
                n_dic['卫星健康状态'] = float(
                    (data_content.strip('\n')[23:42]))
                n_dic['TGD'] = float(
                    (data_content.strip('\n')[42:61]))
                n_dic['IODC'] = float(
                    (data_content.strip('\n')[61:80]))

        n_dic_list.append(n_dic)
    with open('北斗卫星.csv', 'w', newline='') as f:
        header = ['数据组数', '卫星PRN号', '历元', '卫星钟偏差(s)', '卫星钟漂移(s/s)', '卫星钟漂移速度(s/s*s)', 'IODE',
                  'C_rs', 'n', 'M0', 'C_uc', 'e', 'C_us', 'sqrt_A', 'TEO', 'C_ic', 'OMEGA', 'C_is', 'I_0', 'C_rc', 'w',
                  'OMEGA_DOT', 'IDOT', 'L2_code', 'PS_week_num', 'L2_P_code', '卫星精度(m)', '卫星健康状态', 'TGD',
                  'IODC',
                  'X', 'Y', 'Z']
        writer = csv.DictWriter(f, fieldnames=header)
        writer.writeheader()
        writer.writerows(n_dic_list)
    f.close()
    prn_x_y_z = []
    with open('北斗卫星.csv', 'rt') as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:

            PRN = str(row["卫星PRN号"])
            TIME = row["历元"]

            year = int(TIME.strip('\n')[2:4])

            month = int(TIME.strip('\n')[5:7])

            day = int(TIME.strip('\n')[8:10])

            hour = int(TIME.strip('\n')[11:13])
            minute = int(TIME.strip('\n')[14:16])
            second = float(TIME.strip('\n')[17:19])
            a_0 = float(row["卫星钟偏差(s)"])
            a_1 = float(row["卫星钟漂移(s/s)"])
            a_2 = float(row["卫星钟漂移速度(s/s*s)"])
            IODE = float(row["IODE"])
            C_rs = float(row["C_rs"])
            δn = float(row["n"])
            M0 = float(row["M0"])
            C_uc = float(row["C_uc"])
            e = float(row["e"])
            C_us = float(row["C_us"])
            sqrt_A = float(row["sqrt_A"])
            TEO = float(row["TEO"])
            C_ic = float(row["C_ic"])
            OMEGA = float(row["OMEGA"])
            C_is = float(row["C_is"])
            I_0 = float(row["I_0"])
            C_rc = float(row["C_rc"])
            w = float(row["w"])
            OMEGA_DOT = float(row["OMEGA_DOT"])
            IDOT = float(row["IDOT"])
            L2_code = float(row["L2_code"])
            PS_week_num = float(row["PS_week_num"])

            TGD = float(row["TGD"])
            IODC = float(row["IODC"])
            t1 = None
            # 1.计算卫星运行平均角速度 GM:WGS84下的引力常数 =3.986005e14，a:长半径
            GM = 398600500000000
            n_0 = m.sqrt(GM) / m.pow(sqrt_A, 3)
            n = n_0 + δn

            # 2.计算归化时间t_k 计算t时刻的卫星位置  UT：世界时 此处以小时为单位
            UT = hour + (minute / 60.0) + (second / 3600)
            # GPS时起始时刻1980年1月6日0点   year是两位数 需要转换到四位数
            if year >= 80:
                if year == 80 and month == 1 and day < 6:
                    year = year + 2000
                else:
                    year = year + 1900
            else:
                year = year + 2000
            if month <= 2:
                year = year - 1
                month = month + 12  # 1，2月视为前一年13，14月

            # 需要将当前需计算的时刻先转换到儒略日再转换到GPS时间
            JD = (365.25 * year) + int(30.6001 * (month + 1)) + day + UT / 24 + 1720981.5

            WN = int((JD - 2444244.5) / 7)  # WN:GPS_week number 目标时刻的GPS周

            t_oc = ((JD - 2444244.5) - (7.0 * WN)) * 24 * 3600.0 - 14  # t_GPS:目标时刻的GPS秒 减去14秒为BDT

            # 对观测时刻t1进行钟差改正,注意：t1应是由接收机接收到的时间

            t_k = -14

            # 3.平近点角计算M_k = M_0+n*t_k
            M_k = M0 + n * t_k  # 实际应该是乘t_k，但是没有接收机的观测时间，所以为了练手设t_k=0

            # 4.偏近点角计算 E_k  (迭代计算) E_k = M_k + e*sin(E_k)
            E = 0;
            E1 = 1;
            count = 0;
            while abs(E1 - E) > 1e-10:
                count = count + 1
                E1 = E
                E = M_k + e * m.sin(E)
                if count > 1e8:
                    print("计算偏近点角时未收敛！")
                    break

                    # 5.计算卫星的真近点角
            V_k = m.atan2((m.sqrt(1 - e * e) * m.sin(E)), (m.cos(E) - e));

            # 6.计算升交距角 u_0(φ_k), ω:卫星电文给出的近地点角距
            u_0 = V_k + w

            # 7.摄动改正项 δu、δr、δi :升交距角u、卫星矢径r和轨道倾角i的摄动量
            δu = C_uc * m.cos(2 * u_0) + C_us * m.sin(2 * u_0)
            δr = C_rc * m.cos(2 * u_0) + C_rs * m.sin(2 * u_0)
            δi = C_ic * m.cos(2 * u_0) + C_is * m.sin(2 * u_0)

            # 8.计算经过摄动改正的升交距角u_k、卫星矢径r_k和轨道倾角 i_k
            u = u_0 + δu
            r = m.pow(sqrt_A, 2) * (1 - e * m.cos(E)) + δr
            i = I_0 + δi + IDOT * (t_k);  # 实际乘t_k=t-t_oe

            # 9.计算卫星在轨道平面坐标系的坐标,卫星在轨道平面直角坐标系（X轴指向升交点）中的坐标为：
            x_k = r * m.cos(u)
            y_k = r * m.sin(u)

            # 10.观测时刻升交点经度Ω_k的计算，升交点经度Ω_k等于观测时刻升交点赤经Ω与格林尼治恒星时GAST之差  Ω_k=Ω_0+(ω_DOT-omega_e)*t_k-omega_e*t_oe
            omega_e = 7.292115e-5  # 地球自转角速度
            OMEGA_k = OMEGA + (OMEGA_DOT - omega_e) * t_k - omega_e * TEO;  # 星历中给出的Omega即为Omega_o=Omega_t_oe-GAST_w

            # 11.计算卫星在地固系中的直角坐标l
            X_k = x_k * m.cos(OMEGA_k) - y_k * m.cos(i) * m.sin(OMEGA_k)
            Y_k = x_k * m.sin(OMEGA_k) + y_k * m.cos(i) * m.cos(OMEGA_k)
            Z_k = y_k * m.sin(i)

            # 12.判断卫星是否为GEO卫星，否则不进行极移改正。
            if PRN in ['C01', 'C02', 'C03', 'C04', 'C05', 'C59', 'C60', 'C61']:

                fi = omega_e * t_k
                five = (m.pi / 180 * 5) * (-1)

                a = np.mat([X_k, Y_k, Z_k])

                a = rz(fi) * rx(five) * a.T

                X_k = str(a[0, 0])
                Y_k = str(a[1, 0])
                Z_k = str(a[2, 0])

                if month > 12:  # 恢复历元
                    year = year + 1
                    month = month - 12

                # print("历元：", year, "年", month, "月", day, "日", hour, "时", minute, "分", second, "秒", "卫星PRN号:",
                #       PRN,
                #       "平均角速度:", n, "卫星平近点角:", M_k, "偏近点角:", E, "真近点角:", V_k, "升交距角:", u_0,
                #       "摄动改正项:", δu,
                #       δr, δi, "经摄动改正后的升交距角、卫星矢径和轨道倾角:", u, r, i, "轨道平面坐标X,Y:", x_k, y_k,
                #       "观测时刻升交点经度:", OMEGA_k, "地固直角坐标系(极移改正)X:", X_k, "地固直角坐标系Y（极移改正）:",
                #       Y_k,
                #       "地固直角坐标系Z（极移改正）:",
                #       Z_k)
                prn_x_y_z.append(PRN + ",")
                prn_x_y_z.append(str(X_k) + ",")
                prn_x_y_z.append(str(Y_k) + ",")
                prn_x_y_z.append(str(Z_k))
                prn_x_y_z.append("\n")


            else:
                prn_x_y_z.append(PRN + ",")
                prn_x_y_z.append(str(X_k) + ",")
                prn_x_y_z.append(str(Y_k) + ",")
                prn_x_y_z.append(str(Z_k))
                prn_x_y_z.append("\n")
    return prn_x_y_z

#批量读取星历文件（.rnx）

rnx_filename=[]
dir=os.listdir()
for i in range(len(dir)):
    line=dir[i]
    extension=line.split(".")
    if extension[len(extension)-1] in "rnx":
        rnx_filename.append(line)
num=1
for i in rnx_filename:
    f = open("北斗卫星位置"+str(num)+".txt", 'w')
    f.writelines(reader_GNSS(i))
    print("文件写入成功！")
    f.close()
    num+=1
    print("完成计算的星历名称为：" + str(i))






 
2025.5.13 更新 
 读取rinex2.11版本的代码如下：   
import re
import math

def gregorian_to_jd(year, month, day, hour, minute, second):
    """ 将公历时间转换为儒略日 (JD) """
    if month <= 2:
        month += 12
        year -= 1

    A = math.floor(year / 100)
    B = 2 - A + math.floor(A / 4)

    JD = math.floor(365.25 * (year + 4716)) + math.floor(30.6001 * (month + 1)) + day + B - 1524.5
    JD += (hour / 24) + (minute / 1440) + (second / 86400)

    return JD

def jd_to_mjd(jd):
    """ 将儒略日 (JD) 转换为修正儒略日 (MJD) """
    return jd - 2400000.5

def read_nav(filepath):
    with open(filepath, 'r') as f:
        lines = f.readlines()

    nav_records = []
    i = 0

    # 跳过头文件，直到遇到数据块的第一行
    while i < len(lines) and 'END OF HEADER' not in lines[i]:
        i += 1
    i += 1  # 跳过"END OF HEADER"行，开始读取数据块

    while i < len(lines):
        block = lines[i:i+8]  # 假设每个数据块有8行

        if len(block) < 8:
            break  # 如果数据块不完整，则跳出循环

        # 在提取数据之前对每行进行替换D->E
        block = [line.replace('D', 'E').replace('d', 'e') for line in block]
        # 第一行提取：卫星PRN、年月日时分秒、钟差参数
        match = re.match(
            r'^\s*(\d+)\s+(\d+)\s+(\d+)\s+(\d+)\s+(\d+)\s+(\d+)\s+([+-]?\d+\.\d+)\s*([+-]?\d+\.\d+[eE][+-]?\d+)\s*([+-]?\d+\.\d+[eE][+-]?\d+)\s*([+-]?\d+\.\d+[eE][+-]?\d+)\s*$',
            block[0].strip()
        )
        if match:
            prn = int(match.group(1))
            year = int(match.group(2))+2000
            month = int(match.group(3))
            day = int(match.group(4))
            hour = int(match.group(5))
            minute = int(match.group(6))
            second = float(match.group(7))

            clock_bias = float(match.group(8))
            clock_drift = float(match.group(9))
            clock_drift_rate = float(match.group(10))
        else:
            print(f"首行匹配失败: {block[0]}")
            i += 8
            continue
        # 计算MJD
        jd = gregorian_to_jd(year, month, day, hour, minute, second)
        mjd = jd_to_mjd(jd)
        # 处理后面7行：每行4个参数
        data = {}
        param_names = [
            'IODE', 'Crs', 'deltan', 'M0',
            'Cuc', 'e', 'Cus', 'sqrtA',
            'TOE', 'Cic', 'OMEGA', 'Cis',
            'i0', 'Crc', 'omega', 'deltaomega',
            'IDOT', 'L2code', 'GPSweek', 'L2Pflag',
            'sACC','sHEA', 'TGD', 'IODC',
            'TTN', 'fit', 'spare1', 'spare2'
        ]

        for j in range(1, 8):
            # 处理每行中的科学计数法，期望每行有 4 个数字
            nums = re.findall(r'([+-]?\d+\.\d+[eE][+-]?\d+)', block[j].strip())

            # 检查是否有 4 个符合要求的数字
            if len(nums) != 4:
                print(f"参数行解析失败: {block[j]}")
            else:
                # 将符合要求的数字转化为浮点数并存储为字典中的键值对
                for idx, param in enumerate(param_names[(j-1)*4:(j)*4]):
                    data[param] = float(nums[idx])

        if len(data) != 28:  # 7行，每行4个参数
            print(f"参数总数不足: {len(data)}，跳过该块")
            i += 8
            continue

        # 整合数据
        record = {
            "mjd": mjd,  # 只返回MJD
            "prn": prn,
            "clock_bias": clock_bias,
            "clock_drift": clock_drift,
            "clock_drift_rate": clock_drift_rate,
            **data  # 每个参数作为字典的一部分
        }
        nav_records.append(record)

        i += 8  # 跳到下一个数据块
    return nav_records 
根据时间简化儒略日（mjd）计算目标时刻的北斗卫星的位置与速度的代码： 
GEO卫星还是一直会出现精度不高的问题，但其他卫星精度都很高。   
import numpy as np
import math

from GNSStools.rinex_reader import read_nav

#constant value
gme_cgcs2000=3.986004418e14
wearth_cgcs2000=7.2921150e-5
fact=1.0
import numpy as np

def rotation_matrix(ldot: bool, iaxis: int, angle: float) -> tuple:
    """
    Generate rotation matrix and its derivative with improved clarity.

    Parameters:
        ldot: If True, compute derivative matrix.
        iaxis: Rotation axis (1=X, 2=Y, 3=Z).
        angle: Rotation angle in radians.

    Returns:
        (rotmat, drotmat): Rotation matrix and its derivative.
    """
    if iaxis not in {1, 2, 3}:
        raise ValueError("iaxis must be 1 (X), 2 (Y), or 3 (Z)")

    # Axis mapping: (i, j) for non-rotation axis elements
    axis_pairs = {1: (2, 3), 2: (3, 1), 3: (1, 2)}
    i, j = axis_pairs[iaxis]

    # Handle small angles for numerical stability
    if abs(angle) < 1e-10:
        sina = angle
        cosa = 1.0 - 0.5 * angle**2
    else:
        sina = np.sin(angle)
        cosa = np.cos(angle)

    # Initialize matrices (Fortran column-major order)
    rotmat = np.eye(3, order='F')  # Start from identity matrix
    drotmat = np.zeros((3, 3), order='F') if ldot else None

    # Fill non-diagonal elements
    rotmat[i-1, i-1] = cosa
    rotmat[j-1, j-1] = cosa
    rotmat[i-1, j-1] = sina
    rotmat[j-1, i-1] = -sina

    # Derivative matrix
    if ldot:
        drotmat[i-1, i-1] = -sina
        drotmat[j-1, j-1] = -sina
        drotmat[i-1, j-1] = cosa
        drotmat[j-1, i-1] = -cosa

    return rotmat, drotmat
def brdxyz_beidou(cmode, EPO, TT ):
    result=np.zeros(6)
    dtsat = 0
    dt=(TT-EPO['mjd'])*86400-14
    if cmode[2]=='y' :
        dtsat=EPO['clock_bias']+(EPO['clock_drift']+EPO['clock_drift_rate']*dt)*dt
    if cmode[0:2]=='nn':
        return
    a=EPO['sqrtA']**2
    xn=math.sqrt(gme_cgcs2000/a/a/a)
    xn=xn+EPO['deltan']

    xm=EPO['M0']+xn*dt
    ex=xm
    e=EPO['e']
    for _ in range(12):
        ex=xm+e*math.sin(ex)

    v0 = 1.0 - e * math.cos(ex)
    vs = math.sqrt(1.0 - e * e) * math.sin(ex) / v0
    vc = (math.cos(ex) - e) / v0
    v = abs(math.asin(vs))
    if vc >= 0:
        if vs < 0:
            v = 2 * math.pi - v
    else:
        if vs <= 0:
            v = math.pi + v
        else:
            v = math.pi - v
    phi = v + EPO['omega']
    ccc = math.cos(2 * phi)
    sss = math.sin(2 * phi)

    du = EPO['Cuc'] * ccc + EPO['Cus'] * sss
    dr = EPO['Crc'] * ccc + EPO['Crs'] * sss
    di = EPO['Cic'] * ccc + EPO['Cis'] * sss

    r = a * (1 - e * math.cos(ex)) + dr
    u = phi + du
    xi = EPO['i0'] + di + EPO['IDOT'] * dt

    xx = r * math.cos(u)
    yy = r * math.sin(u)



    vel=np.zeros(3)


    pos = np.zeros(3)
    pos[0] = xx
    pos[1] = yy

    xnode = EPO['OMEGA'] + (EPO['deltaomega'] - wearth_cgcs2000 * 0) * dt
    xnode = xnode - wearth_cgcs2000 * EPO['TOE']
    rot=np.zeros((3,3))
    drot=np.zeros((3,3))
    rot,drot=rotation_matrix(True,3,-xnode)
    rottmp=np.zeros((3,3));drottmp=np.zeros((3,3))
    rottmp,drottmp=rotation_matrix(True,1,-xi)
    rot=np.dot(rot,rottmp)

    rot_save=np.zeros((3,3))


    if EPO['i0']<0.4:
        rottmp,drottmp=rotation_matrix(False,1,-5.0/180*np.pi)
        rot=np.dot(rottmp,rot)
        rot_save=rot
    rottmp,drottmp=rotation_matrix(True,3,wearth_cgcs2000*dt)
    rot=np.dot(rottmp,rot)
    if EPO['i0']<0.4:
        rot_save=np.dot(rottmp,rot_save)
    else:
        rot_save=rottmp
    pos=np.dot(rot,pos)
    vel=np.zeros(3)
    if cmode[1]=='y' :
        d_ex = xn / (1.0 - e * np.cos(ex))
        d_v = np.sin(ex) * d_ex * (1.0 + e * np.cos(v)) / (1.0 - e * np.cos(ex)) / np.sin(v)
        d_u = d_v + 2.0 * (EPO['Cus'] * ccc - EPO['Cuc'] * sss) * d_v
        d_r = a * e * np.sin(ex) * d_ex + 2.0 * (EPO['Crs'] * ccc - EPO['Crc'] * sss) * d_v
        d_i = EPO['IDOT'] + 2.0 * (EPO['Cis'] * ccc - EPO['Cic'] * sss) * d_v
        d_omega = EPO['deltaomega']

        # 计算速度在轨道平面中的分量
        xpdot = d_r * np.cos(u) - r * np.sin(u) * d_u
        ypdot = d_r * np.sin(u) + r * np.cos(u) * d_u

        vel[0]=xpdot
        vel[1]=ypdot
        vel[2]=0

        vel=np.dot(rot,vel)

        vel[0]=vel[0]+pos[0]*wearth_cgcs2000
        vel[1]=vel[1]-pos[0]*wearth_cgcs2000

        xtmp=np.zeros(3)
        xtmp[0] = yy * np.sin(xi) * np.sin(xnode) * d_i
        xtmp[1] = -yy * np.sin(xi) * np.cos(xnode) * d_i
        xtmp[2] = yy * np.cos(xi) * d_i

        xtmp=np.dot(rot_save,xtmp)
        xtmp[0] += (-xx * np.sin(xnode) - yy * np.cos(xnode) * np.cos(xi)) * d_omega
        xtmp[1] += (xx * np.cos(xnode) - yy * np.sin(xnode) * np.cos(xi)) * d_omega

        xtmp=np.dot(rot_save,xtmp)

        vel=vel+xtmp

    result=[np.concatenate((pos,vel)),dtsat]


    return result





 
学习不易，如有转载，请与博主联系
