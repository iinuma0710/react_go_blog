# 付録A 駆け足で学ぶ Go の基礎

## A.1 Hello World
まずは，Go のプロジェクトを作成します．

```bash
$ go mod init helloworld
```

GitHub で外部に公開する場合には ```github.com/(アカウント名)/(リポジトリ名)``` を指定します．
次に ```main.go``` を作成します．

```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("Hello World")
}
```

コンパイルして実行すると，次のように表示されます．

```bash
$ go build
$ ./helloworld 
Hello World
```

Go のプログラムには次のような決まり事があります．

- 最初にパッケージ名を示す
- ```main``` パッケージは必須である
- ```main``` パッケージの ```main()``` 関数から実行が開始される

## A.2 リテラル・変数宣言
リテラルには整数や浮動小数点数，ブール型，文字列，```nil``` などがあります．
変数宣言には何通りかの方法があります．

```go
// 型を明示する変数宣言
var num1 int = 123

// 右辺の値から型を推論する場合
var num2 = 123
```

関数内で変数の代入と宣言を同時に行う場合には，次のように変数を宣言できます．

```go
num3 := 123
```

Go では宣言して使われていない変数があるとエラーとなります．

### A.2.1 シャドーイング
Go では，コードブロックの内部で変数を再宣言できます．
これをシャドーイングと呼び，コードブロック外ですでに定義されている変数の値には影響を及ぼしません．

## A.3 名前
Go には次のような命名規則があります．

- パッケージ名はできるだけ1単語ですべて小文字で付ける
- パッケージレベルの関数や変数，型は CamelCase か camelCase を用いる
    - CamelCase ならパブリックな要素，camelCase ならプライベートな要素になる
- 関数内でのみ参照する変数はできるだけ短い小文字で命名し，1文字でも良い
- エラーは ```err```，コンテキストは ```ctx```

## A.4 コメント
コメントは行コメント ```//``` か ブロックコメント ```/* */``` を使います．
空行無しで関数館宣言の直前に行コメントを付与すると，それらの要素の説明としてドキュメント内で利用されます．

## A.5 型と変換
Go では，リテラルの状態では型を持たず，変数に代入された時に型が決まります．
型を指定しない場合には ```int``` となります．
また，暗黙的な型変換はなく，同じ数値でも明示的に型変換しなければ他の変数には代入できません．

| 数値の種類 | 8ビット | 16ビット | 32ビット | 64ビット | ハードウェア依存 | 型省略時 |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 符号あり | int8 | int16 | int32 | int64 | int | int |
| 符号あり | uint8 | uint16 | uint32 | uint64 | uint |  |
| 符号あり |  |  | float32 | float64 |  | float64 |

```go
var i int = 123
// 数値同志の変換は、かっこでくくり型を前置します
var f float64 = float64(i)
// 64 ビット OS で 64 ビットの int と、int64 も明示的な変換が必要です
var i64 int64 = int64(i)
// bool への変換は比較演算子を使います
var b bool = i != 0
```

プリミティブ型同士の変換は次のように行います．
文字列からの変換は ```strconv``` の ```Parse``` がつく関数，文字列への変換では ```Format``` がつく関数を利用します．

| 変換元 | 整数型 | 浮動小数点数型 | ブール型 | 文字列 |
| :--: | :--: | :--: | :--: | :--: |
| 整数型 | 型変換 | 型変換 | 比較演算子 | FormatInt()，FormatUint() |
| 浮動小数点数型 | 型変換 | 型変換 | 比較演算子 | FormatFloat() |
| ブール型 |  |  |  | FormatBool() |
| 文字列型 | ParseInt()，ParseUint() | ParseFloat() | ParseBool() |  |

```go
// 文字列との変換は strconv パッケージを利用
in := 12345
// strconv の数値入力は int64, uint64, float64 なので
// それ以外の変数を使う時は型変換が必要
s := strconv.FormatInt(int64(in), 10)
// s = "12345"

// Parse 系はエラー変換失敗時にエラーを返す
// 成功時の err は nil
f, err := strconv.ParseFloat("12.3456", 64)
// f = 12.3456
// err = nil
```

## A.6 ポインタ
変数を格納したメモリ上のアドレスを扱う機能をポインタ，ポインタの指す実体を値と呼びます．

- 変数のポインタ型には ```*``` を前置する
- 既存の変数のポインタを取り出すには ```&``` を用いる
- ポインタから参照先の値を取り出す (デリファレンス) には ```*``` を用いる

```go
// ポインターの参照先となる普通の変数
var i int = 10
// ポインターを格納する変数（デフォルト値は nil）
var p *int
// p には i のアドレスが入る
p = &i
// p 経由で i の値を取得
fmt.Println(*p)
```

ポインタ型は ```nil``` で初期化され，```nil``` の参照先を取り出そうとすると，プログラムがパニックを起こして終了します．
リテラルにはポインタが割り当てられません．
一方，関数やスライス，マップは参照型と呼ばれ，```*``` を付けていなくてもポインタと同様の動作をします．
これらのコピーを行う場合には，その参照のみがコピーされます．

## A.7 ゼロ値
値を設定せずに宣言だけした変数は，「ゼロ値」で初期化されます．

| 型 | ゼロ値 |
| :--: | :--: |
| 数値 | 0 |
| ブール型 | false |
| 文字列 | 空文字列 |
| ポインタ，インタフェース | nil |

このように，ゼロ値で変数を初期化することにより，バグやセキュリティホールを予防しています．

## A.8 スライス
Go には配列もありますが，スライスのほうがよく用いられます．

|  | 型名 | 要素数 | リテラル |
| :--: | :--: | :--: | :--: |
| 配列 | 要素の型の前にブラケットを前置 | ブラケットの中に記述 | ブレースを後置してインスタンスを作成 |
| スライス | 要素の型の前にブラケットを前置 | 不定 | ブレースを後置でいてインスタンスを作成 |

スライスは，参照している配列と配列中の要素の長さ，及び容量の限界値からなるデータで，```appned()``` 関数で要素を追加することが可能です．
要素の長さが容量の限界値に達すると，自動的に2倍のサイズのメモリを確保して配列を載せ替えます．

```go
/* 配列 */
// 要素数 3 の整数の配列
var nums [3]int = [3]int{1, 2, 3}
// 要素の値を取り出して表示
fmt.Println(nums[0])
// 長さを取得
fmt.Println(len(nums))


/* スライス */
var nums1 []int

// 1, 2, 3 の要素を持つスライスを作成して代入
nums2 := []int{1, 2, 3}

// あるいは既存の配列やスライスからも範囲アクセスでスライス作成
nums3 := nums[0:2] // 配列から
nums4 := nums2[1:3] // スライスから

// 配列と同じようにブラケットで要素取得可能，範囲外アクセスはパニック
fmt.Println(nums2[1]) // 2

// 要素の割り当ても可能
nums2[0] = 100

// 長さも取得可能
fmt.Println(len(nums2)) // 3

// スライスに要素を追加，再代入が必要
nums2 = append(nums2, 4)
```

Go では，スライスの範囲外にアクセスしようとするとパニックとなってプログラムが終了する設計になっています．

## A.9 マップ
Go のマップは，

- 型は ```map[キーの型]値の型```
- ブレースを付けるか ```make()``` 関数で初期化する
- 値の割り当てと取得はブラケットを利用

```go
// HTTP ステータスを格納
hs := map[int]string{
    200: "OK",
    404: "Not Found",
}
// make で作る
authors := make(map[string][]string)

// ブラケットで要素アクセス，代入
authors["Go"] = []string{"Robert Griesemer", "Rob Pike", "Ken Thompson"}

// データ取得
status := hs[200]
fmt.Println(status)
// "OK"

// 存在しない要素にアクセスするとゼロ値
fmt.Println(hs[0]) // panic

// あるかどうかの情報も一緒に取得
status, ok := hs[304]
// status = ""
// ok = false
```

## A.10 制御構文
### A.10.1 if
Go では，条件節は必ずブール型でなければなりません．

```go
// if 文 /if else/else の基本的な書き方
if statusCode == 200 {
    fmt.Println("no error")
} else if statusCode < 500 {
    fmt.Println("client error")
} else {
    fmt.Println("server error")
}
```

また，条件節で変数を宣言して，ブロック内で利用することもできます．

```go
// データ取得とチェックを同時に行う
if result, ok := cache[input]; ok {
    fmt.Println("cached value", result)
}
```


### A.10.2 for
```range``` を用いてスライスやマップの全要素を取り出すループを作成できます．
スライスではインデックスと値が，マップではキーと値が取り出されます．

```go
// スライスやマップの書く要素に対してループ
scketches := []string{"Dead Parrot", "Killer joke", "Spanish Inquisition", "Spam"}
for i, s := range scketches {
    fmt.Println(i, s)
}
```

変数を1つだけ書くと，スライスのインデックス，あるいはマップのキーを取り出せます．
また，ブランク識別子 ```_``` を用いて．値のみを取り出すこともできます．

```go
// 1 変数だけ書けばインデックス飲みを受け取れる
for i := range scketches {
    fmt.Println(i)
}

// ブランク識別子でインデックスを読み飛ばして値だけを使う
for _, s := range scketches {
    fmt.Println(s)
}
```

```break``` でループを抜けたり，```continue``` でループの先頭に戻ることもできます．

```go
for _, s := range scketches {
    // もしスケッチ名が K から始まっていたら読み飛ばす
    if strings.HasPrefix(s, "K") {
        continue
    }

    // もしスケッチ名が n で終わっていたらループ終了
    if strings.HasSuffix(s, "n") {
        break
    }
}
```

ブール型の要素を1つだけ持つループを記述することも可能で，条件式を省略すると ```break`` で抜けるまで回り続ける無限ループとなります．

```go
counter := 0
for counter < 10 {
    fmt.Println(" ブール値が true の間回り続けるループ ")
    counter += 1
}

end := time.Now().Add(time.Second)
for {
    fmt.Println("break や return で抜けないと終わらないループ ")
    if end.Before(time.Now()) {
        break
    }
}
```

C 言語の ```for``` 文に似たループも記述可能です．

```go
for i := 0; i < 10; i++ {
    fmt.Println("10 回繰り返す ")
}
```

### A.10.3 switch
式を ```switch``` に書き，```case``` 節に式の値ごとの処理を記述します．

```go
switch s {
case "running":
    fmt.Println(" 実行中 ")
case "stop":
    fmt.Println(" 停止中 ")
default:
    fmt.Println(" その他 ")
}
```

他の言語とは異なり，```fallthrough``` を記述しない限り，最初にマッチした ```case``` 節の処理が完了すると ```switch``` 文を抜けます．

```go
switch s {
case "running":
    fallthrough // "running の時も実行中と表示される "
case "run":
    fmt.Println(" 実行中 ")
case "stop":
    fmt.Println(" 停止中 ")
default:
    fmt.Println(" その他 ")
}
```

## A.11 関数
Go の関数では，返り値が必要なのに値を返さなかったり，返り値の数や型が間違っているとエラーになります．
同じ型の返り値が続く場合は型を省略でき，後ろの型が前にもあるものとして扱われます．

```go
func calc(x, y int) int {
    return x + y
}
```

返り値に名前を付けておいて，同名の変数に値を代入し ```return``` を実行すると，その変数が返り値として返されます．
返り値が複数個になったり，名前を付けたりする場合には，返り値をカッコで括ります．

```go
func calcAge(y int, m time.Month, d int) (age int, err error) {
    b := time.Date(y, m, d, 0, 0, 0, 0, time.Local)
    n := time.Now()
    if b.After(n) {
        err = errors.New(" 誕生日が未来です ")
        return
    }
    for {
        b = time.Date(y+age+1, m, d, 0, 0, 0, 0, time.Local)
        if b.After(n) {
            return
        }
        age++
    }
}
```

Go の名前付き関数はパッケージレベルでしか定義できませんが，無名関数であれば関数内で定義したり，コールバック関数として他の関数の引数にすることもできます．
無名関数を格納する変数の型は ```func(int, int) int``` となります．(引数や返り値名は入れても省略しても OK です．)

```go
m := func(x, y int) int {
return x * y
}

// def-func5
// 名前付きで定義した関数以外に、無名関数として定義したものを渡せる
doCalc(10, 20, m)

// その場で定義した関数も渡せる
doCalc(10, 10, func(x, y int) int {
    return x * y
})
// def-func5

// 関数を受け取る関数
func doCalc(x, y int, f func(int, int) int) {
    fmt.Println()
}
```

### A.11.1 defer で後処理の関数の実行予約
Python の ```with``` 句のように，スコープを抜けるタイミングで後処理を予約実行してくれる仕組みがあります．

```go
// ファイルを開く
f, err := os.Create("sample.txt")
if err != nil {
    fmt.Println("err", err)
    return
}
// この関数のスコープを抜けたら自動でファイルをクローズ
defer f.Close()
io.WriteString(f, "hello world")
```

## A.12 エラー処理
Go に例外処理はなく，関数の返り値 ```err``` が ```nil``` か否かでエラーを判定します．

- 失敗する可能性のある関数の返り値を ```error``` 型を返す
- 成功時には ```nil``` を，失敗時にはそこに詳細なエラーを割り当てて返す
- 関数を呼び出した後は ```if err != nil``` でエラーをチェックし，追加の情報をラップしたり，そのまま return で呼び出し元に返したりする

## A.13 構造体
構造体はいくつかのデータをまとめて扱えるようにしたもので，```type``` キーワード，構造体名，```struct``` キーワードで作成し，ブレース内にメンバとなる変数 (フィールド) を列挙します．
外部パッケージから利用する場合には，構造体のフィールドも大文字とします．
構造体内にはメソッドを定義することはできません．

```go
type Book struct {
    Title       string
    Author      string
    Publisher   string
    ReleasedAt  time.Time
    ISBN        string
}
```

構造体は型であってメモリ上に実体を持たないので，データを格納するためにはインスタンスを作成する必要があります．

```go
// インスタンス作成（フィールドはすべてゼロ値に初期化）
var b Book

// フィールドを初期化しながらインスタンス作成
b2 := Book{
    Title: "Twisted Network Programming Essentials",
}

// フィールドを初期化しながらインスタンス作成，変数にはポインターを格納
b3 := &Book{
    Title: " カンフーマック――猛獣を飼いならす 310 の技 ",
}
```

以下に示すように，JSON タグを用いてデータをダンプしたり，フィールドにマッピングしたりする方法は，業務アプリケーションの実装でよく用いられます．

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "os"
    "time"
)

type Book struct {
    Title       string      `json:"title"`
    Author      string      `json:"author"`
    Publisher   string      `json:"publisher"`
    ReleasedAt  time.Time   `json:"release_at"`
    ISBN        string      `json:"isbn"`
}

func main() {
    f, err := os.Open("book.json")
    if err != nil {
        log.Fatal("file open error: ", err)
    }
    d := json.NewDecoder(f)
    var b Book
    d.Decode(&b)
    fmt.Println(b)
}
```

## A.14 ライブラリのインポート
Go では ```import``` 文を用いてライブラリをインポートします．
中央集権的なライブラリのリポジトリは存在せず，GitHub や GitLab などのサーバで公開されているライブラリを取得します．

```bash
$ go get github.com/rs/zerolog
go: downloading github.com/rs/zerolog v1.26.1
go: added github.com/rs/zerolog v1.26.1
```

## A.15 ウェブアプリケーション
Go のウェブアプリケーションは次のような構造になっています．

```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "os"
    "github.com/rs/zerolog/log"
)

func main() {
    // ①
    http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, World")
        log.Info().Msg("receive hello world request")
    })

    // ②
    fmt.Println("Start listening at :8080")

    // ③
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        // ④
        fmt.Fprintf(os.Stderr, "")
        io.WriteString(os.Stderr, err.Error())
        os.Exit(1)
    }
}
```

1. URLごとの処理を登録
2. 画面にテキストを表示
3. ```http``` パッケージの ```ListenAndServe()``` 関数を使ってサーバーを起動
4. ポートがすでに使われている場合のエラー処理

次のコマンドでサーバが起動します．

```bash
$ go run main.go

# 実行ファイルを作成してからでも良い
$ go buid
```

これで ```http://localhost:8080/hello``` にアクセスするとテキストが表示されます．

## A.16 まとめ
以上が，基本的な Go の構文になります．