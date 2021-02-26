# web 前端
## boostrap
* boostrap 表格
  * ```js
     // angular.forEach($scope.data.config_list,function (item) {
                        //     console.log(JSON.stringify(item,null,4))
                        // }) 
    ```
   ```js
      //		DefaultMultipartHttpServletRequest defaultMultipartHttpServletRequest = (DefaultMultipartHttpServletRequest)inv.getRequest();
  //		Map map = defaultMultipartHttpServletRequest.getParameterMap();
  //		Map <String,MultipartFile> mapfile = defaultMultipartHttpServletRequest.getFileMap();
  //		System.out.println(map.keySet());
  //		System.out.println(mapfile.keySet());
  //
  //		for(String key : mapfile.keySet()){
  //			System.out.println(mapfile.get(key).getOriginalFilename());
  //		}
  //		String [] values = (String[])map.get("productconfig");
  //		for(int i=0;i<values.length;i++){
  //			System.out.println(values[i]);
  //		}
  //		System.out.println("*************************8");
  //		System.out.println(defaultMultipartHttpServletRequest.getParameter("productconfig"));
  //		System.out.println(defaultMultipartHttpServletRequest.getFile("listiconfile").getOriginalFilename());
  //		System.out.println(defaultMultipartHttpServletRequest.getFile("detailiconfile").getOriginalFilename());
 ```
 	@Post("/createConfig2")
	@HttpFeatures(charset = "utf-8")
	public String addCondfig(Invocation inv ) throws IOException {
		ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
		IOUtils.copyLarge(inv.getRequest().getInputStream(),byteArrayOutputStream);
		String data = new String(byteArrayOutputStream.toByteArray(), Charsets.UTF_8);
		JSONObject pc = null;
		try {
			 pc = new JSONObject(data);
			logger.info("update_broker_list_data:{}", pc.toString());
			System.out.println(data);
			productConfigLists.add(pc);

			JSONObject ret =new JSONObject();
			ret.put("code",0);
			ret.put("data",pc);
			return  "@"+ret.toString();
		} catch (JSONException e) {
			logger.error("hit an error {}",e);
			return "{\"code\":-1,\"msg\":\"Hit an impossible json error!!\"}";
		}
	}
```