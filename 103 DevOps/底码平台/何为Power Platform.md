# 何为Power Platform?

[离一](https://www.zhihu.com/people/chi-yi-62-92)

数据从业者

7 人赞同了该文章

PL-100是对于Microsoft Power Platform使用情况的一个测试，分很多等级如PL-100,PL-200,PL-400,PL-600,DA-100。这一个月学习下来，power platform给我的感觉就是**低代码**(实现网页设计，图表设计，流程自动化等不需要涉及到很多代码，入手相对简单)，**多功能**(power app可以设置自己的小应用，或者给公司处事流程创建一个app使用界面)；

**Power Platform介绍**

Power Platform的核心组件有Power App, Power Automate, Power BI, Power Virtual Agents.

**(1) Power App**是操作者可以在网页开发APP的一款应用，主要由Canvas App(画布应用)，Model Driven App(模型驱动应用)组成，Canvas App更多的是设计一些个性化的APP(灵活性更好，轻量级的应用)，Model Driven App则是一个有标准化框架的APP,根据需要设计出里面的组件，最后将这个组件放入这个框架中，便可以设计出复杂的APP；下面给出两个设计界面的截图方便理解。

![img](https://pic3.zhimg.com/80/v2-6d98cdb890092237e9d7f731eebebdde_1440w.webp)

上面是Power App的一个开发环境，最后APP的应用可以实现对电脑信息表的查询，增加，删除，修改等操作；

![img](https://pic2.zhimg.com/80/v2-3b890e9505937ebf218ab7c6151a8ed9_1440w.webp)

上面是一个简单的模型驱动应用，里面除了实现对数据的增删查改，还可以放入的dashboard,自动化流等，功能可以更复杂。

**(2)Power Automate**: power automate主要是将一些重复操作的流程做成自动化，比如每周一给公司所有成员发邮件；当数据集中有记录增加时，给负责人发邮件等；当用户按下应用中的某一个按钮时，后台可以自动实现某些操作；功能有点类似于RPA，尽可能简化一些手工操作。

![img](https://pic4.zhimg.com/80/v2-53fe145dd318dc8a6dfdd2c69bb36753_1440w.webp)

如上图一样不断追加不同操作，最后实现某个流的一个过程；

**Power BI**: Power BI和Tableau功能比较相像，可以在其中设计Dashboard来展现数据，分析数据。

**Power Virtual Agents（聊天机器人）：**用户可以根据图形界面创建出一些智能对话机器人，可以将这个智能对话机器人放到自定义网页或者Teams应用或者某个画布应用中去。本文主要介绍Power App和Power Automate的知识点。

**Power App中Canvas App**

1.常用的公式场景（仅展示案例中常用的）



![img](https://pic4.zhimg.com/80/v2-6a6b50a6ac0aff1e1b6db1e5a4129bb7_1440w.webp)

**(1)对于插入的图标（例如圆形的填充色）实现不同的职位设置不同的填充颜色：**

Switch(ThisItem.职位,"经理",RGBA(0, 0, 255, 0.5),"组长",RGBA( 0, 139, 139, 1 ),"职员",RGBA( 184, 134, 11, 0.5 ))

Switch和If的用法：If( Slider1.Value = 25, "Result1", "Result2" )

Switch( Slider1.Value, 20, "Result1", 10, "Result2", 0, "Result3", "DefaultResult" ) 如果Slider1.Value为20则返回Result1，如果为10则返回Result2，如果为0则返回Result3，如果都不是则返回DefaultResult（可以放也可以不放在公式中）；

**(2)控制插入标签要显示的内容：**标签的Text属性设置为如ThisItem.姓名（姓名为数据源中的字段） 显示文本粘贴函数Concatenate(ThisItem.部门, " - " ,ThisItem.职位)或者用ThisItem.部门& " - " &ThisItem.职位;

根据不同的情况设置不同的Header: 通过表单的状态显示

If(Form1.Mode=FormMode.Edit,"编辑员工信息", Form1.Mode=FormMode.View,"查看员工信息", Form1.Mode=FormMode.New,"新建员工信息")

**(3)对跳转链接图标（Icon）的属性设置：**如跳转到详情页Icon的OnSelect属性设置，

ResetForm(Form1);ViewForm(Form1);Navigate(员工新增)

ResetForm(Form1)是重置表单,ViewForm(Form1)将表单设为可读状态,

Navigate是导航到某一窗页；Refresh(‘datasource’)刷新数据源

控制Icon是否可见：If(Form1.Mode=FormMode.View,false,true) 当处于编辑模式下是不可见的；

以及控制提交选项Icon时,输入无效信息不可提交：

If(!IsBlank(ErrorMessage1.Text)|| !IsBlank(ErrorMessage2.Text)|| !IsBlank(ErrorMessage7.Text) ,Disabled,Edit)

**(4)表单（Form）**

表单一般与Gallery或者下拉列表，Lookup，First函数等结合使用；

**表单(Form)与Gallery结合使用**：表单的Item属性设置为Gallery控件的Selected属性（Item设置为Gallery1.Selected）,比如可以实现选中Gallery中的某一条记录，会在另一个表单中显示关于这条记录的详细信息。

**与函数结合**：Form的Item设置为First('Device-Order-Data')或Lookup(Accounts, "Fabrikam" in name)，表单会默认显示数据集中的某一部分数据。

可以通过一些Icon来更改Form的模式（编辑/新建），最后通过SubmitForm函数提交更改；

比如：

新建表单Icon的OnSelect属性：ResetForm(Form1);NewForm(Form1);Navigate(员工新增)

修改表单Icon的OnSelect属性：ResetForm(Form1);EditForm(Form1);Navigate(员工新增)

浏览表单Icon的OnSelect属性：ResetForm(Form1);ViewForm(Form1);Navigate(员工新增)

最后通过SubmitForm(Form1)这种形式提交上面的更改；

删除表单记录Icon的OnSelect属性可以为Remove(员工信息,Gallery1.Selected)

**返回Icon的OnSelect属性**，有Back()返回上一页面，或者Navigate(‘PageName’)导航到别的页面，Navigate()函数中也可以放参数；

**控制控件的位置**（例如控制删除按钮控件的位置）

根据表单提交Icon是否显示调整控件位置：If(Icon1.Visible,500,570)

**控制表单中的字段是否可以编辑：**

当表单处于编辑/查看状态时，是可以编辑的，否则是不可以编辑的；

If(Form1.Mode=FormMode.Edit,DisplayMode.View,Form1.Mode=FormMode.View, DisplayMode.View, Form1.Mode=FormMode.New, DisplayMode.Edit)

**(5)表单中怎么实现搜索功能：**

**简单：**先插入文本输入控件作为搜索框

将表单的Items属性改为Filter(员工信息, StartsWith(姓名,TextInput1.Text))

TextInput1.Text指搜索框内输入的文本；（实时搜索，每输入一个字就会搜一遍）

**进阶设置：**将搜索框内的信息设置为全局可以使用的变量 Set(gblSearchText,TextInput1.Text)

表单的Items设置进一步改为Filter(员工信息,StartsWith(姓名,gblSearchText)) 用设置的变量来搜索，同时把DelayOutput设置为True,为false表示在键盘上输入文字会立即作出响应，如果设置为True,会把延长时间拉长；

**(6)如何实现数据筛选功能**

插入组合框（Combo box）控件：下拉列表，有允许多个选择，允许搜索设置；

组合框的Items可以设置为：["IT","develop","software"] 自己定义的下拉选项，不够灵活；

还可以设置从数据源中筛选：Distinct(员工信息.部门,部门)或者Distinct(员工信息,部门)

然后对Gallery的Items属性进一步增加部门为所选部门的条件：

Filter(员工信息, StartsWith(姓名,gblSearchText) && 部门 = ComboBox1.Selected.Result)

**注意 &表示连接符；&&表示并且的意思**

**筛选的优化**：当一个都不选时，默认展现所有：对公式进行更改

Filter(员工信息, StartsWith(姓名,gblSearchText) && (IsBlank(ComboBox1.Selected.Result)||部门 = ComboBox1.Selected.Result) )

当下拉选框选中多个值时，默认只会筛选出最后一个，怎么将两个都筛选出：将前面条件改为in，或者改为单选模式；

Filter(员工信息, StartsWith(姓名,gblSearchText) && (IsBlank(ComboBox1.Selected.Result)||部门 in ComboBox1.SelectedItems) ) 因为是复选框下下拉的多个选项，所以是SelectedItems

**(7)变量(Variable)：**通过设置变量可以把一组公式、文本等内容存储到变量中，在其他地方引用变量达到想要的效果；

**全局变量(Global Variables)：**用Set进行设置如Set(gblSearchText,TextInput1.Text) 该变量的设置可以在任何屏幕中使用(scoped for application)；

**上下文变量(Context Variables)：**UpdateContext进行设置，只能在当前页面使用(scoped for screen)；UpdateContext( { Name: "Lily", Score: 10 } ) 设置Name变量和Score变量;

关于Context variable会涉及到的三个函数：Patch, Navigate, UpdateContext;

关于全局变量：用到Collect,Set会比较多；

**Power App中Model Driven App**

**(1)模型驱动应用制作过程大致分为3个阶段(从组件设计开始)：数据建模：**在Dataverse中基于需要的数据构建表与表之间的关系；**定义业务流程：**业务流程中的每个阶段要做什么；引领用户完成某些操作；**设计应用界面：**设置窗体，视图类型；**最后组装：**从空白开始创建模型驱动应用，选择新型应用设计器，直接往其中加页面；另一种是Classic app designer(经典模型驱动应用)

**(2)模型应用包含的组件：数据：**dataverse中表，列，关系 **UI:**用户与用户之间如何进行交互，有窗体，视图等 **逻辑：**业务规则，工作流，业务流程 **可视化：**图标，仪表板，power bi

**(3)Model Driven App的组件：**（包含对Dataverse的部分组件介绍）

从Dataverse入手准备Model Driven App的组件，在Dataverse中新建/上传一个表；

**Relationship:** 表选项卡下的relationship新建的过程,点击add relationship,选择关系类型（一对多，多对一，多对多），建立表与表之间的关系。

**Business Rule:** 定义业务规则，比如对价格字段进行设置，在右边的选项卡里点击Properties,在其中输入规则名，Entity对应内容，以及下面的rule填好（这一步相当于填写Condition Expression）；定义好condition之后需要加上action, action有add recommendation,

add lock/unlock, show error message, set field value, set default value, add business rule,

add set visibility; 整一个business rule的表达式相当于if… then…

**Views:** 主要配置的是Active表，往视图里面增加自己想要显示的字段，并且整理字段顺序；这个View是在model driven app打开后直接显示的表；

**Forms:** 会默认显示3种，QuickViewForm, Card, Main,主要修改Main；表单是点击某个选项后弹出的界面；

**Main form：**点击记录时，会跳转到一个新的页面，或者在创建/编辑记录时会跳转的全新页面就是主窗体定义的；可以插入Timeline控件或者新建业务流程；

**Quick View form:**用于显示和当前记录相关的记录，在添加快速视图控件时，会显示指定表的快速视图窗体来显示效果，显示的记录不会跳转到单独页面；

**Quick Create form:**区别于上面的主要是会弹出该窗体，窗体的界面较小，能够添加的控件也是有限的；

**Card form:**在手机端显示视图的一种样子；不会显示字段的标签名字，而是直接显示实际的值（比如：姓名(不显示) 直接显示李祺(这样)）所以Card form中会直接显示比较有意义的字段，让用户清晰的看到数据；

窗体需要保存并发布；

**Business Flow(业务流程)** 位置：在Dataverse下面的流中进行创建，流程是基于相关表创建的，比如基于设备订单表创建的一个业务流程，指引用户完成某些操作；

**Power Automate**

**1.概念理解：**

**(1) Power Apps和Dataverse相当于是应用的前端界面和数据存储；**如果要完成一个应用场景，需要让数据在不同的环境中进行流转，这时候Power Automate会比较有用。

Power Automate提供三种自动化的能力:

第一种通过AI builder提供支持的AI能力；

第二种API自动化：某些系统会有API,可以通过Power Automate实现系统之间的流程交互；

第三种RPA自动化能力：某些系统没有API，可以通过RPA方式，让没有API的系统之间也可以进行流程交互。

**(2)上面的三种能力体现在下面的5种自动化类型上；**

第一种自动化云端流(automated cloud flow)，由指定事件触发，比如sharepoint中新增数据时，SQL库中新增数据时，有新的form表单提交时，当这些事件发生时会自动执行某些任务，这就属于自动化云端流。

第二种是即时云端流(instant flow/button flow)，需要用户手动触发形成的流程；比如用户在Power app中点击某个按钮，背后执行一些自动化的流程，就称之为即使云端流；

第三种计划云端流(recurring flow)，定时任务，时间周期可以是时分秒等；比如每周一早上自动发邮件等；

这三种区别是触发方式不同，流程内执行的操作和自动化任务没有任何区别，都是云端流；

第四种桌面流（RPA流程），本身没有触发器，一般与前面三者结合使用来达到自动化的流程；比如像在一个网站上（没有API）执行某些操作，或者本地系统（SAP）执行某些重复操作；

第五种业务流程，主要在power app中应用，并且这个流程一般和Dataverse表进行关联，目的用于引导用户完成某些操作；比如之前的设备订单流程；

**(3)具体介绍云端流（前面三种自动化流程）核心：触发器，操作和条件控制；**

第一步就是要确定你的流程是由什么触发的，可以是前面提及的某个时间，按钮触发或者计划触发等；(Trigger)

第二步：在流程触发之后，想执行什么样的操作，比如说发邮件还是增加数据等;(Action)

第三步是否加条件控制，条件控制是在流程过程中，用条件控制来控制我们要执行的操作；

同样也会有flow checker(流程检查器)，测试，发布;

**2.Power Automate实践：**

**(1)** **即时云端流实践(常与Power App应用结合，通过按钮触发某一个流程)**

做一个向EXCEL中追加数据的Flow；(手动触发)

1.在Power Automate选项卡下的创建选择——即时云端流

2.基本流程先是点击手动触发流，增加操作，选择在EXCEL表中插入新列

在列字段选择的时候，里面的字段有两种输入形式，一种是动态内容：有系统自己会动态更新或者获取的，比如触发时间，地址等；如果前面有自己设置的转换时间或者一些触发时要填的内容，也会在动态内容下面显示，第二种是表达式，根据公式最后得到的结果；

3.整个流的优化：手动触发流操作下可以设置触发时，自己自定义的文本或者其他文件，在后面的动态内容中会显示；

4.对于日期的转化（字段的变形）,可以在操作中增加操作，或者在输入内容时设置表达式等;

**(2) 计划云端流实践（时间触发）**

目标：每周把办公室座位数的数据发给相关人员；

OneDrive for Business创建一个EXCEL,然后把数据转为表格，读取表格中的数据发送邮件；读取数据如下：办公室座位最多的是哪座城市，其座位数有多少，所有城市总的座位数有多少；

1.设置前面的任务进行频率 Recurrence设置

2.设置初始化变量-总座位数（我们要求的）

3.连接EXCEL: 设置为列表中存在的行

4.接着是应用到每一个设置，选择前面的Value(初始值) 后面再设置增量变量总座位数，从EXCEL中读取数据时会将数字读取为文本，所以进行算术运算需要int,转为数值类型；

Int(items(‘应用到每一个’)[‘座位数’]) 表示从应用到每一个操作中获取当前遍历的行，并从中读取座位数，并将座位数转化为int类型，最后再次运行流程，会显示循环的次数，最后一次便是累加的和（增量变量）

接着获取最大办公室的城市及最大办公室的座位数

同样的需要在前面设置初始变量：最大办公室(字符型) 最大座位数(整数型)

应用到每一个选项下有增量变量得到的办公室总座位数，然后通过条件判断，可以得到最大座位数的城市及其座位数；

最后就是发送邮件流：在发送邮件内容中引用前面的变量；

最后 保存 验证下 验证成功；

**(3) 自动化云端流实践（事件触发）**

目的：用户在SharePoint列表中新增数据时，会触发一个流，要求经理或者相关人员审批，审批通过后会将审批状态更新到sharepoint列表上；

第一步：创建项，把站点信息填上;

第二步：查找审批(已经有审批内置模板)，找到启动并等待审批

第三步：判断审批状态；结果等于Approve;是的话，设置审批状态为允许，否设置审批状态为拒绝；

第四步：做一个发送电子邮件的操作

至此，关于power platform的小分享已经结束，欢迎大家扫码关注下面的公众号，后期还会分享关于Tableau,Python,R语言,EXCEL,数据分析思维等知识。



编辑于 2021-11-07 11:15



https://zhuanlan.zhihu.com/p/430388126