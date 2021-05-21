### Build

  docker run --rm --name slate -v C:/Users/larry/Documents/projects/slate/build:/srv/slate/build -v C:/Users/larry/Documents/projects/slate/source:/srv/slate/source slatedocs/slate
  

本地调试
./slate.sh serve

编译
./slate.sh build

./build为编译后的文件。 

将build目录copy更新到github代码仓库即可自动发布。
https://github.com/kine-api-docs/kine-api-docs.github.io.git

https://github.com/kine-api-docs/kine-api-docs.github.io
