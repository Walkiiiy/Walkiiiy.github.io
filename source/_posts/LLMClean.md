---
title: LLMClean
date: 2025-06-16 15:01:26
tags:
---
**文献阅读：结合LLM构建具有上下文感知的数据错误检测**
**from"LLMClean: Context-Aware Tabular Data Cleaning via LLM-Generated OFDs"by Fabian Biester,Mohamed Abdelaal,Daniel Del Gaudio**
## 总览
- 通过本体功能依赖（OFDs）表达上下文知识，来提升数据清洗质量。
- 传统方法的上下文模型构建依赖人工，需领域专家提供语义信息，手工标注费时且容易出错，模型难以快速适应新数据和环境变动，不适合实时/大规模场景。
- 提出LLMClean使用LLM从原始数据中自动生成上下文模型:
  * 无需额外元数据；
  * 自动提取语义信息；
  * 自动推导OFD 规则；
  * 应用于数据验证与错误检测；
- 在IoT、医疗、工业等3类数据集进行实验，结果呈现自动化生成的上下文模型与人工专家构建的模型清洗能力相当，且效率和可扩展性更优。
## OFD与FD
FD是简单函数依赖，A->B，A相同的数据B一定相同。OFD是在合理范围内的依赖关系，由A 所属的概念 → 推理出 B 的合理值。这里的**理解并推理约束的范围**给了LLM发挥的空间。
举例对比：
- 医院与邮政编码
```
ZipCode → City
```
含义：同一个邮政编码的城市应该一致。
OFD 表达：
```
ZipCode →∼ City（通过本体推理）
```
含义：允许同义变体、“洛杉矶”与“Los Angeles”在本体中是等价概念。
**OFD 更宽容、也更语义合理**，能处理“格式不同但语义一致”的情况。
- 传感器与能力参数
无法用 FD 表达：
```
ds18b20 → MaxValue = 80
```
因为 FD 不能处理“值范围”或“元数据”。
OFD 表达：
```
Sensor_ID → Capability.MaxValue
```
含义：某传感器型号决定其最大支持的温度/压力等能力值。
**OFD 可以处理“属性关系”，FD 只能处理“值关系”。**
- 时间顺序约束
A → B 的数据必须满足时间先后关系：t₁ < t₂
#### OFD 表达：
```
Device_A → Device_B ⇒ Timestamp_A < Timestamp_B
```

## 整体架构
![LLMClean_structure](/images/LLMClean_structure.png)
- 1.输入脏数据集，包含错误、缺失、不一致项的表格数据。
- 2.**LLM-Based Context Model Generation**
   * 使用大语言模型（如 GPT 或 LLaMA）对数据列名进行解析和建模；
   * 生成具有语义结构的上下文模型（Context Model）；
   * 论文提供了一个 IoT 元模型，用于支持 RDF 图建模与OFD规则自动生成
    ![LLMClean_ModalGeneration](/images/LLMClean_ModalGeneration.png)
- 3.**OFDs Extraction**
   * 从上下文模型中提取本体功能依赖（OFDs）；
   * 这些依赖规则用于数据验证；
   * LLMClean支持七种OFD类型，用于建模各种语义上下文。
- 4.**OFDs-Based Error Detection**
   * 通过执行OFD检查，识别违反依赖规则的记录（即可能含错误的数据行）；
- 5.**Error Correction**
   * 将错误位置信息**传入外部修复工具**如 Baran、HoloClean，自动生成修复建议。
- 6.输出修复后的高质量数据集，可用于下游 ML 应用。

**需要着重区别OFDextraction和Model generation:Model generation定义一个列是什么，属于哪些类，该列的类型有哪些包含关系和值类型。OFDextraction则根据model分析各列之间的约束关系。**
比如结合图2，研究的IOT数据建模有以下类型：
  * `ssn:System`：系统整体（如家庭监控平台）
  * `ssn:Device`：包括感知/控制设备
  * `ssn:Sensor`：每个传感器
  * `iot-lite:Location`：物理部署点
  * `iot-lite:Metadata`：能力参数
  * `iot-context:Measurement`：采集值 + 时间
  * `iot-context:MonitoringComponent`：设备监控单元

Model实际上就是形成三元组：
```
<Sensor_1> <hasCapability> <MaxValue>
<Device_1> <hasLocation> <Room1>
<Sensor_1> <hasDeployment> <Device_1>
```
而OFDextraction就是根据以下约束类型检查数据是否存在约束关系：
| OFD类型                        | 描述             | 示例说明                   |
| ---------------------------- | -------------- | ---------------------- |
| **Denial Dependency (DD)**   | 否定某些组合不可共存     | ¬(A=SensorX ∧ B=RoomY) |
| **Matching Dependency (MD)** | 值相似 ⇒ 另一列值也应相似 | SensorName ∼→ Location |
| **Device-Link Dependency**   | 传感器与设备绑定       | SensorA → DeviceB      |
| **Temporal Dependency**      | 时间顺序依赖（A先于B）   | A → B ⇒ tA < tB        |
| **Location Dependency**      | 设备与位置绑定        | DeviceA → Room1        |
| **Monitoring Dependency**    | 设备被监控器实时跟踪     | A → CPU\_Load/NetSpeed |
| **Capability Dependency**    | 传感器具备特定能力参数    | SensorX → Min/MaxValue |

比如，在后续的纠错环节中，若两个记录中 SensingDevice 相同，但 Device 不同 ⇒ 违反了 Device-Link Dependency；若某传感器采集的温度值超出其最大值 ⇒ 违反 Capability Dependency。
## Automated Context Modeling 上下文模型自动构建
利用大语言模型，从表格数据中自动构建上下文模型Context Model
![LLMClean_ModalGeneration_1](/images/LLMClean_ModalGeneration_1.png)
#### IOT数据集
- 步骤 1：Column Mapping，列到本体概念的映射
使用 LLM 识别列名如 `device_id`, `location`, `timestamp`属于哪个概念：`ssn:Sensor`, `iot-lite:Location`, `iot-context:Measurement`, 等。如某些概念缺失，使用synthetic column generation,人工合成列补全。
- 步骤 2：Dataset Transformation，数据集结构重构
包括以下 3 个子步骤：
   - 1.Sensor Splitting，传感器拆分：有些行记录多个传感器的读数（例如温度和 CO₂）,拆成单条记录，每条记录一个测量项：
```
(T1, C1, L1, t1) → (Temp, T1, L1, t1), (CO2, C1, L1, t1)
```
   - 2.Column Generation,列补全若缺少“SensorID”, “MinValue”, “MaxValue”等概念列,LLM会生成列名,用 default 或 synthetic 值填充。
   - 3.Column Renaming,标准化列名
```
"Sensor\_name" → "sensor"
"place" → "location"
```
- 步骤 3：Sensor Capability 信息提取
自动从网络（如 Wikipedia、Wikidata）或 LLM 查询各传感器的能力范围，如 Min/Max 温度。可人工补充增强（提供 GUI 接口）
- 步骤 4：Context Model 构建，输出 RDF 图
将清洗后的结构转化为 RDF triples（本体图），每行数据生成一组实体关系三元组，用于后续 OFD 推理和错误检测。
#### 非IOT数据集
对于非IoT数据集，由于缺少设备/传感器上下文，因此采用精简流程：
- 步骤 1：列对提取Column Pair Generation，生成所有列对，如 (ZipCode, City)、(HospitalName, Owner)，使用 LLM 判断列对间是否存在语义依赖关系,如匹配或否定依赖。
- 步骤 2：概念识别与关系建模Concept Mapping，判定某列是否是另一列的属性（如ZipCode属于City）,或者是否是两个独立概念。
- 步骤 3：构建 RDF 图，每一列 → 映射为 RDF 图中的一个 concept 节点,依赖关系 → 映射为边（predicate），形成语义图,建立概念层次结构（例如 Owner → Hospital → Address）。
## 结果：
![LLMClean_res](/images/LLMClean_res.png)
比对不同修补模型在hospital和IOT数据集上的效果
Baran、ML-Impute、Mean-Mode为三种不同修补工具
Numerical是连续型数值纠错，Categorical离散值或者标签类型纠错。