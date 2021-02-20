# 参考
- https://www.youtube.com/watch?v=BdzYdN_Zd9Q&list=WL&index=2
- https://tutorialedge.net/golang/go-grpc-beginners-tutorial/



# 何がしたい？
protoファイルからgoファイルを生成し呼び出す



# protoファイルからgoファイルを生成するためのライブラリインストール
```
go get -u github.com/golang/protobuf/protoc-gen-go
```
- (公式)https://developers.google.com/protocol-buffers/docs/gotutorial
- https://qiita.com/marnie_ms4/items/4582a1a0db363fe246f3#protoc%E3%81%AB%E3%82%88%E3%82%8Bdocument%E7%94%9F%E6%88%90
### PATHを通す
```
export PATH=$PATH:$GOPATH/bin
```
- `proto-gen-go`を使用するためにパスを通す必要がある。上記を`zshrc`等に記載する

### GOBINとは
- ※話の流れとは関係無いが調べたためメモ
- 以下コマンドで確認可能
```
go env
```

###### GOBINの設定方法
https://text.baldanders.info/golang/go-env/


# protoファイルを定義する
### option go_packageを指定する
```
option go_package = "git@github.com/kskgit/try-gRPC/chat";
```
- `option go_package` = 他ファイルがこのファイルを呼び出したいときに指定するパス（たぶん）
https://developers.google.com/protocol-buffers/docs/reference/go-generated#package
### messageを定義する
```go
message Message {
  string body = 1;
}
```
- `message` = リクエストメッセージ
- https://developers.google.com/protocol-buffers/docs/overview#simple
### serviceを定義する
```go
service ChatService {
  rpc SayHello(Message) returns (Message) {}
}
```
- `service` = 定義した`message`を取得する際の処理を記載する
- https://developers.google.com/protocol-buffers/docs/overview#services



# protoファイル呼び出す処理を定義する
```go
import (
	"fmt"
	"log"
	"net"

	"github.com/kskgit/try-gRPC/chat"
	"google.golang.org/grpc"
)

func main() {
	fmt.Println("Go gRPC Beginners Tutorial!")
	lis, err := net.Listen("tcp", fmt.Sprintf(":%d", 9000))
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	s := chat.Server{}

	grpcServer := grpc.NewServer()

	chat.RegisterChatServiceServer(grpcServer, &s)

	if err := grpcServer.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %s", err)
	}
```
- `RegisterChatServiceServer`
- `grpcServer.Serve(lis)`



# protoファイルからgoファイルを生成する
### go mod initでパッケージ名を定義する
- 現状だと上記の`"github.com/kskgit/try-gRPC/chat"`でパッケージを呼び出せないため、`go mod init PACKAGE_NAME`でパッケージを初期化しておく
- https://qiita.com/uchiko/items/64fb3020dd64cf211d4e#1-go-mod-init-%E3%81%A7%E5%88%9D%E6%9C%9F%E5%8C%96%E3%81%99%E3%82%8B
### コマンド実行
```
protoc --go_opt=module=github.com/kskgit/try-gRPC/chat --go_out=plugins=grpc:chat chat.proto
```







https://kk-river108.hatenablog.com/entry/2018/10/19/214826
https://blog.ebiiim.com/posts/grpc-with-go-mod/
https://developers.google.com/protocol-buffers/docs/reference/go-generated#invocation



