## **LAB5 Python& PS 在FPGA上加速模擬SOC (硬體驗證)**

* lab3 使用testbench.v 單純的驗證.v
* lab4 在caravel SOC 驗證，缺點耗時
* lab5 FPGA模擬各IP(lab4中caravel soc ip) 配合ps python加快模擬

**LAB5 目的**  
1. 從LAB4-1了解MMIO&Firmware Code將LAB4-1的功能轉換為Harward
2. 將verilog testbench用python Code 取代
3. testbench 內的功能轉換為FPGA硬體


## **簡單執行**
### 1. 安裝包下載
https://github.com/bol-edu/caravel-soc_fpga-lab/tree/main/labi
### 2. 執行cmd
![image](https://hackmd.io/_uploads/ByEiIT0ra.png)
### 3. 將python檔案丟入租借的FPGA模擬
![image](https://hackmd.io/_uploads/ryrHPaCra.png)
### 4. 模擬結果如下
* Counter_wb
 ![image](https://hackmd.io/_uploads/ryYV_aAST.png)
 ![image](https://hackmd.io/_uploads/r1bfYTCB6.png)


* Counter_la.hex
![image](https://hackmd.io/_uploads/HyGf20CB6.png)
![image](https://hackmd.io/_uploads/rkczhRABp.png)

* gcd_la.hex
![image](https://hackmd.io/_uploads/SkeN2C0Sp.png)
![image](https://hackmd.io/_uploads/BJnIaCRSa.png)
   
   
## **硬體驗證步驟的觀念及架構**
SOC 架構可參考  https://hackmd.io/RAv93zJOSK2O11xULdYZOw

因為TESTBENCH還是由軟體的方式驗證SOC CODE 因此會耗費大量時間
LAB5將caravel SOC 內各IP實體化為FPGA，透過python控制FPGA 


## **硬體架構觀念**
### **RISC-V 用MMIO-Register 控制 MPRJ-IO ，PYTHON 讀取後與FPGA溝通**
**1. la的信號pin 溝通 SOC的RISC-V 和 USER PROJECT**  
 (la 的信號參考下面主題 soc程式介紹) 
 
**2. python code作為testbench同功能 用來測試硬體FPGA**  
 lab4中的verilog simulation 可以直接看chip signal
 但lab5用FPGA驗證所以使用python 透過AXI-LITE MMIO ADDRESS 控制FPGA

**3. RISC-V 跟外部TESTBENCH 溝通透過MPRJ-IO**  
  

#### 結構圖如下

![image](https://hackmd.io/_uploads/BJqScE1Ua.png)
**1. Caravel soc:**
如上圖所示，將上圖的soc整塊電路放在此block

**2. Spiflash: (save rom code)**
在verilog 中，原本只是一個behavior model，於FPGA中用硬體實現
BRAM內放置的firmware code透過spiflash的io pin實現cpu&soc溝通

**3. Read_romcode:**
將firmware code從系統的memory讀入FPGA中的BRAM

**4. ResetControl:**
用來確認FPGA的系統環境都已經READY :例如BRAM已經存取firmware等，release CARAVEL SOC的RESET PIN，使cpu & soc開始執行運算動作

**5. Caravel_ps:**
執行後的結果透過此module，CPU與SOC之間相互傳輸，可以把它視為mprj_io 控制器
BLOCK需要提供一個axlite map 給ps-cpu，讀取mprj_io的值以此獲取chip的各種狀況

由上述 1 ~ 3 BLOCK
當CPU reset 結束後，就會透過spiflash io pin 從BRAM讀取firmware code 輸入Caravel block

## **SOC程式介紹**
* ### **Firware Code (counter_la.c)**

---

檔案位置
 labi/vvd_srcs/caravel_soc/counter_la/counter_la.c (RISC-V 執行的 firmware code)   
 
**1. mprj 的位置定義**
設定每一個mprj屬性 如output/input pin
defs.h 有 memory map io (mprj 的位置)

![image](https://hackmd.io/_uploads/Bk40nHJUa.png)


**2. 起始設定  mprjio = 0xAB41 ，終點設定  mprjio = 0xAB51**
counter_la 是一個user project內的hardware
check counter_la data >0x1F4 輸入mprjio = 0XAB41
python code 將會確認mprjio 是否為0xAB51(CHECKBIT)
![image](https://hackmd.io/_uploads/ryv42SJUp.png)  

* ### **Main Design (user project)**

---
檔案位置
labi/vvd_srcs/caravel_soc/rtl/user/user_proj_example.counter.v    

**1. la data 設定counter value**
![image](https://hackmd.io/_uploads/HyKyP8kLp.png)



























