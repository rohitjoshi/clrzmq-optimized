# clrzmq-optimized &mdash; optimized 0MQ Bindings for .NET and Mono

-----------------------------------------------------------
Update: These changes have been merged with master branch. So please use master branch https://github.com/zeromq/clrzmq
----------------------------------------------------------
Welcome to the clrzmq-optimized wiki!

This forked version of clrzmq is optimized for low end devices. You can see the different in CPU usage.

clrzmq:

https://docs.google.com/open?id=0Bz_UeFnokXJVZVdNV1ZMWDJsWDg

SocketProxy.Receive() uses 13142.42 CPU ticks

clrzmq-optimized:

https://docs.google.com/open?id=0Bz_UeFnokXJVekI4OEhNN3pUR1E

SocketProxy.Receive() uses 2696.54 CPU ticks

This project aims to provide the full functionality of the underlying ZeroMQ API to CLR projects.

Bundled libzmq version: **3.2.1-beta2**  
Legacy libzmq version supported: **2.2.0 (stable)**

## Getting Started

The quickest way to get started with clrzmq is by using the [NuGet package][clrzmq-nuget]. The NuGet packages include a copy of the native libzmq.dll, which is required to use clrzmq.

You may also build clrzmq directly from the source. See the Development Environment Setup instructions below for more detail.

To get an idea of how to use clrzmq, have a look at the following example.

### Example server

```c#
using System;
using System.Text;
using System.Threading;
using ZMQ;

namespace ZMQGuide
{
    class Program
    {
        static void Main(string[] args)
        {
            // ZMQ Context, server socket
            using (ZmqContext context = ZmqContext.Create())
            using (ZmqSocket server = context.CreateSocket(SocketType.REP))
            {
                server.Bind("tcp://*:5555");
                
                while (true)
                {
                    // Wait for next request from client
                    string message = server.Receive(Encoding.Unicode);
                    Console.WriteLine("Received request: {0}", message);

                    // Do Some 'work'
                    Thread.Sleep(1000);

                    // Send reply back to client
                    server.Send("World", Encoding.Unicode);
                }
            }
        }
    }
}
```

### Example client

```c#
using System;
using System.Text;
using ZMQ;

namespace ZMQGuide
{
    class Program
    {
        static void Main(string[] args)
        {
            // ZMQ Context and client socket
            using (ZmqContext context = ZmqContext.Create())
            using (ZmqSocket client = context.CreateSocket(SocketType.REQ))
            {
                client.Connect("tcp://localhost:5555");

                string request = "Hello";
                for (int requestNum = 0; requestNum < 10; requestNum++)
                {
                    Console.WriteLine("Sending request {0}...", requestNum);
                    client.Send(request, Encoding.Unicode);

                    string reply = client.Receive(Encoding.Unicode);
                    Console.WriteLine("Received reply {0}: {1}", requestNum, reply);
                }
            }
        }
    }
}
```

More C# examples can be found in the [0MQ Guide][zmq-guide] or in the [examples repository][zmq-example-repo]. Tutorials and API documentation specific to clrzmq are on the way.

## Development Environment

On Windows/.NET, clrzmq is developed with Visual Studio 2010. Mono development is done with MonoDevelop 2.8+.

### Windows/.NET

clrzmq depends on `libzmq.dll`, which will be retrieved automatically via NuGet. If you require a specific version of libzmq, you can compile it from the [0MQ sources][libzmq].

#### clrzmq

1. Clone the source.
2. Run `build.cmd` to build the project and run the test suite.
3. The resulting binaries will be available in `/build`.

#### Alternate libzmq (optional)

If you want to use a custom build of `libzmq.dll`, perform the following steps:

1. Delete or rename the `src/.nuget/packages.config` file. This prevent the NuGet package from being retrieved.
2. Remove any folders matching `src/packages/libzmq-*` that may have been downloaded previously.
3. Copy the 32-bit and 64-bit (if applicable) build of `libzmq.dll` to `lib/x86` and `lib/x64`, respectively.

Note that PGM-related tests will fail if a non-PGM build of libzmq is used.

### Mono

**NOTE**: Mono 2.10.7+ is required **for development only**, as the NuGet scripts and executables require this version to be present.
If you choose to install dependencies manually, you may use any version of Mono 2.6+.

#### Mono 2.10.7+ configuration

NuGet relies on several certificates to be registered with Mono. The following is an example terminal session (on Ubuntu) for setting this up correctly.
This assumes you have already installed Mono 2.10.7 or higher.

```shell
$ mozroots --import --sync

$ certmgr -ssl https://go.microsoft.com
$ certmgr -ssl https://nugetgallery.blob.core.windows.net
$ certmgr -ssl https://nuget.org
```

This should result in a working Mono setup for use with NuGet.

#### libzmq

Either clone the [ZeroMQ repository][libzmq] or [download the sources][zmq-dl], and then follow the build/install instructions for your platform.
Use the `--with-pgm` option if possible.

#### clrzmq

1. Clone the source.
2. Run `nuget.sh`, which downloads any dependent packages (e.g., Machine.Specifications for acceptance tests).
3. Run `make` to build the project.
4. The resulting binaries will be available in `/build`.

**NOTE**: The combination of 0MQ, MSpec, and Mono currently has issues, so the test suite does not automatically run.  
**NOTE**: `clrzmq` only supports x86 builds on Mono at this time

## Issues

Issues should be logged on the [GitHub issue tracker][issues] for this project.

When reporting issues, please include the following information if possible:

* Version of clrzmq and/or how it was obtained (compiled from source, NuGet package)
* Version of libzmq being used
* Runtime environment (.NET/Mono and associated version)
* Operating system and platform (Win7/64-bit, Linux/32-bit)
* Code snippet demonstrating the failure

## Contributing

Pull requests and patches are always appreciated! To speed up the merge process, please follow the guidelines below when making a pull request:

* Create a new branch in your fork for the changes you intend to make. Working directly in master can often lead to unintended additions to the pull request later on.
* When appropriate, add to the AcceptanceTests project to cover any new functionality or defect fixes.
* Ensure all previous tests continue to pass (with exceptions for PGM tests)
* Follow the code style used in the rest of the project. ReSharper and StyleCop configurations have been included in the source tree.

Pull requests will still be accepted if some of these guidelines are not followed: changes will just take longer to merge, as the missing pieces will need to be filled in.

## License

This project is released under the [LGPL][lgpl] license, as is the native libzmq library. See LICENSE for more details as well as the [0MQ Licensing][zmq-license] page.

[clrzmq-old]: https://github.com/zeromq/clrzmq-old
[clrzmq-nuget]: http://packages.nuget.org/Packages/clrzmq
[libzmq]: https://github.com/zeromq/libzmq
[zmq-guide]: http://zguide.zeromq.org/page:all
[zmq-example-repo]: https://github.com/imatix/zguide/tree/master/examples/C%23
[zmq-dl]: http://www.zeromq.org/intro:get-the-software
[zmq-license]: http://www.zeromq.org/area:licensing
[issues]: https://github.com/zeromq/clrzmq/issues
[lgpl]: http://www.gnu.org/licenses/lgpl.html
