# 【Matlab】海底声学模拟（Bellhop）以及滤波器的设计

> 来源：[CSDN 原文](https://blog.csdn.net/m0_49684834/article/details/128335058)  
> 发布时间：2022-12-16  
> 标签：无

---

BELLHOP 模型是通过高斯波束跟踪方法(Porter 和 Bueker，1987 年)，计算 水平非均匀环境中的声场。高斯波束跟踪方法对于高频水平变化问题特别有吸 引力，这是简正波、波数积分和抛物线模型不可替代的。高斯束射线跟踪法的基 本思想是将高斯强度分布与每条声线联系起来，该声线为高斯声束的中心声线, 这些声线能较平滑的过渡到声影区，也能较平滑的穿过焦散线，所提供的结果与 全波动模型的结果更为一致。

---

一、设计要求 
  某单波束测深仪最大测量水深为300米，请根据《水声学原理》和《数字信号处理》相关知识，仿真设计该单波束测深仪的数字信号处理系统（包括模拟滤波器参数、采样频率、量化精度等工作参数；FIR/IIR滤波器设计，并对数字信号进行：匹配滤波；底检测；底跟踪和声呐图绘制等处理）。 
 
 （PS：需要全部代码文件文件请点击这里，需要Bellhop使用说明书请点击这里。） 
 
二、采样数据模拟生成 
1.理想条件下声呐采样波形生成 
1.1假设出的理想条件： 
（1）基于射线声学理论 
（2）几何衰减按球面波传播衰减规律衰减，不考虑吸收衰减 
（3）仅考虑水底的反射 
（4）考虑在高斯白噪声背景下 
（5）整个空间声速分布均匀 
1.2在假设信号发射的时刻为零时刻的前提下，输入的参数说明 
表1 输入参数说明 
 C=1500  声速  单位：m/s  f0=15e3  信号频率  单位：Hz  fs=f0*10  采样率，最高频率的5倍  单位：Hz  Tao=5/f0  信号脉宽  单位：s  Sample_time=0.1  采样时长  单位：s  SNR=60  信噪比  单位：dB  H=10  水深  单位：m  H1=5  发射换能器水深  单位：m  H2=5  接收换能器水深  单位：m  D=1  接收与发射换能器水平距离  单位：m  Re_coef_bottom  水底反射系数  单位：无  
1.3生成发射信号并模拟反射环境 
data_generation.m 
function receive_signal = data_generation(c,fs,f0,Tao,Sample_time,SNR,H,H1,H2,D,Re_coef_bottom)
%**********************************通信声呐水听器数据生成函数************************************
%几点假设：
%（1）基于射线声学理论
%（2）几何衰减按球面波传播衰减规律衰减，不考虑吸收衰减
%（3）仅考虑水底的反射
%（4）考虑在高斯白噪声背景下
%（5）整个空间声速分布均匀
%***************************************输入参数说明******************************************
% c                  声速             Unit：m/s
% fs                 采样率           Unit：Hz
% f0                 信号频率         Unit：Hz
% Tao                信号脉宽         Unit：s
% Sample_time        采样时长         Unit：s      假设信号发射的时刻为零时刻
% SNR                信噪比           Unit：dB
% H                  水深                    Unit：m
% H1                 发射换能器水深           Unit：m
% H2                 接收换能器水深           Unit：m
% D                  接收与发射换能器水平距离  Unit：m
% Re_coef_bottom     水底反射系数  存在半波损失
Ts = 1/fs;      %采样时间间隔
sample_num = fix(Sample_time*fs);       %采样总点数
nTs = (0:sample_num-1)/fs;              %离散的采样时刻
sample_num1 = fix(Tao*fs);              %信号的采样点数
nTs1 = (0:sample_num1-1)/fs;            %信号的离散的采样时刻
receive_signal = zeros(size(nTs));      %用于存储水听器接收的信号
d0 = sqrt((H1-H2)^2+D^2);     %两个换能器的直线距离
S0 = 1/d0;                    %直达波的声压幅值
Noise_var = 10^(-SNR/20)*S0;  %白噪声的方差
%% 计算到达回波信号
real_time = d0/c;                                     %信号的实际到达时刻
signal_start_time = fix(real_time*fs)+1;              %信号第一个采样点的时刻
phase = (signal_start_time*Ts-real_time)/Ts*2*pi;     %得到信号的第一个采样点的相位 
receive_signal(signal_start_time:(signal_start_time+sample_num1-1)) = S0*sin(2*pi*f0*nTs1+phase); %模拟采样
receive_signal = receive_signal + Noise_var*randn(size(nTs));     %加入噪声
M=power(D/(2*H-H1-H2),2);
d1=sqrt(M+1)*(H-H1);
d2=sqrt(M+1)*(H-H2);
dib=  d1+d2  %反射波总路程
Sib=1/dib*Re_coef_bottom;   %反射波的声压幅值
real_time = dib/c;                                                   %信号的实际到达时刻
signal_start_time = fix(real_time*fs)+1;                             %信号第一个采样点的时刻
phase = (signal_start_time*Ts-real_time)/Ts*2*pi;                    %得到信号的第一个采样点的相位 
receive_signal(signal_start_time:(signal_start_time+sample_num1-1)) =...
receive_signal(signal_start_time:(signal_start_time+sample_num1-1)) + Sib*sin(2*pi*f0*nTs1 + phase); %模拟采样                  
receive_signal = receive_signal/S0;    %将直达波的幅值归一化
 
调用上述函数，主运行函数（test.m） 
close all;
clear all;
clc;
c = 1500;            %声速             Unit：m/s
f0 = 15e3;           %信号频率         Unit：Hz
fs = f0*10;          %采样率 最高频率的5倍   Unit：Hz
Tao = 5/f0;         %信号脉宽         Unit：s
Sample_time = 0.1;   %采样时长         Unit：s      假设信号发射的时刻为零时刻
SNR = 40;            %信噪比           Unit：dB
H = 10;       %水深                    Unit：m
H1 = 5;       %发射换能器水深           Unit：m
H2 = 5;       %接收换能器水深           Unit：m
D = 1;       %接收与发射换能器水平距离  Unit：m
Re_coef_bottom = 1;   %水底反射系数  
%%
sample_num = fix(Sample_time*fs);       %采样总点数
nTs = (0:sample_num-1)/fs;              %离散的采样时刻
%生成水听器接收数据
receive_signal0 =data_generation(c,fs,f0,Tao,Sample_time,SNR,H,H1,H2,D,Re_coef_bottom);
figure;
plot(nTs,receive_signal0);
xlabel('时间s');
ylabel('幅度');
string = ['水听器接收的仿真数据,信噪比SNR=',num2str(SNR),'dB'];
title(string);
 
运行后得到图像： 
 
  得到理想情况下的仿真数据。 
2.Bellhop射线模型与参数设计 
2.1Bellhop模型的介绍： 
  BELLHOP 模型是通过高斯波束跟踪方法(Porter 和 Bueker，1987 年)，计算 水平非均匀环境中的声场。高斯波束跟踪方法对于高频水平变化问题特别有吸 引力，这是简正波、波数积分和抛物线模型不可替代的。高斯束射线跟踪法的基 本思想是将高斯强度分布与每条声线联系起来，该声线为高斯声束的中心声线, 这些声线能较平滑的过渡到声影区，也能较平滑的穿过焦散线，所提供的结果与 全波动模型的结果更为一致。 
2.2Bellhop模型的使用： 
（1）确定env文件。 
  具体操作请参考Bellhop使用说明书。 
（2）设计程序 
  设计程序实现任意信号输入，实现接收信号的Bellhop模型模拟。 
  bharr2ir.m 
function ir_vec=bharr2ir(delay, amp, sampling_rate)

valid_delay_index=find(delay>0);
if isempty(valid_delay_index)
    disp('[bharr2ir]Error: Zero path simulated by BELLHOP.');
    return;
end
delay_vec=delay(valid_delay_index);
amp_vec=amp(valid_delay_index);

%单位冲激响应进行采样
delay_min=min(delay_vec);
delay_max=max(delay_vec);

cir_length=round(delay_max*sampling_rate);
ir_vec=zeros(cir_length, 1);

%find individual ray paths
for icn=1: length(delay_vec)
    %calculate the arrival index
    %init_delay gives some zeros priror to the first path
    arr_id=round(delay_vec(icn)*sampling_rate);
    
    %generate impulse response. Note that sometime, multiple returns can be
    %generate for the same delay in BELLHOP. 
    ir_vec(arr_id)=ir_vec(arr_id)+amp_vec(icn);
end
 
 ambientnoise_psd.m 
function [npsd_db]=ambientnoise_psd(windspeed, fc)
%考虑海洋湍流和风力

%Turbulance noise湍流噪声
ANturb_dB=17-30*log10(fc/1000);

%Ambient noise in dB (wind driven noise)
ANwind_dB=50+7.5*sqrt(windspeed)+20*log10(fc/1000)-40*log10(fc/1000+0.4);

%Thermo noise in dB (wind driven noise)热噪声
ANthermo_dB=-15+20*log10(fc/1000);

%Total noise PSD
npsd_db=10*log10(10^(ANturb_dB/10)+10^(ANwind_dB/10)+10^(ANthermo_dB/10));
 
signal_mf.m 
function est_ir_vec=signal_mf(rx_signal, tx_source, cir_length)

ttl_samples=length(tx_source);

%generate matched-filter from the source signal
txsig_flip=conj(tx_source(end:-1:1));

%matched-filtering
%division by ttl_samples is necessary for normalization
est_ir_vec=conv(txsig_flip, rx_signal)/ttl_samples; 
 uw_isi.m 
function [y, adj_ir_vec]=uw_isi(ir_vec, sl_db, tx_source, nv_db)
%与单位冲激响应函数卷积并加入噪声
%发送信号与信道冲激响应进行卷积
tx_sig=conv(tx_source, ir_vec);

%接收信号信噪比
SNR=sl_db-nv_db;

%信道输出
y=awgn(tx_sig,SNR);
adj_ir_vec=ir_vec;
 
 Bellohop.m 
function[rx_signal,Ts]=Bellohop()%rx_signal信号序列 fs采样频率
clear; close all; clc; 

%采样率
sampling_rate=2000e3;

%发射机中心频率
%该值和计算噪声有关
fc=200e3;
windspeed=5;

radio=sampling_rate/fc  %采样率与中心频率的比值
%发射机参数
sl_db=200;  %发射机声源级
bw=8e3;     %带宽

%通过Actup的arr文件获取所需的幅值和时延（单位冲激响应）
%BELLHOP run ID
env_id='env';
%read BELLHOP arr file:
[ amp1, delay1, SrcAngle, RcvrAngle, NumTopBnc, NumBotBnc, narrmat, Pos ] ...
    = read_arrivals_asc( [env_id '.arr'] ) ;
[m,n]=size(amp1);
amp=amp1(m,:);
delay=delay1(m,:);

%Step 1: 创建任意波形（这里以正弦波为例）
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
F_Serial_Signal=[zeros(1,1),ones(1,1),zeros(1,10)];  %待发送串行数据
Signal_L=length(F_Serial_Signal);                    %发送数据长度
communication_rate=40e3;                       %通信速率 
Communication_radio= sampling_rate/communication_rate; %采样倍数
signal=repmat(F_Serial_Signal,Communication_radio,1);
signal2=reshape(signal,1,Signal_L*Communication_radio);      %调整后的数据
signal_length=length(signal2);                 %数据长度
t=0:1/sampling_rate:(signal_length-1)/sampling_rate;      %时间
modulation_signal=cos(2*pi()*fc*t); %载波信号
          


tx_source=signal2.*modulation_signal;          %发送

%Step 2:加入噪声和多径影响
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Narrmx=10; %limit ourselves to use the first Narrmax paths

%对单位冲激响应进行采样
ir_vec=bharr2ir(delay, amp, sampling_rate);
%strongest tap in dB
maxamp_db=20*log10(max(abs(amp)))+sl_db;%声源级（dB）加由于信道衰减的能量（dB），表示接收到的信号的能量

%噪声级
[npsd_db]=ambientnoise_psd(windspeed, fc);%npsd_db是噪声的能量谱密度（单位频带内的能量）转换成dB，
nv_db=npsd_db+10*log10(bw);%相当于能量谱密度乘以带宽，即该频带内的噪声的能量
% maxamp_db和nv_db的差值应该为信噪比
disp(['Strongest tap strength=' num2str(maxamp_db,'%.1f') ' dB; Noise variance=' num2str(nv_db,'%.1f') 'dB']);

%考虑多径和噪声影响后的接收信号响应
[rx_signal, adj_ir_vec]=uw_isi(ir_vec, maxamp_db, tx_source, nv_db);

Ts=1/sampling_rate;

t2=0:Ts:(length(rx_signal)-1)*Ts;

figure(1)
subplot(2,1,1)
plot(t,tx_source)
title('发送信号')
xlabel('时间/s')
ylabel('幅值(V)')
set(gca, 'Fontsize', 16);

subplot(2,1,2)
plot(t2,rx_signal)

title('接收到的信号')
xlabel('时间/s')
ylabel('幅值(V)')
set(gca, 'Fontsize', 16);
end
 
效果实现如下： 
 
2.3绘制冲激响应图 
 plotting.m 
clear
clc
filename = 'env.arr';
Minimum_range=100  %（接收水听器的水平方向上接收范围最小值，m）----R 
Maximum_range=1000 %（接收水听器的水平方向上接收范围最大值，m）---RB 

[ amp1, delay, SrcAngle, RcvrAngle, NumTopBnc, NumBotBnc, narrmat, Pos ]... 
 = read_arrivals_asc(  filename );
%%单位冲激响应
[m,n]=size(amp1);
amp = abs(amp1); %取模  
x = delay(m,:); %获取第50个接收机的时延和幅值
y = amp(m,:);
figure(1)
stem(x,y)
grid on
xlabel('相对时延/s')
ylabel('幅度')
title('单位冲激响应')

%%归一化冲激响应
Amp_Delay = [x;y];
Amp_Delay(:,all(Amp_Delay==0,1))=[]; %去掉0值
Amp_Delay=sortrows(Amp_Delay',1);  %按照时延从小到大排序
normDelay = Amp_Delay(:,1)-Amp_Delay(1,1);%归一化时延
normAmp = Amp_Delay(:,2)/Amp_Delay(1,2);%归一化幅度
figure(2)
stem(normDelay,normAmp,'^')
grid on
xlabel('相对时延/s')
ylabel('归一化幅度')
title('归一化冲激响应')

figure(3)
mum=1:m;
ReRange = Minimum_range+(Maximum_range-Minimum_range)/m*mum;
for i=1:min(narrmat)
plot(delay(:,i),ReRange,'o')
hold on
end
hold off
grid on
colorbar
xlabel('时延(sec)')
ylabel('Range(m)')
title(filename)
 
生成结果如下： 
 
 
三.滤波器设计方案 
对于本次问题采样IIR滤波器： 
切比雪夫Ⅱ型滤波器，种类为带通滤波器 
Fstop1=180KHz 
Fpass1=190KHz 
Fpass2=210KHz 
Fstop2=220KHz 
Astop1=90 
Apass=1 
Astop2=90 
阶：22阶 
节：11 
稳定滤波幅值响应 
FS：采样频率为2000KHz 
滤波器的幅值响应图像： 
 
 滤波器设计代码如下： 
function Hd = filter_2nd
%FILTER_2ND 返回离散时间滤波器对象。
% MATLAB Code
% Generated by MATLAB(R) 9.11 and DSP System Toolbox 9.13.
% Generated on: 14-Dec-2022 16:14:37
% Chebyshev Type II Bandpass filter designed using FDESIGN.BANDPASS.
% All frequency values are in kHz.
Fs = 2000;  % Sampling Frequency
Fstop1 = 180;         % First Stopband Frequency
Fpass1 = 190;         % First Passband Frequency
Fpass2 = 210;         % Second Passband Frequency
Fstop2 = 220;         % Second Stopband Frequency
Astop1 = 90;          % First Stopband Attenuation (dB)
Apass  = 1;           % Passband Ripple (dB)
Astop2 = 90;          % Second Stopband Attenuation (dB)
match  = 'stopband';  % Band to match exactly
% Construct an FDESIGN object and call its CHEBY2 method.
h  = fdesign.bandpass(Fstop1, Fpass1, Fpass2, Fstop2, Astop1, Apass, ...
                      Astop2, Fs);
Hd = design(h, 'cheby2', 'MatchExactly', match);

% [EOF] 
 
 滤波后结果如图： 
 
 
 学习不易，转载请联系博主。
