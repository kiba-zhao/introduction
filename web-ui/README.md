# web前端开发说明 #

前端技术用于构建与用户之间交互的应用操作页面．

根据情况前端一般需要包含以下几种模式：
  * [开发模式](#开发模式)
  * [调试模式](#调试模式)
  * [部署模式](#部署模式)
  
## 开发模式 ##
开发阶段可能会出现服务接口未能直接使用，而这种情况需要前端能够在mock接口来支持开发工作的进行．

> mock接口方案： [json-server](https://github.com/typicode/json-server)

webpack config示例：

``` json
{
  devServer: {
    before(app) {
      app.use('/api', jsonServer.defaults(), jsonServer.rewriter(routes), router);
    }
  }
}
```

## 调式模式 ##
前端功能或页面开发完成，需要与后端接口交互或调试，已确保应用功能正确可用．

> 调试方案：　[webpack-dev-server](https://github.com/webpack/webpack-dev-server)

webpack config示例：

``` json
{
    devServer: {
        proxy: { '/api': { target: 'http://backend', pathRewrite: { '^/api': '' } } }
    }
}
```

## 部署模式 ##
部署运行前端项目，需要直接请求真实的后端接口．

> 部署方案： [打包nginx的docker镜像](https://registry.hub.docker.com/_/nginx)

Dockerfile示例：

``` dockerfile
FROM nginx:1.17-alpine

ADD ./dist/ /usr/share/nginx/html
ADD ./nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
```

nginx示例配置：

``` nginx
worker_processes  auto;

events {
    worker_connections  1024;
}

http {
  include             mime.types;
  default_type        application/octet-stream;

  sendfile on;

  keepalive_timeout   65;

  gzip on;
  gzip_static on;
  gzip_disable "msie6";

  gzip_vary on;
  gzip_types text/plain text/css application/javascript;

  server {
    listen            80;
    index             index.html index.htm;

    location /api/ {
      proxy_pass  http://web-gate/;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location / {
      alias /usr/share/nginx/html/;
      try_files $uri $uri/ /index.html;
    }

  }                 

}
```
