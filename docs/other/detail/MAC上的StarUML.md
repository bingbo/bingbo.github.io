mac上的uml建模工具staruml

### 安装

这里直接从官网上下载安装即可，下载地址：http://staruml.io/download

### 破解

1. 在mac上找到LicenseManagerDomain.js文件
2. 编辑修改/Applications/StarUML.app/Contents/www/license/node/LicenseManagerDomain.js
    
    ```bash
    vim /Applications/StarUML.app/Contents/www/license/node/LicenseManagerDomain.js
    ```

3. 在validate函数中，添加如下信息
    
    ```node
    // edit by 0xcb
    return {
        name: "0xcb",
        product: "StarUML",
        licenseType: "vip",
        quantity: "ibingbo.com",
        licenseKey: "later equals never!"
    };
    ```
4. 打开staruml,help->Enter license

    输入name: "0xcb"和licenseKey: "later equals never!"
    
