# docker-example

## Best Practices Used
1. [An Exhaustive Guide to Writing Dockerfiles for Node.js Web Apps](https://blog.hasura.io/an-exhaustive-guide-to-writing-dockerfiles-for-node-js-web-apps-bbee6bd2f3c4/)
    * Using an appropriate base image (carbon for dev, alpine for production).
    * Optimising for Docker cache layers — placing commands in the right order so that npm install is executed only when necessary.
    * #ProTips — 1) Using COPY over ADD 2) Handling CTRL-C Kernel Signals using init flag.
2. [Docker and Node.js Best Practices](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md)
    *  [Environment Variables](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md#environment-variables)
        ```
        -e "NODE_ENV=production"
        ```
        Run with `NODE_ENV` set to `production`. This is the way you would pass in secrets and other runtime configurations to your application as well.

        In case of multiple Environment Variables, the variable info can be stored in a file and passed during runtime.
        ```
        --env-file=env_file_name
        ```
    * [Handling Kernel Signals](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md#handling-kernel-signals)

        Node.js was not designed to run as PID 1 which leads to unexpected behaviour when running inside of Docker. For example, a Node.js process running as PID 1 will not respond to `SIGINT` (`CTRL-C`) and similar signals. As of Docker 1.13, you can use the `--init` flag to wrap your Node.js process with a [lightweight init system](https://github.com/krallin/tini) that properly handles running as PID 1.

        ```
        docker run -it --init node
        ```

    * [Non-root User](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md#non-root-user)

        By default, Docker runs container as root which inside of the container can pose as a security issue. You would want to run the container as an unprivileged user wherever possible. The node images provide the `node` user for such purpose. The Docker Image can then be run with the `node` user in the following way:

        ```
        -u "node"
        ```

        Alternatively, the user can be activated in the `Dockerfile`:

        ```Dockerfile
        FROM node:6.10.3
        ...
        # At the end, set the user to use when running this image
        USER node
        ```    

    * [CMD](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md#cmd)

        When creating an image, you can bypass the `package.json`'s `start` command and bake it directly into the image itself. First off this reduces the number of processes running inside of your container. Secondly it causes exit signals such as `SIGTERM` and `SIGINT` to be received by the Node.js process instead of npm swallowing them.

        ```Dockerfile
        CMD ["node","index.js"]
        ```
3. [node-docker-good-defaults](https://github.com/BretFisher/node-docker-good-defaults)
    * **Small image and quick re-builds**. `COPY` in `package.json` and run `npm install` before `COPY` in your source code. This saves big on build time and keep container lean.
4. [How to build a smaller Docker image](https://medium.com/@gdiener/how-to-build-a-smaller-docker-image-76779e18d48a)
    * Each instructions contained in the `Dockerfile`, create a layer. Layers use space and as the levels increase, the final image size also increases. This is because the system keeps all the changes between the various statements.
    It may be a good practice to combine the various instructions in a single line. But beware, it’s important to design the Dockerfile to use the layers cache system as much as possible.
    * Alpine Linux is built around `musl`, `libc` and `busybox`. This makes it smaller and more resource efficient than traditional GNU/Linux distributions. In other words, a smaller and more secure Linux distribution.
    
## Files required
* .dockerignore
* Dockerfile

## .dockerignore
The `.dockerignore` file is a special file that can be placed within the build context directory. The build context directory is the directory that we specify at the end of a docker build command. The file itself is a simple text file that contains a list of glob patterns for files and directories to exclude from the final build image.
By leveraging the `.dockerignore` file, we can exclude files and directories we do not need within our final image.  
If a line in `.dockerignore` file starts with `#` in column 1, then this line is considered as a comment and is ignored before interpreted by the CLI.

```
node_modules
npm-debug
.vscode
.npmignore
.eslintrc.json
.gitignore
.git
*.md
```