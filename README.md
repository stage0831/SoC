# LAB1
## vitis_hls part
### 前置動作
先使用下面指令將lab資料存入ubuntu。
- git clone https://github.com/bol-edu/course-lab_1 ~/course-lab_1

接著在course-lab_1(lab的資料夾中)開啟vivado_hls
- vivado_hls

new一個新的project，名稱通常取作hls_ip。

接著更改part selection為：
- xc7z020clg400-1

進到project後，要加入或撰寫新的cpp file或是.h file以及cpp的testbench，建議使用new source file。
要注意include的檔案名稱的大小寫，通常會被自動改為小寫，需更改成符合自己的.h檔。

### 設定directive
下一步是在設定directive的動作，有以下兩種方式

1. 使用#pragma 如下圖：
![](https://hackmd.io/_uploads/S1x17RHJT.png)

被註解掉的那行是Block-level Protocol，只有在top moudle會需要定義，因為只有top module會跟外面的IP對接。 常用的是
- ap_ctrl_hs 
- ap_ctrl_chain (high-performance 可pipeline)
- ap_ctrl_none  (data-driven 有資料就動)
- ap_hs
- ap_fifo

而其他的port會有Port-level Protocol，若funtion輸入是一筆資料則為Scalar input，若是輸入一個pointer則為pointer to array，而function的return value也算是Scalar，因此也是使用s_axilite。
- Scalar input (不是連續的那種)：s_axilite (AXI_Lite)
- pointer to array :使用MEM來實現，interface為m_axi (AXI_Master)，但一樣是使用s_axilitex來傳遞base address



2. 使用directive如下圖：
![](https://hackmd.io/_uploads/Bkr0ECSkp.png)
- 紅色筆位置點選來選擇要做的directive
- 藍色筆圈的檔案可以看目前的設定如下圖：

![](https://hackmd.io/_uploads/rk8rBCSJ6.png)

通常使用第二種方式的時機是在調整架構時，會嘗試不同的設定來跑。
而若是要將自己的設計交給別人時會使用第一種方式，因為第一種方式才不會遺失directive的設定。若沒有directive的設定，別人要使用時也不知道directive該設什麼。

### 模擬
- 要先到Project -> Project Settings -> Synthesis Settings - Top Function name，設定top function。
- 要跑模擬有時需要註解掉某些#pragma
- 接著跑C Simulation、C Synthesis 以及 Co-simulation。
- 要跑Co-simulation似乎只需先跑C Synthesis，可以不用先跑C Simulation(有待驗證)。
- 在跑Co-simulation前可以設定Dump trace，all的話可以看到全部的波型，port似乎是只有io部分有。
- testbench跑出來的結果會存在TopModuleName_csim.log檔。(要繳交) 

### 匯出RTL
- 須把註解掉的#pragma還原。
- 在simulation的綠色三角形中有個export IP的選項可以匯出RTL
- 匯出後會加上一些控制訊號變成AXI的interface如下圖

![](https://hackmd.io/_uploads/B1ulrsLyp.png)

這樣到了Block Design的環節才能夠與其他東西對接

## vivado part
### 前置動作
在course-lab_1(lab的資料夾中)開啟vivado
- vivado

new一個新的project，名稱通常取作vivado。

選擇RTL Project，並選擇xc7z020clg400-1。

進入後，選擇
- setting -> IP -> Repository -> add(加號) -> 選擇hls_ip

在彈出的視窗中選擇top module

### creat block design
點選在左邊的Flow Navigator底下的
- IP INTEGRATOR -> Create Block Design

在右邊的Diagram視窗下按add(加號)並搜尋
- ZYNQ

接著在同樣的Diagram視窗下按add(加號)並搜尋要用的top module。
然後按下重新整理鍵。

接著按下Diagram視窗下的綠色提示的
- Run Block Automation

跳出視窗按下ok後，連點兩下ZYNQ進到ZYNQ Processing System，點選
- Clock Configuration

調整FCLK_CLK0的頻率(100)。


再按下
- Run Connection Automation

來完成接線

在Sources中
- 右鍵design_1 -> create HDL wrapper -> Let vivado manage wrapper and auto-update

會看到下圖的訊息
![](https://hackmd.io/_uploads/Sy42Cq816.png)

### Generate FPGA bitstream
點選在左邊的Flow Navigator底下的

- PROGRAM AND DEBUG -> Generate Bitstream

跳出視窗按ok，下一個Launch Runs也是ok，下一個Bitstream Generation Compeleted也ok(預設是Open Implemented Design

接著到MobaXterm的ubuntu輸入(粗體字的部分需修改)
- cd ~/**course-lab_1**
- cp ./vivado/vivado.runs/impl_1/**design_1_wrapper.bit** ./**Multip2Num.bit**
- cp ./vivado/vivado.gen/sources_1/bd/design_1/hw_handoff/**design_1.hwh** ./**Multip2Num.hwh**

### 租借onlineFPGA
用MobaXterm連到租借平台
- boledupynq
- GlNDat

用拿到的網址和密碼進入jupyter notebook，並上傳檔案
- .hwh
- .bit
- .ipynb

按下Run得到結果


# LAB2
## issue (II violation)
https://zhuanlan.zhihu.com/p/447488933

在實作lab2的stream的部分時，在做完C synthesis時觀察report的時候發現有個II violation如下
![](https://hackmd.io/_uploads/BJmwopYyp.png)


### 問題與解法
- Latency (cycles): 表示一個loop總共要跑多少cycles才能跑完。
- iteration: 表示loop中的一次循環。
- iteration interval (II):代表一個loop中，兩次的iteration間隔的cycles，如下圖
- 
II = 3
![](https://hackmd.io/_uploads/Sy4iT6tkp.png)

II = 1
![](https://hackmd.io/_uploads/rkmaTpFka.png)

---

可使用兩種方法來解決
1. 加上directive
2. 修改c code

要找到問題來源
- guidance -> 點選那條warning
![](https://hackmd.io/_uploads/rydDoaK1p.png)

再開啟schedule viewer
- solution -> open schedule viewer

可以看到XFER_LOOP的II=11
![](https://hackmd.io/_uploads/H1tM-AFyT.png)

然而並不是因為這部分的指令要花費11個週期才能完成，而是因為data dependency造成這條指令無法被pipeline成II=1的情況。

解決方法是
- 將XFER_LOOP設定directive為unroll

![](https://hackmd.io/_uploads/r1qXHRFJa.png)


這樣HLS會將電路複製N份，讓資源足夠做出II=1的pipeline，此種方法只能在迴圈次數是固定的情況下使用，如下圖，在FIR.h中N被宣告為11這個常數，因此這個方法才有效。
![](https://hackmd.io/_uploads/H1l5ICtya.png)

![](https://hackmd.io/_uploads/B1HrD0tya.png)
