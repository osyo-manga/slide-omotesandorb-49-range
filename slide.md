## 表参道.rb #49
- - -

## ここがつらいよ無限 Range

---

#### 自己紹介
- - -

* なまえ  : おしょー
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* Rails 歴 1年半
* 趣味で Ruby にパッチを投げています
* 好きなメソッド：tap
* 好きな gem：rspec
* 好きなエディタ：Vim

---

### 今日話すこと
- - -

## ここがつらいよ無限 Range


---

#### Range の基本
- - -

* `(first...last)` みたいなシンタックスで定義
  * `Range.new(frst, last)` と同等
* `(1..5)` だと `5` を含む
* `(1...5)` だと `5` を含まない
* `#===` や `#cover?` で範囲チェック出来る

```ruby
# 数値で範囲を定義
# 終端が含まれる
pp (1..10).cover? 10    # => true

# 終端は含まれない
pp (1...10).cover? 10   # => false

# 文字列でも定義出来る
pp ("a".."z") === "x"  # => true
```


---

# 無限 Range の軌跡

---


#### Ruby 2.5 以前
- - -

* `Float::INFINITY` を使って擬似的に無限を定義
* 特定の値でのみ使用できる

```ruby
# 数値に対して使用できる
# 終端に上限がない
pp (1..Float::INFINITY) === 1000          # => true
pp (1..Float::INFINITY) === 100000000     # => true
# 先端の場合は負の値を渡す
pp (-Float::INFINITY..1) === -100000000   # => true

# 先頭から任意の数の値を取り出す
pp (1..Float::INFINITY).take(5)
# => [1, 2, 3, 4, 5]

# ただし、文字列みたいな Range には使用できない
pp ("a"..Float::INFINITY).take(5)
```

---

#### Ruby 2.6
- - -

* 終端無限を定義するシンタックスを追加
* `(1..)` のように終端の値を省略して定義
* `(1..)` は `(1..nil)` と同等

```ruby
# 数値も OK
pp (1..).take(5)
# => [1, 2, 3, 4, 5]

# 文字列みたいな Range でも OK
pp ("a"..).take(5)
# => ["a", "b", "c", "d", "e"]

require "time"

# Time でも使える
pp (Time.parse("2019/05/01")..) === Time.parse("2020/07/10")
# => true
```

---

#### Ruby 2.7
- - -

* 先端無限を定義するシンタックスを追加(予定)
* `(1..)` と同様に `(..1)` と定義
* `(..)` みたいに両方省略出来ない

```ruby
pp (..20) === 3
# => true

require "time"

pp (..Time.parse("2019/05/01")) === Time.parse("1900/07/10")
# => true
```
---

### どういう時に無限 Range を使うの

---

### String#[] や Array#[]
- - -

* String や Array の添字アクセスに Range を渡す
* その Range の範囲を取得できる

```ruby
# 5 〜 10文字目
pp "abcdefghijklmnopqrstuvwxyz"[5..10]
# => "abcdefghijk"

# 10文字目よりも前
pp "abcdefghijklmnopqrstuvwxyz"[..10]
# => "abcdefghijk"

# 10文字目以降
pp "abcdefghijklmnopqrstuvwxyz"[10..]
# => "klmnopqrstuvwxyz"

# Array#[] にも使える
pp [1, 2, 3, 4, 5][2..]
# => [3, 4, 5]
```

---

### case when
- - -

```ruby
require "time"

def gengo(time)
	case time
	when (...Time.parse("1989/01/08"))
		"平成以前"
	when (Time.parse("1989/01/08")..Time.parse("2019/04/30"))
		"平成"
	when (Time.parse("2019/05/01")..)
		"令和"
	end
end

pp gengo(Time.parse("1989/01/07"))   # => "平成以前"
pp gengo(Time.parse("1989/01/08"))   # => "平成"
pp gengo(Time.parse("2019/04/30"))   # => "平成"
pp gengo(Time.parse("2019/05/01"))   # => "令和"
pp gengo(Time.now)                   # => "令和"
```

---

### パターンマッチ
- - -

```ruby
def validation user
  case user
  in { name: /^[a-z]+$/, age: (20..) }
    :OK
  else
    :NG
  end
end

p validation(name: "homu", age: 14)
# => :OK
p validation(name: "mami", age: 24)
# => :NG
```

---

#### ActiveRecord の where に渡す
- - -

* where に Range を渡すと BETWEEN が追加される
* が、無限 Range の場合はうまく動作しない…

```ruby
# BETWEEN クエリが追加される
puts User.where(age: 20..60).to_sql
# => SELECT "users".* FROM "users" WHERE "users"."age" BETWEEN 20 AND 60

# 無限 Range の場合は BETWEEN 20 AND NULL になる
puts User.where(age: 20..).to_sql
# => SELECT "users".* FROM "users" WHERE "users"."age" BETWEEN 20 AND NULL
```

* [Ruby2.6のInfiniteRangeをActiveRecordで利用した時の挙動を各RDBMSに対して調査した - Qiita](https://qiita.com/color_box/items/7fa900909745787ac821)
* [BETWEEN 1 AND NULLとしたときの挙動について - Qiita](https://qiita.com/neko_the_shadow/items/b9406e69333543358819)

---

# ここから本編

---

#### 終端 Range に対する #end
- - -

* (1..10).end
  * => 10                                     <!-- .element: class="fragment" -->
* (1..Float::INFINITY).end
  * => Infinity                                     <!-- .element: class="fragment" -->
* (1..).end
  * => nil                                     <!-- .element: class="fragment" -->

---

#### 終端 Range に対する #last
- - -

* (1..10).last
  * => 10                                     <!-- .element: class="fragment" -->
* (1..Float::INFINITY).last
  * => Infinity                                     <!-- .element: class="fragment" -->
* (1..).last
  * => Error: in `last': cannot get the last element of endless range (rangeerror)                                     <!-- .element: class="fragment" -->

---

#### 終端 Range に対する #last + 引数
- - -

* (1..10).last(3)
  * => [8, 9, 10]                                        <!-- .element: class="fragment" -->
* (1..Float::INFINITY).last(3)
  * => 無限ルーーープ                                     <!-- .element: class="fragment" -->
* (1..).last(3)
  * => Error: in `last': cannot get the last element of endless range (rangeerror)                                     <!-- .element: class="fragment" -->

---

#### (1..).first(3) と (..1).last(3)
- - -

* (1..).first(3)
  * => [1, 2, 3]                                        <!-- .element: class="fragment" -->
* (..1).last(3)
  * => Error: in `each': can't iterate from NilClass (TypeError)                                        <!-- .element: class="fragment" -->
* [Feature #15747: `(..1).last(2)` should return array but raise TypeError - Ruby master - Ruby Issue Tracking System](https://bugs.ruby-lang.org/issues/15747)

---


#### 数値の Range に対する #size
- - -

* (1..10).size
  * => 10                                        <!-- .element: class="fragment" -->
* (1..Float::INFINITY).size
  * => Infinity                                        <!-- .element: class="fragment" -->
* (1..).size
  * => Infinity                                        <!-- .element: class="fragment" -->

---

#### 文字列の Range に対する #size
- - -

* ("a".."z").size
  * => nil                                        <!-- .element: class="fragment" -->
* ("a"..).size
  * => nil                                        <!-- .element: class="fragment" -->
* (.."z").size
  * => nil                                        <!-- .element: class="fragment" -->

---

#### まとめ1
- - -

```ruby
p (1..10).end                # => 10
p (1..Float::INFINITY).end   # => Infinity
p (1..).end                  # => nil

p (1..10).last               # => 10
p (1..Float::INFINITY).last  # => Infinity
p (1..).last                 # Error: in `last': cannot get the last element of endless range (rangeerror)

p (1..10).last(3)                # => [8, 9, 10]
p (1..Float::INFINITY).last(3)   # Infinite loop
p (1..).last(3)                  # Error: in `last': cannot get the last element of endless range (rangeerror)

p (1..).first(3)             # => [1, 2, 3]
p (..1).last(3)              # Error: in `each': can't iterate from NilClass (TypeError)
```

>>>

```ruby
# begin / end は nil を返します
p (..10).begin  # => nil
p (1..).end     # => nil

# しかし、 first / last は例外が発生します
p (..10).first  # Error: in `first': cannot get the first element of beginless range (RangeError)
p (1..).last    # Error: in `last': cannot get the last element of endless range (RangeError)

# 数値の場合は Infinity を返す
p (1..10).size      # => 10
p (1..).size        # => Infinity
p (..1).size        # => Infinity
# それ以外の場合は全部 nil を返す
p ("a".."z").size   # => nil
p ("a"..).size      # => nil
p (.."z").size      # => nil
```


---

#### まとめ2
- - -

* begin / end と first / last はかなり違う                                        <!-- .element: class="fragment" -->
  * 更に min / max もある…                                        <!-- .element: class="fragment" -->
* 先端終端が値として欲しい場合は begin / end を使う                                        <!-- .element: class="fragment" -->
* 引数を渡す場合は first / last を使う                                        <!-- .element: class="fragment" -->
* 任意の Range が無限かどうか判定するのは地味にむずい                                        <!-- .element: class="fragment" -->
* 無限 Range の場合はうっかり無限ループしてしまう可能性があるので注意しよう                                        <!-- .element: class="fragment" -->

---

# 宣伝

---

### Ruby Hack Challenge Holiday 
- - -

* [Ruby Hack Challenge Holiday](https://rhc.connpass.com/event/140200/) というイベントをやっています
* Ruby 本体を自分でビルドしたりハックしたりするイベント       <!-- .element: class="fragment" -->
* Ruby のコミッタの方が主催しているので Ruby について聞けるチャンス       <!-- .element: class="fragment" -->
* Ruby 本体の開発に興味がある人はぜひ！！       <!-- .element: class="fragment" -->
* 次回は 09/01 開催予定       <!-- .element: class="fragment" -->


---


## ご清聴
## ありがとうございました
