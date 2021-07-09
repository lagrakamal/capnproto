# Tutorial: Creating Manifest for capnproto, to run it inside a graphene:

I'm trying in this tutorial to explain how to install required stuff and how to create a manifest to be able to run a test appication secured by "Graphene-Oscarlab".  

## installing Intel-SGX

To install intel-SGX just follow the instructions here.
If you are going to run graphene with SGX make sure you installed Intel SGX driver

## Installing Graphene-oscarlab

To install graphene just follow the instructions here.
Be sure your ubuntu kernel is not low than 5.9 and you installed all "Common dependencies" before running graphene application without SGX and "Dependencies for SGX" if you are going to run graphene with SGX.

I tested the graphene on ubuntu with kernel version 5.11.



### Create a simple manifest for capnproto and run the adressbook test in Graphene.

after you installed all required stuff. Just move to the Graphene directory where is capnproto Example. The path will be like that:

~/graphene/Examples/capnproto


and if you take a look on Readme file, you will be asked to install the following prerequisites: `sudo apt install -y libcapnp-dev capnproto`.

After making files a manifest will be automatic generated.
You can now run an example

#### run Adressbook in non-SGX Graphene

~~~run without SGX
graphene-direct addressbook write
~~~

#### run Adressbook in Graphene with SGX

~~~run with SGX
graphene-sgx addressbook write
~~~

#### Entrypoint
~~~manifest
libos.entrypoint = "[PATH]"
~~~

In our created manifest it will look something like that:

~~~manifest
libos.entrypoint = {{ adressbook }}
~~~

This helps to specify which application should execute in Graphene.

#### loader.preload

~~~manifest
loader.preload = "file:{{ graphene.libos }}"
~~~

In our created manifest it will look something like that:

~~~manifest
loader.preload = "file:/usr/local/lib/x86_64-linux-gnu/graphene/libsydb.so"
~~~

This helps to specify which library should be loaded before starting the app in Graphene.

### Command-line arguments

we going to run the Graphene in a simultion mode,
So in our created manifest, it will look something like that:

~~~manifest
loader.insecure__use_cmdline_argv = true
~~~

This will read application arguments directly from the command line, don't use this on production.

It can be specified:

~~~manifest
loader.argv_src_file = "file:file_with_serialized_argv"
~~~

### Environment variables


~~~manifest
loader.env.[ENVIRON] = "[VALUE]"
~~~

this adds/overwrites a single environment variable and can be used multiple times to specify more than one variable.



~~~manifest
loader.env.LD_LIBRARY_PATH = "/lib/:/lib/x86_6-linux-gnu:/usr//lib/x86_-linux-gnu"
~~~
this will specify paths to search for libraries


### Log level

~~~manifest
loader.log_level = "[none|error|warning|debug|trace|all]"
(Default: "error")

loader.log_file = "[PATH]"
~~~

In our created manifest, the Default value will be setted:

~~~manifest
loader.log_level = "error"
~~~


