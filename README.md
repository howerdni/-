# -
# DIgSILENT based 电网动态仿真测试平台-苏州电网

## **1. 项目简介**

本项目旨在构建一个基于 **DIgSILENT PowerFactory** 的苏州电网仿真平台，能够进行潮流分析、短路计算，动态稳定性（机电、电磁暂态）仿真，平台包含 **HVDC-VSC**（特高压柔性直流输电）、**HVDC-LCC**（特高压常规直流输电）及**NE**（新能源）等新型电力系统模型。
苏州电网规模庞大，本平台通过等值建模技术，优化电网的计算效率，同时保留关键节点的动态特性，为电力规划、稳定性分析和新能源接入评估提供有力支持。

## **2. 苏州电网概述**

苏州电网位于中国华东地区，承担着江苏省重要的电力供应任务。其主要特点包括：

- **高负荷密度**：作为长三角经济圈的核心城市之一，苏州拥有高工业和商业负荷。
- **交直流混合系统**：电网中集成了 **LCC-HVDC**（特高压常规直流输电--锦苏直流）和 **VSC-HVDC**（白鹤滩柔性直流）输电技术，实现高效远距离输电与灵活调节。
- **新能源接入**：大规模风电和光伏等新能源逐步接入，对电网的调度和稳定性提出了更高要求。
- **高电压等级**：电网主要采用 **500kV** 和 **220kV** 电压等级，并配备多个直流输电接入点，以满足区域内外的电力需求。

## **3. 苏州电网主要分区**

苏州2027年负荷预计在32000MW~33000MW，本地机组供应其电力负荷的一半左右，其余电力均靠外来交直流输电供应。
苏州电网可划分为多个区域，各区域的负荷特点、机组分布及直流输电接入情况如下：

### **(1) 南部区域（木渎、吴江）**

- **负荷中心**：工业和商业负荷密集，昼夜负荷波动较大。
- **主要机组**：本地机组以燃气机组为主，承担调峰功能兼供电功能。
- **直流接入**：锦屏－苏南特高压直流输电的受端换流站位于该区域，接入500kV电网,承担主要的直流输电任务,通过500kV主变送到负荷端，**VSC-HVDC（白鹤滩—江苏工程--混合级联特高压直流工程）**，其1/6容量落于木渎电网。
- **主要变电站**：木渎、越溪、吴江，笠泽。

### **(2) 苏州城区**

- **负荷中心**：商业和居民负荷较高，具有明显的峰谷差。
- **主要机组**：本期燃气机组和分布式能源较为普遍。
- **直流接入**： **VSC-HVDC（白鹤滩—江苏工程--混合级联特高压直流工程）**，其1/6容量落于苏州城区电网。
- **主要变电站**：车坊、玉山。

### **(3) 昆山-太仓片区**

- **负荷中心**：电子产业园和大型制造企业较多，负荷增长迅速。
- **主要机组**：太仓地区燃煤机组和燃气轮机较多，多条 **500kV** 线路与上海电网互联。
- **主要变电站**：石牌、全福、太仓。

### **(4) 张家港-常熟区域**

- **负荷中心**：重工业和化工企业集中，负荷需求稳定且大。
- **主要机组**：大容量燃煤机组和燃气轮机。
- **直流接入**：**VSC-HVDC（白鹤滩—江苏工程--混合级联特高压直流工程）**，其1/6容量落于苏州城区电网。
- **主要变电站**：常熟北，常熟南，昭文。

## **4. 等值建模方法**

在苏州电网的 DIgSILENT 仿真平台中，我们采用**基于实际数据的等值方法**，对 **220kV** 电磁环网进行等值建模，确保仿真计算的**效率与准确性**。具体方法如下：

### **4.1 等值策略**

- **保留 500kV 主变 220kV 侧的母线**，确保 500kV 及以上电网的潮流和动态特性不受影响。
- **保留所有有机组接入的 220kV 母线**，使发电机的动态行为可用于仿真分析。
- **对其他 220kV 电磁环网进行等值**，减少不必要的计算量，同时保留等效的负荷和电压支撑能力。

### **4.2 等值方法**

等值方法主要包括**拓扑简化**、**导纳等值**和**负荷分配**。

### **4.3 等值网络算法简介**

在苏州电网的 DIgSILENT 平台中，我们采用**等值网络建模技术**来优化计算效率，同时保留关键节点的动态特性。以下是该方法的核心步骤：

#### **4.3.1 生成电网的邻接矩阵**

为了表示电网拓扑结构，我们构建了一个稀疏邻接矩阵，用于存储节点间的连接关系：

- **节点唯一标识**：采用 `(bus_name, vol_rank)` 作为节点的唯一标识，确保不同电压等级的母线不会混淆。
- **线路连接关系**：线路的连接关系存储为邻接矩阵（Adjacency Matrix），并使用稀疏存储格式（如 LIL/CSR）来优化计算性能。
- **变压器连接关系处理**：变压器的连接关系单独处理，以确保高压侧和低压侧的正确映射。

#### **4.3.2 识别电网中的变压器和关键节点**

- **变压器筛选**：通过遍历 TCard（两绕组变压器）数据，筛选出两绕组变压器并建立对应的连接关系。
- **广度优先搜索（BFS）**：采用 BFS 算法，在邻接矩阵上找到与给定起始母线电磁环网相连的所有节点，形成等值区域。

#### **4.3.3 计算节点导纳矩阵（Y 矩阵）**

导纳矩阵（Admittance Matrix, Y-Matrix）是潮流计算的核心：

- **导纳参数计算**：通过线路和变压器的阻抗参数（ $r$, $x$, $g$, $b$）计算自导纳和互导纳。
- **稀疏矩阵存储**：采用稀疏矩阵格式（如 LIL/CSR）存储，提升计算效率。
- **负荷矩阵构建**：结合潮流数据，构造负荷矩阵（S-Matrix），用于计算等值负荷分布参数。

#### **4.3.4 采用戴维南等值方法计算等值阻抗**

为了在减少网络规模的同时保留重要节点的动态特性，我们使用**戴维南等值（Thevenin Equivalent）**方法：

- **端口选择**：选取等值区域的关键母线作为端口。
- **等值阻抗矩阵计算**：

$$
Z_{\text{eq}} = M_A^T \cdot Y^{-1} \cdot M_A
$$

其中， $M_A$ 为端口选择矩阵， $Y^{-1}$ 是导纳矩阵的逆矩阵。

- **等值导纳矩阵转换**：

$$
Y_{\text{eq}} = Z_{\text{eq}}^{-1}
$$

  进一步转换为阻抗矩阵，并计算等值线路长度（Equivalent Line Length），用于与实际输电线路进行对比。

#### **4.3.5 计算等值负荷分布**

采用等值负荷分配方法，计算剩余母线的功率负荷：

$$
S_B = S_B - Y_{BE} \cdot Y_{EE}^{-1} \cdot S_E
$$

其中：

- $S_B$ 为等值区域的负荷分布
- $Y_{BE}$ 和 $Y_{EE}$ 分别为导纳矩阵的子矩阵
- $S_E$ 为被等值掉的区域的负荷

计算出的等值负荷分布可以用于动态仿真，保证等值后的系统仍然与原系统保持一致性。

### **4.4 算法优势**

- **计算效率高**：通过稀疏矩阵和等值建模技术，显著减少计算量，适用于大规模电网仿真。
- **保留关键特性**：在简化电网模型的同时，保留了关键节点的机组动态行为和电气特性，确保仿真结果的准确性。
- **灵活性强**：支持自定义节点顺序和多端口等效计算，适应不同的仿真需求。

### **4.5 应用场景**

- **电力系统规划**：辅助进行电网扩展和优化设计，评估不同方案的可行性和经济性。
- **稳定性分析**：评估电网在不同扰动下的动态响应，评估系统的稳态和暂态稳定。
- **新能源接入评估**：分析大规模新能源（如风电、光伏）、HVDC等电力电子设备和装置接入对电网的影响，优化新能源、新型电力系统装置与电网的匹配。

通过上述等值网络算法，苏州电网 DIgSILENT 仿真平台能够高效、准确地模拟和分析复杂的电力系统，为新型电力规划、运行和优化提供有力支持。

## **5. 机组动态控制系统模型与 BPA 对应关系**

本平台使用的机组动态控制系统模型与 BPA 的对应关系如下：

- **BPA 的 FV 卡（模型 FVUI）** 对应 **ESST1A（PSS/E）**
- **BPA 的 FQ 卡（模型 FQUI）** 对应 **ESAC2A（PSS/E）**
- **BPA 的 SI 卡（PSS 模型 SIUI）** 对应 **PSS2A（PSS/E）**
- **BPA 的 GS, TB 卡（原动机模型 GSTBUI）** 对应 **IEEEG1 模型**

在 DIgSILENT 平台上，对所有的同步机辅助控制系统均进行了模型等效替换，确保动态特性与原系统保持一致。具体对应关系如下表所示：

| BPA 模型卡 | 对应 DIgSILENT 模型 |
|------------|----------------------|
| FV 卡 (FVUI) | ESST1A/PSS/E               | 
| FQ 卡 (FQUI) | ESAC2A/PSS/E               | 
| SI 卡 (SIUI) | PSS2A/PSS/E                | 
| GS, TB 卡 (GSTBUI) | IEEEG1/PSS/E               | 

### **5.1 模型等效替换说明**

在本平台的 DIgSILENT PowerFactory 环境中，所有同步发电机的辅助控制系统均采用上述对应的 PSS/E 模型进行了等效替换。此举确保了以下几点：

- **动态特性一致性**：等效模型准确反映了原 BPA 模型的动态响应特性，保证仿真结果的可靠性。
- **计算效率优化**：通过标准化模型的使用，简化了系统配置，提高了仿真的计算效率。
- **兼容性与扩展性**：采用标准化的 PSS/E 模型，方便后续的系统扩展和模型升级。

### **5.2 模型功能简介**

- **ESST1A（PSS/E）**：用于模拟同步机的励磁系统，确保电压调节的稳定性。
- **ESAC2A（PSS/E）**：用于模拟同步机的励磁-无功调节功能，提高系统的电压稳定性。
- **PSS2A（PSS/E）**：用于模拟同步机的电力系统稳定器，增强系统阻尼，防止振荡。
- **IEEEG1（PSS/E）**：用于模拟同步机的调速器-原动机动态特性，保证机组的转速和功率调节性能。

### **5.3 实施效果**

通过上述模型等效替换，苏州电网 DIgSILENT 仿真平台在以下方面取得显著成效：

- **提高仿真精度**：确保动态仿真结果与实际电网运行特性高度一致。
- **增强系统稳定性分析能力**：通过准确的动态模型，能够更有效地评估电网在各种扰动下的稳定性。
- **支持复杂电网结构**：适应包含 HVDC、可再生能源等复杂组件的大规模电网仿真需求。

综上所述，机组动态控制系统模型的等效替换为苏州电网 DIgSILENT 仿真平台提供了坚实的基础，确保了仿真分析的高效性和准确性。

## **6. 平台功能与应用**

本平台对常规直流输电、柔性直流输电以及风电接入（包括双馈和全功率风电）的暂态仿真模型均提供了比 BPA 更为详细的模型设计。其模块化和精细化的建模方法不仅提升了仿真的准确性和效率，还满足了新型电力系统不断发展的仿真需求。以下是本平台在这些方面的具体优势和应用：

### **6.1 暂态仿真模型**

本平台涵盖了多种关键电力系统组件的暂态仿真模型，具体包括：

- **常规直流输电（HVDC-LCC）**：提供详细的LCC控制模型，能够准确模拟直流电流的控制和故障响应。
- **柔性直流输电（HVDC-VSC）**：采用先进的电压源换流器模型，支持动态调节和灵活控制以及保护策略，适应复杂的电力仿真需求。
- **风电接入模型**：
  - **双馈感应发电机（DFIG）模型**：
  - **全功率风力发电机模型**：提供全面的动态仿真能力，适用于大规模风电场的集成和仿真分析。

### **6.2 模块化与精细化设计**

本平台的仿真模型具备高度的模块化和精细化特性，具体体现在：

- **模块化设计**：各类设备和系统的模型模块化，便于灵活组合和扩展，适应不同的仿真需求。
- **精细化建模**：每个模型都包含详细的动态特性参数，能够精确反映实际设备的运行行为和响应特性。

### **6.3 仿真能力**

本平台不仅支持机电暂态仿真，还具备强大的电磁暂态仿真能力：

- **机电暂态仿真**：模拟发电机与电网的相互作用，评估系统在大规模扰动下的动态稳定性。
- **电磁暂态仿真**：分析系统在快速扰动（如开关操作、故障清除）下的电压和电流瞬态响应，确保电网的瞬态稳定性。

### **6.4 满足新型电力系统仿真需求**

随着电力系统向更高的智能化和可再生能源集成方向发展，本平台通过以下方式满足新型电力系统的仿真需求：

- **支持复杂电网结构**：能够模拟包含 HVDC、可再生能源（如风电、光伏）等多种复杂组件的大规模电网，适应现代电力系统的多样化需求。
- **灵活的仿真场景**：既适用于传统电力系统的机电分析，也能应对新型电力系统的电磁暂态分析，提供全面的仿真支持。
- **高效的计算性能**：通过优化的等值建模技术和稀疏矩阵计算，提升了大规模电网仿真的计算效率，缩短仿真时间。

### **6.5 应用实例**

以下是本平台在不同领域的具体应用实例：

- **电力系统规划与优化**：辅助电力系统规划人员进行电网扩展、设备选型和优化设计，评估不同方案的可行性和经济性。
- **稳定性分析与评估**：为电力系统的稳定性分析提供精确的仿真工具，评估系统在各种扰动下的动态响应，确保电网的稳态和暂态稳定。
- **新能源接入与调度**：支持大规模可再生能源接入的仿真分析，优化新能源与电网的匹配，提高电网的灵活性和适应性。

#### **6.5.1 双馈风机接入模型示例**

以双馈感应发电机（DFIG）为例，其接入模型详细包括以下组件：

- **机组模型**：
  - **DFIG_2.5MW**：模拟 2.5MW 双馈感应发电机的动态行为。
  
- **控制模型**：
  - **PQ 控制与同步（PQ Control & Synchronization）**：实现有功和无功功率的控制与同步。
  - **保护与同步（Protection & Synchronization）**：确保在故障情况下的保护机制与系统同步。
  - **最大功率跟踪（MPT）**：优化风机在不同风速下的功率输出。
  - **超频率功率减载（Over-Frequency Power Reduction）**：在频率过高时减少功率输出，保护系统稳定。
  - **转子电流控制（Irot-Control）**：调节转子电流以维持系统稳定。
  - **轴系模型（Shaft）**：模拟风机轴系的动态特性。
  - **速度控制器（Speed-Controller）**：控制风机转速以匹配电网需求。
  - **风机模型（Turbine）**：模拟风机的动力学行为。
  - **变桨控制（Pitch Control）**：调整风机叶片角度以优化功率输出。
  - **保护控制中包含 Crowbar 元器件的投切逻辑**：在特定故障情况下，通过 Crowbar 电路保护发电机转子，防止过电压。

通过上述详细的模型设计，双馈风机的动态行为得到了全面的模拟，确保了风电并网后的系统稳定性和可靠性。

### **6.6 总结**

通过模块化和精细化的模型设计，本平台在暂态仿真方面展现出显著优势。无论是常规直流输电、柔性直流输电，还是风电接入，本平台均能够提供高精度、高效率的仿真支持，满足现代电力系统多样化和复杂化的需求。未来，随着电力系统的不断演进，本平台将持续优化和扩展其仿真模型，进一步提升仿真能力和应用范围。

通过上述功能和应用，苏州电网 DIgSILENT 仿真平台不仅提升了电力系统仿真的精度和效率，还为电力规划、运行和优化提供了强有力的技术支持，助力实现电力系统的高效、安全和可持续发展。

### Platform-Figure
![image](https://github.com/howerdni/Suzhou-PowerGrid-Simulation/blob/main/39d7d82b5db988ae2fba12780f38d06.png)
![image](https://github.com/howerdni/Suzhou-PowerGrid-Simulation/blob/main/95c1726f32cd441d82d17a923328532.png)

## **7. 风机一般控制**
### **7.1 概述**
在现代风电机组中，电力电子变换器是核心部件之一，负责实现风力机与电网之间的能量变换与控制。常见的风电机组按照发电机与变换器的拓扑结构，可分为**双馈风电机组**（Doubly-Fed Induction Generator，DFIG）与**全功率风电机组**（Full Converter Wind Turbine）两大类：

- **双馈风机（DFIG）**  
  通过电力电子变换器对异步发电机的转子侧进行控制，定子直接接入电网。变换器容量通常约为机组额定容量的 30%～40%，但可以对机组的输出功率和无功功率进行有效调控。

- **全功率风机（Full Converter）**  
  发电机与电网之间通过全功率变换器完全隔离，变换器容量与机组额定容量相当。机侧变换器负责发电机的转矩和励磁控制，网侧变换器负责与电网的能量交换，以及维持直流母线电压的恒定。

无论是双馈风机还是全功率风机，它们的电力电子变换器控制通常采用“内环-外环”级联的矢量控制策略：
- **外环**：控制有功/无功功率、直流母线电压或机端电压等慢变量。  
- **内环**：控制变换器电流等快速变量，实现对变换器输出电压或电流的精确调节。  

后续小节将分别介绍双馈风机和全功率风机中电力电子变换器的典型控制策略与结构。
