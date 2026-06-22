
**强制访问控制（Mandatory Access Control, MAC）** 是一种由系统集中控制的访问控制策略。系统通过比较主体的安全许可级别（Clearance）和客体的安全标签（Classification/Label）来决定是否允许访问。
- 区别于 DAC，MAC 中普通用户（甚至资源所有者）**无法自主修改**安全属性或传递权限。
- 常用于军事、政府及高度注重信息安全的系统中。

## 安全标签体系

MAC 系统为所有实体分配严格的安全标签。通常包含两个维度：
1. **等级（Hierarchical Level）**：如绝密（Top Secret）、机密（Secret）、秘密（Confidential）、无密级（Unclassified）。
2. **类别（Category/Compartment）**：如特定项目名或部门名，用于横向隔离（Need-to-Know 原则）。

## 核心安全模型

### 1. Bell-LaPadula (BLP) 模型

BLP 模型是用于保障系统**保密性（Confidentiality）** 的经典 MAC 模型，防止信息流向低密级主体。
- **简单安全属性（Simple Security Property）**：**向下读（No Read Up）**。主体只能读取密级低于或等于自身许可级别的客体。
- **星属性（\*-Property）**：**向上写（No Write Down）**。主体只能向密级高于或等于自身许可级别的客体写入数据。(**防止高权限主体主动泄露高密信息**)

### 2. Biba 模型

Biba 模型与 BLP 相对应，专注于保障系统的**完整性（Integrity）**，防止低完整性数据污染高完整性数据。
- **简单完整性属性（Simple Integrity Property）**：**向上读（No Read Down）**。主体只能读取完整性级别高于或等于自身的主体产生的数据。
- **完整性星属性（Integrity \*-Property）**：**向下写（No Write Up）**。主体只能修改完整性级别低于或等于自身的客体。(**防止低可信度主体主动修改高可信度数据**)
