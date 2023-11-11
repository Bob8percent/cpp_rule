# cpp_rule

<details><summary>

## templateの型推論
```c++
template <typename T>
void func(ParamType Type);
```
4つ注意点がある

</summary><div>

### ParamTypeが参照, ポインタの場合(ユニヴァーサル参照ではない)
```c++
// 参照
template <typename T>
void func(T& Type);

int x = 27; func(x);			// T: int, ParamType: int&
const int cx = x; func(cx);		// T: const int, ParamType const int&
const int& rx = cx; func(rx);	// T: const int, ParamType const int&

// ポインタ
template <typename T>
void func(T* Type);

int x = 27; func(&x);			// T: int, ParamType: int*
const int* p = x; func(p);		// T: const int, ParamType: const int*
```
とても直観的。

### ParamTypeがユニヴァーサル参照の場合
```c++
// ユニヴァーサル参照
template <typename T>
void func(T&& Type);

// 左辺値
int x = 27; func(x);			// T: int, ParamType: int&
const int cx = x; func(cx);		// T: const int, ParamType: const int&
const int& rx = x; func(rx);	// T: const int&, ParamType: const int&
// 右辺値
func(27);						// T: int, ParamType: int&&
```
特殊なのは、`PamamType`が左辺値の場合。

### ParamTypeが値渡しの場合
```c++
// 値渡し
template <typename T>
void func(T Type);

int x = 27; func(x);			// T and ParamType: int
const int cx = x; func(cx);		// T and ParamType: int
const int& rx = x; func(rx);	// T and ParamType: int
const* char const p = "aaa"; func(p);	// T and ParamType: const char* (pのconst性が無視)
```
値渡しの場合、仮引数は実引数のコピーなので別物なので、参照性/const/volatileは無視される

### 配列型, 関数型はポインタ型に推論される
```c++
// 値渡し
template <typename T>
void func(T Type);

// 配列型, ポインタ型
const char name[] = "Sato Jonathan";
func(name);	// T and ParamType: const char*

// 関数型, ポインタ型
void somefunc(int, double);	// 型はvoid(int, double)
func(somefunc);				// T and ParamType: void(*)(int, double)

// 参照で解決！
template <typename T>
void func(T& Type);

func(name);		// T and ParamType: const char[14]
func(somefunc);	// T and ParamType: void(&)(int, double)
```
配列型や関数型は参照渡しによってポインタ型に変換されない
</div></details>

<details><summary>

## autoの型推論
```c++
int x = 0;
auto&& uref = x;	// uref: int&
```
1点を除いて、templateと挙動は同じ。

</summary><div>

### intializer_listの挙動

> autoの場合
```c++
auto x1 = 27;	// int
auto x2(27);	// int
auto x3{27};	// int
auto x4 = {27};	// initializer_list<int>
```
> templateの場合
```c++
template<typename T>
func1(T Type);

template<typename T>
func2(initializer<T> Type);

func1({1, 3, 5});	// error
func2({1, 3, 5});	// OK
```
templateはinitializer_listを推論できない

### 例外
```c++
// 関数
auto func()	// error
{
	return {1, 3, 5};
}

// ラムダ式
auto l2 = [&v](const auto& newVal){ v = newVal;};
l2({1, 3, 5});	// error
```
関数の戻り値, 仮引数に`auto`を使うと`initializer_list`を推論できない == `template`と挙動が同じ
</div></details>

<details><summary>

## decltypeの型推論
```c++
int& i;	// decltype(i): int&
```
decltypeの主要用途と、1つの注意点。

</summary><div>

### 主要用途
autoの推論の規則をdecltypeの規則にする
```c++
// auto の規則により参照が外れる(戻り値の型: int)
auto authAndAccess(std::vector<int>& v, std::size_t i)
{
	return v[i];
}

// decltype の規則によって推論する(int&)
decltype(auto) authAndAccess(std::vector<int>& v, std::size_t i)	// 戻り値の型はdecltype(v[i])
{
	return v[i];
}
```

### 注意点
名前でなく, (複雑な)左辺値式を仮引数とした場合、参照型となる
```c++
// 名前の場合, 戻り値の型はint
decltype(auto) func1()
{
	int x = 0;
	return x;	// decltype(x): int
}

// 複雑な左辺値式の場合, 戻り値の型はint&
decltype(auto) func2()
{
	int x = 0;
	return (x);	// decltype((x)): int&
}
```
</div></details>

<details><summary>

## autoのメリット/デメリット
`auto`の使用するべきタイミングを見極める

</summary><div>

- メリット
	- 初期化の強制
	```c++
	int x;		// 初期化しなくてもコンパイルできる
	auto x1;	// error!
	auto x2 = 0;	// OK!
	```
	- 型の不一致を防げる
	```c++
 	std::unordered_map<std::string, int> m;

 	// 型の不一致! 暗黙の型変換が発生してしまう。
 	for(const std::pair<std::string, int>& e : m)	// std::unordered_mapのキーがconstになる仕様を知らなかった…
 	{
		...
 	}
 	// OK!
 	for(const auto& e : m)
 	{
		...
 	}
	```
	- リファクタリングを容易にする
	```c++
 	auto func()
 	{
 		// 変更前
		//int x = 0;
 		// 変更後
 		long x = 0;

 		return x;
 	}
 	auto gx = func();	// ここを書き換える必要がない
	```
 - デメリット
	- 推論規則を理解しないといけない([autoの型推論](URL "#autoの型推論"), [templateの型推論](URL "#templateの型推論"))
 
</div></details>

## 引用文献
Effective Modern C++　―C++11/14プログラムを進化させる42項目　Scott Meyers著、千住 治郎訳
