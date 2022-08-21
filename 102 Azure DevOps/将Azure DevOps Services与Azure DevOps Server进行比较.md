# Azure DevOps Services与Azure DevOps Server进行比较

- 2022/07/20
- 7 个参与者

**Azure DevOps Services | Azure DevOps Server 2020 | Azure DevOps Server 2019 | TFS 2018**

**云产品/服务**Azure DevOps Services提供可缩放、可靠且全球可用的托管服务。 它由 99.9% 的 SLA 提供支持，由我们的 24/7 运营团队监视，并在全球本地数据中心提供。

**本地产品/服务**Azure DevOps Server**基于SQL Server后端构建**。 客户通常需要将数据保留在其网络中时，通常会选择本地版本。 或者，当他们想要访问与Azure DevOps Server数据和工具集成的SQL Server Reporting Services 时。

尽管这两种产品/服务都提供相同的[基本服务](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/services?view=azure-devops)，但与Azure DevOps Server相比，Azure DevOps Services提供了以下附加优势：

- 简化的服务器管理。
- 立即访问最新和最伟大的功能
- 改进了与远程站点的连接。
- 从资本支出 (服务器等) 转换到运营支出 (订阅) 。

若要确定哪种产品（云或本地）满足需求，请考虑以下主要差异。

## Azure DevOps Services与Azure DevOps Server的基本差异

选择所需的平台，或者考虑从本地迁移到云时，请考虑以下方面：

- [范围和缩放数据](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/about-azure-devops-services-tfs?view=azure-devops#scope-scale-data)
- [身份验证](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/about-azure-devops-services-tfs?view=azure-devops#authentication)
- 用户和组
- [管理用户访问权限](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/about-azure-devops-services-tfs?view=azure-devops#manage-user-access)
- [安全性和数据保护](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/about-azure-devops-services-tfs?view=azure-devops#security-data)

**特定功能区域的差异**
虽然Azure DevOps Services是Azure DevOps Server的托管版本，但功能之间存在一些差异。 Azure DevOps Services不支持某些Azure DevOps Server功能。 例如，**Azure DevOps Services不支持与SQL Server Analysis Services集成以支持报告**。

以下两个领域在支持方面有所不同：

- [流程自定义](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/about-azure-devops-services-tfs?view=azure-devops#process-customization)
- [报告](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/about-azure-devops-services-tfs?view=azure-devops#reporting)

你是在Azure DevOps Server考虑移动吗？ 阅读 [迁移选项](https://docs.microsoft.com/zh-CN/azure/devops/migrate/migrate-from-tfs?view=azure-devops) 以了解选项。



## 范围和缩放数据

随着业务的增长，可能需要纵向扩展Azure DevOps实例。

### Azure DevOps Services使用组织和项目进行缩放

Azure DevOps Services与Azure DevOps Server略有不同。 目前只有两个选项用于范围和缩放数据：组织和项目。

 Azure DevOps Services中的组织 (获取自己的 URL， `https://dev.azure.com/fabrikamfiber` 例如) ，并且它们始终只有一个项目集合。 组织可以在集合中有多个项目。

我们建议在Azure DevOps Services中创建组织，无论在何处创建集合，Azure DevOps Server。 以下方案适用：

- 可以为每个组织购买Azure DevOps Services用户 - 付费用户只能访问付款所在的组织。 如果你有需要访问多个组织的用户，Visual Studio订阅可能是一个有吸引力的选项。 Visual Studio订阅者可以免费添加到任意数量的组织。 我们还考虑了其他方法，以便向分组到单个组织的许多组织提供访问权限。
- 当前必须一次管理组织。 如果有多个组织，此过程可能会很麻烦。

了解详细信息：[在 Azure DevOps 中规划组织结构](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/plan-your-azure-devops-org-structure?view=azure-devops)。

### Azure DevOps Server使用部署、项目集合和项目进行缩放

Azure DevOps Server提供以下三个选项来界定和缩放数据：部署、项目集合和项目。 最简单的情况是部署只是服务器。

但是，部署可能更为复杂，其中包括：

- 在单独的计算机上拆分SQL的双服务器部署
- 具有大量服务器的高可用性场

Project集合充当用于安全和管理以及物理数据库边界的容器。 它们还用于对相关项目进行分组。

最后，项目用于封装各个软件项目的资产，包括源代码、工作项等。

了解详细信息：[在 Azure DevOps 中规划组织结构](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/plan-your-azure-devops-org-structure?view=azure-devops)。



## 身份验证

使用 Azure DevOps Services，可通过公共 Internet (（例如 `https://contoso.visualstudio.com`) ）进行连接。 根据组织设置，可以使用 [Microsoft 帐户](https://www.microsoft.com/account)凭据或[Azure AD](https://docs.microsoft.com/zh-cn/azure/active-directory/active-directory-whatis)凭据进行身份验证。 还可以设置Azure AD，要求使用多重身份验证、IP 地址限制等功能。

建议将组织配置为使用Azure AD而不是 Microsoft 帐户。 此方法在许多方案中提供更好的体验，以及用于增强安全性的更多选项。

了解详细信息：[关于使用 Azure AD 访问Azure DevOps Services](https://docs.microsoft.com/zh-CN/azure/devops/organizations/accounts/access-with-azure-ad?view=azure-devops)。

使用 Azure DevOps Server，可以连接到 Intranet 服务器 (例如 `https://tfs.corp.contoso.com:8080/tfs`) 。 使用 Windows 身份验证和 Active Directory (AD) 域凭据进行身份验证。 此过程是透明的，你永远不会看到任何类型的登录体验。



## 管理用户和组

在Azure DevOps Services中，可以使用类似的机制[来提供对用户组的访问权限](https://docs.microsoft.com/zh-CN/azure/devops/organizations/accounts/manage-azure-active-directory-groups?view=azure-devops)。 可以将Azure AD组添加到Azure DevOps Services组。 如果使用 Microsoft 帐户而不是Azure AD，则必须一次[添加一个用户](https://docs.microsoft.com/zh-CN/azure/devops/organizations/accounts/add-organization-users?view=azure-devops)。

在Azure DevOps Server中，通过将 Active Directory (AD) 组添加到各种Azure DevOps组， (例如单个项目) 的参与者组，为用户提供对部署的访问权限。 AD 组成员身份保持同步。当用户在 AD 中添加和删除时，他们也会获得并失去对Azure DevOps Server的访问权限。



## 管理用户访问权限

在Azure DevOps Services和Azure DevOps Server中，可以通过将用户分配到[访问级别](https://docs.microsoft.com/zh-CN/azure/devops/organizations/security/access-levels?view=azure-devops)来管理对功能的访问权限。 所有用户都必须分配到单个访问级别。 在云和本地产品/服务中，你可以免费访问无限数量的利益干系人的工作项功能。 此外，无限数量的Visual Studio订阅者可以免费访问所有基本功能。 只需为需要访问权限的其他用户付费。

在Azure DevOps Services中，必须将[访问级别分配给](https://docs.microsoft.com/zh-CN/azure/devops/organizations/accounts/add-organization-users?view=azure-devops)组织中的每个用户。 Azure DevOps Services在登录时验证Visual Studio订阅者。 无需Visual Studio订阅即可免费向五个用户分配基本访问权限。

若要为更多用户提供基本访问权限或更高访问权限，请 [为组织设置计费](https://docs.microsoft.com/zh-CN/azure/devops/organizations/billing/set-up-billing-for-your-organization-vs?view=azure-devops) 并为 [更多用户付费](https://docs.microsoft.com/zh-CN/azure/devops/organizations/billing/buy-basic-access-add-users?view=azure-devops)。 否则，所有其他用户都获得利益干系人访问权限。

Azure AD组授予对用户组的访问权限。 首次登录时会自动分配访问级别。 对于配置为使用 Microsoft 帐户登录的组织，必须显式为每个用户分配访问级别。

在Azure DevOps Server中，所有使用都在荣誉系统上。 若要根据用户的许可证设置访问级别，请在管理页上指定其 [访问级别](https://docs.microsoft.com/zh-CN/azure/devops/organizations/security/change-access-levels?view=azure-devops) 。 例如，仅分配未经许可的用户利益干系人访问权限。

具有Azure DevOps Server客户端访问许可证 (CAL) 的用户可以具有基本访问权限。 Visual Studio订阅者可以具有基本或高级访问权限，具体取决于订阅。 Azure DevOps Server不会尝试验证这些许可证或强制实施合规性。



## 安全性和数据保护

许多实体在考虑迁移到云时想要详细了解数据保护。 我们致力于确保Azure DevOps Services项目保持安全和安全。 我们提供了技术功能和业务流程来履行这一承诺。 还可以采取措施来保护数据。 在我们的 [数据保护概述](https://docs.microsoft.com/zh-CN/azure/devops/organizations/security/data-protection?view=azure-devops)中了解详细信息。



## 流程自定义

可以根据支持的流程模型，以两种不同的方式自定义工作跟踪体验：

- Azure DevOps Services：使用支持 WYSIWYG 自定义的**继承**过程模型
- Azure DevOps Server：可以选择**继承**过程模型或**本地 XML** 进程模型，该模型支持通过导入或导出工作跟踪对象的 XML 定义文件进行自定义
- Azure DevOps Server 2018 及更早版本：你只能访问**本地 XML** 进程模型

虽然 **本地 XML** 进程模型选项非常强大，但它可能会导致各种问题。 主要问题是现有项目的流程不会自动更新。

例如，Azure DevOps Server 2013 引入了多个新功能，这些功能依赖于新的工作项类型和其他进程模板更改。 从 2012 升级到 2013 时，每个项目集合都会获取包含这些更改的每个“框中”进程模板的新版本。 但是，这些更改不会自动合并到现有项目中。 相反，在完成升级后，必须使用 [“配置功能](https://docs.microsoft.com/zh-cn/previous-versions/azure/devops/reference/upgrade/configure-features-after-upgrade) ”向导或更手动的过程将更改包含在每个项目中。

为了帮助你避免Azure DevOps Services这些问题，始终禁用自定义进程模板和**witadmin.exe**工具。 此方法使我们能够在每个Azure DevOps Services升级的情况下自动更新所有项目。 同时，产品团队正在努力使自定义过程成为可能，以便我们可以轻松、持续地支持。 我们最近引入了这些更改中的第一个，更多的更改正在进行中。

使用新的流程自定义功能，可以直接在 Web 用户界面 (UI) 进行更改。 如果要以编程方式自定义进程，可以通过 REST 终结点执行此操作。 这样自定义项目时，我们会使用Azure DevOps Services升级发布其基本进程的新版本时，它们会自动更新。

若要了解详细信息，请参阅 [自定义工作跟踪体验](https://docs.microsoft.com/zh-CN/azure/devops/reference/customize-work?view=azure-devops)。



## 分析和报告

Azure DevOps Services和Azure DevOps Server提供了许多工具，可让你深入了解软件项目的进度和质量。 包括以下工具：

- 云和本地平台中提供的[仪表板](https://docs.microsoft.com/zh-CN/azure/devops/report/dashboards/dashboards?view=azure-devops)和轻型[图表](https://docs.microsoft.com/zh-CN/azure/devops/report/dashboards/charts?view=azure-devops)。 这些工具易于设置和使用。
- [Analytics 服务和](https://docs.microsoft.com/zh-CN/azure/devops/report/powerbi/what-is-analytics?view=azure-devops)[Analytics 小组件](https://docs.microsoft.com/zh-CN/azure/devops/report/dashboards/analytics-widgets?view=azure-devops)。 Analytics 服务针对快速读取访问和基于服务器的聚合进行优化。
- [Microsoft Power BI集成](https://docs.microsoft.com/zh-CN/azure/devops/report/powerbi/overview?view=azure-devops)，它支持将 Analytics 数据引入Power BI报表，并提供简单性和功能的组合。
- [OData 支持](https://docs.microsoft.com/zh-CN/azure/devops/report/extend-analytics/quick-ref?view=azure-devops)，允许你直接从受支持的浏览器查询 Analytics 服务，然后根据需要使用返回的 JSON 数据。 可以生成跨多个项目或整个组织的查询。 若要了解有关 Analytics 服务的详细信息，请参阅 [我们的报告路线图](https://docs.microsoft.com/zh-CN/azure/devops/report/powerbi/reporting-roadmap?view=azure-devops)。



## Visual Studio Team Services 现已命名为 Azure DevOps Services。

VSTS 中的许多特色服务现在在 Azure DevOps Services 和 Azure DevOps Server 2019 中作为独立服务提供。 可以将服务单独获取或全部作为Azure DevOps Services。 如果你是Azure DevOps订阅者，则你有权访问所有服务。

| VSTS 功能名称 | Azure DevOps服务名称 | 说明                                                         |
| :------------ | :------------------- | :----------------------------------------------------------- |
| 生成 & 版本   | Azure Pipelines      | 适用于任何语言、平台和云的持续集成和持续交付 (CI/CD) 。      |
| 代码          | Azure Repos          | 项目的无限云托管专用 Git 和 Team Foundation 版本控制 (TFVC) 存储库。 |
| 工作          | Azure Boards         | 使用看板、积压工作、团队仪表板和自定义报告进行工作跟踪。     |
| 测试          | Azure Test Plans     | 一次性计划和探索性测试解决方案。                             |
| 包 (扩展)     | Azure Artifacts      | Maven、npm、Python、通用包和NuGet来自公共和专用源的包源。    |

Azure DevOps Services和Azure DevOps Server 2019 都使用新的导航用户界面，垂直边栏将转到主要服务区域：**Boards**、**Repos**、**Pipelines**等。 若要了解详细信息，请参阅 [Azure DevOps中的 Web 门户导航](https://docs.microsoft.com/zh-CN/azure/devops/project/navigation/?view=azure-devops)。

 备注

可以从用户界面禁用选择服务。 有关详细信息，请参阅 [打开或关闭服务](https://docs.microsoft.com/zh-CN/azure/devops/organizations/settings/set-services?view=azure-devops)。

仍`visualstudio.com`可用于访问Azure DevOps Services。 我们已移动到新域名作为新 `dev.azure.com` 组织的主要 URL。 该 URL 为 `https://dev.azure.com/{your organization}/{your project}`. 如果要将 URL 更改为主要 `dev.azure.com` URL，则组织管理员可以从组织设置页执行此操作。

## 相关文章

- [基本服务](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/services?view=azure-devops)
- [客户端-服务器工具](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/tools?view=azure-devops)
- [软件开发角色](https://docs.microsoft.com/zh-CN/azure/devops/user-guide/roles?view=azure-devops)
- [Azure DevOps Services定价](https://azure.microsoft.com/pricing/details/devops/azure-devops-services/)
- [Azure DevOps Server定价](https://azure.microsoft.com/pricing/details/devops/server/)



https://docs.microsoft.com/zh-CN/azure/devops/user-guide/about-azure-devops-services-tfs?view=azure-devops