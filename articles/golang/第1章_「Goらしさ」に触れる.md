# 第1章 「Go らしさ」に触れる
ここでは，「Go らしい」コードの書き方について見ていきます．

## 1.1 変数やパッケージ，メソッドなどに名前を付けるには
Go における命名規則は，[Effective Go](https://golang.org/doc/effective̲go.html) に公式見解が示されています．
ここでは，それをベースに [Go Review Comments](https://github.com/golang/go/wiki/CodeReviewComments) や標準ライブラリにおける命名方法を見ていきます．

### 1.1.1 変数名
変数名が複数の単語で構成されるときには，```maxLength``` のように MixedCaps 形式を用いることになっています．
また，他のパッケージから参照する変数は，```MaxLength``` のように先頭を大文字にします．  
URL や ID，HTTP，API などの頭字語は，全て大文字か小文字に統一して表記します．
他の単語と組み合わせる場合には，```ServerHTTP``` や ```stackID``` のようにします．  
```Error``` インタフェースを満たす ```Error``` 型として振る舞う型名には，```MarshalerError``` のように ```Error``` を接尾語に用います．
一方，```Error``` 型の変数は ```Err```，あるいは ```err``` で始まります．

bufio/scan.go
```go
var (
    ErrTooLong = errors.New("bufio.Scanner: token too long")
    ErrNegativeAdvance = errors.New("bufio.Scanner: SplitFunc returns negative advance count")
    ErrAdvanceTooFar = errors.New("bufio.Scanner: SplitFunc returns advance count beyond input")
)
```

Go の変数名は短い名前が好まれ，特にローカル変数では request を表す ```req``` や ```for``` 文のインデックスを表す ```i``` のように極力短い変数名を用います．
一方，宣言した場所と離れた場所で用いられる変数やグローバル変数などには，説明的な変数名を付ける必要があります．
たとえば，バッファサイズの最小値を表す変数は ```smallBufferSize``` として宣言されています．  
ただし，Go では全ての型が静的に決まるため，説明的変数名であったとしても，変数名に型名を含める必要はありません．
むしろ，情報密度が下がるため忌避すべきとされています．

### 1.1.2 パッケージ名
Go のパッケージ名は，```bytes``` や ```http``` のように，内容が想像しやすい全て小文字の1単語を用いるとされています．
ただし，```util``` や ```common``` のような汎用的な名前は内容が想像しづらいため避けるべきです．
複数の用途で用いられるパッケージは，```encoding/json``` や ```encoding/xml``` のようにフォルダ分けしてやるのが一般的です．
同様に，複数の単語でパッケージ名を構成したい場合にも，フォルダ分けが有効です．  
パッケージ名は基本的に全て外部に公開されますが，```internal``` パッケージだけはモジュール外からは読み込めません．
また，テストに用いるパッケージだけは，```(パッケージ名)_test``` のように例外的にスネークケースを用います．  
Go では，パッケージから必要な要素のみを取り出して使うことはなく，パッケージ名を付けて利用します．
そのため，パッケージ名と関数名に重複があるのは冗長とされています．

### 1.1.3 インタフェース名
1つのメソッドのみを持つインタフェースでは，```io.Reader``` や ```fmt.Stringer``` のように ```er``` を接尾辞に付けます．
ただし，```Open``` メソッドのみを持つ ```Driver``` インタフェースのように，末尾に ```er``` がつかないインタフェース名もあります．
インタフェースが複数のメソッドで構成される場合，そのインタフェースの目的を適切に説明する名前を選択します．

### 1.1.4 レシーバ名
自身を表すレシーバには，レシーバの型を反映した名前を付けます．
通常，```Request``` の ```r``` や ```Regexp``` の ```re``` のように1文字か2文字で命名します．
Go では，型ごとに一貫したレシーバ名を付けることとされています．

## 1.2 定数の使い方
Go では，異なる数値型の演算を許さない一方，定数はより柔軟な設計になっています．

### 1.2.1 型のない定数を定義する
Go における定数はリテラルとほぼ同義で，変数や引数などに代入して利用する値のことを指します．
定数同士の演算も可能で，この演算はコンパイル時に静的に実行され，実行ファイルのバイナリに埋め込まれます．
代入されたり型が指定されるまで型は不定となります．

```go
var a = 1       // イコールの右側の 1 が定数
var b = 1 + 100 // 定数の中で演算子を使った計算も可能
```

型を示さずに ```const``` キーワードを使うと，定数に名前を付けられます．

```go
const (
    a = 1       // イコールの右側の 1 が定数
    b = 1 + 2   // 演算もできる
    c = 9223372036854775807 + 1 // uint64 を超える数値も定義可能
    d = "hello world"   // 整数以外に、浮動小数点数や文字列やブール型も扱える
    e = iota + 10       // const 宣言でのみ iota も使える
)
```

定数を ```var``` に割り当てたり，関数に引数として渡したりする場合，型が指定されていなければ，それぞれの定数のデフォルトとなる型が設定されます．
型名が指定されているか，型が決まっている関数の引数に渡すした場合，その型となり，このタイミングで変数の値の範囲がチェックされます．

```go
var a = 1               // a の型は定数のデフォルトの型（ここでは int）になる
var b int32 = 1 + 100   // 明示的に書くことも可能
function(1000)          // 関数の引数に渡すリテラルも定数だが、関数の引数の型に合わせて型が決定
```

定数で利用可能な演算は ```iota``` によるインクリメントやなど簡単なものに限られます．

### 1.2.2 型付き const 変数を定義する
定数名の横に型名を書くことで，型付の定数を定義できます．
プリミティブ型から作成した定義型も指定可能で，値の書き換えはできなくなります．

```go
type ErrorCode int

const (
    f       int       = 10
    code    ErrorCode = 10
)
```

定数はコンパイル時に結果がわかっている必要があるため，関数の返り値を ```const``` 指定できません．
右辺の初期値の決定に関数を使う場合は ```var``` を使い，関数呼び出しに対しては暗黙的に ```init()``` 関数が作成され，パッケージ内の関数が呼び出される前に実行されて初期化されます．
また，構造体のインスタンス，マップ，スライスなどの複合型を扱うこともできません．
それ以外は，型なしの定数を変数に束縛するのと同じ動作となります．

```go
const (
    g int32 = 4294967295 + 1            // これは型の範囲を超えるためエラーになる
    h []int = []int{1, 2, 3}            // 配列やスライスはエラー
    i map[string]int = map[string]int{  // マップもエラー
        "Tokyo": 10,
        "Kanagawa": 11,
        "Chiba": 12,
    }
    j = function() // 関数の返り値もエラー
)
```

### 1.2.3 他言語との違い
C++ の ```const``` は構造体やクラスにおいて「属性値の変更を許さない」という意味になります．
また，JavaScript や TypeScript では「変数の再代入を許さない」という意味になります．
TypeScript では ```readonly``` 修飾子によって配列の値を変更禁止にできますが，```const``` 変数では再代入が禁止なだけで値の書き換えは可能となっています．

### 1.2.4 定数で error 型のインスタンスを提供する
Go では，失敗の可能性がある関数の返り値にエラー (なければ ```nil```) を返すのが一般的です．
このエラーのインスタンスを予め定義しておけば，エラー処理を簡単に実現できます．
しかし，エラーを表す構造体のインスタンスは ```const``` 指定できません．

```go
const (
    // New() 関数の返り値は実行時まで決まらないので const にできない
    ErrDatabase = errors.New("Database Error")
)
```

これに対して，適当なプリミティブ型の定義型を作成し，メソッドを定義して ```error``` インタフェースを満たして，そのインスタンスを ```const``` 指定することで解決できます．

```go
type errDatabase int

func (e errDatabase) Error() string {
    return "Database Error"
}

const (
    ErrDatabase errDatabase = 0
)
```

これで，```if``` 文を使って簡単にエラーの判定ができるようになります．

```go
err := OpenDB("postgres://localhost:5432")
if err == ErrDatabase {
    log.Fatal("DB 接続エラー ")
}
```

ただし，標準ライブラリでもエラーの定義は ```var``` を使って行われています．

```go
var ErrConnDone = errors.New("sql: connection is already closed")
```

そのため，いくらでもエラーの中身を書き換えることができてしまいます．
しかし，Go では性善説に基づいて実装者を束縛するような機能はあまり提供されていません．

## 1.3 iota を用いて列挙型を実現する
Go には ```enum`` キーワードはありませんが，いくつかの要素を組み合わせて列挙型に近い機能を実現できます．

- 種別を表す型を ```type``` キーワードで作成
- 定数のリストを ```const``` で作成
- 定数に割り当てる値として ```iota``` を利用

### 1.3.1 それぞれユニークな値を持つ定数を定義
列挙型相当の定数には，ベースとして ```int``` がよく用いられます．

```go
type CarType int
```

```iota``` は参照されるたびにインクリメントされる定数です．
定数宣言の中では，変数名のみを記述するとその直前の値がそのまま指定されます．
すなわち，次のように記述すると，それぞれの定数にユニークな整数値が設定されます．

```go
const (
    Sedan CarType = iota + 1
    Hatchback
    MPV
    SUV
    Crossover
    Coupe
    Convertible
)
```

上記で定義した定数は，通常の定数と同様に扱えます．

```go
var t CarType
t = SUV
```

```iota``` の初期値は0ですが，通常は1を足して1をオリジンとします．

### 1.3.2 iota の挙動
正確には，```iota``` は「行が変わるごと」にインクリメントされます．
また，ブロックを抜けるとリセットされます．

```go
const (
    a = iota    // 0
    b           // 1
    c           // 2
    _           // 3 だが使われない
    // 空行は無視
    d           // 4
    e = iota    // 5
)

const (
    f = iota    // 再び 0
    g           // 1
    h           // 2
)
```

### 1.3.3 組み合わせてフラグとして利用する定数の実現
ビットシフトによって，組み合わせを表現するフラグとしても利用できます．

```go
type CarOption uint64

const (
    GPS CarOption = 1 << iota
    AWD
    SunRoof
    HeatedSeat
    DriverAssist
)
```

上記のように2進数の値をセットして，論理和や論理積を取るとフラグとして利用できます．

```go
// 組み合わせて設定
var o CarOption
o = SunRoof | HeatedSeat

if o&SunRoof != 0 {
    fmt.Println(" サンルーフ付き ")
}
```

### 1.3.4 iota を使うべきでないとき
サーバのレスポンスとしてクライアントに値を返す場合など，外部プロセスで値を利用するときには ```iota``` を使うべきではありません．
これは，外部プロセスが ```iota``` で設定した値の変更に追従できないことが原因です．

### 1.3.5 文字列として出力可能にする
```stringer``` コマンドを利用して，整数値を文字列に変換することもできます．

```go
$ go install golang.org/x/tools/cmd/stringer@latest
```

列挙型相当の定数を記述したファイルに，以下のコメントを追記します．

```go
//go:generate stringer -type=CarOption
//go:generate stringer -type=CarType
```

```go generate``` コマンドを実行すると，```String()``` メソッドを含む ```cartype_string.go``` と ```caroption_string``` が生成され，整数定数から文字列に変換できるようになります．

```go
c := Convertible
fmt.Printf(" 握力王の愛車は %s です \n", c)
// 握力王の愛車は Convertible です
```

```stringer``` 以外にも，サードパーティ製の [```enumer```](github.com/alvaroloes/enumer@latest) を使うと，定数一覧を返す ```CarOptionValues()``` や文字列から
定数にする ```CarOptionString(s string)```，及び ```MarshalJSON()```， ```UnmarshalJSON()``` が生成されます．

## 1.4 Go のエラーを扱う
### 1.4.1 Go のエラーは値
Go のエラーは ```error``` 型のインタフェースを満たす組み込み型です．
このインタフェースに ```nil``` がセットされていれば，エラーがないことを示しています．

```go
type error interface {
    Error() string
}
```

一般的によく用いられる実装は，```errors.New()``` を使ってエラーを生成します．
その実体は ```errorString``` という構造体で，```Error() string``` を備えています．

```go
func New(text string) error {
    return &errorString{text}
}
type errorString struct {
    s string
}
func (e *errorString) Error() string {
    return e.s
}
```

### 1.4.2 panic() は使わない
Go で ```panic()``` を呼び出すと，プログラム全体が停止してしまいます．
エラーの値をチェックするのがエラーハンドリングの基本です．

### 1.4.3 Go のエラーハンドリング
一般的に，エラーの発生し得る関数では，返り値の一番最後に ```error``` を定義します．

```go
func ReadFile(name string) ([]byte, error) {
    // ...
}
```

関数を呼び出した後，ガード節によってエラーをハンドリングしてやることで，エラーがない状態で以降の処理を実行できます．

```go
f, err := os.Open("important.txt")
if err != nil {
    // エラーハンドリング
}

r := bufio.NewReader(f)
l, err := r.ReadString('\n')
if err != nil {
    // エラーハンドリング
}
// ここではエラーは発生していない
```

### 1.4.4 他の言語との違い
例えば，Java では例外機構が備わっており，```throw``` で例外を発生させて，```catch``` で例外を補足してハンドリングします．
一方，Go では関数の戻り値によってエラーの判定を行います．

## 1.5 変数は短縮形式 := と var のどちらを使うべきか
```var``` で宣言した変数は，値を代入しなければゼロ値で初期化されます．
値を代入する場合には，```:=``` の短縮記法を積極的に使うようにしましょう．
ただし，明示的に別の型にしたい場合には，どちらでも構いません．

## 1.6 関数のオプション引数
Go の文法では同じ型であれば可変長引数を設定できますが，異なる型で可変長引数を取ることはできません．
ここでは，次のように異なる型の可変長引数を実現する関数を作成してみます．

```go
type Portion int

const (
    Regular Portion = iota  // 普通
    Small   // 小盛り
    Large   // 大盛り
)

type Udon struct {
    men         Portion
    aburaage    bool
    ebiten      uint
}

// 麺の量、油揚げ、海老天の有無でインスタンス作成
func NewUdon(p Portion, aburaage bool, ebiten uint) *Udon {
    return &Udon{
        men:        p,
        aburaage:   aburaage,
        ebiten:     ebiten,
    }
}

// 海老天 2 本入りの大盛り
var tempuraUdon = NewUdon(Large, false, 2)
```

### 1.6.1 別名の関数によるオプション引数
最も愚直な方法で，組み合わせを展開して，それぞれ別名の関数で実装する方法です．

```go
func NewKakeUdon(p Portion) *Udon {
    return &Udon{
        men:        p,
        aburaage:   false,
        ebiten:     0,
    }
}

func NewKitsuneUdon(p Portion) *Udon {
    return &Udon{
        men:        p,
        aburaage:   true,
        ebiten:     0,
    }
}

func NewTempuraUdon(p Portion) *Udon {
    return &Udon{
        men:        p,
        aburaage:   false,
        ebiten:     3,
    }
}
```

コンストラクタではこの実装がしばしば見られます．  
類似の方法に，多くの引数を取れる関数を1つ用意しておき，ラッパ関数を用途別に用意する方法もあります．

### 1.6.2 構造体を利用したオプション引数
この方法では，比較的少ないコード量で大量のオプションを指定できますが，ゼロ値やデフォルト値の実装が面倒になるという欠点があります．
可変長引数と組み合わせて，オプションがない場合には引数ごと省略可能にしたり，あるいはポインターにして ```nil``` を渡せるようにするなどの派生パターンもあります．

```go
type Option struct {
    men         Portion
    aburaage    bool
    ebiten      uint
}
func NewUdon(opt Option) *Udon {
    // ゼロ値に対するデフォルト値処理は関数 / メソッド内部で行う
    // 朝食時間は海老天 1 本無料
    if opt.ebiten == 0 && time.Now().Hour() < 10 {
        opt.ebiten = 1
    }
    return &Udon{
        men:        opt.men,
        aburaage:   opt.aburaage,
        ebiten:     opt.ebiten,
    }
}
```

### 1.6.3 ビルダを利用したオプション引数
通信を行う API やロガー周り，コマンドライン引数のパーサでよく用いられます．

```go
type fluentOpt struct {
    men         Portion
    aburaage    bool
    ebiten      uint
}

func NewUdon(p Portion) *fluentOpt {
    // デフォルトはコンストラクタ関数で設定
    // 必須オプションはここに付与可能
    return &fluentOpt{
        men:        p,
        aburaage:   false,
        ebiten:     1,
    }
}

func (o *fluentOpt) Aburaage() *fluentOpt {
    o.aburaage = true
    return o
}

func (o *fluentOpt) Ebiten(n uint) *fluentOpt {
    o.ebiten = n
    return o
}

func (o *fluentOpt) Order() *Udon {
    return &Udon{
        men:        o.men,
        aburaage:   o.aburaage,
        ebiten:     o.ebiten,
    }
}

func useFluentInterface() {
    oomoriKitsune := NewUdon(Large).Aburaage().Order()
}
```

先述の構造体を利用した方法にオプション引数を追加したもので，コード量は多くなり，終了メソッド (ここでは ```Order()```) を呼び出す必要があります．
一方，```Fluent``` インタフェース形式の API を利用して，

- 同一項目をパラメーター違いで複数回受け取るのが比較的シンプルに実現できる
- 未指定だったのかゼロ値と同じ値を設定したのかをこれも容易に判別できる
- 1つのメソッドで複数の引数に値を設定することができる

というメリットがあります．

### 1.6.4 Functional Option パターンを使ったオプション引数
構造体のメソッドを使わず，独立した関数を利用する方法です．

```go
type OptFunc func(r *Udon)

func NewUdon(opts ...OptFunc) *Udon {
    r := &Udon{}
    for _, opt := range opts {
        opt(r)
    }
    return r
}

func OptMen(p Portion) OptFunc {
    return func(r *Udon) { r.men = p }
}

func OptAburaage() OptFunc {
    return func(r *Udon) { r.aburaage = true }
}

func OptEbiten(n uint) OptFunc {
    return func(r *Udon) { r.ebiten = n }
}

func useFuncOption() {
    tokuseiUdon := NewUdon(OptAburaage(), OptEbiten(3))
}
```

上記のように，返り値の関数を型定義しておいてそれを使うと，GoDoc 上でまとめて表示されます．
メリットとして，パッケージの作成者以外でもオプションを自作できることが挙げられますが，パッケージ外からの呼び出し時に記述量が多くなるというデメリットがあります．

### 1.6.5 どの実装方法を選択すべきか
構造体を使った方法を基本として，必要に応じてそれ以外の方法を実装するという方法が望ましいと言えます．

## 1.7 プログラムを制御する引数
ここでは，コマンドライン引数と環境変数をつかって，プログラム起動時にオプションを渡す方法を見ていきます．

### 1.7.1 コマンドライン引数
数値や文字列を受け取るだけであれば，標準ライブラリの ```flag``` パッケージで十分です．

```go
var (
    FlagStr = flag.String("string", "default", " 文字列フラグ ")
    FlagInt = flag.Int("int", -1, " 数値フラグ ")
)
```

POSIX スタイルの短縮記号と長い引数名への対応，短縮記号をつなげた記法，必須フラグのチェック，サブコマンドを持つ，といった場合には，

- gopkg.in/alecthomas/kingpin.v2
- github.com/spf13/cobra

などのライブラリを使うのがおすすめです．
以下は，前者を用いてサブコマンドや共通フラグを設定した場合の例を示しています．

```go
var (
    defaultLanguage = kingpin.Flag("default-language", "Default language").String()
    
    generateCmd = kingpin.Command("create-index", "Generate Index")
    inputFolder = generateCmd.Arg("INPUT", "Input Folder").Required().ExistingDir()
    
    searchCmd = kingpin.Command("search", "Search")
    inputFile = searchCmd.Flag("input", "Input index file").Short('i').File()
    searchWords = searchCmd.Arg("WORDS", "Search words").Strings()
)
```

### 1.7.2 環境変数
環境変数はプロセスの持つ変数で，プログラム実行時に OS からプログラムに渡されます．
アプリケーションをパッケージングしたあとでも，デプロイ環境などに合わせて設定可能なため，クラウドではよく用いられます．  
Go では，```os``` パッケージの ```os.Environ()``` や ```os.LookupEnv()``` などで取得できます．
または，```kingpin.v2``` の ```Envar(name)``` で取得する方法や，```kelseyhightower/envconfig``` のように構造体のタグを使って環境変数情報を構造体にマッピングする方法もあります．

```go
import (
    "github.com/kelseyhightower/envconfig"
)

type Config struct {
    Port        uint16 `envconfig:"PORT" default:"3000"`
    Host        string `envconfig:"HOST" required:"true"`
    AdminPort   uint16 `envconfig:"ADMIN_PORT" default:"3001"`
}
```

## 1.8 メモリ起因のパフォーマンス低下を解消する
Go のパフォーマンスで差が出やすいのは，スライスやマップのメモリ確保の部分です．

### 1.8.1 スライスのメモリ確保を高速化する
Go でよく用いられるスライスに ```append``` したときには，

- 確保済みのメモリが不足した場合には OS に要求
- 確保済みのものから新しい配列用にメモリを割り当てる
- 新しい配列に古い配列の内容をコピー
- 古い配列にアクセスするスライスがなくなったらメモリを解放

という処理が行われています．
Go のランタイム内部では，確保済みメモリの使い回しなど，OS とのやり取りを極力減らすようにメモリ管理を行っていますが，最終的なリクエストは OS に行きます．
場合によっては，スワップアウトや優先度の低いプロセスを終了させることでメモリを確保しようとして，OS の処理が重くなってしまうことがあります．
これを考慮して，スライスで確保されるメモリ量はヒューリスティックによって決定されいます．  
ただ，予め必要な要素数が確実がわかっている場合や，最大数や最小数がわかっている場合には，```make()``` で明示的にメモリを確保しておくと，パフォーマンスの改善に繋がります．

```go
// 正確な長さがわかっている場合
s1 := make([]int, 1000)
fmt.Println(len(s1)) // 1000
fmt.Println(cap(s1)) // 1000

// 正確な長さがわからないが最大量の見込みがつく場合，キャパシティだけ増やす
s2 := make([]int, 0, 1000)
fmt.Println(len(s2)) // 0
fmt.Println(cap(s2)) // 1000
```

使用するメモリの下限がわかっている場合にも，キャパシティにその値を設定しておくと，メモリの再割当ての回数を大幅に減らせます．

### 1.8.2 マップのメモリ確保を高速化する
マップの背景にはバケットと呼ばれる，8つのデータを保持できるデータ構造を持ち，要素が増えるとバケット数も増えていきます．
マップに関してもスライスと同様に，```make()``` で初期化が可能です．

```go
m := make(map[string]string, 1000)
fmt.Println(len(m)) // 0
```

ここまで，メモリの最適化手法を見てきましたが，最適化は必ず計測してから行うようにしましょう．

### 1.8.3 defer の落とし穴
```defer``` はリソースの確保直後に解放処理を記述し，予約実行してくてくれる便利な機能ですが，```for``` ループと組み合わせる場合には注意が必要です．
次のようなコードでは，```f.Close()``` がループの完了まで実行されないため，線形にリソースの消費量が増えていきます．
そのため，```defer``` は使わず，ファイルを使い終わったあとに ```f.Close()``` する必要があります．

```go
for _, fname := range files {
    f, err := os.Open(fname)
    if err != nil {
        return err
    }

    // この書き方だと Close() は全部のループを抜けるまで実行されない
    // defer を使わずにファイルを使った後に自分で f.Close() を呼ぶ
    defer f.Close()

    data, _ := io.ReadAll(f)
    result = append(result, data)
}
```

また，メソッドによってはエラーを返す場合もありますが，```defer``` ではそのエラーを取りこぼしてしまいます．
この場合には，無名関数でエラーをくくって，そのエラーを名前付きの返り値に代入してやります．

```go
func deferReturnSample(fname string) (err error) {
    var f *os.File
    f, err = os.Create(fname)
    if err != nil {
        return fmt.Errorf(" ファイルオープンのエラー %w", err)
    }
    defer func() {
        // Close のエラーを拾って名前付き返り値に代入
        // すでに err に別のものが入る可能性があるときはさらに要注意
        err = f.Close()
    }()
    io.WriteString(f, "defer のエラーを拾うサンプル ")
    return
}
```

## 1.9 文字列の結合方法
Go では文字列を結合する方法がいくつか用意されていますが，ナイーブに ```+``` で結合するとパフォーマンスが低下します．
Go の文字列は不変であるため，```for``` ループで複数の文字列を結合しようとした場合には，その都度，中間の文字列がメモリ上に確保されます．
そのため，速度が低下するという問題があります．  
大量の文字列を結合するには，```strings``` パッケージの ```Builder``` を使い，予め結合後のサイズがわかっていれば ```Grow()``` で内部で利用するバッファサイズを指定することも可能です．

```go
var builder strings.Builder
builder.Grow(100) // 最大 100 文字以下と仮定できる場合
    for i, word := range src {
    if i != 0 {
        builder.WriteByte(' ')
    }
    builder.WriteString(word)
}
log.Println(builder.String())
```

結合する文字列が少なければ，1つの式でまとめて書いてしまうのも効率は悪くありません．

## 1.10 日時の取り扱い
### 1.10.1 日時の time.Time の取得
Go で日時とタイムゾーンを扱うには ```time.Time``` を使います．

```go
// システムロケールの現在時刻の time.Time インスタンス取得
now := time.Now()

// 指定日時の time.Time インスタンス取得
tz, _ := time.LoadLocation("America/Los_Angeles")
future := time.Date(2015, time.October, 21, 7, 28, 0, 0, tz)

fmt.Println(now.String())
fmt.Println(future.Format(time.RFC3339Nano))
```

タイムゾーンデータベースからタイムゾーンを取得するには ```LoadLocation``` を利用し，ローカルや UTC のタイムゾーンの取得には ```Local``` や ```UTC``` を使います．
```FixedZone()``` で好きな時差のタイムゾーンを作ることもできます．

```go
// 定義済みのローカルタイムゾーン
now := time.Date(1985, time.October, 26, 9, 0, 0, 0, time.Local)

// 定義済みの UTC タイムゾーン
past := time.Date(1955, time.November, 12, 6, 38, 0, 0, time.UTC)
```

タイムゾーンの情報には，夏時間の開始日時など，頻繁に更新される情報が含まれているので，OS を更新して，その情報を参照するのがベストです．
あるいは，Go 1.15 であれば ```time/tzdata``` でタイムゾーンの情報をアプリケーションにバンドルできるようになったので，そちらを利用する方法も可能です．

### 1.10.2 時間を表す time.Duration
時間を表す型として，```int64``` をベースに，1が 1ns を表す ```time.Duration``` が提供されています．
```time.Duration``` を作成するには，

- ```time.Time``` 同士の差を Sub() メソッドで計算
- ```time.Second``` などの既存の時間のインスタンスの積により作成

といった方法で作成します．
内部的には同じでも，```int64``` と ```time.Duration``` は型が異なるため，直接演算はできません．
数値リテラルであればコンパイル時に方が自動判定されますが，関数に渡したり変数同士で演算したりする場合には，```time.Duration``` にキャストする必要があります．

```go
// 5 分を作成
// Nanosecond, Millisecond, Second, Minute, Hour が定義済み
fiveMinute := 5 * time.Minute

// int とは型違いで直接演算できないので、即値との計算以外は time.Duration への明示的なキャストが必要
// キャストがないと、次のエラーが発生する
// invalid operation: seconds * time.Second (mismatched types int and time.Duration)
var seconds int = 10
tenSeconds := time.Duration(seconds) * time.Second

// Time の演算で Duration 作成
past := time.Date(1955, time.November, 12, 6, 38, 0, 0, time.UTC)
dur := time.Now().Sub(past)
```

作成した ```time.Duration``` のインスタンスを ```time.Time``` に加算して，その分をオフセットした新しい ```time.Time``` を作成することができます．
```Truncate()``` メソッドで5分単位で切り捨てることもできます．
次のサンプルは，毎時 0 分に既製のバッチのプログラムがファイルを出力するコードで，毎時0分0秒のタイムスタンプを文字列で取得できます．

```go
// 1 時間にまとめてバッチで読み込むファイル名を取得
filepath := time.Now().Truncate(time.Hour).Format("20060102150405.json")
```

```time.Time``` には，```Add()``` や ```AddDate()``` メソッドしかないため，引き算の代わりに「マイナスの ```time.Duration``` を ```Add()``` に渡す」か「マイナスの年月日を ```AddDate()``` に渡す」必要があります．

```go
// 5 分後と 5 分前の時刻
fiveMinuteAfter := time.Now().Add(fiveMinute)
fiveMinuteBefore := time.Now().Add(-fiveMinute)
```

また，次のようなメソッドが用意されています．

- ```time.Ticker``` : 指定された時間ごとに継続して発生するイベント
- ```time.Timer``` : ワンショットタイマー
- ```context.WithTimeout``` : 指定時間以内に処理できないと中断されるタイムアウト処理
- ```time.Sleep``` : 指定時間処理を停止

```go
// 3 秒停止
fmt.Println("3 秒スリープスタート ")
time.Sleep(3 * time.Second)
fmt.Println("3 秒スリープ完了 ")

// 10 秒間待つ
fmt.Println("10 秒停止スタート ")
timer := time.NewTimer(10 * time.Second)
defer timer.Stop()
<-timer.C
fmt.Println("10 秒停止完了 ")
```

### 1.10.3 ウェブフロントエンドとのデータの交換
Go では ```Format()``` メソッドで時刻を文字列にして，```time.Parse()``` や ```time.ParseInLocation()``` で文字列から ```time.Time``` のインスタンスを作成しています．
フォーマット文字列は，毎回自分で作成しなくても RFC822Z や RFC3339 などのフォーマットが予め用意されています．  
JavaScript との互換性を考慮して，```time.RFC3339Nano``` でナノ秒まで含めた形式で出力するのがおすすめです．
このフォーマット文字列で出力された日時の情報は，JavaScript の ```Date``` コンストラクタや ```Date.parse()``` 関数で処理でき，逆に JavaScript の ```Date``` インスタンスの ```toISOString()``` で生成した文字列は，```time.RFC3339Nano``` を指定した ```time.Parse()``` でパースできます

### 1.10.4 翌月を計算するときのハマりどころ
年月日の加減算を行うために ```AddDate()``` メソッドが用意されています．

```go
jst, _ := time.LoadLocation("Asia/Tokyo")
now := time.Date(2021, 6, 8, 20, 56, 00, 000, jst)
nextMonth := now.AddDate(0, 1, 0)
fmt.Println(nextMonth)
// 2021-07-08 20:56:00 +0900 JST
```

もし，日付が範囲外や存在しない値を指定された場合には，存在する値に自動で変換されます．
これを正規化と読んでいます．

```go
normal := time.Date(2021, 6, 31, 00, 00, 00, 000, jst)
fmt.Println(normal)
// 2021-07-01 00:00:00 +0900 JST
```

ここで，5月31日の翌月を求めようとして ```AddDate(0, 1, 0)``` とすると，7月1日になってしまいます．

```go
now := time.Date(2021, 5, 31, 00, 00, 00, 000, jst)
nextMonth := now.AddDate(0, 1, 0)
fmt.Println(nextMonth)
// 2021-07-01 00:00:00 +0900 JST
```

これを防ぐためにはひと工夫必要です．

```go
func NextMonth(t time.Time) time.Time {
    year1, month2, day := t.Date()
    first := time.Date(year1, month2, 1, 0, 0, 0, 0, time.UTC)
    year2, month2, _ := first.AddDate(0, 1, 0).Date()
    nextMonthTime := time.Date(year2, month2, day, 0, 0, 0, 0, time.UTC)
    if month2 != nextMonthTime.Month() {
        return first.AddDate(0, 2, -1) // 翌月末
    }
    return nextMonthTime
}
```

## 1.11 まとめ
ここでは，Go の文化や考え方を，他の言語との比較を交えつつ見てきました．
1つの文の裏で多くの処理が走る Java とは異なり，Go は動的に挙動が変わる余地が少なく，書いたまま動くシンプルさを売りにしています．
また，Go は全てのユーザに同じフォーマットでコードを書くことを求めているのも特徴です．