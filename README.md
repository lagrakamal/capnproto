# Tutorial: Creating Manifest for capnproto, to run it inside a graphene:

I'm trying in this tutorial to explain how to install prerequirements and create a manifest to be able to run a test appication in your Graphene.  

## installing Intel-SGX

To install intel-SGX just follow the instructions here.
If you are going to run Graphene with SGX make sure you installed Intel SGX driver

## Installing Graphene-oscarlab

To install Graphene from git just follow the instructions here.
Be sure your Ubuntu kernel is not low than 5.9 and also you installed all "Common dependencies" before running graphene application without SGX and "Dependencies for SGX" if you are going to run Graphene with SGX.

I worked on Ubuntu with kernel version 5.11.

### Create a simple manifest for capnproto and run the adressbook test in Graphene.

after installing all required stuff. We ar going to build capnproto simple Adressbook Example. Move to the Graphene directory then to the Examples directory and then capnproto.

The Path 
$Prefix/graphene/Examples/capnproto


If you take a look on Readme file, you will be asked to install the following Library: `sudo apt install -y libcapnp-dev capnproto`.

After making files a manifest will be automatic generated.
You can now run an example

~~~build addressbook examples and the manifest
make
~~~

#### run Adressbook in non-SGX Graphene

~~~run without SGX
graphene-direct addressbook write
~~~

#### run Adressbook in Graphene with SGX

~~~run with SGX
graphene-sgx addressbook write
~~~

#### Manifest syntax

##### Entrypoint
~~~manifest
libos.entrypoint = "[PATH]"
~~~

In our created manifest it will look something like that:

~~~manifest
libos.entrypoint = "addressbook"
~~~

This helps to specify which application should execute in Graphene.

##### loader.preload

~~~manifest
loader.preload = "file:{{ graphene.libos }}"
~~~

In our created manifest it will look something like that:

~~~manifest
loader.preload = "file:/usr/local/lib/x86_64-linux-gnu/graphene/libsydb.so"
~~~

This helps to specify graphene libOS to be loaded before starting the app in Graphene.


##### Log level

~~~manifest
loader.log_level = "[none|error|warning|debug|trace|all]"
(Default: "error")

loader.log_file = "[PATH]"
~~~

In our created manifest, the Default value will be setted:

~~~manifest
loader.log_level = "error"
~~~

##### Command-line arguments

we going to run the Graphene in a simulation mode,
so in our created manifest, it will look something like that:

~~~manifest
loader.insecure__use_cmdline_argv = true
~~~

This will read application arguments directly from the command line, don't use this on production.

It can be specified:

~~~manifest
loader.argv_src_file = "file:file_with_serialized_argv"
~~~

##### Environment variables!!!!!!!!!


~~~manifest
loader.env.[ENVIRON] = "[VALUE]"
~~~

this adds/overwrites a single environment variable and can be used multiple times to specify more than one variable.



~~~manifest
loader.env.LD_LIBRARY_PATH = "/lib/:/lib/x86_6-linux-gnu:/usr//lib/x86_-linux-gnu"
~~~
this will specify paths to search for libraries


##### FS mount point

~~~manifest
fs.mount.[identifier].type = "[chroot|tmpfs]"
fs.mount.[identifier].path = "[PATH]"
fs.mount.[identifier].uri  = "[URI]"
~~~

This syntax specifies how file systems are mounted inside the library OS. For dynamically linked binaries.

identifier: you can choose it by your self it must be unique to identfy the mount. 
chroot: All host files and sub-directories found under [URI] are forwarded to the Graphene instance and placed under [PATH]
tmpfs: Temporary in-memory-only files. These files are not backed by host-level files. The tmpfs files are created under [PATH]

usually at least one chroot mount point of Glibc library is required in the manifest:

In our created manifest it look something like that:

fs.mount.lib.type = "chroot"
fs.mount.lib.path = "/lib"
fs.mount.lib.uri = "file:/usr/local/lib/x86_64-linux-gnu/graphene/runtime/glibc"



###### Python for example:

The recommended usage is to provide an absolute path, and mount the executable at the entrypoint.

~~~manifest
libos.entrypoint = "/usr/bin/python3.8"

fs.mount.python.type = "chroot"
fs.mount.python.path = "/usr/bin/python3.8"
fs.mount.python.uri = "file:/usr/bin/python3.8"
# Or, if using a binary from your local directory:
# fs.mount.python.uri = "file:python3.8"
~~~

##### Enclave size

sgx.enclave_size = "256M"

This helps to specifies the size of the enclave set during enclave creation time. (default: 256M)


##### Number of threads

sgx.thread_num = 4

This helps to specifies the maximum number of threads that can be created inside the enclave (default: 4)

### How to run the other capnproto Examples

1. Install Cap'n Proto in ($PREFIX below):
Example : /path/to/graphene/Examples/capnproto

~~~install from git
git clone https://github.com/sandstorm-io/capnproto.git (rename src for simplicity)
cd src/c++
autoreconf -i
./configure
make -j6 check
sudo make install
~~~

to build capnproto samples (Adressbook and calculator-server and client)

~~~install from git
mkdir src/build
cd src/build
cmake ../c++ -DCMAKE_INSTALL_PREFIX=$PREFIX
cmake --build . --target install
~~~

~~~install from git
export PATH=$PREFIX/bin:$PATH
mkdir ../build-samples
cd ../build-samples
cmake ../c++/samples
cmake --build .
~~~

to build (hello and test Examples)

~~~install from git
mkdir ../hello_example
cd ../hello_example
cmake ../c++/samples/hello_example
cmake --build .
~~~

~~~install from git
mkdir ../test_example2
cd ../test_example2
cmake ../c++/samples/test_example2
cmake --build .
~~~

based on adressbook manifest, we create manifests for our created Examples to be able to run it in graphene.









