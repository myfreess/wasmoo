# 安装Wasm版的MoonBit工具链

目前native的MoonBit工具链支持三个主流平台：x86 Windows, x86 Linux, arm Darwin

得益于`wasm_of_ocaml`项目，现在我们可以将MoonBit的编译器工具链编译为wasm文件(使用一个nodejs脚本启动)，在支持nodejs的平台上都可以使用。

本文主要面向x86 Darwin(Intel Mac)用户与arm Linux用户，x86 Darwin平台因为过于老旧，目前已经不再提供native的MoonBit工具链，而arm Linux平台的MoonBit工具链暂时还未添加

## 前置需求

安装bash, curl, git, nodejs以及rust工具链, 然后按照以下说明依次执行相应命令。

由于wasm_of_ocaml编译出的wasm文件需要支持wasm-gc的运行时才能正常运行，在安装nodejs时需要选择较高版本。

## 下载Wasm版MoonBit工具链

首先在任意位置新建一个临时目录，然后执行以下命令下载最新的Wasm版MoonBit工具链压缩包:

```shell
curl -fSL -O https://github.com/moonbitlang/moonbit-compiler/releases/latest/download/moonbit-wasm.tar.gz
tar -zxvf moonbit-wasm.tar.gz
```

接着需要设置`MOON_HOME`环境变量，这个环境变量的值是存放MoonBit工具链相关文件的目录，默认为`~/.moon`。

```shell
export MOON_HOME="$HOME/.moon"
```

## 安装对应版本moon

MoonBit的构建系统`moon`使用Rust编写并已经开源，自行构建安装即可。关键在于需要安装到正确的版本(因为构建系统和编译器的许多地方是强耦合的)，在wasm版MoonBit工具链的压缩包里有一个文件`moon_version`记录了该版本`moon`对应的git commit sha，使用`git reset --hard`即可切换到对应commit。

moonrun是MoonBit官方提供的wasm运行时，为了方便跑测试我们把它也安装上。

```shell
mkdir -p $HOME/.moon
MOON_VERSION=$(cat ./moon_version)
BIN_DIR="$MOON_HOME/bin"
mkdir -p "$BIN_DIR"
git clone https://github.com/moonbitlang/moon
pushd moon
git reset --hard "$MOON_VERSION"
cargo build --release
cp target/release/moon "$BIN_DIR"
cp target/release/moonrun "$BIN_DIR"
popd
```

## 安装moonc/moonfmt/mooninfo/运行时依赖

```shell
sed -i '1 i #!'"$(which env) -S node --stack-size=4096" moonc.js
sed -i '1 i #!'"$(which env) -S node --stack-size=4096" moonfmt.js
sed -i '1 i #!'"$(which env) -S node --stack-size=4096" mooninfo.js
cp moonc.js moonfmt.js mooninfo.js moonc.assets moonfmt.assets mooninfo.assets "$BIN_DIR" -r
mv "$BIN_DIR/moonc.js" "$BIN_DIR/moonc"
mv "$BIN_DIR/moonfmt.js" "$BIN_DIR/moonfmt"
mv "$BIN_DIR/mooninfo.js" "$BIN_DIR/mooninfo"
chmod +x "$BIN_DIR/moonc"
chmod +x "$BIN_DIR/moonfmt"
chmod +x "$BIN_DIR/mooninfo"
cp lib include "$MOON_HOME" -r
```

## 安装core

> 由于目前moonc实现bundle的机制，wasm版的moonc所产生的.core文件会和native版本有一定不同. 

```shell
tar -xf core.tar.gz --directory="$MOON_HOME/lib/core"
pushd "$MOON_HOME/lib/core"
moon bundle --target all
popd
```

## 将`$HOME/.moon/bin`添加到`PATH`

+ bash

```shell
echo "export PATH=\"$MOON_HOME/bin:"'$PATH"' >> ~/.bashrc
```

+ zsh

```shell
echo "export PATH=\"$MOON_HOME/bin:"'$PATH"' >> ~/.zshrc
```

## 总结