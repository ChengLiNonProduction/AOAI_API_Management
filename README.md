# Rate Limit 与资源管理

如果使用 Azure OpenAI 的企业级用户发现单一资源的 TPM (Token Per Minute) 无法满足他们的需求，可以通过创建多个 AOAI 资源的方式，并使用 Azure API Management 管理多个终结点，将工作负载分散到多个资源中，从而提高整体 TPM。

以下是具体的实现步骤。

## 创建多个 AOAI 资源
1. 在不同区域创建多个 AOAI 资源，以两个为例
  > ![AOAI resource](./img/AOAI%20resource.png)
2. 在 AOAI 资源概览页面，获取相应的 Endpoint, Key
  > ![key & endpoint](./img/key%20endpoint.png)

## 创建 API 管理服务
1. 在 Azure 门户中选择 **创建资源**，搜索 **API Management** 并点击 **Create**
  > ![create APIM](./img/create%20APIM.png)
2. 在 **创建 API 管理** 页，输入以下设置
  - **Region:** 从可用的 API 管理服务位置选择附近的地理区域
  - **Resource name:** API 管理服务的名称，需全球唯一，创建后不能更改
  - **Administrator email:** 填入您自己的邮箱，APIM 服务创建完成之后将发邮件通知
  - **Organization name:** 填入您或组织的名称，通知邮件的标题中将使用此名称
  - **Pricing tier:** 选择 Developer
  > ![APIM setting](./img/APIM%20setting.png)
3. 点击 **Review + Create**

## 导入 API
1. 导航到 API 管理实例，选择 **APIs** -> **Add API** -> **HTTP** API
  > ![HTTP API](./img/HTTP%20API.png)
2. 在 **创建 HTTP API** 窗口中，选择 **Full**
  - **Products:** 选择 Unlimited
  - **Gateways:** 选择 Managed
  > ![API setting](./img/create%20API%20Full.png)
3. 点击 **Create**

## 向 HTTP API 添加操作
1. 选择上一步中创建的API，点击 **Add Operation** -> **Frontend** 添加操作
  - **Display name：** 显示在开发人员门户中的名称
  - **URL:** 方法选择 POST，路径填入 ```/chat/completions?api-version={api-version}```
  > ![add operation](./img/add%20operation.png)
2. 继续在 **Frontend** 中选择 **Response** 添加响应
  - 200 OK
  - 400 Bad Request
  - 500 Internal Server Error
3. 在当前窗口的 **Representation** 下添加表现形式
  - **Content Type:**
    ```
    application/json
    ```
  - **Sample:**  
    ```
    {"messages":[{"role":"system","content":"You are a helpful assistant."},{"role":"user","content":"Hi"}],"temperature":0.9,"stream":true}
    ```
  - **Definition:**
    ```
    ChatCompletionsPostRequest-1
    ```
  > ![add representation](./img/add%20representation1.png)
4. 最后在 **Template** 中填入 API version
  > ![template](./img/template.png)

## 创建和发布产品
