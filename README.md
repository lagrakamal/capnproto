# Tutorial: Creating Graphene manifests to run capnproto Examples:

We are going in this tutorial to explain how to install all required materials and how to create manifests to be able to run our capnproto Examples in Graphene.  

## installing Intel-SGX

To install intel-SGX just follow the instructions [https://github.com/intel/linux-sgx](here).
If you are going to run Graphene with SGX make sure you installed Intel SGX driver

## Installing Graphene-oscarlab

To install Graphene just follow the instructions [https://graphene.readthedocs.io/en/latest/quickstart.html](here).
Be sure your Ubuntu kernel is not lower than 5.9 and also you installed all "Common dependencies" before running graphene application without SGX, and "Dependencies for SGX" if you are going to run Graphene with SGX.

I worked on Ubuntu with kernel version 5.11.

### Create a simple manifest for capnproto and run the adressbook test in Graphene.

After installing all required stuff. We are going to build capnproto simple Adressbook Example provided from Graphene. 
Move to /path/to/graphene directory, then to the Examples directory and then capnproto directory.

The Path 
$Prefix/graphene/Examples/capnproto


If you take a look on Readme file, you will be asked to install the following Library: `sudo apt install -y libcapnp-dev capnproto`.

~~~build addressbook examples and the manifest
make
~~~

To enable SGX

~~~build addressbook examples and the manifest
make SGX=1
~~~

After making files a manifest will be automatic generated.
You can now run Adressbook example direct or without SGX


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

This helps to specify which application should execute in Graphene.

~~~manifest
libos.entrypoint = "[PATH]"
~~~

In our created manifest it will look something like that:

~~~manifest
libos.entrypoint = "addressbook"
~~~

##### loader.preload

This helps to specify Graphene libOS to be loaded before starting the app in Graphene.

~~~manifest
loader.preload = "file:{{ graphene.libos }}"
~~~

In our created manifest it will look something like that:

~~~manifest
loader.preload = "file:/usr/local/lib/x86_64-linux-gnu/graphene/libsydb.so"
~~~


##### Log level

~~~manifest
loader.log_level = "[none|error|warning|debug|trace|all]"
(Default: "error")

loader.log_file = "[PATH]"
~~~
Set to "debug" if you want to get information about Graphene’s operation and internals while runnig your app. it will enable all messages of type error, warning and debug

In our created manifest, the Default value will be setted:

~~~manifest
loader.log_level = "error"
~~~

##### Command-line arguments

we going to run the Graphene in a simulation mode,
so in our created manifest, it will look like the following:

~~~manifest
loader.insecure__use_cmdline_argv = true
~~~

The following will read application arguments directly from the command line, don't use this on production.

It can be specified:

~~~manifest
loader.argv_src_file = "file:file_with_serialized_argv"
~~~

##### Environment variables

This adds/overwrites a single environment variable and can be used multiple times to specify more than one variable.

~~~manifest
loader.env.[ENVIRON] = "[VALUE]"
~~~


It will look in the manifest like the following:

~~~manifest
loader.env.LD_LIBRARY_PATH = "/lib/:/lib/x86_6-linux-gnu:/usr//lib/x86_-linux-gnu"
~~~



##### FS mount point

The following syntax specifies how file systems are mounted inside the library OS. For dynamically linked binaries.

~~~manifest
fs.mount.[identifier].type = "[chroot|tmpfs]"
fs.mount.[identifier].path = "[PATH]"
fs.mount.[identifier].uri  = "[URI]"
~~~

identifier: you can choose it by yourself, but it must be unique to identfy the mount. 
chroot: All host files and sub-directories found under [URI] are forwarded to the Graphene instance and placed under [PATH]
tmpfs: Temporary in-memory-only files. These files are not backed by host-level files. The tmpfs files are created under [PATH]

usually at least one chroot mount point of Glibc library is required in the manifest:

In our created manifest it look like the following:

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

The following helps to specifies the size of the enclave set during enclave creation time.

sgx.enclave_size = "256M"

(default: 256M)


##### Number of threads

This helps to specifies the maximum number of threads that can be created inside the enclave 

sgx.thread_num = 4

(default: 4)

### How to build and run other capnproto Examples (Adressbook | calculator)

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

Then build capnproto samples (Adressbook and calculator-server and client) using the following instructions:

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

You just builded Examples "Adressbook, Calculator-server and  client" provided from capnproto.

#### build hello and test Examples to experience how capnproto function

##### Example1: printing "hello capnproto!"

compile firstly the provieded schema:

~~~compile schema
capnp compile -ocapnp ../c++/samples/hello_example/hello.capnp
~~~

~~~compile schema
@0xdc4c36538a23a17f;

struct Hello
{
    
    mystring   @0    :Text;
}
~~~

Nice, now create a directory for hello example, then make files:

~~~make files
mkdir ../hello_example
cd ../hello_example
cmake ../c++/samples/hello_example
cmake --build .
~~~

##### Example2: test Example using objects

follow the same previous instructions for test_example2:

~~~compile schema
capnp compile -ocapnp ../c++/samples/test_example2/myproto.capnp
~~~

~~~compile schema
@0x9a2868d7410f3819;

struct MyProto
{
    id @0 :UInt32;
    name @1 :Text;
    pets @2 :List(Text);
}
~~~

~~~install from git
mkdir ../test_example2
cd ../test_example2
cmake ../c++/samples/test_example2
cmake --build .
~~~

based on adressbook manifest, we can create easly needed manifests for our created Examples, to be able to run it in Graphene. 
Just change the entrypoint in the manifest to the path, where the hello_examples || test_example2 is executable.

As a result you will be able to run the Examples in Graphene.

### Creating a manifests for calculator Example.

To get an introduction what calculator Example is, take a look on capnproto documentation.

we built in previous steps the calculator example server and client.
Lets create a manifest for the calculator Example and run it in Graphene

calculator example is using a library "libcapnp". to run this in Graphene we should edit the manifest and mount the host Os directory to the library "libcapnp"

This will look like the following:

~~~manifest for calculator Example
fs.mount.libcapnp.type = "chroot"
fs.mount.libcapnp.path = "/lib"
fs.mount.libcapnp.uri = "file:/usr/local/lib"
~~~

then change the entrypoint to the calculator-server executable application:

libos.entrypoint = "path/to/calculator-server"

you can do the same steps to create a manifest for the client.

Nice, can now use graphene-direct or graphene-sgx like in the previous steps, to run the Example in Graphene.

### More information

For more informations about manifest syntax visit the Graphene official documentation[https://graphene.readthedocs.io/en/latest/quickstart.html](here).
For more Graphene Examples take visit [https://github.com/enclaive/grphn_wiki](git@grphn_wiki) and [https://github.com/enclaive/grphn_wiki/wiki](Wiki tutorials) 













