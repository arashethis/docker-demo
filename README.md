### 制作的 Docker 容器步骤
1. 新建文件`.dockerignore`
1. 新建文件 `Dockerfile`
1. 创建 image 文件
    ```bash
    docker image build -t koa-demo .
    #或者
    docker image build -t koa-demo:0.0.1 .
    ```
    1. -t参数用来指定 image 文件的名字
    1. 冒号指定标签。如果不指定，默认的标签就是latest
    1. 最后的那个点表示 Dockerfile 文件所在的路径
1. 使用 image 文件生成容器并运行
    ```bash
    $ docker container run -p 8000:3000 -it koa-demo /bin/bash
    # 或者
    $ docker container run -p 8000:3000 -it koa-demo:0.0.1 /bin/bash
    ```
    1. `-p`参数：容器的 3000 端口映射到本机的 8000 端口
    1. `-it`参数：容器的 Shell 映射到当前的 Shell，在本机窗口输入的命令会传入容器
    1. `koa-demo:0.0.1`：image 文件的名字（如果有标签，还需要提供标签，默认是 latest 标签）
    1. `/bin/bash`：容器启动后内部第一个执行的命令。这里是启动 Bash
1. 此时可执行命令启动 koa 程序
    ```bash
    node index.js
    ```
1. 访问 http://127.0.0.1:8000，网页显示"Hello World"

    > Node 进程运行在 Docker 容器的虚拟环境里面，进程接触到的文件系统和网络接口都是虚拟的，与本机的文件系统和网络接口是隔离的，因此需要定义容器与物理机的端口映射（map）
1. 退出容器和删除容器
    1. `Ctrl + C` 停止 Node 进程，
    1. 按下 `Ctrl + D` （或者输入 exit）退出容器
    1. 用 `docker container kill` 终止容器运行。
        ```bash
        # 在本机的另一个终端窗口，查出容器的 ID
        docker container ls

        # 停止指定的容器运行
        docker container kill [containerID]
        ```
    1. 删除容器文件
        ```bash
        # 查出容器的 ID
        docker container ls --all

        # 删除指定的容器文件
        docker container rm [containerID]
        ```
    1. 使用 `docker container run `命令的`--rm`参数，在容器终止运行后自动删除容器文件
        ```bash
        docker container run --rm -p 8000:3000 -it koa-demo /bin/bash
        ```
1. 使用 CMD 命令，修改 Dockerfile 文件如下
    ```
    FROM node:12
    COPY . /app
    WORKDIR /app
    RUN npm install --registry=https://registry.npm.taobao.org
    EXPOSE 3000
    CMD node index.js
    ```
    1. RUN命令在 image 文件的构建阶段执行，执行结果都会打包进入 image 文件
    1. CMD命令则是在容器启动后执行
    1. 另外，一个 Dockerfile 可以包含多个RUN命令，但是只能有一个CMD命令
    1. 指定了CMD命令以后，`docker container run` 命令不能附加命令（比如 `/bin/bash`），否则它会覆盖 CMD 命令
        ```bash
        docker container run --rm -p 8000:3000 -it koa-demo:0.0.1
        ```