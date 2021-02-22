# 何がしたい？
公式チュートリアルを通じて簡単なgrcpサーバとクライアント間の通信を行いたい

（公式チュートリアル）https://grpc.io/docs/languages/go/basics/



# ①事前準備
### ライブラリインストール
```
go get google.golang.org/protobuf/cmd/protoc-gen-go \
         google.golang.org/grpc/cmd/protoc-gen-go-grpc
```
- (公式)https://developers.google.com/protocol-buffers/docs/gotutorial
- https://qiita.com/marnie_ms4/items/4582a1a0db363fe246f3#protoc%E3%81%AB%E3%82%88%E3%82%8Bdocument%E7%94%9F%E6%88%90
### PATHを通す
```
export PATH=$PATH:$GOPATH/bin
```
- `proto-gen-go`を使用するためにパスを通す必要があるため、上記を`zshrc`等に記載する
### go envの設定
```
export GO111MODULE=on
```
- 普通にこれ実行するだけだと毎回リセットされるため`zshrc`等に書き込んでおく
### GOBINとは
- ※話の流れとは関係無いが調べたためメモ
- 以下コマンドで確認可能
```
go env
```
###### GOBINの設定方法
https://text.baldanders.info/golang/go-env/



# ②serviceを定義する
```proto
service RouteGuide {
  // A simple RPC.
  //
  // Obtains the feature at a given position.
  //
  // A feature with an empty name is returned if there's no feature at the given
  // position.
  rpc GetFeature(Point) returns (Feature) {}

  // A server-to-client streaming RPC.
  //
  // Obtains the Features available within the given Rectangle.  Results are
  // streamed rather than returned at once (e.g. in a response message with a
  // repeated field), as the rectangle may cover a large area and contain a
  // huge number of features.
  rpc ListFeatures(Rectangle) returns (stream Feature) {}

  // A client-to-server streaming RPC.
  //
  // Accepts a stream of Points on a route being traversed, returning a
  // RouteSummary when traversal is completed.
  rpc RecordRoute(stream Point) returns (RouteSummary) {}

  // A Bidirectional streaming RPC.
  //
  // Accepts a stream of RouteNotes sent while a route is being traversed,
  // while receiving other RouteNotes (e.g. from other users).
  rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
}
```
### serviceとは
- `message`に定義したデータを取得したり更新したりするメソッド
- https://developers.google.com/protocol-buffers/docs/overview#services

### server-side streaming RPC
- `return`の引数の頭に`stream`を付けることで、サーバサイドストリーミング通信を行う
- クライアント側はサーバからの全てのレスポンスが返ってくるまで処理をストップする

### client-side streaming RPC
- メソッドの引数の頭に`stream`を付けることで、クライアントサイドストリーミング通信を行う
- サーバ側はクライアントからの全てのリクエストを受信するまでストップする

### Bidirectional streaming RPC
- `return`・メソッド引数の頭に`stream`を付けることで、双方向ストリーミング通信を行う
- 全てのリクエストを待ってレスポンスを返すことも、一つのリクエストに毎回レスポンスを返すことも出来る

# ③クライアントとサーバーのコードを生成する
```
$ protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    routeguide/route_guide.proto
```
- `--go_out = OUTPUT_PATH`
	- 必須パラーメータ
	- 出力先のディレクトリを指定する
	- `route_gide.pr.go`が出力される
		- 全てのリクエスト・レスポンスメッセージの入力・取得に関するコードが含まれている

- `--go_opt=paths=source_relative`
	- protoファイルと同じフォルダ配下に作成する
	- 指定しなかった場合protoファイルに記載している`go_package`オプションに記載したパスがそのまま記載される
	- https://developers.google.com/protocol-buffers/docs/reference/go-generated

- `--go-grpc_out = OUTPUT_PATH`
	- `route_gide_grpc.pr.go`が出力される
	- クライアントがメソッドを呼び出すためのコードやサーバがメソッドを実装するためのコードが含まれている

- `--go-grpc_opt=paths=source_relative`
	- `--go_opt=paths=source_relative`の`go-grpc_out`用



# 参考
- https://www.youtube.com/watch?v=BdzYdN_Zd9Q&list=WL&index=2
- https://tutorialedge.net/golang/go-grpc-beginners-tutorial/
- https://grpc.io/docs/languages/go/basics/









https://kk-river108.hatenablog.com/entry/2018/10/19/214826
https://blog.ebiiim.com/posts/grpc-with-go-mod/
https://developers.google.com/protocol-buffers/docs/reference/go-generated#invocation

================================

### option go_packageを指定する
```
option go_package = "git@github.com/kskgit/try-gRPC/chat";
```
- `option go_package` = 他ファイルがこのファイルを呼び出したいときに指定するパス（たぶん）
https://developers.google.com/protocol-buffers/docs/reference/go-generated#package


# ③messageを定義する
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



# ③protoファイル呼び出す処理を定義する
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
- `lis, err := net.Listen("tcp", fmt.Sprintf(":%d", 9000))` = クライアントからのリクエストを受け付けるポートを定義する
- `RegisterChatServiceServer`
- `grpcServer.Serve(lis)`
- `grpc.NewServer()`
- https://grpc.io/docs/languages/go/basics/#starting-the-server



# ④protoファイルからgoファイルを生成する
### go mod initでパッケージ名を定義する
- 現状だと上記の`"github.com/kskgit/try-gRPC/chat"`でパッケージを呼び出せないため、`go mod init PACKAGE_NAME`でパッケージを初期化しておく
- https://qiita.com/uchiko/items/64fb3020dd64cf211d4e#1-go-mod-init-%E3%81%A7%E5%88%9D%E6%9C%9F%E5%8C%96%E3%81%99%E3%82%8B
### コマンド実行
```
protoc --go_opt=module=github.com/kskgit/try-gRPC/chat --go_out=plugins=grpc:chat chat.proto
```
