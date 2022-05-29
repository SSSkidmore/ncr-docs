# Compiling Ganeti


## Dependencies

```bash

apt install autoconf automake autopoint \
    autotools-dev bridge-utils build-essential \
    cmake curl debootstrap g++ gcc ghc gzip \
    hlint hscolour kpartx libghc-adjunctions-dev \
    libghc-ansi-terminal-dev libghc-ansi-wl-pprint-dev \
    libghc-async-dev libghc-attoparsec-dev libghc-base-orphans-dev \
    libghc-base64-bytestring-dev libghc-basement-dev libghc-bifunctors-dev \
    libghc-blaze-builder-dev libghc-call-stack-dev libghc-case-insensitive-dev \
    libghc-clock-dev libghc-colour-dev libghc-comonad-dev \
    libghc-contravariant-dev libghc-cryptonite-dev libghc-curl-dev \
    libghc-distributive-dev libghc-exceptions-dev libghc-extensible-exceptions-dev \
    libghc-free-dev libghc-hashable-dev libghc-hinotify-dev \
    libghc-hostname-dev libghc-hslogger-dev libghc-hunit-dev \
    libghc-integer-logarithms-dev libghc-invariant-dev libghc-io-streams-dev \
    libghc-io-streams-haproxy-dev libghc-json-dev libghc-kan-extensions-dev \
    libghc-lens-dev libghc-lifted-base-dev libghc-memory-dev \
    libghc-monad-control-dev libghc-network-bsd-dev libghc-network-dev \
    libghc-network-uri-dev libghc-old-locale-dev libghc-old-time-dev \
    libghc-parallel-dev libghc-primitive-dev libghc-profunctors-dev \
    libghc-psqueue-dev libghc-quickcheck2-dev libghc-random-dev \
    libghc-readable-dev libghc-reflection-dev libghc-regex-base-dev \
    libghc-regex-pcre-dev libghc-regex-posix-dev libghc-scientific-dev \
    libghc-semigroupoids-dev libghc-semigroups-dev libghc-snap-core-dev \
    libghc-snap-server-dev libghc-splitmix-dev libghc-statevar-dev \
    libghc-syb-dev libghc-tagged-dev libghc-temporary-dev \
    libghc-test-framework-dev libghc-test-framework-hunit-dev libghc-test-framework-quickcheck2-dev \
    libghc-th-abstraction-dev libghc-transformers-base-dev libghc-transformers-compat-dev \
    libghc-type-equality-dev libghc-unix-compat-dev libghc-unordered-containers-dev \
    libghc-utf8-string-dev libghc-vector-dev libghc-void-dev \
    libghc-xml-dev libghc-zlib-bindings-dev libghc-zlib-dev \
    libpcre3-dev libssl-dev make m4 pandoc perl pylint pylint3 python3-coverage \
    python3-simplejson python3-openssl python3-pyparsing python3-pycurl \
    python3-pygments python3-psutil python3-paramiko python3-pyinotify \
    python3-bitarray python3-yaml python3-mock python3-pep8 python3-pycodestyle \
    python3-dev python3-sphinx qemu qemu-system qemu-system-x86 \
    qemu-utils rsync seabios ovmf socat vim wget nano unzip zlib1g-dev \
    arping drbd-utils lvm2 fping ndisc6 cabal-install graphviz pycodestyle \
    gawk

ln -s /usr/bin/pycodestyle /usr/bin/pep8


```


## Clone, Patch, Configure and Build

```bash

git clone https://github.com/ganeti/ganeti

cd ganeti

git checkout tags/v3.0.2

patch -p1 < ../rapi_osparams.patch

./autogen.sh

./configure IP_PATH=/bin/ip --prefix=/opt/ganeti/3.0.2 --localstatedir=/var \
  --sysconfdir=/etc \
  --with-export-dir=/var/lib/ganeti/export \
  --with-iallocator-search-path=/usr/local/lib/ganeti/iallocators,/usr/lib/ganeti/iallocators \
  --with-os-search-path=/srv/ganeti/os,/usr/local/lib/ganeti/os,/usr/lib/ganeti/os,/usr/share/ganeti/os \
  --with-extstorage-search-path=/srv/ganeti/extstorage,/usr/local/lib/ganeti/extstorage,/usr/lib/ganeti/extstorage,/usr/share/ganeti/extstorage \
  --docdir=/usr/share/doc/ganeti \
  --enable-restricted-commands \
  --disable-symlinks \
  --with-haskell-flags="-optl -Wl,-z,relro -optl -Wl,--as-needed" \
  --with-user-prefix=gnt- \
  --with-group-prefix=gnt- \
  --with-backup-dir=/var/backups

make -j4

make install

ls /opt/ganeti/3.0.2

```

## Users and Groups

```bash

# Groups
addgroup --quiet --system "gnt-admin"
addgroup --quiet --system "gnt-confd"
addgroup --quiet --system "gnt-daemons"
addgroup --quiet --system "gnt-luxid"
addgroup --quiet --system "gnt-masterd"
addgroup --quiet --system "gnt-metad"
addgroup --quiet --system "gnt-rapi"

# Users
adduser --quiet --system --ingroup "gnt-confd" --no-create-home --disabled-password --disabled-login --home /var/lib/ganeti "gnt-confd"
adduser --quiet --system --ingroup "gnt-confd" --no-create-home --disabled-password --disabled-login --home /var/lib/ganeti "gnt-masterd"
adduser --quiet --system --ingroup "gnt-luxid" --no-create-home --disabled-password --disabled-login --home /var/lib/ganeti "gnt-masterd"
adduser --quiet --system --ingroup "gnt-masterd" --no-create-home --disabled-password --disabled-login --home /var/lib/ganeti "gnt-masterd"
adduser --quiet --system --ingroup "gnt-metad" --no-create-home --disabled-password --disabled-login --home /var/lib/ganeti "gnt-metad"
adduser --quiet --system --ingroup "gnt-rapi" --no-create-home --disabled-password --disabled-login --home /var/lib/ganeti "gnt-rapi"

# Group memberships
adduser --quiet "gnt-confd" "gnt-daemons"
adduser --quiet "gnt-masterd" "gnt-admin"
adduser --quiet "gnt-masterd" "gnt-confd"
adduser --quiet "gnt-masterd" "gnt-daemons"
adduser --quiet "gnt-masterd" "gnt-masterd"
adduser --quiet "gnt-metad" "gnt-daemons"
adduser --quiet "gnt-rapi" "gnt-admin"
adduser --quiet "gnt-rapi" "gnt-daemons"

```


## systemd Configurations

TODO


## Initialization 

TODO