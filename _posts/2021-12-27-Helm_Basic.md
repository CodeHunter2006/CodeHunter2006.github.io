---
layout: post
title: "Helm Chart Basic"
date: 2021-12-27 22:00:00 +0800
tags: K8S
---

![Helm](/assets/images/2021-12-27-Helm_Basic_1.jpeg)

Helm chart 是在 K8S spec 的基础上，利用 Go template 实现了"模板+参数"的方式部署集群(包含多种资源)，方便集群部署的管理。

# 基本概念

- **chart**
  以 chart 文件夹中的所有文件及其 import 的文件构成了一个静态模板，这个模板经过参数填写就可以部署到 K8S 集群中，这就是一个 chart。
  chart 可以用 VCS(如 git) 控制起来。
- **release**
  chart 每次部署会生成一个 release 实例，chart 和 release 的关系就像面向对象中 class 和 object 的关系。
  release 保存在 k8s 对应 namespace 下的 secrete 中，名称格式：`sh.helm.release.v1.<release-name>.v<version>`
- **repository**
  用于存储 chart 的仓库
- **derective**
  在 template 中，以`{{ xxx }}`形式定义指令脚本，最终的渲染结果会以指令的返回值占位替换。
  指令的基本格式为`functionName arg1 arg2...`
- **pipelines**
  与 Linux 中的概念相同，以`|`作为连续指令的参数

# 文件夹结构

- `Chart.yaml` 描述 Chart 基本信息
- `values.yaml` 渲染模板时使用的参数，这些参数可以在`helm install`或`helm upgrade`时被覆盖
- `templates/` 各种资源模板，可以经过渲染变为最终的资源 Spec
  - `_helpers.tpl` 一般以`.tpl`作为 helper 后缀
  - 每种资源可以由多个独立的 Spec 描述文件组成，用`---`在 yaml 内分割表示
- `charts/` subChart

# 模板渲染

- value 优先级(覆盖)关系：
  `--set` > `--values` > `values.yaml` > `template`

- 模板渲染语法是基于 Go`text/template` 的渲染语法

- 在渲染过程中，helm 会持续检查 resource spec 定义文件的语法，一旦发现语法问题会报错停止

- `name: {{ .Release.Name }}-configmap`
  这里被`{{ }}`包住的部分是 template 指令

- Built-in Object
  在指令中可以使用 Object 的值，Object 可以嵌套，字段命名参照 Go 的对外暴露变量名

  - Release
  - Values
  - Chart
  - Files
    Access to files, except for template files.
  - Capabilities
  - Template

- 删除多于空格或换行符

  - `{{-` 删除左边多余空格
  - `-}}` 删除右边多余空格

- `with ... end`
  切换当前的 scope(第一个.前面隐含的对象)

- 通常 resource name 应该和 release name 相关，避免写死发生冲突

## 常用语法

- 注释
  `{{- /* import wv template configMap */ -}}`

- 调用函数时可以用`$_`表示一个返回参数占位

- `{{- $files := .Files }}`
  定义变量

- 在`range`范围内，如果需要`---`分隔多个文件，注意把`---`放在最后(如`end`之前)。
  如果放在中间，会报错

## 常用函数

- `tpl $tpl $`
  以文本内容为模板，返回模板套用后的结果，`$`表示将当前对象传给模板做 top 对象

- `toYaml $x`
  将对象转换为 yaml 格式的文本

- `nindent 2`
  忽略左边的 pandding 空格，向右缩进两个空格

- `range`
  遍历 array 或 map 类型的对象

  ```
  {{- range $k,$v := .data }}
  {{ $k }}: |
  {{ tpl $v $ | indent 2 }}
  {{- end}}
  ```

- `printf "%s%s" $.folderName "/*.yaml"`
  字符串连接，中间无空格

- `.Files.Glob` 遍历文件; `.Files.Get` 打开并读取文件内容
  ```
  {{- $files := .Files }}
  {{- $pattern := printf "%s%s" $.Values.folders "/*.yaml" }}
  {{- range $path, $_ := .Files.Glob $pattern }}
    {{ $files.Get $path }}
  {{- end }}
  ```

# 常用命令

- `helm list`
  查看已安装的 chart

- `helm get manifest helmName`
  显示 helmChart 清单，显示结果以`---`分割 yaml 文件。

- `helm create mychart`
  创建示例模板

- `helm install helmName chartFolderPath`
  以 helmName 为名，部署 helmChart 的一个 release

- `-n/--namespace`
  在`xxx`namespace 下，部署 chart 定义的资源。
- `--dry-run`
  dry-run 测试
- `--debug`
  显示更详细的 debug 信息
- `--set foo=bar`
  设置 values.yaml 参数
  - `--set`在一条命令中可以多次出现
  - 一次设置多个参数用`key1=val1,key2=val2`形式
  - 可以删除已存在的 value:`--set livenessProbe.httpGet=null`
- `-f/--values myvals.yaml`
  通过 yaml 文件的形式设置参数

- `helm uninstall helmName`
  卸载 helmName

# 参考

[Go template language](https://pkg.go.dev/text/template)</br>
[Sprig template library](https://masterminds.github.io/sprig/)</br>
[Template Function List](https://helm.sh/docs/chart_template_guide/function_list/)</br>
