## 1、Swagger、OpenAPI、springdoc-openapi 的关系

很多项目里大家习惯把接口文档叫做 Swagger，但现在更准确的说法是：
- OpenAPI：接口文档规范
- Swagger UI：展示接口文档的网页页面
- swagger-annotations：提供 Java 注解，例如`@Operation` 、`@Schema`
- springdoc-openapi：Spring Boot 项目中常用的 OpenAPI 文档生成工具
在 Spring Boot 项目中，一般通过 springdoc-openapi 扫描 Controller、DTO、VO、注解信息，自动生成接口文档

## 2、推荐依赖配置
### 2.1、Spring Boot 3.x 推荐依赖

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.8.17</version>
</dependency>
```

### 2.2、常用访问地址

启动项目后，一般可以访问：
```bash
http://localhost:8080/swagger-ui.html
http://localhost:8080/v3/api-docs
```
说明：
- `/swagger-ui.html`：Swagger UI 页面
- `/v3/api-docs`：OpenAPI JSON 文档

### 2.3、Spring Boot 2.x 项目说明

如果项目还是 Spring Boot 2.x ，一般不要直接使用 springdoc-openapi 2.x
Spring Boot 2.x 更适合使用 springdoc-openai 1.8.0 这一类版本

## 3、常用注解导包

新版 OpenAPI 注解一般来自下面这些包：
```Java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.Hidden;
import io.swagger.v3.oas.annotations.tags.Tag;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.ArraySchema;
import io.swagger.v3.oas.annotations.media.ExampleObject;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import io.swagger.v3.oas.annotations.security.SecurityScheme;
```
注意：
新版注解包名一般都是：
```Java
io.swagger.v3.oas.annotations
```
老版注解包名一般是：
```Java
io.swagger.annotations
```
新项目建议优先使用新版 `io.swagger.v3.oas.annotations`

## 4、常用注解快速总结

|           注解           |       作用        |               常用位置               |
| :--------------------: | :-------------: | :------------------------------: |
|         `@Tag`         | Controller 分组说明 |           Controller 类           |
|      `@Operation`      |     接口方法说明      |          Controller 方法           |
|      `@Parameter`      |     请求参数说明      |               方法参数               |
|       `@Schema`        | DTO / VO / 字段说明 |               类、字段               |
|     `@ApiResponse`     |     单个响应说明      |          Controller 方法           |
|    `@ApiResponses`     |     多个响应说明      |          Controller 方法           |
|       `@Content`       |   请求体或响应体内容类型   | `@ApiResponse`、`@RequestBody` 内部 |
|     `@ArraySchema`     |  数组或 List 类型说明  |          `@Content` 内部           |
|    `@ExampleObject`    |     请求或响应示例     |          `@Content` 内部           |
|       `@Hidden`        |     隐藏接口或字段     |             类、方法、字段              |
|   `@SecurityScheme`    |     定义认证方式      |               配置类                |
| `@SecurityRequirement` |    声明接口需要认证     |         Controller 类或方法          |
> **💡 一句话记忆：** `@Tag` 管 `Controller`，`@Operation` 管接口，`@Parameter` 管参数，`@Schema` 管 `DTO/VO` 字段，`@ApiResponse` 管响应说明。

## 5、@Tag：Controller 分组说明
### 5.1、作用

`@Tag` 用于给 Controller 分组，Swagger UI 页面会按照 `@Tag` 的 name 进行接口分类展示

### 5.2、示例

```Java
@Tag(name = "设备管理",description = "设备新增、修改、查询、删除相关接口")
@RestController
@RequestMapping("/api/devices0")
public class DeviceController{

}
```

## 6、@Operation：接口方法说明
### 6.1、作用

`@Operation` 用于描述某一个接口方法

### 6.2、示例

```Java
@Operation(summary = "查询设备详情",description = "根据设备ID查询设备的基础信息，房间信息和在线状态")
@GetMapping("/{id}")
public DeviceVO getDeviceDetail(@PathVariable Long id){
	return deviceService.getDeviceDetail(id);
}
```

### 6.3、常用属性

```Java
@Operation(
	summary = "接口简短说明",
	description = "接口详细说明",
	tags = {"设备管理"},
	operation = "getDeviceDetail"
)
```

## 7、@Parameter：请求参数说明
### 7.1、作用

`@Parameter` 用于描述请求参数，例如：
- `@PathVariable`
- `@RequestParam`
- Header 参数
- Cookie 参数

### 7.2、@PathVariable 示例

```Java
@GetMapping("/{id}")
public DeviceVO detail(
	@Parameter(description = "设备ID",required = true,example = "1001")
	@PathVariable Long id
){
	return deviceService.detail(id);
}
```

### 7.3、@RequestParam 示例

```Java
@Operation(summary = "分页查询设备列表")
@GetMapping("/page")
public PageResult<DeviceVO> pageDevices(
	@Parameter(description = "设备名称，支持模糊查询")
	@RequestParam(required = false) String deviceName,
	
	@Parameter(description = "设备状态：ONLINE在线，OFFLINE离线")
	@RequestParam(required = false) String status,
	
	@Parameter(description = "页码，从1开始",example = "1")
	@RequestParam Integer pageNo,
	
	@Parameter(description = "每页数量",example = "10")
	@RequestParam Integer pageSize
){
	return deviceService.pageDevices(deviceName,status,pageNo,pageSize);
}
```

### 7.4、常用属性

```Java
@Parameter(
	name = "deviceName",
	description = "设备名称",
	required = false,
	example = "巡检机器人001"
)
```

### 7.5、使用建议

>如果是简单的 DTO 请求体，字段说明优先卸载 DTO 的 `@Schema` 上。
>如果是路径参数、查询参数，建议用 `@Parameter` 标明含义

## 8、@Schema：DTO / VO / 字段说明
### 8.1、作用

`@Schema` 是最常用的实体说明注解。它可以用在：
- DTO 类
- VO 类
- 字段
- 枚举
- 请求参数
- 响应参数

### 8.2、DTO 示例

```Java
@Schema(description = "设备新增请求")
@Data
public class DeviceCreateRequest {

    @Schema(description = "设备名称", example = "巡检机器人001", requiredMode = Schema.RequiredMode.REQUIRED)
    private String deviceName;

    @Schema(description = "设备编号", example = "ROBOT-20260703-001")
    private String deviceCode;

    @Schema(description = "房间ID", example = "101")
    private Long roomId;

    @Schema(description = "设备状态", example = "ONLINE", allowableValues = {"ONLINE", "OFFLINE"})
    private String status;
}
```

### 8.3、VO 示例

```Java
@Schema(description = "设备详情响应")
@Data
public class DeviceVO {

    @Schema(description = "设备ID", example = "1001")
    private Long id;

    @Schema(description = "设备名称", example = "巡检机器人001")
    private String deviceName;

    @Schema(description = "设备编号", example = "ROBOT-001")
    private String deviceCode;

    @Schema(description = "设备状态", example = "ONLINE")
    private String status;
}
```

### 8.4、常用属性

```Java
@Schema(description = "字段说明")
@Schema(example = "示例值")
@Schema(hidden = true)
@Schema(allowableValues = {"ONLINE", "OFFLINE"})
@Schema(requiredMode = Schema.RequiredMode.REQUIRED)
```

### 8.5、必填字段推荐写法

不太建议使用老写法：
```Java
@Schema(required = true)
```
更推荐使用：
```Java
@Schema(requiredMode = Schema.RequiredMode.REQUIRED)
```
同时配合参数校验注解：
```Java
@Schema(description = "设备名称",requiredMode = Schema.RequiredMode.REQUIRED)
@NotBlank(message = "设备名称不能为空")
private String deviceName;
```
说明：
- `@Schema` ：主要影响接口文档
- `@NotBlank`、`@NotNull`：主要影响后端参数校验

## 9、@RequestBody：请求体说明
### 9.1、注意点

这里有两个 `@RequestBody`，很容易混淆
Spring 请求体注解：
```Java
org.springframework.web.bind.annotation.RequestBody
```
Swagger/OpenAPI 的请求体说明注解：
```Java
io.swagger.v3.oas.annotations.parameters.RequestBody
```

### 9.2、简洁推荐写法

普通项目里，推荐请求体字段说明直接写在 DTO 上：
```Java
@Operation(summary = "新增设备")
@PostMapping
public Boolean createDevice(@RequestBody DeviceCreateRequest request) {
    return deviceService.createDevice(request);
}
```
然后在 DTO 里写：
```Java
@Schema(description = "设备新增请求")
@Data
public class DeviceCreateRequest {

    @Schema(description = "设备名称", example = "巡检机器人001")
    private String deviceName;
}
```

### 9.3、完整写法

如果需要对请求体写更详细说明，可以这样写：
```Java
@Operation(summary = "新增设备")
@PostMapping
public Boolean createDevice(
        @io.swagger.v3.oas.annotations.parameters.RequestBody(
                description = "设备新增请求参数",
                required = true
        )
        @org.springframework.web.bind.annotation.RequestBody DeviceCreateRequest request
) {
    return deviceService.createDevice(request);
}
```

### 9.4、使用建议

普通 CRUD 接口没必要写得特别复杂
优先把说明写在 DTO 字段的 `@Schema` 上，接口方法保持简洁

## 10、@ApiResponse/@ApiResponse：响应说明
### 10.1、作用

用于描述接口可能返回的响应结果，例如：
- `200`：成功
- `400` ：请求参数错误
- `401`：未登录
- `403`：无权限
- `404`：资源不存在
- `500` 服务器内部异常

### 10.2、简单示例

```Java
@Operation(summary = "查询设备详情")
@ApiResponses({
        @ApiResponse(responseCode = "200", description = "查询成功"),
        @ApiResponse(responseCode = "400", description = "请求参数错误"),
        @ApiResponse(responseCode = "404", description = "设备不存在"),
        @ApiResponse(responseCode = "500", description = "服务器内部异常")
})
@GetMapping("/{id}")
public DeviceVO getDeviceDetail(@PathVariable Long id) {
    return deviceService.getDeviceDetail(id);
}
```

### 10.3、指定响应体类型

```Java
@Operation(summary = "查询设备详情")
@ApiResponse(
        responseCode = "200",
        description = "查询成功",
        content = @Content(
                mediaType = "application/json",
                schema = @Schema(implementation = DeviceVO.class)
        )
)
@GetMapping("/{id}")
public DeviceVO getDeviceDetail(@PathVariable Long id) {
    return deviceService.getDeviceDetail(id);
}
```

### 10.4、使用建议

如果项目有统一返回结构，例如：
```Java
Result<DeviceVO>
```
很多情况下框架可以自动推断返回结构
只有当文档展示不准确，泛型展示混乱，接口非常重要时，再手动补 `@ApiResponse`

## 11、@Content：请求体/响应体内容类型
### 11.1、作用

`@Content` 一般不会单独使用，通常配合：
- `@ApiResponse`
- Swagger 的 `@RequestBody`

### 11.2、示例

```Java
@ApiResponse(
        responseCode = "200",
        description = "成功",
        content = @Content(
                mediaType = "application/json",
                schema = @Schema(implementation = DeviceVO.class)
        )
)
```

### 11.3、常用 mediaType

```Java
application/json
multipart/form-data
application/octet-stream
text/plain
```

## 12、@ArraySchema：数组或List类型说明
### 12.1、作用

用于描述数组或集合返回值

### 12.2、示例

```Java
@Operation(summary = "查询设备列表")
@ApiResponse(
        responseCode = "200",
        description = "查询成功",
        content = @Content(
                mediaType = "application/json",
                array = @ArraySchema(schema = @Schema(implementation = DeviceVO.class))
        )
)
@GetMapping("/list")
public List<DeviceVO> listDevices() {
    return deviceService.listDevices();
}
```

### 12.3、使用建议

如果方法返回值已经明确是：
```Java
List<DeviceVO>
```
一般可以不用额外写 `@ArraySchema`，只有自动识别不准确时再补

## 13、@ExampleObject：请求或响应示例
### 13.1、作用

用于给接口文档添加 JSON 示例，方便前端、测试人员理解请求格式

### 13.2、示例

```Java
@Operation(summary = "新增设备")
@io.swagger.v3.oas.annotations.parameters.RequestBody(
        description = "设备新增请求",
        required = true,
        content = @Content(
                mediaType = "application/json",
                schema = @Schema(implementation = DeviceCreateRequest.class),
                examples = @ExampleObject(
                        name = "新增巡检机器人",
                        value = """
                                {
                                  "deviceName": "巡检机器人001",
                                  "deviceCode": "ROBOT-001",
                                  "roomId": 101,
                                  "status": "ONLINE"
                                }
                                """
                )
        )
)
@PostMapping
public Boolean createDevice(@RequestBody DeviceCreateRequest request) {
    return deviceService.createDevice(request);
}
```

### 13.3、使用建议

这些接口比较适合加示例：
- 新增接口
- 修改接口
- 登录接口
- 上传文件接口
- 下发任务接口
- 回调接口
- 第三方对接接口

## 14、@Hidden：隐藏接口或字段
### 14.1、隐藏接口

```Java
@Hidden
@GetMapping("/internal/test")
public String internalTest(){
	return "OK";
}
```

### 14.2、隐藏字段

```Java
@Schema(hidden = true)
private String internalToken;
```

### 14.3、使用建议

适合隐藏：
- 内部调试接口
- 临时测试接口
- 不想暴露给前端的接口
- 敏感字段
- 内部流转字段

## 15、@SecurityScheme/@SecurityRequirement：认证配置
### 15.1、作用

如果项目使用 Token、JWT、Bearer Token 等认证方式，可以在 Swagger UI 中配置认证信息

### 15.2、Bearer Token 配置示例

```Java
@Configuration
@SecurityScheme(
        name = "BearerAuth",
        type = SecuritySchemeType.HTTP,
        scheme = "bearer",
        bearerFormat = "JWT"
)
public class OpenApiConfig {

}
```
需要导入：
```Java
import io.swagger.v3.oas.annotations.security.SecurityScheme;
import io.swagger.v3.oas.annotations.enums.SecuritySchemeType;
import org.springframework.context.annotation.Configuration;
```

### 15.3、Controller 上使用

```Java
@SecurityRequirement(name = "BearerAuth")
@Tag(name = "设备管理")
@RestController
@RequestMapping("/api/devices")
public class DeviceController {

}
```

### 15.4、方法上使用

```Java
@Operation(summary = "删除设备")
@SecurityRequirement(name = "BearerAuth")
@DeleteMapping("/{id}")
public Boolean deleteDevice(@PathVariable Long id) {
    return deviceService.deleteDevice(id);
}
```

### 15.5、使用建议

如果大部分接口都需要登录，可以把 `@SecurityRequirement` 放在 Controller 类上。
如果只有少部分接口需要认证，可以放在具体接口方法上。

## 16、完整 Controller 示例

```Java
package com.example.robot.controller;

import com.example.robot.model.request.DeviceCreateRequest;
import com.example.robot.model.vo.DeviceVO;
import com.example.robot.service.DeviceService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@Tag(name = "设备管理", description = "巡检机器人设备相关接口")
@RestController
@RequestMapping("/api/devices")
@RequiredArgsConstructor
public class DeviceController {

    private final DeviceService deviceService;

    @Operation(summary = "新增设备", description = "新增一台巡检机器人设备")
    @ApiResponse(responseCode = "200", description = "新增成功")
    @PostMapping
    public Boolean createDevice(@RequestBody DeviceCreateRequest request) {
        return deviceService.createDevice(request);
    }

    @Operation(summary = "查询设备详情", description = "根据设备ID查询设备详情")
    @ApiResponses({
            @ApiResponse(
                    responseCode = "200",
                    description = "查询成功",
                    content = @Content(
                            mediaType = "application/json",
                            schema = @Schema(implementation = DeviceVO.class)
                    )
            ),
            @ApiResponse(responseCode = "404", description = "设备不存在")
    })
    @GetMapping("/{id}")
    public DeviceVO getDeviceDetail(
            @Parameter(description = "设备ID", required = true, example = "1001")
            @PathVariable Long id
    ) {
        return deviceService.getDeviceDetail(id);
    }

    @Operation(summary = "删除设备", description = "根据设备ID删除设备")
    @ApiResponses({
            @ApiResponse(responseCode = "200", description = "删除成功"),
            @ApiResponse(responseCode = "404", description = "设备不存在")
    })
    @DeleteMapping("/{id}")
    public Boolean deleteDevice(
            @Parameter(description = "设备ID", required = true, example = "1001")
            @PathVariable Long id
    ) {
        return deviceService.deleteDevice(id);
    }
}
```

## 17、完整 DTO 示例

```Java
package com.example.robot.model.request;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import lombok.Data;

@Schema(description = "设备新增请求")
@Data
public class DeviceCreateRequest {

    @Schema(description = "设备名称", example = "巡检机器人001", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "设备名称不能为空")
    private String deviceName;

    @Schema(description = "设备编号", example = "ROBOT-001", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "设备编号不能为空")
    private String deviceCode;

    @Schema(description = "房间ID", example = "101", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotNull(message = "房间ID不能为空")
    private Long roomId;

    @Schema(description = "设备状态", example = "ONLINE", allowableValues = {"ONLINE", "OFFLINE"})
    private String status;
}
```

## 18、完整 VO 示例

```Java
package com.example.robot.model.vo;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

@Schema(description = "设备详情响应")
@Data
public class DeviceVO {

    @Schema(description = "设备ID", example = "1001")
    private Long id;

    @Schema(description = "设备名称", example = "巡检机器人001")
    private String deviceName;

    @Schema(description = "设备编号", example = "ROBOT-001")
    private String deviceCode;

    @Schema(description = "房间ID", example = "101")
    private Long roomId;

    @Schema(description = "设备状态", example = "ONLINE")
    private String status;
}
```

## 19、老版 Swagger 注解和新 OpenAPI 注解对照

|    老 Swagger 注解     |              新 OpenAPI 注解               |      说明       |
| :-----------------: | :-------------------------------------: | :-----------: |
|       `@Api`        |                 `@Tag`                  | Controller 分组 |
|   `@ApiOperation`   |              `@Operation`               |     接口说明      |
|     `@ApiParam`     |              `@Parameter`               |     参数说明      |
|     `@ApiModel`     |                `@Schema`                | DTO / VO 类说明  |
| `@ApiModelProperty` |                `@Schema`                |     字段说明      |
|   `@ApiResponse`    |             `@ApiResponse`              |  响应说明，注意包名不同  |
|   `@ApiResponses`   |             `@ApiResponses`             | 多响应说明，注意包名不同  |
|    `@ApiIgnore`     | `@Hidden` / `@Parameter(hidden = true)` |    隐藏接口或参数    |
