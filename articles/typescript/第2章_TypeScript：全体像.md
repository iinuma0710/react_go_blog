# 第2章 TypeScript：全体像
ここでは，TSC (TypeScript Compiler) について見ていきます．

## 2.1 コンパイラ
プログラムとは，プログラマの書いた一連のテキストを含むファイルのことです．
このテキストは，

1. **コンパイラ** と呼ばれる特別なプログラムによって解析され，
2. **抽象構文木** (AST Abstract Syntax Tree) に変換し，
3. コンパイラが，AST をさらに**バイトコード**と呼ばれるより低レベルな表現に変換し，
4. バイトコードを**ランタイム**と呼ばれる別のプログラムに入力として与える

と言う手順を踏んで，評価したり結果を出力したりします．
しかし，TypeScript のコンパイラは，バイトコードを出力する代わりに，JavaScript のコードを生成する点で独特であると言えます．
そして，生成された JavaScript のコードは，通常通りブラウザや Node.js を介して実行されるのです．  
そして TypeScript のコンパイラは，AST を生成してから，JavaScript のコードを生成するまでの間に，**型チェック** (typecheck) を行います．
型チェックは**型チェッカー** (typechecker) によって行われ，コードが型安全であることを検証します．
上記をまとめると，TypeScript のプログラムを実行する手順は次のようになります．

1. TypeScript ソースプログラム → TypeScript AST
2. AST が型チェッカーでチェックされる
3. TypeScript AST → JavaScript
4. JavaScript → JavaScript AST
5. AST → バイトコード
6. バイトコードをランタイムで評価

この内，1から3は TSC，4から6はブラウザや Node.js のランタイムによって行われます．
また，3の JavaScript への変換では，プログラマの記述した型は参照されません．
これによって，型の扱い方でアプリケーションが破壊されるのを防ぐことができます．

## 2.2 型システム
型システム (type system) は，プログラマが作成したプログラムに型を割り当てるために型チェッカーが使用するルールで，一般的には2種類の型システムがあります．
1つ目は明示的に全ての型をコンパイラに示すもの，2つ目が型を自動的に推論するものです．  
TypeScript はその両方が可能な言語となっています．
型を明示的に示す場合には ```:``` の後ろに型を指定してやります．

```ts
let a: number = 1
let b: string = 'hello'
let c: boolean[] = [true, false]
```

一方，型の指定を省略して，型を推論させることもできます．

```ts
let a = 1
let b = 'hello'
let c = [true, false]
```

### 2.2.1 TypeScript vs JavaScript
TypeScript と JavaScript の型システムを比較した表を以下に示します．

| 型システムの特徴 | JavaScript | TypeScript |
| :--: | :--: | :--: |
| 型のバインド | 動的 | 静的 |
| 自動的な型変換 | される | (多くの場合)されない |
| 型チェックのタイミング | 実行時 | コンパイル時 |
| エラー表面化のタイミング | (多くの場合)されない | (多くの場合)コンパイル時 |

#### 2.2.1.1 型はどのようにバインドされるか？
「型が動的にバインドされる」とは，型を確定するのにプログラムを実行する必要があるということを意味します．
TypeScript は**漸進的型付言語** (gradually typed language) と呼ばれ，コンパイル時に全ての型がわかっていなくても動作する仕様になっています．
ただし，基本的には全ての型を静的に決定すべきで，JavaScript から移行する途中のような限られた場面でのみ，この仕様を活用します．

#### 2.2.1.2 型は自動的に変換されるか？
JavaScript は弱く型付けられた言語で，数値と配列を足し合わせるような不正な演算をしようとしても，なんとか結果を出力しようとします．
たとえば，```3 + [1]``` は，

1. ```3``` が数値であり，```[1]``` が配列であることを認識
2. ```+``` で両者を連結したいと推定
3. ```3``` を暗黙的に文字列 ```"3"``` に変換
4. ```[1]``` を暗黙的に文字列 ```"1"``` に変換
5. 型変換の結果を結合して ```"31"``` を出力

という手順で，```"31"``` と出力されます．
これを明示的に行うと，```(3).toString() + [1].toString()``` となります．  
一方 TypeScript では，このように不正と思われる操作に対してはエラーを出力します．
先述のような JavaScript の振る舞いは，バグの原因を切り分けるのに大きな障害となります．
個々人でコーディングしているならまだしも，チーム開発では致命的です．
そのため，型変換の必要がある場合には，それを明示的に示す必要があるのです．

#### 2.2.1.3 型はいつチェックされるか？
TypeScript では，コンパイル時にコードの型チェックを行います．
そのため，VSCode の TypeScript 拡張機能を使うと，プログラムを書きながらエラー箇所を確認することができます．

#### 2.2.1.4 エラーハイツ表面化するか？
JavaScript ではプログラムを実行するまで，例外を投げたり暗黙の方変換を行ったりしません．
そのため，最悪の場合，サービスをデプロイしてからバグが発見されることになります．  
一方，TypeScript は構文と型に関するエラーをコンパイル時，あるいはエディタの拡張機能を使っていればコードを書いてすぐにスローします．
ただし，スタックオーバフローやネットワークの切断，不正なユーザ入力などは実行してみないとわかりません．
しかし，これによってテストケースを削減することができます．

## 2.3 コードエディタのセットアップ
TypeScript のコンパイラを使うには，[Node.js](https://nodejs.org) が必要です．
適当なフォルダを作って，その中で TypeScript の環境を作ります．

```bash
# 新しいフォルダーを作ります
$ mkdir chapter-2
$ cd chapter-2

# 新しい npm プロジェクトを初期化します（表示されるプロンプトに従います）＊2
$ npm init

# TSC，TSLint，および Node.js 用の型宣言をインストールします
$ npm install --save-dev typescript tslint @types/node
```

### 2.3.1 tsconfig.json
TypeScript のプロジェクトには，

- どのファイルをコンパイルするか？
- コンパイル結果をどこに出力するか？
- どのバージョンの JavaScript を出力するか？

などの情報を格納した ```tsconfig.json``` をルートディレクトリに配置する必要があります．
ここでは，```tsconfig.json``` に以下の内容を書き込みます．

```json
{
    "compilerOptions": {
        "lib": ["es2015"],
        "module": "commonjs",
        "outDir": "dist",
        "sourceMap": true,
        "strict": true,
        "target": "es2015"
    },
    "include": [
        "src"
    ]
}
```

それぞれのオプションの意味は次のとおりです．

- include 
  TSC が TypeScript ファイルを探すディレクトリ 
- lib  
  コードを実行する環境に存在しているとTSC が想定すべき API．
  これには，ES5 の ```Function.prototype.bind```，ES2015 の ```Object.assign```，DOM の ```document.querySelector``` のようなものが含まれる．
- module  
  CommonJS や SystemJS，ES2015など TSC がコンパイルする先のモジュールシステム．
- outDir  
  生成した JavaScript コードを格納するフォルダ
- strict  
  不正なコードをチェックするときに，できるだけ厳格にチェックする．
  このオプションによって，すべてのコードが適切に型付けされていることが強制される．
- target  
  コンパイル先の JavaScript バージョン

他にも多くのオプションを指定できます．
詳しくは [TypeScript の Web サイトにある公式ドキュメント](http://bit.ly/2JWfsgY) を参照してください．
また，tsconfig.json のオプション追加はコマンドラインからも可能です．
こちらも，詳しくは ```./node_modules/.bin/tsc --help``` の出力を実行してみてください．

### 2.3.2 tslint.json
tslint.json には，スベースを使うかタブを使うかといったような，作成するコードのスタイル規約を設定します．
次のコマンドで，デフォルトの設定が書き込まれた tslint.json が生成されます．

```bash
$ ./node_modules/.bin/tslint --init
```

このファイルを自分のコーディングスタイルに合わせて書き換えます．

```json
{
    "defaultSeverity": "error",
    "extends": [
        "tslint:recommended"
    ],
    "rules": {
        "semicolon": false,
        "trailing-comma": false
    }
}
```

TSLint で利用可能なオプションは[ドキュメント](https://palantir.github.io/tslint/rules/)を参照してください．
カスタムルールの追加や React 用の外部プリセットをインストールできます．  
ただし，TSLint は2019年に非推奨となりました．
今後は，ESLint + TypeScript プラグインを使うことが推奨となっています．

## 2.4 index.ts
最後に，簡単なサンプルプログラムを動かしてみます．
```chapter_02``` ディレクトリに，以下の構成でファイルを作成します．

```
chapter-2/
├── node_modules/
├── src/
│　└── index.ts
├── package.json
├── tsconfig.json
└── tslint.json
```

```src/index.ts``` の中身は，次の1行だけです．

```ts
console.log('Hello TypeScript!')
```

これをコンパイルして実行します．

```bash
# TypeScript コードを TSC でコンパイル
$ ./node_modules/.bin/tsc

# Node.js を使ってコードを実行
$ node ./dist/index.js
Hello TypeScript!
```

これで，動作確認ができました．

## 2.5 練習問題
(省略)