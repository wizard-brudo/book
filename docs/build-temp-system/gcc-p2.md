
{{ include('../packages/gcc/README.md') }}

## Подготовка

???+ warning "Предупреждение"
	Распаковка указанных пакетов должна производиться из каталога `{{ include('../packages/gcc/.name') }}`. Проверьте текущее местоположение, прежде чем выполнить команды ниже.

```bash
tar -xf ../{{ include('../packages/mpfr/.filename') }}
mv -v {{ include('../packages/mpfr/.name') }} {{ include('../packages/mpfr/.name_short') }}
tar -xf ../{{ include('../packages/gmp/.filename') }}
mv -v {{ include('../packages/gmp/.name') }} {{ include('../packages/gmp/.name_short') }}
tar -xf ../{{ include('../packages/mpc/.filename') }}
mv -v {{ include('../packages/mpc/.name') }} {{ include('../packages/mpc/.name_short') }}
tar -xf ../{{ include('../packages/isl/.filename') }}
mv -v {{ include('../packages/isl/.name') }} {{ include('../packages/isl/.name_short') }}
```

Смените пути установки библиотек:

```bash
sed -e '/m64=/s/lib64/lib/' \
    -e '/m32=/s/m32=.*/m32=..\/lib32$(call if_multiarch,:i386-linux-gnu)/' \
    -i.orig gcc/config/i386/t-linux64
```

Пакет GCC требует использовать отдельную директорию для сборки. Создайте её:

```bash
mkdir -v build
cd       build
```

Разрешим сборку `libgcc` с поддержкой многопоточности:

```bash
mkdir -pv $LIN_TGT/libgcc
ln -s ../../../libgcc/gthr-posix.h $LIN_TGT/libgcc/gthr-default.h
```

Настройка:

```bash
../configure                   \
    --build=$(../config.guess) \
    --host=$LIN_TGT            \
    --prefix=/usr              \
    CC_FOR_TARGET=$LIN_TGT-gcc \
    --with-build-sysroot=$LIN  \
    --enable-initfini-array    \
    --disable-nls              \
    --disable-decimal-float    \
    --disable-libatomic        \
    --disable-libgomp          \
    --disable-libquadmath      \
    --disable-libssp           \
    --disable-libvtv           \
    --disable-libstdcxx        \
    --enable-languages=c,c++   \
    --disable-multilib
```

### Для multilib

замените параметр `--disable-multilib` на `--enable-multilib --with-multilib-list=m64,m32`

### Значения параметров

`--enable-initfini-array` - параметр заставляет использовать некоторые внутренние структуры данных, которые необходимы, но не могут быть обнаружены при построении кросс-компилятора.

`--disable-decimal-float, --disable-threads, --disable-libatomic, --disable-libgomp, --disable-libquadmath, --disable-libssp, --disable-libvtv, --disable-libstdcxx` - параметр отключают поддержку десятичных расширений с плавающей запятой, потоковой передачи, libatomic, libgomp, libquadmath, libssp, libvtv и стандартной библиотеки C++ соответственно. Эти функции не будут скомпилированы при сборке кросс-компилятора и не являются необходимыми для кросс-компиляции временной libc.

`--enable-languages​​=c,c++` - опция включает поддержку компиляторов C и C++. Это единственные языки, которые нужны сейчас.

## Сборка

```bash
make
```

## Установка

```bash
make DESTDIR=$LIN install
```

Некоторые программы используют команду `cc`, а не `gcc`. Создайте символическую ссылку на `gcc`:

```bash
ln -sv gcc $LIN/usr/bin/cc
```
