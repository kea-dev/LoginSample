# How to Build and Deploy this project.

Build like this:

```shell
docker run -it  --rm  --pid=host -v $(pwd):/app:rw --workdir /app lakruzz/lamj mvn clean package install
```
To access the container:

``` shell
docker run -it  --rm  --pid=host -v $(pwd):/app:rw --workdir /app lakruzz/lamj /bin/bash
```



