# Azure DevOps Server体系结构概述

- 2022/06/25
- 5 个参与者

**Azure DevOps Server 2020 |Azure DevOps Server 2019 |TFS 2018**

若要最好地规划和管理部署，应首先了解Azure DevOps Server的基础体系结构。 了解体系结构可帮助你维护部署的总体运行状况，并有助于确保你的开发团队需要的服务器和服务的总体可用性。

可以通过多种方式部署Azure DevOps Server：在**一台服务器上;在多个服务器上**;或者在一个域或工作组中或跨域部署。 或者，可以选择使用 Azure DevOps Services，其中部署的所有服务器元素由 Microsoft 托管。

 了解体系结构可帮助你决定最能满足你的业务需求的拓扑。 无论选择哪种拓扑，如果了解基础体系结构Azure DevOps Server，则可以更好地管理物理和逻辑要求。 本文简单概述了各种体系结构，并提供了有关示例部署的详细信息的链接。 它还提供有关本地部署的服务、数据库、配置信息、网络端口和协议的技术信息。

若要了解Azure DevOps Server的体系结构及其如何影响部署，应考虑以下事项：

- Azure DevOps的逻辑应用程序、数据和客户端层，以及是要对应用程序和数据层使用一个或多个服务器，还是希望使用 Azure DevOps Services 在云中托管的应用程序和数据层
- 托管这些层的物理或虚拟服务器的位置
- Team Foundation Build 以及环境中运行的生成计算机的数量和位置，包括可能需要多少个支持开发做法，或者是否要使用Azure Pipelines云服务来生成和部署软件应用程序
- Azure DevOps代理服务器的潜在需求

此外，还必须考虑这些实体之间的交互。 例如，如果选择使用托管Azure DevOps Server服务，则必须确保客户端可以在端口 443 上访问该服务。 如果选择在本地部署Azure DevOps Server，则必须知道Azure DevOps Server使用的 Web 服务、数据库和对象模型。 此外，还必须知道默认情况下Azure DevOps Server使用哪些网络端口和协议，以及可以自定义的网络端口。 最后，必须了解必须在Azure DevOps Server中设置哪些权限，以及部署所依赖的组件和程序。

除了自己的服务，Azure DevOps Server还依赖于其他服务才能正常运行。 有关这些服务的详细信息，请参阅Azure DevOps Server数据仓库[Azure DevOps Server概念](https://docs.microsoft.com/zh-cn/azure/devops/server/architecture/key-concepts?view=azure-devops-2020)和[组件](https://docs.microsoft.com/zh-CN/previous-versions/azure/devops/report/sql-reports/components-data-warehouse)。 有关安装要求和依赖项的详细信息，请参阅[Azure DevOps Server安装指南](https://docs.microsoft.com/zh-CN/azure/devops/server/install/get-started)。

 重要

除非按照Microsoft 支持部门指示，否则不应手动修改任何Azure DevOps Server数据库，否则应遵循手动备份数据库的过程。 任何其他修改都可能使你的服务协议失效。



## Azure DevOps Services

![Azure DevOps Services](https://docs.microsoft.com/zh-cn/azure/devops/server/media/architecture/vsts_architecture_intro.png?view=azure-devops-2020)

Microsoft 提供了使用Azure DevOps Services的选项，它可以为你托管Azure DevOps Server的所有服务器端方面。 你的源代码、工作项、生成配置和团队功能都可以在云中托管。 从体系结构的角度来看，这大大简化了Azure DevOps Server的使用，因为需要考虑的体系结构的唯一方面是客户端组件及其 Internet 访问。

使用 Azure DevOps Services 时，请使用 Web 浏览器通过 Microsoft 帐户连接到服务。 可以创建项目，将成员添加到团队，并像在本地安装Azure DevOps Server一样工作，而无需管理服务器开销。 Azure DevOps Services托管云中的应用程序层、数据层和生成服务器。

若要详细了解云服务与本地部署，请查看[Azure DevOps Services与Azure DevOps Server](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/about-azure-devops-services-tfs)。



## 对象模型

使用托管体系结构或本地部署的体系结构，可以通过编写基于其服务器或客户端对象模型的应用程序来扩展Azure DevOps的特性和功能。 在所有部署类型中，你可以编写扩展客户端功能的应用程序。 但是，如果要扩展服务器功能，应用程序必须在应用程序层服务器上运行。 若要扩展客户端功能，必须在与团队资源管理器相同的计算机上运行应用程序。

![The Azure DevOps Server object model](https://docs.microsoft.com/zh-cn/azure/devops/server/media/architecture/object_model.png?view=azure-devops-2020)



## 本地部署的 Web 服务和数据库

Azure DevOps Server包括一组 Web 服务和数据库，这些服务以及数据库在托管逻辑应用程序、数据和客户端层的服务器上单独安装和配置Azure DevOps。 某些功能（如任务板和积压工作团队功能）完全基于 Web，只能通过 Web 门户（基于客户端 Web 的服务）进行访问。 其他功能（如版本控制功能）可通过 Web 门户或客户端应用程序访问。 下图提供了用于Azure DevOps Server本地部署的 Web 服务、应用程序和数据库的概要视图。

![Azure DevOps Server main service tiers](https://docs.microsoft.com/zh-cn/azure/devops/server/media/architecture/tfs_services_tiers.png?view=azure-devops-2020)

![Optional Azure DevOps Server services](https://docs.microsoft.com/zh-cn/azure/devops/server/media/architecture/optional_tfs_services.png?view=azure-devops-2020)

![Azure DevOps Server clients](https://docs.microsoft.com/zh-cn/azure/devops/server/media/architecture/tfs_client_model.png?view=azure-devops-2020)



## 集合级别服务

集合级服务为项目集合级别的操作提供功能。 可以使用其中一些服务创建扩展Azure DevOps Server的应用程序。 有关为Azure DevOps Server创建应用程序的详细信息，请参阅[开发扩展](https://docs.microsoft.com/zh-CN/azure/devops/extend/index)。

 备注

一些服务会在多个级别中显示。 例如，注册表服务在集合级别和服务器级别工作，将会出现在这两个列表中。

**框架服务：**

- 注册表服务
- 注册服务 (与早期版本的Azure DevOps Server) 兼容
- 属性服务
- 事件服务
- 安全性服务
- 位置服务
- 标识管理服务
- 版本控制 Web 服务
- 工作项跟踪 Web 服务
- Team Foundation Build Web 服务
- Lab Management Web 服务
- VMM Administration Web 服务
- 测试代理控制器 Web 服务



## 服务器级服务

服务器级服务 (也称为应用程序级服务) 为Azure DevOps Server作为软件应用程序的操作提供功能。 可以使用其中一些服务创建扩展Azure DevOps Server的应用程序。

**框架服务：**

- 注册表服务
- 事件服务
- Project收集服务
- 属性服务
- 安全性服务
- 位置服务
- 标识管理服务
- 管理服务
- 集合管理服务
- 目录服务



## 数据层

数据层包含数据、存储过程及其他关联的逻辑。 使用Azure DevOps Services时，将使用 SQL Server Azure 托管数据层。 在Azure DevOps Server的本地部署中，逻辑数据层由SQL Server中的以下操作存储组成。 这些存储区可能位于一台物理服务器上或分布于多台服务器上。 可以使用其中一些操作存储创建扩展Azure DevOps Server的应用程序。

- 配置数据库 (TFS_Configuration)
- 应用程序仓库 (TFS_Warehouse)
- Analysis Services 数据库 (TFS_Analysis)
- 项目集合的数据库 (TFS_CollectionName)

下表提供了Azure DevOps Server在本地部署中使用的数据库列表。 除非另有说明，否则您可以将此列表中的所有数据库从安装它们的原始服务器和实例中移到和还原到其他服务器或实例中。

> | 数据库名称            | 说明                                                         | 服务器                                                       |
> | :-------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
> | **TFS_Configuration** | 此数据库存储Azure DevOps Server的资源目录和配置信息。 此数据库包含Azure DevOps Server的操作存储。 | 安装并配置Azure DevOps Server时使用的SQL Server实例。        |
> | **TFS_Warehouse**     | 此数据库存储报表的数据。                                     | 安装并配置Azure DevOps Server时使用的SQL Server实例。        |
> | **TFS_Analysis**      | 此多维数据库存储项目集合中的聚合数据。                       | 安装并配置SQL Server Analysis Services时使用的SQL Server实例。 |
> | **项目集合的数据库**  | 每个项目集合的一个数据库，其中包含该集合中所有项目的数据。   | 与Azure DevOps Server兼容的SQL Server实例。                  |



## 客户端层

客户端层通过服务对象模型与应用层通信，并使用为该层列出的相同 Web 服务。 无论是在本地部署Azure DevOps Server，还是使用Azure DevOps Services，都是如此。 除了该模型之外，客户端层还包括 Visual Studio 行业合作伙伴 (VSIP) 组件、Microsoft Office 集成、命令行接口及签入策略框架。



## Configuration

托管服务取决于本地部署的客户端服务以及与云中托管的应用层和数据层的 Internet 连接。 Azure DevOps Server的本地部署取决于SQL Server、Internet Information Services (IIS) 和Windows操作系统。 根据所选拓扑，Azure DevOps Server也可能依赖于SQL Server Reporting Services或SharePoint产品。 因此，Azure DevOps Server的配置信息可以存储在以下任何位置：

- IIS 数据存储区。
- Azure DevOps Server的配置文件。
- Reporting Services 的数据源（例如，TFSREPORTS 数据）。
- Azure DevOps Server的配置数据库。 Azure DevOps Server注册表是配置数据库的一部分。
- Windows 注册表。

有关不同本地部署拓扑以及这些资源存储位置的示例，请参阅 [简单拓扑示例](https://docs.microsoft.com/zh-cn/azure/devops/server/architecture/examples-simple-topo?view=azure-devops-2020)、 [中等拓扑示例](https://docs.microsoft.com/zh-cn/azure/devops/server/architecture/examples-moderate-topo?view=azure-devops-2020)和 [复杂拓扑示例](https://docs.microsoft.com/zh-cn/azure/devops/server/architecture/examples-complex-topo?view=azure-devops-2020)。 在维护Azure DevOps Server的本地部署时，必须考虑这些配置源。 若要以任意方式更改配置，可能需要修改存储在多个位置中的信息。 您可能还需要更改数据层和客户端层的配置信息。 Azure DevOps Server包括管理控制台和多个命令行实用工具，可帮助你进行这些更改。 有关详细信息，请参阅 [管理任务快速参考](https://docs.microsoft.com/zh-cn/azure/devops/server/admin/admin-quick-ref?view=azure-devops-2020)。



## Active Directory 和组标识的同步

在 Active Directory 域中运行Azure DevOps的本地部署中，发生以下任何事件时，将同步组和标识信息：

- 应用程序层服务器启动。
- Active Directory 组将添加到Azure DevOps组。

超过计划作业中指定的时间段。 默认值为 1 小时，Azure DevOps Server所有组每 24 小时更新一次。

标识管理服务 (IMS) 与 Active Directory 同步，并且更改的标识会从服务器传播到客户端。 默认情况下，所有组在 24 小时内更新，但是您可以对此进行自定义，以便更好地满足您的部署需要。 有关详细信息，请参阅[Azure DevOps Server的信任和林注意事项](https://docs.microsoft.com/zh-CN/azure/architecture/reference-architectures/identity/adds-forest)。 有关不使用 Active Directory 的本地部署，请参阅[工作组中的管理Azure DevOps Server](https://docs.microsoft.com/zh-CN/previous-versions/ms252507(v=vs.110))。



## 组和权限

在本地部署中，Azure DevOps Server具有可在项目、集合或服务器级别设置的默认组和权限集。 你可以创建自定义组，并在组和各个级别自定义权限。 但是，添加到Azure DevOps Server的用户或组不会自动添加到本地部署Azure DevOps Server可以依赖的两个组件：SharePoint产品和Reporting Services。 如果你的部署使用这些程序，则必须向其添加用户和组，并授予相应的权限，让这些用户或组在Azure DevOps Server中的所有操作中正常运行。 有关详细信息，请参阅[Azure DevOps Server中管理用户或组](https://docs.microsoft.com/zh-CN/azure/devops/organizations/security/set-project-collection-level-permissions)。

对于托管部署，通过 Microsoft 帐户和团队成员资格的组合来控制访问权限。 有关详细信息，请参阅[Azure DevOps Services概述](https://docs.microsoft.com/zh-CN/azure/devops/overview)。



## 网络端口和协议

默认情况下，Azure DevOps Server的本地部署配置为使用特定的网络端口和协议。 下图显示了简单部署中Azure DevOps Server的网络流量。

![Simple on-premises installation](https://docs.microsoft.com/zh-cn/azure/devops/server/media/architecture/ports-protocols.png?view=azure-devops-2020)

同样，Azure DevOps Server的托管服务配置为使用特定的网络端口和协议。 下图演示托管部署中的网络通信。

![Hosted Azure DevOps Server](https://docs.microsoft.com/zh-cn/azure/devops/server/media/architecture/hosted-tfs.png?view=azure-devops-2020)

 

下图显示了更复杂的部署中的网络流量，其中包括用于Visual Studio实验室管理的组件。 (请注意，TFS 2017 及更高版本已弃用实验室管理。)

![Application tier](https://docs.microsoft.com/zh-cn/azure/devops/server/media/architecture/ic630774.png?view=azure-devops-2020)

![Virtual environments](https://docs.microsoft.com/zh-cn/azure/devops/server/media/architecture/ic406389.png?view=azure-devops-2020)

![virtual machines](https://docs.microsoft.com/zh-cn/azure/devops/server/media/architecture/ic738717.png?view=azure-devops-2020)

虚拟机使用端口 80 在与实验室管理代理下载有关的方面与任何测试控制器进行通信。 如果发生任何通信问题，请检查此端口是否启用。



## 默认网络设置

默认情况下，Azure DevOps部署中的计算机之间的通信使用下表所示的协议和端口。 如果端口号后跟星号 (*)，则可自定义该端口。

| 层和服务                                                     | 协议                                                         | 端口                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 应用层 - Web 服务                                            | HTTP/HTTPS                                                   | 8080/443*                                                    |
| 应用程序层 - SharePoint产品管理                              | HTTP                                                         | 如果SharePoint产品随 Azure DevOps Server 一起安装，则为 17012*;否则，随机生成 |
| 应用程序层 - SharePoint产品和Reporting Services              | HTTP Windows Management Instrumentation (WMI) 服务（安装过程中需要使用该服务指定和验证 Reporting Services 的 URL） | 80* 动态端口                                                 |
| 数据层                                                       | MS-SQL TCP                                                   | 1433*                                                        |
| 数据层 (SQL Server Analysis Services)                        | MS-AS                                                        | 默认值（2382 或 2383）* 默认端口因已安装的 SQL Server 版本和实例类型而异。 使用 SQL Server 配置管理器确定你的部署所使用的端口。 |
| Azure DevOps代理服务器 - 客户端到代理                        | HTTP                                                         | 8081*                                                        |
| Azure DevOps代理服务器 - 代理到应用程序层                    | HTTP/HTTPS                                                   | 8080/443*                                                    |
| 客户端层 - Reporting Services                                | HTTP                                                         | 80*                                                          |
| 客户端层 - Web 服务                                          | HTTP/HTTPS                                                   | 8080/443*                                                    |
| 生成控制器到应用程序层 HTTP/HTTPS                            | 8080/443                                                     |                                                              |
| 生成代理到应用层                                             | HTTP/HTTPS                                                   | 8080/443                                                     |
| Release Management 服务器                                    | HTTP 或 HTTPS                                                | 1000*                                                        |
| Release Management 客户端                                    | HTTP 或 HTTPS                                                | 1000*                                                        |
| Release Management 代理                                      | HTTP 或 HTTPS                                                | 1000*                                                        |
| 测试控制器到应用层                                           | HTTP/HTTPS                                                   | 8080/443*                                                    |
| 应用层到测试控制器                                           | .NET 远程处理                                                | 6901*                                                        |
| 应用层到域名系统 (DNS)                                       | DNS 动态更新                                                 | 53                                                           |
| 应用层 – Virtual Machine Manager                             | HTTP                                                         | 8100                                                         |
| 测试控制器到测试代理                                         | .NET 远程处理                                                | 6910*                                                        |
| 测试代理到测试控制器                                         | .NET 远程处理                                                | 6901*                                                        |
| 生成控制器到生成代理                                         | HTTP 上的 SOAP                                               | 9191                                                         |
| 隔离环境中的实验室代理到实验室代理                           | TCP 套接字                                                   | 9050                                                         |
| 生成代理到生成控制器                                         | HTTP 上的 SOAP                                               | 9191                                                         |
| Virtual Machine Manager 管理员控制台 – Virtual Machine Manager | HTTP                                                         | 8100                                                         |
| Virtual Machine Manager– Virtual Machine Manager 主机        | Windows 远程管理 (WinRM)，用于执行操作 后台智能传输服务 (BITS)，用于传输数据 | 80，用于执行操作 443，用于传输数据                           |
| Virtual Machine Manager– Virtual Machine Manager 库服务器    | WinRM，用于执行操作 BITS，用于传输数据                       | 80，用于执行操作 443，用于传输数据                           |
| 应用层 – Virtual Machine Manager 主机                        | 分布式组件对象模型/Windows Management Interface (DCOM/WMI) 通信，用于传输数据 | 135 在 49152 至 65535 范围内动态指派                         |
| 客户端层 – Virtual Machine Manager 主机                      | 与虚拟机之间的基于主机的连接。                               | 2179 用于执行基于主机的连接                                  |
| 托管的服务                                                   | HTTPS                                                        | 443                                                          |



## 可自定义的网络设置

如上表所示，可以通过修改Azure DevOps Server以使用自定义端口来更改本地部署中的应用程序、数据和客户端层之间的通信。 下表介绍一些从 HTTP 到 HTTPS 的端口更改示例。

 备注

若要将Azure DevOps Server配置为使用 HTTPS 和安全套接字层，不仅必须为 HTTPS 网络流量启用端口，而且还要执行许多其他任务。 有关详细信息，请参阅[为Azure DevOps Server设置具有安全套接字层 (SSL) 的 HTTPS](https://docs.microsoft.com/zh-cn/azure/devops/server/admin/setup-secure-sockets-layer?view=azure-devops-2020)。

| 服务                     | 协议         | 端口         |
| :----------------------- | :----------- | :----------- |
| 使用 SSL 的 Web 服务     | HTTPS        | 由管理员配置 |
| SharePoint管理中心 HTTPS | 由管理员配置 |              |
| SharePoint 产品          | HTTPS        | 443          |
| Reporting Services       | HTTPS        | 443          |
| 客户端 Web 服务          | HTTPS        | 由管理员配置 |
| 发布管理                 | HTTPS        | 由管理员配置 |

## 相关文章

- [关键概念和术语](https://docs.microsoft.com/zh-cn/azure/devops/server/architecture/key-concepts?view=azure-devops-2020)



https://docs.microsoft.com/zh-cn/azure/devops/server/architecture/architecture?view=azure-devops-2020