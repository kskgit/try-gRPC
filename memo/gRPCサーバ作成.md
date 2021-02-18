# 参考
- https://www.youtube.com/watch?v=BdzYdN_Zd9Q&list=WL&index=2
- https://tutorialedge.net/golang/go-grpc-beginners-tutorial/

# コンパイル用のライブラリインストール
```
go get -u github.com/golang/protobuf/protoc-gen-go
```
- (公式)https://developers.google.com/protocol-buffers/docs/gotutorial
- https://qiita.com/marnie_ms4/items/4582a1a0db363fe246f3#protoc%E3%81%AB%E3%82%88%E3%82%8Bdocument%E7%94%9F%E6%88%90

### PATHを通す
```
export PATH=$PATH:$GOPATH/bin
```
- 使用前に上記のパスを通す必要がある


# protoファイルを作成する
## protoファイルにoption go_packageを指定する
```
option go_package = "git@github.com/kskgit/try-gRPC/chat";
```

### option go_packageとは
- 他ファイルがこのファイルを呼び出したいときに指定するパス（たぶん）
https://developers.google.com/protocol-buffers/docs/reference/go-generated#package


### import pathとは
>The import path is used to determine which import statements must be generated when one .proto file imports another .proto file
- あるファイルが別のファイルを呼び出す際に使用する
- https://developers.google.com/protocol-buffers/docs/reference/go-generated#package

# protoc --go_out=plugins=grpc:chat chat.proto
## これは何をやってるの？


## GOBINとは
- 以下コマンドで確認可能
```
go env
```

### GOBINの設定方法
https://text.baldanders.info/golang/go-env/

https://kk-river108.hatenablog.com/entry/2018/10/19/214826
https://blog.ebiiim.com/posts/grpc-with-go-mod/
