# docker-example

## Files required
* .dockerignore
* Dockerfile

## .dockerignore
The `.dockerignore` file is a special file that can be placed within the build context directory. The build context directory is the directory that we specify at the end of a docker build command. The file itself is a simple text file that contains a list of glob patterns for files and directories to exclude from the final build image.
By leveraging the `.dockerignore` file, we can exclude files and directories we do not need within our final image.  
If a line in `.dockerignore` file starts with `#` in column 1, then this line is considered as a comment and is ignored before interpreted by the CLI.

```
.git
node_modules
npm-debug
.vscode
```

