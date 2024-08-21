#he zyieo.github.io

# **== 关于家用nas的一点思考 ==**

## 0. 复合方案的前提
 
单机nas，性能与功耗之间必然存在冲突，很难进行取舍。
通常复合方案可以较好地解决，发挥各自所长。

> 注意到用户真实面对的是服务和数据，落实在物理机还是虚拟机上对用户而言是被隔离的。

2024年8月19日  
## 1. 不要任何事情都要全体成员一起参与。

因此，借助这个特点，对于各类服务可以，其运行的硬件可以有一定的选择自由。可以根据需求特点，当需要运行某一服务时，选择开通对应的实体物理机。而非启动任何服务，都需要打开所有的物理机（单机），以至于物理系统始终处于较高的功耗上。


docker只要有一个实例启动，等于绑定了所有docker的硬件一起参与。
引起不必要的功耗。
异构主机方案，解绑了。
着眼点在于，服务、数据。各机提供各自能力范围内的服务，各机同时提供各自的数据给外界共享。
由用户根据需要决定选择哪个机子进行服务。
按用户参与的同步方式，划分为伴随型、自动型。

* 异构的是物理机，统一的是面对用户的服务和数据。 *

## 2. 用户使用的策略是关键，同时考虑用自动化的方式，启停重型机服务。


- 一种模式是3机自由选择，或者建立一个统一的操作页面。
- 一种是串列成一台大服务器，只是根据应用需求，点亮其中不同模块。

### 2.1. 辅以自动化实现数据流的方向。这个需要使用脚本。

各物理机彼此独立运行，数据备份策略由各机自行用脚本完成。admin实现规划好数据备份策略，及在各机的实现，然后部署到相关物理机。
好处是互相影响不大，附带的好处是易于扩展。无论AB机，增加一个实例对上游机是透明的。

### 2.2. 一些其他考虑

- 数据重要程度不同
- 数据使用模式不同
- 数据规模不同

== 根据这些不同，规划出存储形式和备份策略不同，方案提供的解决手段应给予支持 ==

2024年8月18日
## 3. 目前的家用解决方案都是照抄企业级方案，专注于技术实现，对总体方案规划有所不足。

对于家用比较注重的功耗问题，多是在主板cpu方面的功耗上进行取舍选择，整体则视而不见。对于家用，动辄4-5个硬盘挂机，甚至联动。硬盘单盘静态功耗3-6w。直接拉高了总体功耗。同时因为有nas长期在线的特点，对于冷备份多半是不屑一顾。


因为nas包含的各种应用，实际上各自的最优解决方案的需求是彼此冲突的。
1. 文件共享，只在工作时间要求有，并且本身也是分级，有些可以通过离线存档。对速度要求没有硬限制，当然越高越好。方便性有一定要求。
2. 同步，区分日常需要人参与型的与偶尔需求。前者使用时开机，数据盘长期在线。后者利用工作机辅助，数据盘偶尔挂载。当然实现的时候互为AB角色 —— 双方功能都提供，但是性能有区别。
3. 照片功能，要求方便性，对主机性能有很大的要求，偶尔性需求，对功耗限制不大。[照片处理服务](https://github.com/photoprism/photoprism)
4.


### 3.1. 使用场景：
1. 长期在线的部分功能，如数据下载， 简单的数据、照片存储； —— A机
2. 有人参与的情况下，使用的全功能的nas服务； —— B机
3. 周期性或者偶尔情况下， 离线保存重要数据，作为AB机的补充。其离线硬盘可以随时取下。 —— C机

### 3.2. 总体方案：
简单地说，轻重搭配的方案，把需要长期在线、维护的工具放在轻型机/低功耗机A上，人在才需要的服务放在日常正常开关机的重型机/普通nas机B上，配合自动脚本进行开机后数据维护。偶尔需要的服务放在工作机上，如工作站，其数据盘可随时离线，保证安全。
工作站C机一般作为客户机， 只对偶尔应用的服务提供支援，比如数据离线备份、
ABC三机出于同一Lan下，组成nas机群。
实质上是把主流nas方案里的docker退化为物理机，并提供不对称冗余。

任何单机都可以提供基本的nas服务，多机提供性能优化。 比如B机在插入新的外置硬盘后，也可以设置出发自动化备份照片与数据。
各单机在功能上开通各自所能开通的功能，
在使用策略上，从性能上使用各自擅长的机子，为方便性考虑，使用就近热机。
冗余的数据在冗余的机子上，同时也提供冗余的服务。

——为避免引起混乱，使用中注意存储数据（永久数据）流向：
数据流向大体是 A=>B=>C=>离线硬盘。

### 3.3. 在配置上，
A： A机使用树莓派、香橙派、机顶盒、电视盒、n1、玩客云之类超低功耗目标机，配合各种变形linux，主要是为支持下载BT，使用一个外挂的u盘作为下载存储空间；以及轻小的文件服务，避免每次都开通B机。最小限度地实现nas功能，主要目标是提供长期在线的便利。性能未必合适。
这是在功耗噪音vs性能方面做的一点取舍选择。

AB机之间协作的方式是B机开通后，使用自动化脚本更新至B机。
更进一步，可以考虑由A机控制B机的启停。

输入：
下载工具bt等客户端
小规模的照片、文件服务器，用于临时、轻小的服务；


输出：
开通smb等文件共享服务。

B：
B机在功耗上要求虽然有所降低，更多考虑足够的性能，但并非主力工作用机。考虑到应用场景，可以使用固态硬盘加速启动降低等待时间，以弥补非长期在线带来的时间延迟。
如果能远程控制B机启停则更佳。
系统与主要服务放在ssd盘上，如果使用紧凑机型，则考虑外挂机械盘作为数据存储主空间。长期外挂硬盘使用笔记本低速2.5吋盘，临时离线数据盘可以使用3.5吋的。这样来优化功耗。

输入：
主要是同步A机新数据
也可提供bt下载，同样考虑固态类的存储器，一般是U盘。

输出：
对外提供文件smb服务、照片服务
插入新硬盘，可触发离线数据备份服务
（定时自动+手动？依据用户习惯设定）

提供多媒体服务， emby dlna 等服务器
[1]李建文,刘笑麟,张锦飞等.提升核电厂冷源安全性的海生物探测技术研究[J].电力安全技术,2017,19(10):32-37.  
 [2]钟云泰.海水循环冷却系统海生物污染的控制[J].华东电力,2004,(08):30-33.  
 [3]阮国萍.核电厂取水口堵塞原因分析与应对策略[J].核动力工程,2015,S1:151-154.  
 [4]朱成军,王海超.核电厂取水明渠增设拦污网的必要性分析[J].机电信息,2016,(03):93-95+97.  
 [5]颜国呈,吕文婷.基于电厂取水外来物堵塞的预防管理[J].科技创新与应用,2016,(14):119.  
 [6]张爱玲,陈晓秋,刘森林,贾祥.我国内陆核电的用水安全[J].水文,2015,(03):69-73+25.  
 [7]王军伟,李世涛.沿海电厂海生物危害及对策[J].广东电力,2015,(09):33-36+84.  
 [8]张声强,缪飞,黄奇然.循环冷却水系统海生物污损化学控制[J].广东化工,2013,(22):101-102+96.  
 [9]孟亚辉,刘磊,郭显久等.核电厂冷源海生物探测预警及决策支撑系统研究[J].大连海洋大学学报,2018,33(01):108-112.  
 [10]Matthew R.Noatch and Cory D.Suski,Nonphysical barriers to deter fish movements[J].Environmental Reviews,2012,20(1):71-82.  
 [11]张立仁,乔娟,任海洋等.复合式驱鱼系统在水电工程中的应用研究[J].人民长江,2014(14):72-25.  
 [12]安德鲁·特恩彭尼.环境保护:拦鱼栅网选择指南[J].国际水力发电,1998(12):40-44.  
 [13]Klinet D.A.,Loeffelman P.H.,Van Hassel J.H.,Anew signal development process and sound system for diverting fish from water intakes[M].1982  
 [14]朱存良.鱼类行为生态学研究进展[J].北京水产,2008(1):20-24.  
 [15]Taft E P,Winchell F C,Amaral S V,eta1.Recent advances in sonic fish deterrence:Proceedings of the 1995 International Conference on Hydropower,July 25,July 28,1995[C].Holden:Alden Research Lab:1724-1733.  
 [16]殷勇勤,吕友林,赵传达等.声学驱鱼技术研究[J].船舶与海洋工程,2017,33(4):26-31.  
 [17]白艳勤,陈求稳,许勇等.光驱诱技术在鱼类保护中的应用[J].水生态学杂志,2013,34(4):85-88.  
 [18]李大鹏,庄平,王明学等.史氏鲟稚鱼的趋光性及不同光照周期对其生长的影响[J].华中农业大学学报,2001(6):564-567.  
 [19]危起伟,杨德国,柯福恩.长江中华鲟超声波遥测技术[J].水产学报,1998(3):20-26.  
 [20]罗清平,袁重桂,阮成旭等.孔雀鱼幼苗在光场中的行为反应分析[J].福州大学学报(自然科学版),2007(4):631-634.  
 [21]王萍,桂福坤,昊常文.光照对眼斑拟石首鱼行为和摄食的影响[J].南方水产,2009,5(5):57-62.  
 [22]Sager D R,Hocutt C H,Stsuffer J R.1987.Estuarine fish responses to strobe light,bubble curtains and strobe Light/bubble-curtain combinations as influenced by water flow rate and flash frequencies[J].Fisheries research,5:383-399.  
 [23]Patrick P H.Responeses of alewife to fashing light.Report to Ontario Hydro Research Division[C].Toronto,1982.  
 [24]赵锡光,何大仁.黑鲷和青石斑鱼对气泡幕反应的比较[J].青岛海洋大学学报(自然科学学版),1997,27(1):33-40.  
 [25]乔云贵,黄洪亮,陈帅等.气泡幕在鱼类行为研究中的应用[J].现代渔业信息,2001,26(12):29-32  
 [26]Breet J R,Alderdice,D F.Research on guiding young salm on at two Brinsh Columbia field ststions[J].Bull Fish Res BdCan,1958(117):l-75.
PhotoPrism：
https://docs.photoprism.app/getting-started/
https://demo.photoprism.app/library/folders
#he zyieo.github.io

https://hamradio.my/2024/05/setting-up-a-network-attached-storage-nas-using-ubuntu-at-home/


