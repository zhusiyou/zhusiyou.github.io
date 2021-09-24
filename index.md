# 动态表单文档
## 介绍
### 愿景
依据此工具提供的能力，通过界面操作，实现低代码完成系统功能的添加、减少系统更新次数、提高开发效率。此工具提供以下能力：
- 表单的构建、渲染、数据的保存
- 查询表单的配置、渲染，并完成数据的查询
- 列表的配置与展示
- 数据明细的展示
### 概念
- **表元数据**：存储数据库中表的元数据信息（如表的中文名称、是表还是视图、表名等），用于构建表单时使用。
- **字段元数据**：存储表的字段定义元数据（如字段的名称、数据类型、长度等），用于构建表单时使用。
- **物理表**：数据库中具体存在的表（如表"table_employee"）。此表可通过表元数据和字段元数据信息由动态表单系统执行ddl语句进行创建。
- **数据源**：对可以返回(key, value)格式的字典表、视图、函数的统一封装，用于下拉框、级联框等表单控件的数据绑定以及查询引擎的实现。
- <span name="env">**系统变量**</span>：动态表单功能使用方在其应用系统中自定义的变量，用于在指定数据库字段默认值等场景下使用，例如某字段需要存储当前登录用户的id，在设计字段时可以设置默认值为${userId}，此值会在保存表单数据时进行解析，下文中还会有具体介绍。

### 设计与实现
[设计图](https://www.processon.com/view/link/612ef399079129550ef48a7e)
[数据处理流程](https://www.kdocs.cn/view/l/chkPIom7OWHp) 

| 表说明       | 表名称       |
| ---------------- | ------------------- |
| 数据字典表       | df_dictionary       |
| 数据源表         | df_meta_data_source |
| 表元数据信息表   | df_meta_table       |
| 字段元数据信息表 | df_meta_field       |
| 表单元数据信息表 | df_meta_form        |

## 安装
### 后端引入
1. 引入maven依赖
```xml
<dependency>
    <groupId>cn.maitian</groupId>
    <artifactId>dynamic-form-spring-boot-starter</artifactId>
    <version>${dynamic-form-version}</version>
</dependency>
```
2. application.yml配置
```yaml
dynamic:
  form:
    #create 项目启动时，如果表不存在，建立动态表单系统相关表，否则不做操作
    #drop-create 项目启动时先删除原有表再新建，可以用于开发环境
    schema-update-mode: create
```


### 前端引入
1. 前端工程安装nodejs包
```
npm install maitian-dynamic-form
```
2. 前端工程建立表/字段元数据管理、表单管理、数据源管理、数据字典管理等页面。
```vue
//以表元数据管理为例 tableManage.vue
<template>
	<div>
		<table-manage />
	</div>
</template>
<script>
import {tableManage} from 'maitian-dynamic-form'
export default {
	name: 'ehrTableManage',
	components: {
		tableManage
	}
}
</script>
```
3. 配置菜单供系统管理人员使用。
## 使用方式
### 用法一
动态表单提供了默认的表单数据保存功能。如果没有复杂业务逻辑处理，可以在构建完表单后，直接配置菜单即可完成功能添加。
1. 前端工程建立一个默认的表单渲染页面(如`defaultForm.vue`)
```vue
//defaultForm.vue
<template>
	<div>
		<dynamic-form />
	</div>
</template>
<script>
import {dynamicForm} from 'maitian-dynamic-form'
export default {
	components: {
		dynamicForm
	}
}
</script>
```
2. 通过系统提供功能构建表单。（如构建了：1-员工表单employee_form; 2-编制申请表单apply_form)

3. 配置菜单。

   | 菜单名称 | 菜单地址                                      |
   | -------- | --------------------------------------------- |
   | 员工管理 | /views/defaultForm.vue?form_key=employee_form |
   | 编制申请管理 | /views/defaultForm.vue?form_key=apply_form |

### 用法二（前端待完善）
前端或服务端有时会有特殊业务逻辑需要处理，这时可以单独建立页面，把表单的`保存`按钮隐藏，用自定义的按钮代替，前端通过表单的`getdata()`方法获取数据，后端可以通过`dataService.saveTableData()`方法将数据保存至表中。下面以employee_form为例给出代码片断。
1. 前端创建employee.vue
```vue
#employee.vue
<template>
	<div>
		<dynamic-form 
			ref="emplyeeForm"
			:formKey="formKey"
			:hiddenSaveBtn="hiddenBtn" />
		<a-button @click="handleSubmit">保存</a-button>
	</div>
</template>
<script>
import {dynamicForm} from 'maitian-dynamic-form'
import {save} from '@/api'
export default {
	components: {
		dynamicForm
	},
	methods: {
		handleSubmit(){
			var formData = this.$refs['employeeForm'].getdata();		
			//TODO: 在这里处理前端逻辑后，再提交到服务端接口			
			save('/employee/save', {
				formData: formData,
				formKey: this.formKey
			})
		}
	}
	data(){
		formKey: 'employee_form',
		hiddenBtn: true,
	}
}
</script>
```
2. 服务端代码，在下边的`before`或`after`位置都可以进行业务逻辑处理
```java
//employeeController
@RestController
@RequestMapping("/employee")
public class EmployeeController{
    @Autowired
    private DataService dataService;

    @PostMapping("/save")
    public Result saveEmployee(@RequestBody FormDataCmdRequest params) {
        //before
        dataService.saveTableData(params);
        //after
        return Result.success();
    })
}
```
## 功能
### 动态表单提供的Services
使用方在将dynamic-form-spring-boot-starter引入后，下面的service会注入到spring容器中，其中DataService用于表单数据的保存，功能会随着与工作流的结合进一步丰富。
- DataService：完成对表单数据的操作
- FormService：完成对表单的相关操作
- TableService：完成对表元数据和字段元数据的操作，以及ddl的执行操作
- DataSourceService：完成对数据源的操作
- DictionaryService：完成对数据字典的操作

> 此外，还会有一些[rest接口](http://172.16.13.81:9090/swagger-ui/index.html#/)注入到spring容器以配合完成动态表单的相关功能，使用方可以不予关注。
### 系统变量处理接口 

> 这里需要根据使用方的实际应用要求进一步完善

`EnvironmentVarHandler`接口是为了解析[概念](#概念)中提到的<a href='#env'>系统变量</a>而定义的，动态表单使用方需要实现此接口，并将其注入到spring容器中，动态表单系统在将表单数据保存到数据库前调用此接口，将设置了**系统变量类型**默认值的字段赋实际值。

```java
//接口实现代码片断
@Component
public class MyEnvironmentVarHandler implements EnvironmentVarHandler {
	
	@Override
	public Object getValue(String variableName){
		if("${userId}".equals(variableName)){
			return SecurityUtils.getUser().getId();
		}
	}
    
    //其他代码
    ...
}
```
## 后续计划
- [查询引擎](http://www.zrpower.cn:8090/#/login)
- 与工作流结合
- 表单格式与控件的丰富，主从表的支持
- [前端自定义事件](https://x-render.gitee.io/form-render/advanced/watch)
- 数据源对视图和api的支持
- 权限过滤

