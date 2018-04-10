# Installing Facebook-Infer on Ubuntu

If you use Mac, it is easy

```
brew install infer
```

But if you use ubuntu, it is a long way to go.

1. download the source code

```
https://github.com/facebook/infer/releases/tag/v0.13.1
```

2. following the instruction here

```
https://my.oschina.net/hibony/blog/690740
```

3. Basically, you need

```
opam 1.2.2
Python 2.7
pkg-config
Java (only needed for the Java analysis)
gcc >= 4.7.2 or clang >= 3.1 (only needed for the C/Objective-C analysis)
autoconf >= 2.63 and automake >= 1.11.1 (if building from git)
```

then run `./configure`, to check what the depency need and install by

```
sudo opam install XXX

Such as atdgen, javalib、oUnit、extlib、camlzip、atdgen、sawja, sqlite
```

4. when configure is done

```
sudo ./build-infer.sh
sudo make install
```

5. to use it, check here: https://github.com/facebook/infer/tree/master/examples