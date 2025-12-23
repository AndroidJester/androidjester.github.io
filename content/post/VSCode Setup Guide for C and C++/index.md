+++
title = "VSCode Setup Guide for C and C++"
description = "Guide for helping beginner in C and C++" 
date = "2023-09-07T22:20:32Z"
# image = "https://www.gravatar.com/avatar/24a44e8bc08cd0c6e2fd43378eeb25b2?s=192&d=identicon&r=PG&f=y&so-version=2"
# math: false
# license: 
tags = [ "Tutorial", "Guide", "C", "C++" ]
categories = [ "Systems Development" ]
hidden = false
comments = false
draft = false
+++

As Visual Studio Code is one of the most used code editor, beginner in C and C++ programming have difficulty setting up this editor for C and C++ programming

Here I would like to help by setting up VSCode with Clangd

Clangd is an LSP that helps coding in C or C++ a easier 

We would be setting up our C project for writing a simple window using the GTK libraries
```c
// main.c
#include <gtk/gtk.h>

static void activate(GtkApplication *app, gpointer user_data) {
  GtkWidget *window;
  GtkWidget *label;

  window = gtk_application_window_new(app);
  label = gtk_label_new("Hello GNOME!");
  gtk_container_add(GTK_CONTAINER(window), label);
  gtk_window_set_title(GTK_WINDOW(window), "Welcome to GNOME");
  gtk_window_set_default_size(GTK_WINDOW(window), 400, 200);
  gtk_widget_show_all(window);
}

int main(int argc, char **argv) {
  GtkApplication *app;
  int status;

  app = gtk_application_new(NULL, G_APPLICATION_FLAGS_NONE);
  g_signal_connect(app, "activate", G_CALLBACK(activate), NULL);
  status = g_application_run(G_APPLICATION(app), argc, argv);
  g_object_unref(app);

  return status;
}
```
This code is from the mesonbuild #[site](https://mesonbuild.com/Tutorial.html) which is the build system we would be using for the project

## What is an LSP? 
An LSP(Language Server Protocol) is a program that looks through your code for syntax errors and in some cases linting errors depending on the linter provided

## Clangd Setup on VSCode
First we would need the Clangd extension as well as the LSP itself
![Clangd extension](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zfmp451goqc14wlb2t8f.png)
You can download the extension from #[here](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd)
The extension provides its own Clangd LSP but in case of issues with that we would like to download and setup the clangd package from the official [site](https://clangd.llvm.org/installation) for both [Windows and Linux](https://github.com/clangd/clangd/releases)

## Linux Setup 
We can download and install the clangd LSP using the available package manager and installer like `apt` for Debian/Ubuntu, `dnf` or `yum` for Red Hat distributions and `pacman` or `yay` for Arch
- I use Arch so the command I am more familar with is `yay`
```bash
yay -S clangd-opt # This downloads and builds the latest clangd available from the Arch User Repository
```

Then inside our VSCode settings we set path to where our clangd would be located
![Settings](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fjmh9lm4uvwrgz3egidf.png)

We try to find the path to clangd using `whereis clangd` or `which clangd`
![Clangd path](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4idok18cr0m2stow0vf4.png)


### Building the project
Clangd would work for most cases however when involving non-standard libraries like our GTK library, we would start to see a lot of errors
![Clangd fails to locate GTK.h](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r4ip5ijdyvlzxmmu7h2p.png)

First we need to know if the library's files are available, we can download them and include them to our project but to make this short we would just use pkg-config and grep command to find the necessary GTK+ library for our little demo which I have installed in my system
![GTK finding](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yv79cf4f2uw2wfvlcmx4.png)

Then we would include this as a dependency to our project through our build tool such as make, CMake or Meson, we would be using meson which is simpler and easy to use.

So we install meson using our package manager
```bash
sudo pacman -S meson
``` 
We setup our meson.build with the project, this lays out the project for compilation

We write the following code for our demo's `meson.build`
```meson
project('demo', 'c')
gtk = dependency('gtk+-3.0')
executable('demo' 'main.c', dependencies: [gtk])
```
What our `meson.build` file will tell the build system is that our project name is demo and it uses the C programming language we have included the gtk+-3.0 as a dependency and we would build an executable with the name demo from our main.c file

This generates the necessary files and parameters for our compilation

A json file with that name `compile_commands.json` is generated and allows us to quickly use our build system to compile our code, this is also needed by clangd to know our dependency files

In our project directory, we setup our build folder using meson but we would need our compile_commands.json to be within the root of the project for clangd to work properly with so we download `bear` for that purpose and use it along meson to get our compile_commands.json 
```zsh
sudo pacman -S bear
bear -- meson setup build
```

![Meson Works](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/iryf88ujd2zgxpnzprix.png)

With that done our compile_commands.json would contain the following code
```json
[
  {
    "arguments": [
      "/usr/bin/cc",
      "-c",
      "-D_FILE_OFFSET_BITS=64",
      "-o",
      "sanitycheckc.exe",
      "sanitycheckc.c"
    ],
    "directory": "/home/android-jester/Documents/learning-concepts/c/demo/build/meson-private",
    "file": "/home/android-jester/Documents/learning-concepts/c/demo/build/meson-private/sanitycheckc.c",
    "output": "/home/android-jester/Documents/learning-concepts/c/demo/build/meson-private/sanitycheckc.exe"
  }
]
```

This however doesn't contain the include files for that we need to compile our project using meson
```bash
bear -- meson compile -C build
```

![Meson Compile](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rx58ocjxioj3wqjeon4x.png)

Funny to say there is some depreciated code in the source file but it will work out

With that done our compile_commands.json look like this
```json
[
  {
    "arguments": [
      "/usr/bin/cc",
      "-Idemo.p",
      "-I.",
      "-I..",
      "-I/usr/include/gtk-3.0",
      "-I/usr/include/pango-1.0",
      "-I/usr/include/glib-2.0",
      "-I/usr/lib/glib-2.0/include",
      "-I/usr/include/sysprof-4",
      "-I/usr/include/harfbuzz",
      "-I/usr/include/freetype2",
      "-I/usr/include/libpng16",
      "-I/usr/include/libmount",
      "-I/usr/include/blkid",
      "-I/usr/include/fribidi",
      "-I/usr/include/cairo",
      "-I/usr/include/pixman-1",
      "-I/usr/include/gdk-pixbuf-2.0",
      "-I/usr/include/gio-unix-2.0",
      "-I/usr/include/cloudproviders",
      "-I/usr/include/atk-1.0",
      "-I/usr/include/at-spi2-atk/2.0",
      "-I/usr/include/at-spi-2.0",
      "-I/usr/include/dbus-1.0",
      "-I/usr/lib/dbus-1.0/include",
      "-fdiagnostics-color=always",
      "-D_FILE_OFFSET_BITS=64",
      "-Wall",
      "-Winvalid-pch",
      "-O0",
      "-g",
      "-pthread",
      "-c",
      "-o",
      "demo.p/main.c.o",
      "../main.c"
    ],
    "directory": "/home/android-jester/Documents/learning-concepts/c/demo/build",
    "file": "/home/android-jester/Documents/learning-concepts/c/demo/main.c",
    "output": "/home/android-jester/Documents/learning-concepts/c/demo/build/demo.p/main.c.o"
  }
]
```
This shows how my code with compile using the gcc compiler installed on my system, I can change anything in this file manually but more importantly we need to see if the clangd extension has recognized our header files

![No Errors](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/belx29txgwsvy9jyqgbx.png)

As we can see the error lines are gone and we can run our executable in the build folder
```bash
./build/demo
```

![GNOME Window](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vvaqn1blbzib3vj7xbp1.png)


## Windows Setup

### MSYS2 MINGW
On Windows, we would use MSYS2 MINGW to setup our environment by installing the GNU GCC compiler and Meson build tool

We can download MSYS2 installer from [here](https://www.msys2.org/)

After going through the installation process, we would have the mingw64 msys2 terminal available.


![MSYS2 MINGW64 Installed](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9w6mf4egkge5nfvfrj7j.png)


### Clangd Installation
Afterward install clangd from the clang-tools-extra inside bash from mingw64
```bash
pacman -S mingw-w64-x86_64-clang-tools-extra
```

![Clang-tools-extra installation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/074tstlxbsqvrf0kmuwl.png)

The tools are a lot bigger on the Windows side because it also contains the needed gcc compiler and libraries, Clang compiler and libraries and other tools

With that we can find the clangd using `whereis clangd` 
![Whereis Clangd](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q10h37yv4bk5a1qaxgzj.png)


### VSCode setup
In VSCode, after installing the clangd extension, we can either use the bundled clangd downloaded from the extension or we can link it to our clangd installed from msys2 mingw64 



![Clangd in VSCode](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ixgtol1w0avcgbdo5621.png)

In VSCode, we can add the following in our user's `settings.json` which contains all our user settings.
```json
"terminal.integrated.profiles.windows": {
	"MSYS2": {
		"path": "cmd.exe",
		"args": [
			"/c",
			"C:\\msys64\\msys2_shell.cmd -defterm -here -no-start -mingw64 -use-full-path"
		]
	},
	...
```


### Conan and Code
Before we start to code we would need to easily download and use some external libraries
Using the guide from [Conan, C++ Open Source Package Manager](https://conan.io/downloads.html), We can setup conan for msys2 mingw-w64

With conan executable setup we need to create our `conanfile.txt` for our project
```toml
[requires]
zlib/1.2.13

[generators]
PkgConfigDeps
MesonToolchain

```

We are simply importing zlib from the conan package center where we can download the needed libraries and generate a native file for our meson

For Windows GTK doesn't work as expected thus we would go for a command line compression tool `zlib` so inside our `meson.build` file we include the following
```meson
project('demo', 'c')
zlib = dependency('zlib')
executable('demo', 'main.c', dependencies: [zlib])
```

And in our project contains our code
```c
//main.c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <zlib.h>

int main(void) {

    char buffer_in [256] = {"A simple string we want to compress, don't worry about the detail just let the library do its thing we will also try to print out the version of ZLIB to show that it works"};

    char buffer_out [256] = {0};

	// Just deflation or compression
    z_stream defstream;
    defstream.zalloc = Z_NULL;
    defstream.zfree = Z_NULL;
    defstream.opaque = Z_NULL;
    defstream.avail_in = (uInt) strlen(buffer_in);
    defstream.next_in = (Bytef *) buffer_in;
    defstream.avail_out = (uInt) sizeof(buffer_out);
    defstream.next_out = (Bytef *) buffer_out;
    deflateInit(&defstream, Z_BEST_COMPRESSION);
    deflate(&defstream, Z_FINISH);
    deflateEnd(&defstream);

  
	// Printing out the compression details
    printf("Uncompressed size is: %lu\n", strlen(buffer_in));
    printf("Compressed size is: %lu\n", strlen(buffer_out));
    printf("ZLIB VERSION: %s\n", zlibVersion());
    return EXIT_SUCCESS;
}
```

### Build and compile
Bear is the program we used in linux to copy our `compile_commands.json` from our build folder however I cannot easily find a prebuilt executable for Windows thus we would manually copy the `compile_commands.json` from the build folder

First we setup the build by getting our dependencies

```bash
conan install . --output-folder=build --build=missing
```

![Conan Downloading the needed files](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n7f2nrl030xc7hksqw49.png)


This generates a build folder which contains the files necessary to build our project using meson 

We use meson and the native meson file generated from the build folder `conan` generated to create our own build folder

```bash
meson setup <build-folder-name> --native-file build/conan_meson_native.ini
```

However you might approach an error like this


![No such compiler found](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rkei7xlv8ic0z2mvi9ig.png)

We can fix that error by simply changing the compiler used to use our gcc compiler included in out clang-tools-extra in our `build/conan_meson_native.ini`

```ini
[properties]

[constants]
preprocessor_definitions = []
# Constants to be overridden by conan_meson_deps_flags.ini (if exists)
deps_c_args = []
deps_c_link_args = []
deps_cpp_args = []
deps_cpp_link_args = []

[project options]
wrap_mode = 'nofallback'
bindir = 'bin'
sbindir = 'bin'
libexecdir = 'bin'
includedir = 'include'
libdir = 'lib'
  
[binaries]
# c = 'cl'
# cpp = 'cl'
# The compiler binary names
c = 'gcc' 
cpp = 'gcc'

[built-in options]
buildtype = 'release'
b_vscrt = 'md'
b_ndebug = 'true'
cpp_std = 'vc++14'
backend = 'ninja'
pkg_config_path = 'C:\Users\Android-Jester\Documents\learning-concepts\C\demo\build'

# C/C++ arguments
c_args = [] + preprocessor_definitions + deps_c_args
c_link_args = [] + deps_c_link_args
cpp_args = [] + preprocessor_definitions + deps_cpp_args
cpp_link_args = [] + deps_cpp_link_args
```

Finally our build would be completed
![Build Complete](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p5dvbnr2y3d7sdb1s7jf.png)

The compile_commands.json file which clangd needs is inside the builddir so we can easily copy it to the root directory, it might contains the following

```json
[
	{
		"directory": "/c/Users/Android-Jester/Documents/learning-concepts/C/demo/builddir",
		"command": "\"gcc\" \"-Idemo.exe.p\" \"-I.\" \"-I..\" \"-fdiagnostics-color=always\" \"-DNDEBUG\" \"-D_FILE_OFFSET_BITS=64\" \"-Wall\" \"-Winvalid-pch\" \"-O3\" -MD -MQ demo.exe.p/main.c.obj -MF \"demo.exe.p\\main.c.obj.d\" -o demo.exe.p/main.c.obj \"-c\" ../main.c",
	    "file": "../main.c",
	    "output": "demo.exe.p/main.c.obj"
	  }
]
```

This includes all the library files needed thus we can compile our code and run it to see the result

```bash
meson compile -C <build-folder-name>
```

And VSCode can recognize our zlib header file
![Clangd works](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zymklchi26xq9ao7lg1y.png)



![Compiled and running](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r6iqmytahujzsay4w1aw.png)



I hope this was a short and informative guide as this is my first post, please if there are any issues or suggestions for improvements with the process, please let me know in the comments
