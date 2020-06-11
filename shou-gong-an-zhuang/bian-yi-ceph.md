# 编译CEPH

## BUILD CEPH

You can get Ceph software by retrieving Ceph source code and building it yourself. To build Ceph, you need to set up a development environment, compile Ceph, and then either install in user space or build packages and install the packages.

### BUILD PREREQUISITES

Tip 

Check this section to see if there are specific prerequisites for your Linux/Unix distribution.

Before you can build Ceph source code, you need to install several libraries and tools:

```text
./install-deps.sh
```

Note 

Some distributions that support Google’s memory profiler tool may use a different package name \(e.g., `libgoogle-perftools4`\).

### BUILD CEPH

Ceph is built using cmake. To build Ceph, navigate to your cloned Ceph repository and execute the following:

```text
cd ceph
./do_cmake.sh
cd build
make
```

Note 

By default do\_cmake.sh will build a debug version of ceph that may perform up to 5 times slower with certain workloads. Pass ‘-DCMAKE\_BUILD\_TYPE=RelWithDebInfo’ to do\_cmake.sh if you would like to build a release version of the ceph executables instead.

Hyperthreading

You can use `make -j` to execute multiple jobs depending upon your system. For example, `make -j4` for a dual core processor may build faster.

See [Installing a Build](https://docs.ceph.com/docs/nautilus/install/install-storage-cluster#installing-a-build) to install a build in user space.

### BUILD CEPH PACKAGES

To build packages, you must clone the [Ceph](https://docs.ceph.com/docs/nautilus/install/clone-source) repository. You can create installation packages from the latest code using `dpkg-buildpackage` for Debian/Ubuntu or `rpmbuild` for the RPM Package Manager.

Tip 

When building on a multi-core CPU, use the `-j` and the number of cores \* 2. For example, use `-j4` for a dual-core processor to accelerate the build.

#### ADVANCED PACKAGE TOOL \(APT\)

To create `.deb` packages for Debian/Ubuntu, ensure that you have cloned the [Ceph](https://docs.ceph.com/docs/nautilus/install/clone-source) repository, installed the [Build Prerequisites](https://docs.ceph.com/docs/nautilus/install/build-ceph/#build-prerequisites) and installed `debhelper`:

```text
sudo apt-get install debhelper
```

Once you have installed debhelper, you can build the packages:

```text
sudo dpkg-buildpackage
```

For multi-processor CPUs use the `-j` option to accelerate the build.

#### RPM PACKAGE MANAGER

To create `.rpm` packages, ensure that you have cloned the [Ceph](https://docs.ceph.com/docs/nautilus/install/clone-source) repository, installed the [Build Prerequisites](https://docs.ceph.com/docs/nautilus/install/build-ceph/#build-prerequisites) and installed `rpm-build` and `rpmdevtools`:

```text
yum install rpm-build rpmdevtools
```

Once you have installed the tools, setup an RPM compilation environment:

```text
rpmdev-setuptree
```

Fetch the source tarball for the RPM compilation environment:

```text
wget -P ~/rpmbuild/SOURCES/ https://download.ceph.com/tarballs/ceph-<version>.tar.bz2
```

Or from the EU mirror:

```text
wget -P ~/rpmbuild/SOURCES/ http://eu.ceph.com/tarballs/ceph-<version>.tar.bz2
```

Extract the specfile:

```text
tar --strip-components=1 -C ~/rpmbuild/SPECS/ --no-anchored -xvjf ~/rpmbuild/SOURCES/ceph-<version>.tar.bz2 "ceph.spec"
```

Build the RPM packages:

```text
rpmbuild -ba ~/rpmbuild/SPECS/ceph.spec
```

For multi-processor CPUs use the `-j` option to accelerate the build.

