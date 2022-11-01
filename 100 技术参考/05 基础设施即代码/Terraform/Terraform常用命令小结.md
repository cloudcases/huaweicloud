# Terraform 常用命令小结

```bash
$ tree .

# 初始化加载模块
# 包括Provider，Provisioner，Module等
$ terraform init

# 更新provider版本
$ terraform init -upgrade=true

# 重启会话可以支持自动补全
$ terraform -install-autocomplete

# 输出当前模板定义的资源关系图
$ terraform graph
$ terraform graph -draw-cycles

# 提前安装graphviz：https://www.graphviz.org/download/ 
$ brew install graphviz

# 输出当前模板定义的资源关系图
# 导出为一张图片
$ terraform graph | dot -Tsvg > graph.svg
$ terraform graph -draw-cycles | dot -Tsvg -o graph.svg

# dot 命令
$ dot -Tpng test.dot -o test.png

# ----------------------------资源管理--------------------------

# 验证模板语法是否正确
# 检查和定位错误出现的详细位置和原因
$ terraform validate

# 资源预览
$ terraform plan
$ terraform plan -var-file="./vars/alpha.tfvars"

# 资源新建和变更
$ terraform apply
$ terraform apply -var-file="./vars/alpha.tfvars"

# 资源展示
$ terraform show

# 资源释放
$ terraform destroy
$ terraform destroy -target=<资源类型>.<资源名称>

# 资源导入
$ terraform import

# 标记资源为“被污染”、取消“被污染”标记
$ terraform taint <资源类型>.<资源名称>
$ terraform untaint <资源类型>.<资源名称>

# 打印出参及其值
$ terraform output

# ----------------------------状态管理--------------------------

# 刷新当前state
$ terraform refresh

# 列出当前state中的所有资源
# 按照 <资源类型>.<资源名称>`的格式列出当前state中存在的所有资源（包括datasource）
$ terraform state list

# 展示某一个资源的属性
$ terraform state show <资源类型>.<资源名称>
# 例子：
$ terraform state show module.cce_cluster.huaweicloud_cce_cluster.cce_cluster

# 获取当前state内容并展示
# 原样展示当前state文件数据
$ terraform state pull

# 移除特定的资源
命令格式为： terraform state rm <资源类型>.<资源名称>
$ terraform state rm 

#变更特定资源的存放地址
$ terraform state mv 
```
