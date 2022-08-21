# Terraform Module 可视化正式发布

2019-12-18 4896

**简介：** 阿里云正式发布 Terraform Module 的可视化操作界面，在命令行操作模式的基础上，增加了基于 Terraform 的在线资源编排的能力，持续帮助开发者和企业降低 Terraform Module 的使用门槛。

**本文已在下方公众号中发布：[Terraform Module 可视化发布](https://mp.weixin.qq.com/s/-88QmyQqIoPRdCFgJEYsfQ)，欢迎大家围观**



## 可视化操作界面

12月12日，阿里云开放平台正式对外推出 Terraform Module 的可视化操作界面：https://api.aliyun.com/#/cli?tool=Terraform，集合所有在 [Terraform Registry](https://registry.terraform.io/browse/modules?provider=alicloud) 上注册过的 Module，对外提供在线运行 Terraform Module 的能力，开发者只需关注 Module 参数本身和所要执行的命令，剩下的工作将由可视化界面来完成。

![_2019_12_15_5_57_44](https://yqfile.alicdn.com/d18a511a1838a6b4c0dae3aad1e812cf6891c9f5.png)



## 可视化五大亮点

**亮点一：完全开放，覆盖全量 Terraform Module**
可视化界面中展示的 Terraform Module 与 Terraform 官方 Registry 中注册的保持一致，任何开发者提交和注册的 Module 都会在界面中展示，并被分享给其他所有开发者使用，最大化发挥 Module 的价值。

**亮点二：按活跃度排序，让最优秀的 Module 站 C 位**
跟官方 Terraform Registry 按 Module 注册时间显示不同的是，可视化界面中的 Module 是按照 Module 的下载量排序后显示的，最优秀的 Module 在最显眼的位置上展示，让开发者和用户更容易发现和使用。

**亮点三： 实时展示 Module 运行过程和结果，保持与命令行一致的操作体验**
可视化界面集成了 Terraform 最重要的三个功能操作：Plan（预览），Apply（创建／变更）和 Destroy（销毁）。用户通过界面填写 Module 对应的参数，可视化界面将自动将这些参数填入 Module 模板中，然后通过点击下方的操作按钮即可实现对 Module 中所定义资源的自动创建和编排。在此过程中，右侧的 CloudShell将会实时的显示当前任务的执行情况，这与通过命令行操作 Module 的体验是完全一致的。

**亮点四：同时具备“在线点击”和“在线命令行”两种操作模式**
可视化界面提供了对 Module 的操作按钮，可实现对 Module 中所定义资源的创建，修改和删除操作。如果想要切换到命令行模式，直接点击右侧的 CloudShell 显示界面，借助 CloudShell 对 Terraform 原生集成，可在 CloudShell 中直接通过 Terraform 原生命令来完成资源的持续管理。

**亮点五：更简单的参数输入，无需关心 Terraform 参数使用语法**
可视化界面将 Terraform 对参数的输入语法进行了简化，提供了最易用的参数输入方式，无需关心 Terraform 自身的使用语法。

阿里云开放平台借助 Terraform 原生的能力，推出的可视化操作界面，持续降低用户和开发者使用 Terraform 成本和门槛，持续带来更简单，更实用和更开放的极致使用体验。Terraform Module 可视化界面只是一个开始，是对命令行操作模式的补充和扩展，未来将在持续满足客户使用需求的前提下，将 Terraform 的能力在阿里云上进一步的释放和扩充，实现阿里云开放能力与 Terraform 开源特性更好的结合。



## 最后

**欢迎所有对 Terraform 和阿里云感兴趣的开发者，积极地加入到阿里云开源生态的建设中来。**动手实践，乐于分享，让自己的想法被更多的人看到，让自己写的 Module 得到更多的人引用。



[Terraform Module 可视化正式发布-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/739730?spm=a2c6h.12873639.article-detail.72.5ce132e69ddZZL)