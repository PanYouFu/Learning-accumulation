# Nginx安装与使用

> Nginx功能丰富，可作为HTTP服务器，也可作为反向代理服务器，邮件服务器

## Mac os 安装Nginx

### 1. 确保 Homebrew 已更新

首先，确保 Homebrew 和相关库是最新的：

```bash
brew update
```

### 2. 安装 Nginx

运行以下命令直接安装 Nginx：

```bash
brew install nginx
```

### 3. 验证 Nginx 安装

安装完成后，你可以运行以下命令来查看 Nginx 是否安装成功以及版本信息：

```bash
nginx -v
```

这会输出类似于：

```yaml
nginx version: nginx/1.xx.x
```

### 4. 启动 Nginx

安装完成后，你可以通过以下命令启动 Nginx：

```bash
brew services start nginx
```

### 5. 访问 Nginx 本地服务器

安装并启动 Nginx 后，可以在浏览器中访问 `http://localhost:8080` 查看是否正常运行。

如果 Nginx 没有自动启动或你想手动启动，可以使用以下命令：

```bash
nginx
```

或者，如果需要停止 Nginx：

```bash
nginx -s stop
```

### 如果遇到依赖问题（如 OpenSSL）

如果在安装过程中遇到与 OpenSSL 相关的问题，可以手动安装 OpenSSL：

```bash
brew install openssl@3 brew link --force openssl@3`
```

然后再重新运行安装命令。

---

## 配置 Nginx

> 可以通过编辑 Nginx 的配置文件来进行进一步的自定义配置。以下是 Nginx 配置的一些常见步骤和文件路径。

### 1. 找到 Nginx 配置文件

在通过 Homebrew 安装 Nginx 后，默认的主配置文件路径是：

```bash
/usr/local/etc/nginx/nginx.conf
```

你可以通过以下命令打开并编辑此文件：

```bash
nano /usr/local/etc/nginx/nginx.conf

# 或者使用你喜欢的编辑器，例如 `vim` 或 `code`（VSCode）：
vim /usr/local/etc/nginx/nginx.conf`
```

*ps: 如果你安装的是 Apple Silicon 版本，路径可能是 `/opt/homebrew/etc/nginx/nginx.conf`。*

### 2. 配置文件结构

Nginx 的配置文件采用模块化结构，通常包含以下部分：

- **`http` 块**：处理 HTTP 请求相关的配置，包含虚拟主机配置。
- **`server` 块**：每个 `server` 块定义一个虚拟主机的配置。
- **`location` 块**：在 `server` 块内，用于指定请求路径和处理逻辑。

### 3. 修改配置示例

#### 修改监听端口

你可以修改 Nginx 监听的端口号。例如，默认端口是 8080，你可以将其改为 80：

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        root   html;
        index  index.html index.htm;
    }
}
```

#### *配置静态文件服务器(前端常用)

如果你想配置 Nginx 来作为一个静态文件服务器，可以修改 `server` 块中的 `root` 路径，指向你想要托管的目录：

```nginx
server {
    listen 8080;
    server_name localhost;

    location / {
        root /path/to/your/static/files;
        index index.html;
    }
}
```

#### *配置反向代理(前端常用)

Nginx 常用作反向代理来代理请求到后端应用。假设你有一个 Node.js 应用运行在 `localhost:3000`，你可以配置 Nginx 将请求转发到该端口：

```nginx
server {
    listen 8080;
    server_name localhost;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 4. 检查配置文件是否正确

每次修改配置文件后，最好先检查文件的语法是否正确。你可以使用以下命令：

```bash
nginx -t
```

如果配置正确，你会看到输出类似：

```bash
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
```

### 5. 重启 Nginx

检查配置无误后，使用以下命令重启 Nginx 以应用新的配置：

```bash
brew services restart nginx
```

### 6. 日志文件

Nginx 的默认日志文件路径为：

- 访问日志：`/usr/local/var/log/nginx/access.log`
- 错误日志：`/usr/local/var/log/nginx/error.log`

可以通过查看日志来排查错误或查看访问记录。

### 总结

1. 找到并编辑配置文件：`/usr/local/etc/nginx/nginx.conf`
2. 进行必要的修改，如更改端口、设置静态文件目录或配置反向代理。
3. 使用 `nginx -t` 检查配置文件的正确性。
4. 通过重启 Nginx 应用新配置。

你可以根据需求对配置进行自定义，Nginx 提供了非常灵活的配置选项。

## 新增配置文件

> 可以 **新增配置文件**，而不必只修改现有的 `nginx.conf` 主配置文件。这种方法通常被称为**模块化配置**，通过将配置拆分成多个文件，便于管理和维护。Nginx 支持使用 `include` 指令加载多个配置文件，这也是推荐的做法。

#### 创建子配置文件

你可以在 Nginx 的配置目录中新增子配置文件。Nginx 默认的配置文件目录是 `/usr/local/etc/nginx/`（或者 `/opt/homebrew/etc/nginx/`，如果你使用 Apple Silicon）。

例如，你可以在 `nginx` 目录下创建一个名为 `my_site.conf` 的配置文件：

```bash
nano /usr/local/etc/nginx/my_site.conf`
```

在该文件中定义你的站点或其他设置。例如：

```nginx
server {
    listen 8081;
    server_name mysite.localhost;

    location / {
        root /path/to/my/site;
        index index.html;
    }
}
```

### 在主配置文件中包含新配置

在 `nginx.conf` 中添加一行 `include`，指向你新创建的配置文件：

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;

    # 引入新的站点配置
    include /usr/local/etc/nginx/my_site.conf;

    # 其他配置...
}
```

或者，如果你有多个配置文件，也可以使用通配符来包含一组文件：

```nginx
http {
    include /usr/local/etc/nginx/sites/*.conf;
}
```

这样你就可以将多个站点或模块的配置文件放在 `sites` 目录中，每个文件的名称以 `.conf` 结尾。例如，你可以将所有站点的配置文件分别命名为 `site1.conf`、`site2.conf`，并将它们都放在 `/usr/local/etc/nginx/sites/` 目录下。

### 检查配置并重启 Nginx

每次新增或修改配置文件后，执行以下命令来检查配置是否正确：

```bash
nginx -t
```

如果没有问题，可以重启 Nginx 以使新配置生效：

```bash
brew services restart nginx
```

### 示例：使用多个站点配置

假设你有两个站点，并为它们分别创建了两个配置文件：

- `/usr/local/etc/nginx/sites/site1.conf`
- `/usr/local/etc/nginx/sites/site2.conf`

配置文件内容类似于：

`site1.conf`：

```nginx
server {
    listen 8080;
    server_name site1.localhost;

    location / {
        root /path/to/site1;
        index index.html;
    }
}
```

`site2.conf`：

```nginx
server {
    listen 8081;
    server_name site2.localhost;

    location / {
        root /path/to/site2;
        index index.html;
    }
}
```

然后在 `nginx.conf` 中添加 `include` 来引入它们：

```nginx
http {
    include /usr/local/etc/nginx/sites/*.conf;
}
```

### 日志配置的拆分

如果你想为每个站点单独配置日志文件，也可以在子配置文件中定义各自的日志路径：

```nginx
server {
    listen 8080;
    server_name site1.localhost;

    access_log /path/to/site1/logs/access.log;
    error_log /path/to/site1/logs/error.log;

    location / {
        root /path/to/site1;
        index index.html;
    }
}
```

### 总结

- 你可以通过创建新的配置文件来管理不同站点或功能模块，而不仅限于修改 `nginx.conf`。
- 使用 `include` 指令引入这些子配置文件，方便维护和扩展。
- 每次修改配置后，使用 `nginx -t` 检查语法，并重启 Nginx 以应用新配置。

这种模块化配置的方式让 Nginx 配置更加灵活和可维护，尤其适合管理多个站点或复杂的服务器配置。

---

## 实践：发布一个React应用

现在有一个前端React项目，打包好后的静态资源地址为`/Users/xx/Documents/demo/demo-react/dist`。使用nginx部署，使得其他用户可以通过`http://ip:8081/`来访问该项目。

### 新增Nginx 配置

在`/usr/local/etc/nginx`路径下新增`webSite_config`文件夹，文件中新增`web-site.conf`文件

`/usr/local/etc/nginx/webShare_config/web-share.conf`

```nginx
server {
  listen 8081;
  server_name localhost;

  root /Users/wj/Documents/demo/demo-react/dist;
  index index.html;

  location / {
    try_files $uri /index.html;
  }
}
```

`try_files` 指令，可以告诉 Nginx：

1. 首先，查找与请求路径匹配的文件（如静态资源）。
2. 如果没有找到该文件，则将请求转向 `index.html`，让前端路由（React Router）处理路由逻辑。
- `$uri`：表示请求的路径（比如 `/about`）。
- `/index.html`：如果没有找到与请求路径匹配的文件，就返回 `index.html`。

在这个配置下，Nginx 会：

1. 检查是否有与请求路径对应的文件存在（如 `/about` 是否有文件存在）。
2. 如果没有找到该文件，则返回 `index.html`，让 React 的前端路由来处理实际的页面渲染。

### 添加新配置

/usr/local/etc/nginx/nginx.conf

```nginx
···
http {
    ···
    # 新增配置
    include /usr/local/etc/nginx/webShare_config/*.conf;
}
```

### 重启Nginx

```bash
brew services restart nginx
--
nginx -s reload
```

关闭nginx

```
nginx -s stop
```

重新加载nginx（重启nginx）

```
nginx -s reload
```

### 访问项目

在同一个局域网内的用户即可通过ip即可访问项目

http://10.65.106.62:8081/

<img title="" src="file:///Users/wj/Library/Application%20Support/marktext/images/2024-10-17-21-12-27-image.png" alt="" width="218">

*PS：为什么需要在同一个局域网内呢*

当你在本地（即局域网）中设置了 Nginx，并通过 `http://10.65.106.62:8081/` 访问时，这是基于你的局域网 IP 地址的，其他与该局域网连接的设备也可以通过你的 IP 地址和端口来访问。

但是，如果其他用户在**不同的网络**，例如通过互联网，或者在其他局域网中，这些用户是无法通过局域网内的 IP 地址（如 `10.65.106.62`）直接访问的，因为局域网的 IP 地址是内网地址，不是全局唯一的，不通过特殊设置是不能被外部网络访问的。

-

为了让局域网外的用户访问你的应用，你需要将应用暴露到互联网，通常有两种方式：

#### 1. 公网 IP（或外网 IP）

如果你有公网 IP，且你的电脑或服务器配置了公网访问权限，那么用户可以通过你的公网 IP 进行访问。

- **公网 IP**：这是一个在全球范围内唯一的 IP 地址，可以通过互联网访问。你可以通过网络服务提供商获取一个公网 IP，或者租用一台云服务器来托管你的应用。

#### 2. 端口映射（NAT 端口转发）

如果你的网络路由器没有直接分配公网 IP，你可以通过**端口映射**将内网服务器的 IP 和端口暴露到外网。

- **端口转发（Port Forwarding）**：这是在路由器上配置的一项功能，允许外部用户通过访问路由器的公网 IP 和指定的端口，转发到你的本地局域网设备。假设路由器有一个公网 IP（例如 `203.0.113.1`），你可以在路由器中设置端口转发，将 `203.0.113.1:8081` 转发到你的本地局域网地址 `10.65.106.62:8081`，这样外部用户就可以访问了。

#### 3. 使用代理或隧道服务（例如 ngrok）

如果你没有公网 IP 或无法配置端口转发，可以使用一些第三方的代理或隧道服务，比如 `ngrok`。这些服务允许你将本地的服务通过他们的服务器暴露到互联网上。

例如，`ngrok` 可以将你的本地 `http://localhost:8081` 暴露到一个互联网可访问的 URL 上，如 `https://random.ngrok.io`，其他用户通过这个 URL 即可访问你的本地项目。

pps:如何配置多个前端项目呢？

[使用nginx部署多个前端项目常见3种方法来实现在一台服务器上使用nginx部署多个前端项目的方法](https://juejin.cn/post/6844904183879958541)

---

## PS：

### 如何配置https

```nginx
server {
    listen 8081 ssl;
    server_name xxx;

    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;

    root /Users/wj/Documents/demo/demo-react/dist;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

#### 2. 解释配置

- **`listen 8081 ssl;`**: 指定监听的端口为 8081，并启用 HTTPS。
- **`server_name xxx;`**: 用你的域名替换 `xxx`。如果没有域名，可以使用本机的 IP 地址或 `localhost`。
- **`ssl_certificate` 和 `ssl_certificate_key`**: 配置 SSL 证书和密钥，确保 HTTPS 正常工作。你可以使用自签名证书或从 CA 机构获取的证书。
- **`root /Users/wj/Documents/demo/demo-react/dist;`**: 指定你的静态文件目录，这里是 React 项目打包后的路径 `/Users/xx/Documents/demo/demo-react/dist`。
- **`index index.html;`**: 设置默认首页为 `index.html`。
- **`try_files $uri /index.html;`**: React 是单页面应用，需要此配置来处理路由。当请求的路径不存在时，Nginx 会将请求重定向到 `index.html`。

#### 如何验证 `root` 路径

- 默认情况下，`root html;` 在 Homebrew 安装的 Nginx 中通常指向 `/usr/local/var/www/html`。
- 你可以通过 `nginx -V` 命令查看 Nginx 的安装路径。
- 可以根据需要将 `root` 修改为自定义的绝对路径，指向你想托管的文件目录。

```nginx
server {
    listen 80;
    server_name mywebapp.com;
    # 自动跳转到 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name mywebapp.com;
    # SSL/TLS 相关配置
    ssl on;
    ssl_certificate /path/to/certfile.crt;
    ssl_certificate_key /path/to/keyfile.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        root /var/www/mywebapp;
        index index.html;
    }
    // 转发路径中含有api的请求到后端服务器
    location /api/ {
        proxy_pass http://127.0.0.1:8000/;
    }
}
```

在上述示例代码中，我们创建了一个名为 mywebapp.com 的虚拟主机，并使用了 SSL/TLS 加密协议。其中，ssl_certificate 和 ssl_certificate_key 分别指定 [SSL 证书](https://cloud.tencent.com/product/symantecssl?from_column=20065&from=20065)和私钥的路径。

### 配置负载均衡

> https://docshome.gitbook.io/nginx-docs/readme/shi-yong-nginx-zuo-wei-http-fu-zai-jun-heng-qi

#### 负载均衡常用手段

##### 轮询

该算法遍历服务器节点列表，并按节点次序每轮选择一台服务器处理请求。当所有节点均被调用过一次后，该算法将从第一个节点开始重新一轮遍历。

##### 加权轮询

在加权轮询中，每个服务器会有各自的 `weight`。一般情况下，`weight` 的值越大意味着该服务器的性能越好，可以承载更多的请求。该算法中，客户端的请求按权值比例分配，当一个请求到达时，优先为其分配权值最大的服务器。

##### IP 哈希（IP hash）

`ip_hash` 依据发出请求的客户端 IP 的 hash 值来分配服务器，该算法可以保证同 IP 发出的请求映射到同一服务器，或者具有相同 hash 值的不同 IP 映射到同一服务器。

**解决服务器session不共享问题**

Session 不共享问题是说，假设用户已经登录过，此时发出的请求被分配到了 A 服务器，但 A 服务器突然宕机，用户的请求则会被转发到 B 服务器。但由于 Session 不共享，B 无法直接读取用户的登录信息来继续执行其他操作。

**灰度发布**

我们可以利用 `ip_hash`，将一部分 IP 下的请求转发到运行新版本服务的服务器，另一部分转发到旧版本服务器上，实现灰度发布。再者，如遇到文件过大导致请求超时的情况，也可以利用 `ip_hash` 进行文件的分片上传，它可以保证同客户端发出的文件切片转发到同一服务器，利于其接收切片以及后续的文件合并操作。

##### 最小连接数（Least Connections）

假设共有 N 台服务器，当有新的请求出现时，遍历服务器节点列表并选取其中连接数最小的一台服务器来响应当前请求。连接数可以理解为当前处理的请求数。

#### 使用upstream模块

PS：

RUN mkdir -p /app/.next/cache/images && chown -R nextjs:nodejs /app/.next

RUN：

这是一个 Dockerfile 指令，用于在构建镜像时执行命令。每个 RUN 指令都会在一个新的镜像层中执行，并将结果保存到镜像中。

mkdir -p /app/.next/cache/images：

mkdir 是一个命令，用于创建目录。

-p 选项表示“父目录”，即如果父目录不存在，则会自动创建所需的父目录。这样可以确保 /app/.next/cache/images 目录及其所有父目录（如 /app/.next 和 /app/.next/cache）都被创建。

/app/.next/cache/images 是要创建的目录的完整路径。

&&：

这是一个逻辑运算符，用于连接两个命令。只有当前一个命令成功执行（返回状态码为0）时，后一个命令才会执行。这可以确保只有在成功创建目录后，才会执行后面的命令。

chown -R nextjs:nodejs /app/.next：

chown 是一个命令，用于更改文件或目录的所有者和所属组。

-R 选项表示“递归”，即将更改应用于指定目录及其所有子目录和文件。

nextjs:nodejs 指定新的所有者和组。这里，nextjs 是新的用户，nodejs 是新的用户组。

- /app/.next 是要更改所有者和组的目录路径。
