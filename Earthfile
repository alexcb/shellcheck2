aarch64-linux-gnu-deps:
    FROM ubuntu:20.04
    
    ENV TARGET aarch64-linux-gnu
    ENV TARGETNAME linux.aarch64
    
    # Build dependencies
    USER root
    ENV DEBIAN_FRONTEND noninteractive
    RUN apt-get update && apt-get install -y ghc automake autoconf build-essential llvm curl file qemu-user-static gcc-$TARGET
    
    # Build GHC
    WORKDIR /ghc
    RUN curl -L "https://downloads.haskell.org/~ghc/8.10.4/ghc-8.10.4-src.tar.xz" | tar xJ --strip-components=1
    RUN ./boot && ./configure --host x86_64-linux-gnu --build x86_64-linux-gnu --target "$TARGET"
    RUN cp mk/flavours/quick-cross.mk mk/build.mk && make -j "$(nproc)"
    RUN make install
    RUN curl -L "https://downloads.haskell.org/~cabal/cabal-install-3.2.0.0/cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz" | tar xJv -C /usr/local/bin
    
    # Due to an apparent cabal bug, we specify our options directly to cabal
    # It won't reuse caches if ghc-options are specified in ~/.cabal/config
    ENV CABALOPTS "--ghc-options;-split-sections -optc-Os -optc-Wl,--gc-sections;--with-ghc=$TARGET-ghc;--with-hc-pkg=$TARGET-ghc-pkg"
    
    # Prebuild the dependencies
    RUN cabal update && IFS=';' && cabal install $CABALOPTS --lib Diff-0.4.0 base-compat-0.11.2 base-orphans-0.8.4 dlist-1.0 hashable-1.3.0.0 indexed-traversable-0.1.1 integer-logarithms-1.0.3.1 primitive-0.7.1.0 regex-base-0.94.0.0 splitmix-0.1.0.3 tagged-0.8.6.1 th-abstraction-0.4.2.0 transformers-compat-0.6.6 base-compat-batteries-0.11.2 time-compat-1.9.5 unordered-containers-0.2.13.0 data-fix-0.3.1 vector-0.12.2.0 scientific-0.3.6.2 regex-tdfa-1.3.1.0 random-1.2.0 distributive-0.6.2.1 attoparsec-0.13.2.5 uuid-types-1.0.3 comonad-5.0.8 bifunctors-5.5.10 assoc-1.0.2 these-1.1.1.1 strict-0.4.0.1 aeson-1.5.5.1

    RUN cabal update
    
    # Copy the build script
    #COPY build /usr/bin

    # push up a new layer so it can be cached for future runs
    SAVE IMAGE --push alexcb/test-shellcheck:aarch64-linux-gnu-deps

x86-64-linux-deps:
    FROM ubuntu:20.04
    
    ENV TARGETNAME linux.x86_64
    
    # Install GHC and cabal
    USER root
    ENV DEBIAN_FRONTEND noninteractive
    RUN apt-get update && apt-get install -y ghc curl file xz-utils
    
    # So we'd like a later version of Cabal that supports --enable-executable-static,
    # but we can't use Ubuntu 20.10 because coreutils has switched to new syscalls that
    # the TravisCI kernel doesn't support. Download it manually.
    RUN curl "https://downloads.haskell.org/~cabal/cabal-install-3.2.0.0/cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz" | tar xJv -C /usr/bin
    
    # Use ld.bfd instead of ld.gold due to
    # x86_64-linux-gnu/libpthread.a(pthread_cond_init.o)(.note.stapsdt+0x14): error:
    #    relocation refers to local symbol "" [2], which is defined in a discarded section
    ENV CABALOPTS "--ghc-options;-optl-Wl,-fuse-ld=bfd -split-sections -optc-Os -optc-Wl,--gc-sections"
    
    # Other archs pre-build dependencies here, but this one doesn't to detect ecosystem movement
    
    RUN cabal update

    # push up a new layer so it can be cached for future runs
    SAVE IMAGE --push alexcb/test-shellcheck:x86-64-linux-deps

x86-64-windows-deps:
    FROM ubuntu:20.04
    
    ENV TARGETNAME windows.x86_64
    
    # We don't need wine32, even though it complains
    USER root
    ENV DEBIAN_FRONTEND noninteractive
    RUN apt-get update && apt-get install -y curl file busybox wine
    
    # Fetch Windows version, will be available under z:\haskell
    WORKDIR /haskell
    RUN curl -L "https://downloads.haskell.org/~ghc/8.10.4/ghc-8.10.4-x86_64-unknown-mingw32.tar.xz" | tar xJ --strip-components=1
    WORKDIR /haskell/bin
    RUN curl -L "https://downloads.haskell.org/~cabal/cabal-install-3.2.0.0/cabal-install-3.2.0.0-x86_64-unknown-mingw32.zip" | busybox unzip -
    RUN curl -L "https://curl.se/windows/dl-7.75.0/curl-7.75.0-win64-mingw.zip" | busybox unzip - && mv curl-7.75.0-win64-mingw/bin/* .
    ENV WINEPATH /haskell/bin
    
    # It's unknown whether Cabal on Windows suffers from the same issue
    # that necessitated this but I don't care enough to find out
    ENV CABALOPTS "--ghc-options;-split-sections -optc-Os -optc-Wl,--gc-sections"
    
    # Precompile some deps to speed up later builds. This list is just copied from `cabal build`
    RUN wine /haskell/bin/cabal.exe update && IFS=';' && wine /haskell/bin/cabal.exe install $CABALOPTS --lib Diff-0.4.0 base-compat-0.11.2 base-orphans-0.8.4 dlist-1.0 hashable-1.3.0.0 indexed-traversable-0.1.1 integer-logarithms-1.0.3.1 primitive-0.7.1.0 regex-base-0.94.0.0 splitmix-0.1.0.3 tagged-0.8.6.1 th-abstraction-0.4.2.0 transformers-compat-0.6.6 base-compat-batteries-0.11.2 time-compat-1.9.5 unordered-containers-0.2.13.0 data-fix-0.3.1 vector-0.12.2.0 scientific-0.3.6.2 regex-tdfa-1.3.1.0 random-1.2.0 distributive-0.6.2.1 attoparsec-0.13.2.5 uuid-types-1.0.3 comonad-5.0.8 bifunctors-5.5.10 assoc-1.0.2 these-1.1.1.1 strict-0.4.0.1 aeson-1.5.5.1
    
    RUN wine /haskell/bin/cabal.exe update
    
    # push up a new layer so it can be cached for future runs
    SAVE IMAGE --push alexcb/test-shellcheck:windows-deps

x86-64-darwin-deps:
    FROM liushuyu/osxcross:latest
    
    ENV TARGET x86_64-apple-darwin18
    ENV TARGETNAME darwin.x86_64
    
    # Build dependencies
    USER root
    ENV DEBIAN_FRONTEND noninteractive
    RUN apt-get update && apt-get install -y ghc automake autoconf llvm curl file
    
    # Build GHC
    WORKDIR /ghc
    RUN curl -L "https://downloads.haskell.org/~ghc/8.10.4/ghc-8.10.4-src.tar.xz" | tar xJ --strip-components=1
    RUN ./boot && ./configure --host x86_64-linux-gnu --build x86_64-linux-gnu --target "$TARGET"
    RUN cp mk/flavours/quick-cross.mk mk/build.mk && make -j "$(nproc)"
    RUN make install
    RUN curl -L "https://downloads.haskell.org/~cabal/cabal-install-3.2.0.0/cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz" | tar xJv -C /usr/local/bin
    
    # Due to an apparent cabal bug, we specify our options directly to cabal
    # It won't reuse caches if ghc-options are specified in ~/.cabal/config
    ENV CABALOPTS "--with-ghc=$TARGET-ghc;--with-hc-pkg=$TARGET-ghc-pkg"
    
    # Prebuild the dependencies
    RUN cabal update && IFS=';' && cabal install $CABALOPTS --lib Diff-0.4.0 base-compat-0.11.2 base-orphans-0.8.4 dlist-1.0 hashable-1.3.0.0 indexed-traversable-0.1.1 integer-logarithms-1.0.3.1 primitive-0.7.1.0 regex-base-0.94.0.0 splitmix-0.1.0.3 tagged-0.8.6.1 th-abstraction-0.4.2.0 transformers-compat-0.6.6 base-compat-batteries-0.11.2 time-compat-1.9.5 unordered-containers-0.2.13.0 data-fix-0.3.1 vector-0.12.2.0 scientific-0.3.6.2 regex-tdfa-1.3.1.0 random-1.2.0 distributive-0.6.2.1 attoparsec-0.13.2.5 uuid-types-1.0.3 comonad-5.0.8 bifunctors-5.5.10 assoc-1.0.2 these-1.1.1.1 strict-0.4.0.1 aeson-1.5.5.1
    RUN cabal update

    # push up a new layer so it can be cached for future runs
    SAVE IMAGE --push alexcb/test-shellcheck:darwin-deps


test-aarch64-linux:
    FROM +aarch64-linux-gnu-deps

    WORKDIR /scratch
    COPY --dir * .
    RUN rm -rf .git
    RUN cabal test
    RUN ./striptests
    RUN cabal update
    RUN mkdir "$TARGETNAME"
    RUN ( IFS=';'; cabal build $CABALOPTS --enable-executable-static )
    RUN find . -name shellcheck -type f -exec mv {} "$TARGETNAME/" ";"
    RUN ls -l "$TARGETNAME"
    RUN "$TARGET-strip" -s "$TARGETNAME/shellcheck"
    RUN ls -l "$TARGETNAME"
    RUN qemu-aarch64-static "$TARGETNAME/shellcheck" --version
    RUN file "$TARGETNAME/shellcheck"


test-x86-64-linux:
    FROM +x86-64-linux-deps

    WORKDIR /scratch
    COPY --dir * .
    RUN rm -rf .git
    RUN cabal test
    RUN ./striptests
    RUN mkdir "$TARGETNAME"
    RUN cabal update
    RUN ( IFS=';'; cabal build $CABALOPTS --enable-executable-static )
    RUN find . -name shellcheck -type f -exec mv {} "$TARGETNAME/" ";"
    RUN ls -l "$TARGETNAME"
    RUN strip -s "$TARGETNAME/shellcheck"
    RUN ls -l "$TARGETNAME"
    RUN "$TARGETNAME/shellcheck" --version
    RUN file "$TARGETNAME/shellcheck"

test-x86-64-windows:
    FROM +x86-64-windows-deps

    WORKDIR /scratch
    COPY --dir * .
    RUN rm -rf .git
    RUN wine /haskell/bin/cabal.exe test
    RUN ./striptests
    RUN mkdir "$TARGETNAME"
    RUN wine /haskell/bin/cabal.exe update
    RUN ( IFS=';'; wine /haskell/bin/cabal.exe build $CABALOPTS )
    RUN find dist*/ -name shellcheck.exe -type f -ls -exec mv {} "$TARGETNAME/" ";"
    RUN ls -l "$TARGETNAME"
    RUN wine "/haskell/mingw/bin/strip.exe" -s "$TARGETNAME/shellcheck.exe"
    RUN ls -l "$TARGETNAME"
    RUN wine "$TARGETNAME/shellcheck.exe" --version

test-x86-64-darwin:
    FROM +x86-64-darwin-deps

    WORKDIR /scratch
    COPY --dir * .
    RUN rm -rf .git
    RUN cabal test
    RUN ./striptests
    RUN mkdir "$TARGETNAME"
    RUN cabal update
    RUN ( IFS=';'; cabal build $CABALOPTS )
    RUN find . -name shellcheck -type f -exec mv {} "$TARGETNAME/" ";"
    RUN ls -l "$TARGETNAME"
    RUN "$TARGET-strip" -Sx "$TARGETNAME/shellcheck"
    RUN ls -l "$TARGETNAME"

all:
    BUILD +test-x86-64-darwin
    BUILD +test-x86-64-windows
    BUILD +test-x86-64-linux
    BUILD +test-aarch64-linux
