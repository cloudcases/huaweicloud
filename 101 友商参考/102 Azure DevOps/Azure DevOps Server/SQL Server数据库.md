# 用于Azure DevOps Server的SQL Server数据库

- 项目
- 2022/06/25
- 4 个参与者

本文内容[Azure DevOps Server 和 SQL Server 数据库之间的交互](https://docs.microsoft.com/zh-cn/azure/devops/server/architecture/sql-server-databases?view=azure-devops-2020#interactions-between-azure-devops-server-and-sql-server-databases)[Azure DevOps Server与SQL Server Reporting Services之间的交互](https://docs.microsoft.com/zh-cn/azure/devops/server/architecture/sql-server-databases?view=azure-devops-2020#interactions-between-azure-devops-server-and-sql-server-reporting-services)[相关文章](https://docs.microsoft.com/zh-cn/azure/devops/server/architecture/sql-server-databases?view=azure-devops-2020#related-articles)

**Azure DevOps Server 2020 |Azure DevOps Server 2019 |TFS 2018**

如果了解SQL Server、SQL Server Reporting Services以及它们如何与Azure DevOps Server交互，则可以更轻松地管理Azure DevOps Server。

下图演示了与SQL Server Reporting Services集成的Azure DevOps Server部署的逻辑体系结构。

![Database relationships with SQL Server Reporting databases, Azure DevOps Server](https://docs.microsoft.com/zh-cn/azure/devops/server/media/databases-no-sharepoint-azure-devops-server.png?view=azure-devops-2020)

在数据库中存储所有数据的一个优点是，它简化了数据管理，因为无需备份单个客户端计算机。 如果熟悉备份SQL Server数据库，则备份和还原Azure DevOps Server数据库类似。 

## Azure DevOps Server 和 SQL Server 数据库之间的交互

下表描述了部署Azure DevOps Server中可能存在的数据库。

**Database**

**使用时间**

**说明**

------

**Tfs_Configuration**

始终

存储描述Azure DevOps Server部署的数据，包括其他数据库的名称和位置。

Tfs_*Collection*

始终

每个项目集合的一个数据库。 每个数据库存储项目的数据 (版本控制、生成和工作项) 在该集合中。

**Tfs_Warehouse**

配置了SQL Server报告

收集所有项目集合中的数据并将其存储在针对报告的优化表中。

**Tfs_Analysis**

配置了SQL Server报告

Analysis Services 数据库，用于将数据从仓库数据库组织到多维数据集结构中。

**ReportServer**

配置了SQL Server报告

存储SQL Server Reporting Services的报表和报表配置数据。

**ReportServer_TempDB**

配置了SQL Server报告

存储SQL Server Reporting Services的临时报告数据。

------

 提示

Azure DevOps Server要求排序规则设置区分大小写、区分重音且不二进制。 如果要将现有SQL Server安装与Azure DevOps Server一起使用，则必须验证排序规则设置是否满足这些要求。 否则，安装Azure DevOps Server会失败。 有关详细信息，请参阅[Azure DevOps Server SQL Server排序规则要求](https://docs.microsoft.com/zh-cn/azure/devops/server/install/sql-server/collation-requirements?view=azure-devops-2020)

SQL Server必须安装在服务器 (或服务器) 上，该服务器与托管逻辑Azure DevOps应用程序层的服务器 (或服务器) 之间配置了适当的信任级别。

## Azure DevOps Server与SQL Server Reporting Services之间的交互

SQL Server Reporting Services被视为Azure DevOps Server逻辑应用程序层的一部分。 但是，Reporting Services不必与该应用程序层的其他逻辑方面（如 SharePoint 产品）安装在同一物理服务器上。

在Azure DevOps Server中配置用户和组权限和组成员身份时，还必须为Reporting Services中的这些用户和组手动配置角色成员身份和权限。 有关详细信息，请参阅[SQL Server Reporting Services角色](https://docs.microsoft.com/zh-cn/azure/devops/server/install/sql-server/reporting-services-roles?view=azure-devops-2020)。

除了在Reporting Services中配置角色成员身份和权限外，还必须管理Azure DevOps Server用于与报表服务器通信的报表读取者帐户。 此帐户通常称为Reporting Services或 *TFSREPORTS* 的数据源帐户。 与Azure DevOps Server的服务帐户一样，报表读取者帐户必须是连接到Azure DevOps Server的每台计算机信任的工作组或域的成员。 有关详细信息，请参阅[安装Azure DevOps Server所需的帐户](https://docs.microsoft.com/zh-cn/azure/devops/server/account-requirements?view=azure-devops-2020)。

 提示

即使使用管理凭据登录，也可能无法访问报表管理器或 http:// *localhost*/Reports 站点，除非将这些站点添加为 Internet Explorer 中的受信任站点或以管理员身份启动 Internet Explorer。 若要以管理员身份启动 Internet Explorer，请选择 **“开始**”，输入 **Internet Explorer**，右键单击结果，然后选择“ **以管理员身份运行**”。

## 相关文章

- [Azure DevOps的报告路线图](https://docs.microsoft.com/zh-CN/azure/devops/report/powerbi/reporting-roadmap)
- [SQL Server Reporting Services角色](https://docs.microsoft.com/zh-cn/azure/devops/server/install/sql-server/reporting-services-roles?view=azure-devops-2020)
- [授予在Azure DevOps Server中查看或创建报表的权限](https://docs.microsoft.com/zh-CN/previous-versions/azure/devops/report/admin/grant-permissions-to-reports)



https://docs.microsoft.com/zh-cn/azure/devops/server/architecture/sql-server-databases?view=azure-devops-2020