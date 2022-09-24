# Terraform Provider 开发指南

2018-08-02 9594

**简介：** 本文主要向大家展示如何为[阿里云 Terraform Provider](https://www.terraform.io/docs/providers/alicloud/index.html) 贡献自己的力量，帮助开发者和志同道合的朋友尽快加入到开源生态的建设中来。

本文主要向大家展示如何为[阿里云 Terraform Provider](https://www.terraform.io/docs/providers/alicloud/index.html) 贡献自己的力量，帮助开发者和志同道合的朋友尽快加入到开源生态的建设中来。

本文面向所有的对Terraform熟悉和感兴趣的朋友，如果您还不了解Terraform，[快快戳这里](https://www.terraform.io/docs/index.html)。

本文会从环境搭建和开发规范两个方面向大家展示如何开放 terraform provider。

## 环境搭建

### 安装 Golang

本地安装go >1.10.0 详见：https://golang.org/dl/

### 安装 Terraform

本地安装 Terraform > 0.11， 详见： https://www.terraform.io/intro/getting-started/install.html

### 安装 Terraform Provider

1. fork或者直接 Clone repo https://github.com/terraform-providers/terraform-provider-alicloud 到：$GOPATH/src/github.com/terraform-providers/terraform-provider-alicloud

   ```
   mkdir -p $GOPATH/src/github.com/terraform-providers
   cd $GOPATH/src/github.com/terraform-providers
   git clone git@github.com:terraform-providers/terraform-provider-alicloud
   ```

2. 编译并 build Provider：

   ```
   cd $GOPATH/src/github.com/terraform-providers/terraform-provider-alicloud
   # make build 会同时build出mac，Linux和windows的provider，并将生成的provider存放在当前目录的`bin`目录下
   make build
   # 如果你只想build单一系统的provider，只需要在执行的时候指定操作系统名称即可
   make mac / make linux / make windows
   # 如果你想开发或者测试，以下命令会将build好的provider安装到 Terrafrom 所在的安装目录，无需自行安装
   Mac： make dev
   Linux: make devlinux
   Windows: make devwin
   ```

3. 设置环境变量，省去每次测试时输入AK的要求

   ```
   # set the creds
   export ALICLOUD_ACCESS_KEY="***"
   export ALICLOUD_SECRET_KEY="***"
   ```

## Resource 开发

Resource 开发是根据阿里云的OpenAPI在terraform provider中实现对阿里云产品和资源的插件。在terraform中，一个阿里云的云资源，如一台ECS instance，一块磁盘，一个VPC，一个SLB实例等，都可以对应一个resource，除此之外，资源与资源之间的关联关系，如磁盘的挂载，EIP的绑定，ECS 实例添加到SLB中等，也可以定义为一个单独的资源，这样带来的好处是，这些关联关系可以在编排模板中被大量的复用，模板结构清晰的同时，降低用户编写模板的复杂度。

### 规划和设计

每个Resource的实现不仅仅是根据[阿里云帮助文档](https://help.aliyun.com/)对OpenAPI的简单调用，还要对产品的设计，功能以及使用有较深的理解，通常遵循如下设计原则：

1. #### 自管理

   **每个resource只管理自身的功能，与其他resource引用或者关联交由关系性资源来完成**。

   如ECS instance只管理实例自身的功能，对于挂哪个数据盘，分配哪个EIP交由attachment资源来完成。

2. #### 松耦合

   **对于资源与资源之间的关联关系，需要单独定义一个逻辑resource，来完成资源与资源之间的关联**。

   如磁盘挂载，eip的挂载，ess与ecs的关联等。

3. #### 参数简洁

   **参数要尽可能简洁，不要有冗余的参数**。

   以RDS instance为例，对于参数zoneId，VpcId以及VSwitchId而言，只需要定义zoneId和VpcId就是冗余参数，因为在调用具体API之前，可以通过vswitchId来获取这两个参数，但某些特殊的场景除外，比如RDS支持Multiple zone，所以zonId也需要保留。

   如果可以设置Default，最后显示设置。

4. #### 语义清晰

   **每个参数字段要具有清晰的语义，帮助用户更好的理解**。

   对于一些公共的，统一的参数，如可用区，slb ID，最好跟其他资源保持一致：`availability_zone`, `load_balancer_id`。 再如，在好多资源中都有包年包月和按量付费之分，计费类型可统一定义为“instance_charge_type”。对于一些语义清楚的参数，如资源名称VpcName，VSwitchName，LoadBalancerName等，在具体的资源中只需用同一字段"name"来定义就好了。

5. #### 参数校验

   **有些参数需要做一些简单的校验，提前提醒客户使用正确的可选值**。除此之外，对于某些参数，要做diff判断，以自动屏蔽某些不起作用的参数。

   如：对于ECS而言，选择`PostPaid`，意味着所有与`PrePaid`相关的参数 `period`，`period_unit`，`renewal_status`，`auto_renew_period` 都将失效，具体表现为在执行`terraform plan`的时候，这些参数不会做diff比较。

### 基本实现

每个Resource需要实现Create，Read，Update，Delete，Import五个功能：

- Create
  - 调用产品创建API，实现对某个资源的创建，并将resource id 写入到state文件中
  - 如果资源没有ID，比如安全组规则，那么resource要自己按照一定的规则自行实现一个ID，对terraform而言，该ID是resource的唯一标识，后续对resource的所有操作都需要依赖于该ID进行
  - **Create 的实现逻辑要尽可能简单，以成功创建资源为目标，太多的复杂逻辑会降低资源创建的成功率**
  - **复杂的功能实现可交由 Update 来完成**
  - **Create之后通常调用Update方法来完成更多功能的实现**
- Update
  - 调用产品的Update或Modify API，实现对resource更多功能的支持以及对已有属性的修改
  - Update之前要打开Partial `d.Partial(true)`，并在每步修改后进行时时更新，如`d.SetPartial("bandwidth")`，以保证后面的每一步成功的更改都能被及时的写入到state 文件中。
  - **Update 之后，调用 Read 方法，实现对resource 所有属性的展示**。
- Read
  - 调用 Describe 或者 List API，实现对已有资源的查询和并将结果写入到state
  - 如果找不到指定的资源，要将resource ID标记为“”，以告诉terraform，当前这个资源已经不存在了。
- Delete
  - 调用Delete API实现对指定资源的删除和释放
  - **为了保证resource成功释放，需要加入retry策略来避免因资源依赖或者异步操作而引起的删除失败问题**
  - **通常resource释放后要再次调用查询API来完成删除操作的验证**
- Import
  - 调用查询API实现对已有资源的导入
  - 通常只需要方法申明，无需多余的实现逻辑，它会借助Read方法来完成对资源的查询和导入

### 基本工作原理

对用户而言，模板中定义资源是Input，控制台看到的通过terraform生产的资源是Output，但对terraform而言，模板是Iuput，资源创建完成后，运行命令的当前目录下的state文件是Output。不管是对用户还是Terraform，当Input和Output不一致时，就意味着**模板中定义的资源来发生变更，这个变更可以是资源属性的简单修改，也可以是某个资源的删除，也可以是某些资源的删除再重建**。

对于Terraform而言，资源的管理是通过不同的命令来进行的。主要的命令包含以下几个：

1. terraform plan

   **实现对资源的预览**。该命令会**对模板和state文件做一个diff，当发现不一致时，会将diff的结果显示出来，供用户预览**。如果资源尚未创建，及当前目录下的state文件为空，该命令直接展示即将创建的所有资源，否则展示要变更的资源。

2. terraform apply

   **实现对资源的创建和更新**。**对于新资源，调用`Create`，完成对资源的创建后，调用 `Update` 完成对某些属性的变更或者直接调用 `Read` 将资源属性写入到State文件中**；对于已有资源，如果terraform plan的结果为空，即没有任何修改，则该命令不会产生任何效果，否则调用`Update`完成对资源的修改，之后再次调用 `Read` 将变更信息同步到State中。

3. terraform destroy

   调用`Destroy`完成对资源的销毁。

4. terraform import

   **调用`Read`方法完成对已有资源（所谓已有资源就是通过非Terraform创建和管理的资源）的导入，将已有资源加入到terraform的管理序列中来**。但要注意，由于已有资源不一定是通过terraform创建的，所有导入成功后，记得运行`terraform plan`进行对比，手动补齐模板，保证模板定义的资源与State保存的一致。

### 开发步骤

在对所要开发的资源和Terraform原理有了熟悉和了解之后，接下来就是具体的开发工作。

1. 资源命名

   开始开发时，首先要定义一个资源名称，这个**资源名称代表了所要开发的resource在provider中的定义**。

   命名规则遵循：`alicloud_<产品简称>_<云资源名>`。其中，产品简称一般跟产品所定义的，或者跟控制台域名前缀保持一致，如slb，ess，ram等；云资源名称一般更API中定义的一致，如instance，security_group等；若当前资源是一个关系资源，通常在云资源名后面，增加一个后缀`_attachment`。

   资源名称定义好之后，该资源对应的文件名即为`resource_alicloud_<产品简称>_<云资源名>`。

2. 定义Schema

   Schema的function name 遵循`resourceAlicloud<产品简称><云资源名>`。Schema的定义包含方法的定义和资源参数的定义。

   方法的定义包含对Create，Read，Update，Delete以及Import的声明，以便在下午中进行具体的实现。

   参数的定义是对当前资源所要包含的所有参数的声明和定义。参数分为入参和出参，每个入参也会被自动申明为出参。每个参数必须声明参数类型`Type`，每个出参通过申明`Computed: true`来标识，每个入参通过`Required: true`或者`Optional: true`来标识。除此之外，入参还可申明参数的可选值，是否为ForceNew，是否有Default值等，具体的详见terraform/helper/schema中支持的资源属性。

3. 注册Resource

   在定义完 Schema 后，要在“provider.go”的ResourcesMap中增加一条kv条目：`<resource name>: <resource schema name>`，即`alicloud_<产品简称>_<云资源名>: resourceAlicloud<产品简称><云资源名>`，该条目表明当前添加的resource是Alicloud Provider中的一个resource，如果没有添加，当运行terraform命令时，provider将无法解析模板中定义的资源。

4. 声明 Sdk Client

   在调用SDK之前，需要在`config.go`文件中声明所要调用的SDK Client，为API的调用提供统一的入口。每个产品线共用同一个Client。每个Client声明需包含以下几个部分：

   - 加载本地的Endpoint，以满足专有云的需求
   - 指明当前Client是否支持STS Token
   - 指明当前Client所使用的UserAgent，便于Terraform使用量的统计

   Client声明过程中需要下载对应的Go SDK。本项目是通过 [Go Vendor](https://github.com/kardianos/govendor)来实现对所依赖的GO SDK的管理的。可使用`govendor fetch <sdk github>` 来加载指定的SDK的文件或者目录，如命令`govendor fetch github.com/aliyun/alibaba-cloud-sdk-go/services/ecs` 会将最新的ecs的GO SDK更新到目录"vendor"下的对应目录中。

5. "Create" 方法

   完成 Sdk Client的声明后，接下来功能开发。首先是创建资源。资源的创建就是调用云资源的Create方法来实现对云资源的生产，在这过程中要注意一下几点：

   - Create的最主要也最重要的目的是保证资源创建成功，并在成功后将资源ID通过`d.SetId(<resource Id>)`写入到State文件中，这样可以保证即使后面的所有操作都失败了，对terraform而言这个资源已经存在了，后面可根据这个ID来继续操作和管理资源，因此资源创建的过程逻辑要尽可能的简单，将一些复杂的附属的操作留在ID设置完成后或者Update方法中去做，如分配公网IP，设置Tag等
   - 关系型资源的创建，如磁盘的挂载，EIP的绑定等，就是调用资源Attach或者Associate或者Bind的接口，完成关系型资源的添加
   - 大部分的API都是异步的，而且存在资源的状态转移，对于这部分资源在完成Create之后，要保证所创建的资源达到最终的可用状态，如Running，InUse，Available等
   - 资源与资源之间往往存在一些相互依赖的关系，而这种关系也往往会因为资源状态不可用导致创建失败，因此需要根据特定的错误码通过Retry来保证资源创建的成功率。
   - 资源创建完成之后，通常是调用Update方法来进一步完善资源，如设置Tag，修改属性等。如果当前没有Update方法，即所有参数都不可修改，则可在资源创建完成之后直接调用Read方法将资源属性写入到State文件。
   - 在实现过程中，一些共用的方法会被抽象在对应产品的service文件中。

6. "Update" 方法

   Update操作是利用生成的资源ID来实现对指定资源的属性修改的过程。在这一过程中，需要注意一下几点：

   - 修改之前，首先需要通过`d.Partial(true)`打开Partial，这么做的目的是为了将每一步的修改实时的更新到state文件中
   - 对于同一个API中多个属性的修改，尽可能只调用一次API来完成多个属性的修改，降低修改的复杂度
   - 对于每个变更的参数，要通过`d.SetPartial()`来将该字段的修改同步到state文件中
   - 和Create一样，某些操作需要借助Retry完提供修改调用的成功率
   - 在所有参数修改完成之后，要记得通过`d.Partial(false)`将Partial关闭。
   - 属性修改完毕后，进入Read方法

7. "Read" 方法

   Read操作是利用生成的资源ID来查询资源的属性，并将这部分属性值通过`d.Set(<参数名>, <参数值>)` 写入到state文件中。当通过资源ID无法查到对应资源时，要将资源ID设置为空，表明该资源已不存在。

8. "Delete" 方法

   Delete操作是利用生成的资源ID调用Delete API来删除指定的云资源。在这一过程中，需要注意一下几点：

   - 因为一些相互依赖关系的存在，会导致某些资源无法一次性删除，此时需要借助Retry来保证资源被成功删除
   - 资源删除操作后，要利用再次调用Describe API来保证资源被成功删除
   - 对于关系型资源，资源的删除对应于关系的解除，如磁盘的卸载，EIP的解绑等

9. "Import" 方法

   Import方法通常不需要开发，默认调用Read方法完成资源属性的查询和导入。

10. 编译测试

    在完成功能的开发后，接下来就是要编译provider并测试开发的功能。运行命令`make dev`或者`make devlinux` 或者`make devwin`build对应操作系统的Provider，之后编写资源模板文件，运行terraform命令，完成资源的创建，更新，删除，并不断完善功能。

## Data Source 开发

Data Source 开发是调用阿里云资源的查询API完成对特定资源的查询和展示。

### 规划和设计

每个Data Source的实现要根据API的字段和资源属性，为用户提供更好的查询体验，最好可以支持模糊查询。参数设计原则除了与Resource设计原则类似外，还应该提供一个参数`output_file`来将查询到的结果输出到文件中，供用户参考。

### 开发步骤

每个Data Source只有一个Read方法需要实现，该方法用来查询并过滤符合条件的所有的resource，然后将过滤的结果以列表的方式展示出来。在具体实现过程如下：

1. 资源命名

   和resource一样，data source的名称代表了所要开发的data source在provider中的定义。命名规则遵循：`alicloud_<产品简称>_<云资源名>`。其中，产品简称更resource一样，云资源名称都是复数形式，因为data source的输出都是列表。

   资源名称定义好之后，该资源对应的文件名即为`data_source_alicloud_<产品简称>_<云资源名>`。

2. 定义Schema

   Schema的function name 遵循`dataSourceAlicloud<产品简称><云资源名>`。Schema的定义包含方法的定义和资源参数的定义。

   方法的定义只包含Read。

   参数的定义是对资源查询接口中所支持的参数的申明和定义。每个data source都必须包含一个统一的入参"output_file"，用来存储所查询到的所有资源；出参类型都是list，list中的每个item包含了所查询资源的属性值。

3. 注册Data Source

   在定义完 Schema 后，要在“provider.go”的DataSourcesMap中增加一条kv条目：`<data source name>: <data source schema name>`，即`alicloud_<产品简称>_<云资源名>: dataSourceAlicloud<产品简称><云资源名>`，该条目表明当前添加的data source是Alicloud Provider中的一个data source，如果没有添加，当运行terraform命令时，provider将无法解析模板中定义的data source。

4. 声明 Sdk Client

   如何没有对应产品线的Client，和resource中指出的一样，需要申明一个sdk Client。

5. "Read" 方法

   Read操作是利用定义的入参，查询并输出满足指定条件的所有资源。和resource一样，查询的数据会被存储在state文件中，所以每个data source也需要一个ID来标识。由于输出结果是list，所以通常当前data source的ID是取所有Item的ID的hash值。资源过滤完毕后，要利用set将数据写入state文件中。

6. 编译测试

   和Resource一样。

## Acceptance Test

在完成模块开发后，要对每个resource和data source的功能进行接收性测试，测试的方法是编写对应resource和data source的测试用例。

测试用例的编写要遵循以下几个原则：

1. 每一个resource和data source对应一个测试文件，文件的命名是在resource或者data source文件名后增加后缀"_test"。

2. 每个测试文件中要实现一个Existing Check方法和Destroy Check方法。

3. 对于支持Import功能的resource，需要提供一个针对Import功能的测试文件。

4. 测试用例要尽可能覆盖所有的应用场景，尤其对一些主要功能的修改，要使用单独的case

5. 测试用例不是完成任务，是每一次功能变更的检验，测试用例也完善，模块的稳定性也高，因为功能的持续更新和迭代必须要保证原有功能的可用。

6. 如何运行测试用例
   以ECS instance 举例，运行如下的测试命令，可以实现对目录`alicloud`下所有以`TestAccAlicloudInstance`为前缀的测试用例的运行：

   ```
   TF_ACC=1 go test ./alicloud -v -run=TestAccAlicloudInstance -timeout=300m
   ```

   其中，`timeout` 表示这个测试用例运行的超时时长。
   如果想要精确到具体的测试用例，补全测试用例的名称即可。

## 文档

文档是用户使用provider的基础，文档的编写非常重要，需遵循如下几个原则：

1. 文档中参数描述要清楚明白，每个参数功能，使用，是否是必填参数，是否支持修改，是否有限定值，是否有默认值，都要显示标明，而且要跟代码中定义的保持一致
2. 文档使用纯英文编写，但不是对帮助文档的直接翻译，要根据具体情况和场景做最友好的阅读体验。如果对英文不自信，可参考帮助文档的国际版 https://www.alibabacloud.com/help
3. 通常，每个文档都要写一个简单的当前资源的使用实例，以供参考
4. 文档的输出参数也要进行说明，而且要跟代码中定义的保持一致
5. 如果当前资源支持import功能，文档最后要显示标注，详见其他resource 文档
6. 如果当前资源有一些暂时不支持的功能，或者使用上有需要提前注意的地方，要在文档中显示添加`Note`
7. resource 文档放在目录`website/docs/r`下，data source 文档放在目录`website/docs/d`下。每新增一个文档，要修改文件`website/alicloud.erb` 来完成侧边栏展示

## 样例

为了更好的帮助用户使用新增的resource，在完成以上工作后，需要为当前的这个resource或者当前这类resource增加一个example，来指导用户编写对应的模板。该样例是一个真实的可运行的模板，通常包含以下四部分：

1. main.tf：资源的定义模板
2. variables.tf：模板所依赖的参数定义和参数值
3. outputs.tf：资源的输出值
4. README.md：模板的简单介绍和使用说明

## 代码提交

完成了以上功能的实现和代码的编写后，接下来最关键的就是代码的提交。值得注意的是，如果您是阿里巴巴公司内部的同学，在代码提交之前，首先需要检查自己的代码中不包含敏感信息，如access key，secret
key等，接着，走provider新版本发布流程。

在完成新版本发布流程后，代码的具体的步骤分为以下几步：

### Commit

为了避免出现代码的错乱，降低review的复杂度，Commit要细化，每一功能点或者一次的修改（不论改动有多小）对应一个Commit。为了防止功能和代码的相互冲突，每个功能点对应一个branch。

### Rebase

提交代码前一定要`Rebase`，具体操作步骤如下：

1. Configuring Your Remotes

   Rebase之前，首先需要配置你的Remote，假定你`origin` 指向了你的fork：

   ```
   $ git remote -v
   origin  git@github.com:YOUR_GITHUB_USERNAME/terraform-provider-alicloud.git (fetch)
   origin  git@github.com:YOUR_GITHUB_USERNAME/terraform-provider-alicloud.git (push)
   ```

   接着，将主仓库配置为你的remote，如起名为“alicloud”

   ```
   $ git remote add alicloud https://github.com/alibaba/terraform-provider-alicloud.git
   
   $ git remote -v
   origin  git@github.com:YOUR_GITHUB_USERNAME/terraform-provider-alicloud.git (fetch)
   origin  git@github.com:YOUR_GITHUB_USERNAME/terraform-provider-alicloud.git (push)
   alicloud  https://github.com/terraform-providers/terraform-provider-alicloud.git (fetch)
   alicloud  https://github.com/terraform-providers/terraform-provider-alicloud.git (push)
   ```

   检查当前仓库的状态

   ```
   $ git status
   On branch YOUR_BRANCH
   Your branch is up-to-date with 'origin/YOUR_BRANCH'.
   nothing to commit, working tree clean
   ```

2. Rebasing Your Branch

   完成commit和remote配置后，即可进行rebase操作：

   ```
   $ git pull --rebase alicloud master
   ```

   如果有冲突，解决冲突后继续 rebase，直到所有冲突都解决，之后再次检查状态：

   ```
   $ git status
   On branch YOUR_BRANCH
   Your branch and 'origin/YOUR_BRANCH' have diverged,
   and have 4 and 1 different commits each, respectively.
     (use "git pull" to merge the remote branch into yours)
   nothing to commit, working tree clean
   ```

3. Uploading Your Code

   完成rebase操作后，上传代码：

   ```
   $ git push origin <your branch>
   ```

   如遇到提交失败，可使用`-f` 或者`--force` 来强制提交

4. Submitting Your Pull Request

   完成代码上传后，在Github控制台将你的代码提交到主仓库的`master`分支，并对PR做一个简单的描述和介绍。同时需要将当前resource或者data source的Acceptance Test的运行结果填写在PR的描述中。

5. Updating Your Pull Request

   如果代码有问题，在Review结束后，需要对已提交的代码做修改，首先将上次提交进行回退：

   ```
   $ git reset HEAD^1
   Unstaged changes after reset:
   ******
   ```

   之后，stash并利用rebase下载最新的代码，以避免不必要的冲突：

   ```
   $ git stash
   Saved working directory and index state WIP on YOUR_BRANH: *******
   HEAD is now at *******
   
   # rebase 下载最新代码
   $ git pull --rebase alicloud master
   ```

   rebase顺利执行之后，恢复stash代码：

   ```
   $ git stash pop
   On branch YOUR_BRANCH
   Changes not staged for commit:
   *****
   ```

   根据review意见，继续修改代码，之后再次进行代码commit，rebase和push即可。

## 写在最后

本文主要讲述了如何为阿里云的Terraform-Provider贡献代码的一些流程和注意事项，欢迎大家积极加入到Terraform Provider的建设中来，谢谢大家。

**版权声明：**本文内容由阿里云实名注册用户自发贡献，版权归原作者所有，阿里云开发者社区不拥有其著作权，亦不承担相应法律责任。具体规则请查看《[阿里云开发者社区用户服务协议](https://developer.aliyun.com/article/768092)》和《[阿里云开发者社区知识产权保护指引](https://developer.aliyun.com/article/768093)》。如果您发现本社区中有涉嫌抄袭的内容，填写[侵权投诉表单](https://yida.alibaba-inc.com/o/right)进行举报，一经查实，本社区将立刻删除涉嫌侵权内容。



https://developer.aliyun.com/article/621240