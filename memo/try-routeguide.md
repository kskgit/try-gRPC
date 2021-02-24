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

### 公式
https://grpc.io/docs/languages/go/basics/#defining-the-service



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

### 公式
https://grpc.io/docs/languages/go/basics/#generating-client-and-server-code


# ④サーバー側を作成する
```go
flag.Parse()
lis, err := net.Listen("tcp", fmt.Sprintf("localhost:%d", *port))
if err != nil {
  log.Fatalf("failed to listen: %v", err)
}
// サーバに渡すオプションを初期化
var opts []grpc.ServerOption

// TLS通信を行う場合の処理
if *tls {
	if *certFile == "" {
		*certFile = data.Path("x509/server_cert.pem")
	}
	if *keyFile == "" {
		*keyFile = data.Path("x509/server_key.pem")
	}
	creds, err := credentials.NewServerTLSFromFile(*certFile, *keyFile)
	if err != nil {
		log.Fatalf("Failed to generate credentials %v", err)
	}
	opts = []grpc.ServerOption{grpc.Creds(creds)}
}

// NewServerでサービスやリクエストを受け付ける前の状態のサーバを生成する
// https://pkg.go.dev/google.golang.org/grpc#NewServer
// ...でスライスを展開している
// https://qiita.com/hnakamur/items/c3560a4b780487ef6065
grpcServer := grpc.NewServer(opts...)
// gRPC serverへサービスとその実装を登録する
// 必ずServe実行前に行う必要がある
// protoから生成されたコードによって
// https://pkg.go.dev/google.golang.org/grpc#Server.RegisterService
pb.RegisterRouteGuideServer(grpcServer, newServer())

// listenerを指定してサーバをスタートする
// https://pkg.go.dev/google.golang.org/grpc#Server.Serve
grpcServer.Serve(lis)
```
