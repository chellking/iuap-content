# 前后端数据交互

## 前端提交数据

1）	检查前端提交请求的路径和后端的服务URL是否对应正确

前端： ctrlBathPath = ctx+'/product';

2）	前端ajax方式提交请求示例

```
$.ajax({
	type : 'GET',
	url : ctrlBathPath+'/page',
	data : queryData,
	dataType : 'json',
	success : function(result) {
		var data = result.data;
		if(data!=null){
			viewModel.mainDataTable.setSimpleData(data.content);
			viewModel.mainDataTable.totalPages(data.totalPages);
		} else {
		}
	}
});
```

## 服务端提供Rest服务路径
1）	分页请求的后端URL映射检查

例如：查询分页请求的后端url为/page，需要和前端的ajax请求对应。

@RequestMapping(value="/page", method= RequestMethod.GET)

2）	后端Controller类的映射路径

@RequestMapping(value = "/product")


