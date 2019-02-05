# Testing Package

Golangの標準パッケージにある自動テストのためのもの
ベンチマークを行うこともできる。

`go test <target package>`コマンドでテストを実行できる。

[公式サイト](https://golang.org/pkg/testing/)

## サンプルコード

```go

// テスト用の関数
func TestApple() {

}

func Example(t *testing.T) {

}

// ベンチマーク用の関数
func BenchmarkFirstCase(b *testing.B) {

}

```

## 基本的な使い方

`go test`コマンドを実行するためには以下の項目を満たす必要がある。

- "\_test.go"で終わるファイルの作成。
    - このファイルはビルドの時に自動的に省かれる。
- 上のファイルの中で関数名が大文字から始まる関数がテスト内容として実行される。
    - `\*testing.T`を引数に取ることができる。
- テスト失敗を告げるにはError, Failまたは関係するメソッドを使う。
- ｀\*testing.T｀のSkipメソッドを使うとそのテストをスキップできる。

ベンチマークを行う際は以下の項目を追加で満たす必要がある。

- `go test`に`-bench`フラグを付ける
- ベンチマークを行う関数は`BenchmarkXxx(*testing.B)`という形式を取る
    - Benchmarkの後に大文字から始まる名前を付けること
    - `\*testing.B`を引数に取ることができる。
        - `\*testing.T`と同様Skip関数が使える。

## サブテスト

Runメソッドを使うことで実行される関数内でサブテスト/サブベンチマーク(以降サブテストと統一)を行うことができる

サブテストには名前を付けることができ、付けた名前は後述する-runオプションなどで利用できる。

ネストすることも可能で、Runメソッド内でRunメソッドを呼び出すとできる。

## テストの前準備/後処理

テストする前に追加の前準備/後処理が必要になるケースがある場合は、テスト用ファイル内に、`func TestMain(m *testing.M)`を定義するといい。

定義した時は各テスト関数は直接呼び出されず、代わりにTestMain関数が呼び出される。
その際はTestMain関数内で`m.Run()`の結果を`os.Exit()`に渡すこと。

```go
func TestMain(m *testing.M) {
  // コマンドライン引数を使用したい時は先にflag.Parse()を呼び出すこと
	os.Exit(m.Run())
}

```

ドキュメントだけでは使い方がよくわからなかったので、Githubで調べたところ、以下のパターンで使われていた。

1. ドキュメントどおり
    ```go
    func TestMain(m *testing.M) {
      os.Exit(m.Run);
    }
    // 変数に介するパターン
    func TestMain(m *testing.M) {
      r := m.Run()
      os.Exit(r);
    }
    ```
1. mを他の関数に渡し、その中で色々する
  [ここから拝借](https://github.com/stan/gitaly/blob/449133f786e6df001c64e20f30ea7ab13e05ba8e/internal/rubyserver/testhelper_test.go)
  ```go
  func hoge(m *testing.M) {
    defer testhelper.MustHaveNoChildProcess()

    m.Run()
  }
  ```
1. 変わったものも
    初め見た時戸惑ったが、goroutineを使っているコードもあった
    [こちらから](https://github.com/nbari/go-sandbox/blob/047e52019186083fcee1035ae0f758a40396d524/functional_tests/main_test.go)
    ```go
    func Test_main(t *testing.T) {
    	go main()
    	exitCode = <-exitCh
    }
    func TestMain(m *testing.M) {
    	m.Run()
    	// can exit because cover profile is already written
    	os.Exit(exitCode)
    }
    ```
    `go main()`はgoroutineというGolangの軽量スレッド内でmain関数を呼び出すという意味になるみたい[Goroutines](https://go-tour-jp.appspot.com/concurrency/1)
    その下の`<-exitCh`もgoroutine関連で[Channels](https://go-tour-jp.appspot.com/concurrency/2)と呼ばれる値の送受信のための型だそうだ


## -runオプション

`-run`オプションを使うとファイル内の特定の関数/サブテストのみ実行できる。

以下、ドキュメントから拝借した。

```hs
go test -run ''      # Run all tests.
go test -run Foo     # Run top-level tests matching "Foo", such as "TestFooBar".
go test -run Foo/A=  # For top-level tests matching "Foo", run subtests matching "A=".
go test -run /A=1    # For all top-level tests, run subtests matching "A=1".```

## 補足事項

補足事項として以下のものがある

- `go test ./...`を使うとプロジェクト内の全てのパッケージのテストを行える
    - ただし、バージョン1.9以降から
    - それ以前のバージョンのときは `go test $(go list ./... | grep -v /vendor/)`と入力するといい
- テストデータは"testdata"と名付けられたディレクトリに入れる
    - そのディレクトリはgoツールから無視される仕様になっている
- 標準エラー出力に何かを出力しても標準出力へ出力される
    - これは仕様で、goコマンドの標準エラーはテストをビルドした時のエラー出力に使われている
