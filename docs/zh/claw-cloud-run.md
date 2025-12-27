# 部署到 Claw Cloud Run

本指南将帮助你在 [Claw Cloud Run](https://claw.cloud/) 上部署 `aistudio-to-api` 服务。

## 部署步骤

1. **登录**: 前往 [https://us-west-1.run.claw.cloud/](https://us-west-1.run.claw.cloud/) 并登录你的账号。
2. **创建应用**: 进入 **App Launchpad**，点击右上角的 **Create App** 按钮。
3. **配置应用**: 填写以下参数：
   - **Application Name**: 填写一个你喜欢的英文名称 (例如 `aistudio-api`)。
   - **Image**: 选择 **Public**。
   - **Image Name**: `ghcr.io/ibenzene/aistudio-to-api:latest`

   **Usage**:
   - **CPU**: `0.5`
   - **Memory**: `1G`

   **Network**:
   - **Container Port**: `7860`
   - **Public Access**: 开启 (Toggle **On**，后面的网址选项不用动)。

   **环境变量 (Environment Variables)**:

   必填参数 `API_KEYS`。其他参数选填 (参考主 README 中的 [配置](../../README_CN.md#-相关配置) 部分)。

   | Name       | Value                 | Description                                        |
   | :--------- | :-------------------- | :------------------------------------------------- |
   | `API_KEYS` | `your-secret-key-123` | **必填**。自定义你的访问密钥，多个密钥用逗号分隔。 |

4. **部署**: 点击 **Create App** 开始部署。

## 访问服务

1. 等待应用运行起来后，在应用详情页点击 **Network** 标签页。
2. 复制 **Public Address** (公网地址)。
3. 在浏览器中访问该地址。输入你设置的 `API_KEYS` 即可进入管理页面。
