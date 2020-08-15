# Spring 高级技术
## multipart上传文件
### 自定义DispatcherServlet配置
* 在AbstractAnnotationConfig-DispatcherServletInitializer将DispatcherServlet注册到Servlet容器中时，会调用customizeRegistration,并将Servlet注册后得到的Registration.Dynamic传递进来，重载customizeRegistration方法，实现对DispatcherServlet的自定义配置   
    ```
            //  multipart  设置临时存储目录 <1>
            @Override
            protected void customizeRegistration(Dynamic registration) {
            registration.setMultipartConfig(
            //        new MultipartConfigElement("/tmp/spittr/uploads", 2097152, 4194304, 0));
            new MultipartConfigElement("/home/bliss/uploads", 2097152, 4194304, 0));
            }
    ```
### 表单请求形式
* 一般数据  
  以&为分隔符的多个name-value对
  ```
  firstName=Charless&lastName=Xavier&email=professor%40xmen.org&username=professorx&password=letmein01
  ```
* multipart形式   
   multipart请求体
    ```java
    -----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
    Content-Disposition: form-data; name="firstName"

    Charles
    -----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
    Content-Disposition: form-data; name="lastName"

    Xavier
    -----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
    Content-Disposition: form-data; name="email"

    charles@xmen.com
    -----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
    "Content-Disposition: form-data; name="username"

    professorx
    -----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
    Content-Disposition: form-data; name="password"

    letmeinO1
    -----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
    Content-Disposition: form-data; name="profilePicture"; filename="me.jpg"

    Content-Type: image/jpeg

    [[ Binary image data goes here ]]
    -----------WebKitFormBoundaryqgkaBn8lHJCuNmiW--
    ```
### multipart前端接口
enctype 表示multipart数据形式提交表单，不是以普通表单数据形式提交
```html
     <form method="POST" th:object="${spitterForm}" enctype="multipart/form-data">
        <div class="errors" th:if="${#fields.hasErrors('*')}">
          <ul>
            <li th:each="err : ${#fields.errors('*')}" 
                th:text="${err}">Input is incorrect</li>
          </ul>
        </div>
        <label th:class="${#fields.hasErrors('firstName')}? 'error'">First Name</label>: 
          <input type="text" th:field="*{firstName}"  
                 th:class="${#fields.hasErrors('firstName')}? 'error'" /><br/>
  
        <label th:class="${#fields.hasErrors('lastName')}? 'error'">Last Name</label>: 
          <input type="text" th:field="*{lastName}"
                 th:class="${#fields.hasErrors('lastName')}? 'error'" /><br/>
  
        <label th:class="${#fields.hasErrors('email')}? 'error'">Email</label>: 
          <input type="text" th:field="*{email}"
                 th:class="${#fields.hasErrors('email')}? 'error'" /><br/>
  
        <label th:class="${#fields.hasErrors('username')}? 'error'">Username</label>: 
          <input type="text" th:field="*{username}"
                 th:class="${#fields.hasErrors('username')}? 'error'" /><br/>
  
        <label th:class="${#fields.hasErrors('password')}? 'error'">Password</label>: 
          <input type="password" th:field="*{password}"  
                 th:class="${#fields.hasErrors('password')}? 'error'" /><br/>

        <label>Profile Picture</label>:
          <input type="file"
                 name="profilePicture"
                 accept="image/jpeg,image/png,image/gif" /><br/>

        <input type="submit" value="Register" />
      </form>
```
### 配置multipart解析器
DispatcherServlet 并没有实现任何解析 multipart 请求数据的功能。它将该任务委托给了 Spring 中  MultipartResolver 策略接口的实现，通过这个实现类来解析 multipart 请求中的内容          
```java
@Bean
public MultipartResolver multipartResolver() throw IOException(){
    return new StandardServletMultipartResolver();
}
```
### 处理multipart请求
定义MultipartFile数据
```java
private MultipartFile profilePicture;

public MultipartFile getProfilePicture() {
    return profilePicture;
  }
```
multipart接口接受文件到本地
```java
//  multipart  接收Multipart <3>
  @Transactional
  @RequestMapping(value="/register", method=POST)
  public String processRegistration(
          @Valid SpitterForm spitterForm,
          Errors errors,
          RedirectAttributes model) throws IllegalStateException, IOException {
    
    if (errors.hasErrors()) {
      return "registerForm";
    }
    Spitter spitter = spitterForm.toSpitter();
//    spitterRepository.save(spitter);
    spitterRepository.saveSpitter(spitter);
    MultipartFile profilePicture = spitterForm.getProfilePicture();
    profilePicture.transferTo(new File("/home/bliss/uploads/spitter/" + spitter.getUsername() + ".jpg"));

//通过addFlashAttribute 结合redirect 可以将processRegistration中的model  中的属性共享给 showSpitterProfile中的model
    model.addFlashAttribute(spitter);
//redirect 防止表单重复提交
    return "redirect:/spitter/" + spitter.getUsername();
//    return "redirect:/spitter/{username}";
  }
```
### 异常处理
将异常映射为HTTP状态码
```java
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(
    @PathVariable("spittleId") long spittleId, 
    Model model) {
  Spittle spittle = spittleRepository.findOne(spittleId);
  if (spittle == null) {
    throw new SpittleNotFoundException();
  }
  model.addAttribute(spittle);
  return "spittle";
}

@ResponseStatus(value=HttpStatus.NOT_FOUND, reason="Spittle Not Found")
public class SpittleNotFoundException extends RuntimeException {
}
```
@ExceptionHandler 处理单个Controller类所有异常
不仅要包括状态码，还要包含所产生的错误
```java
public class DuplicateSpittleException extends RuntimeException {
}
```
```java
  @RequestMapping(method=RequestMethod.POST)
  public String saveSpittle(SpittleForm form, Model model) {
    try {
//      spittleRepository.save()
      spittleRepository.saveSpittle(new Spittle(null, form.getMessage(), new Date(),
          form.getLongitude(), form.getLatitude()));
      return "redirect:/spittles";
    } catch (DuplicateSpittleException e) {
      return "error/duplicate";
    }
  }
```

```
//不用写try-catach
@RequestMapping(method=RequestMethod.POST)
public String saveSpittle(SpittleForm form, Model model) {
  spittleRepository.save(new Spittle(null, form.getMessage(), new Date(), 
    form.getLongitude(), form.getLatitude()));
  return "redirect:/spittles";
}
  //  它能处理同一个控制器中所有处理器方法所抛出的异常  DuplicateSpittleException,不用在每一个可能抛出 DuplicateSpittleException 的方法中添加异常处理代码
  @ExceptionHandler(DuplicateSpittleException.class)
  public String handleNotFound() {
    return "error/duplicate";
  }
```
全局多个Controller类异常处理@ControllerAdvie+ExceptionHandler
```java
//ControlAdvice 注解的类，包含
// @ExceptionHnadler
//@InitBinder
// @ModelAtribute
// 注解，并且这些注解所标注的方法会应用到整个应用程序所有控制器带有@RequestMapping注解的方法上
@ControllerAdvice
public class AppWideExceptionHandler {

//  任何控制器方法抛出了DuplicateSpittleException 都将调用方法
  @ExceptionHandler(DuplicateSpittleException.class)
  public String handleNotFound() {
    return "error/duplicate";
  }

}
```
### 跨重定向传输数据
一般，当一个requestmaping方法执行后，该方法所指定的模型数据会复制到请求中，作为请求中的属性，请求会转发forward到视图上进行渲染。控制器方法和视图处理的是同一个请求，所以在转发过程中，请求属性（model数据）会保存。
但redirect，当一个requestmaping方法执行后的结果为redirect时，此时原始请求结束，会发起一个新的请求到新的requestmaping方法，这个请求属性没有任何模型数据。
* url模板重定向
  ```java
    @RequestMapping(value="/register", method=POST)
  public String processRegistration(
      @Valid Spitter spitter, 
      Errors errors) {
    if (errors.hasErrors()) {
      return "registerForm";
    }

    spitterRepository.save(spitter);
    return "redirect:/spitter/" + spitter.getUsername();
    <!-- 直接在url中传递参数 -->
  }
  
  @RequestMapping(value="/{username}", method=GET)
  public String showSpitterProfile(@PathVariable String username, Model model) {
    Spitter spitter = spitterRepository.findByUsername(username);
    model.addAttribute(spitter);
    return "profile";
  }   
  ```

  ```java
    @RequestMapping(value="/register", method=POST)
  public String processRegistration(
      @Valid Spitter spitter, 
      Errors errors,
      Model model) {
    if (errors.hasErrors()) {
      return "registerForm";
    }

    spitterRepository.save(spitter);
    model.addAttribute("username",spitter.getUsername());
    model.addAttribute("spitterId",spitter.getId());
    //    return "redirect:/spitter/" + spitter.getUsername();
        return "redirect:/spitter/{username}";
        <!-- 将username作为占位符填充到url模板，不是直接连接到重定向string中，不安全的字符都会进行转义处理 -->
    }

    @RequestMapping(value="/{username}", method=GET)
    public String showSpitterProfile(@PathVariable String username, Model model) {
        Spitter spitter = spitterRepository.findByUsername(username);
        model.addAttribute(spitter);
        return "profile";
    }
  ```
  
* flash  RedirectAttribute属性    
  flash属性会携带这些数据直到下一次请求，才会消失
    ```java
    @RequestMapping(value="/register", method=POST)
    public String processRegistration(
        Spitter spitter, RedirectAttribute model) {
    spitterRespository.save(spitter);
    model.addAttribute("username", spitter.getUsername());
    model.addFlashAttribute("spitter", spitter);
    return "redirect:/spitter/{username}";
    }
    @RequestMapping(value="/{username}", method=GET)
    public String showSpitterProfile(
        @PathVariable String username, Model model) {
    if (!model.containsAttribute("spitter")) {
        model.addAttribute(spitterRepository.findByUsername(username));
    }
    return "profile";
    }
    ```
    