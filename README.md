# Build and Use
## Prerequisites
Those tools need to be installed.
* git
* cmake
* ninja
* arc

## Structure
The README assumes that the following structure is present (or will become present) under `~/llvm`
```
.
├── README.md
├── arc
├── build
├── build_10
├── install
├── install_10
└── llvm-project
```

## Check-out LLVM into llvm-project
Clone from Github (https://github.com/llvm/llvm-project).

`git clone https://github.com/llvm/llvm-project.git`

If you want to work on master branch, skip the next steps.

## Apply const-correctness patch for release/10.x branch
Checkout the release 10.x branch since the const-correctness patch is designed for this version.

`git checkout release/10.x`

Apply the const-correctness patch from arc (https://reviews.llvm.org/D54943). This requires to have an Phabricator account which can be linked from yout Gihub Account. The command will guide you what needs to be done.

`arc patch D54943`

## Build LLVM
Change into the root directory `~/llvm`. Choose the installation directory in the cmake configure step wisely. It makes sense to have one build and one install directory per release.

```bash
mkdir build && cd build
cmake -G Ninja -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra" -DCMAKE_INSTALL_PREFIX="~/llvm/install" -DCMAKE_BUILD_TYPE="release" ../llvm-project/llvm
cmake --build ..
cmake --install ..
```

## Run clang-tidy
Clang-tidy runs best when provided with a compilation database where it can find all compilation flags.

### Create a compilation database
In case of cmake this is a build in option. In case of bazel an own solution is required. Change into the project root and execute the following commands. There is no need to build the project. This will generate a compilation database for your project.
```bash
mkdir build && cd build
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 -G Ninja .. 
```

### Run clang-tidy via the python helper script (recommended)
This is an advanced command with applying fixes. Checkout all options with the `-h` parameter. This command must run in the project's root directory so clang-tidy is able to find all referenced files. The python script needs to point to the binaries of clang-tidy and clang-apply-replacements (for applying fixes suggested by clang-tidy). The `-checks` option allows using wildcards an suppressions (i.e. `-*` means disable all checks). With `-p` one sets the directory where the `compile_commands.json` is found.
```bash
~/llvm/install/share/clang/run-clang-tidy.py -p build -clang-tidy-binary ~/llvm/install/bin/clang-tidy -clang-apply-replacements-binary ~/llvm/install/bin/clang-apply-replacements -checks="cppcoreguidelines*,-clang-analyzer*" -header-filter=".*" -fix
```

### Run clang-tidy as standalone tool (not recommended)
One can also directly invoke clang-tidy, this needs the files being specificied directly.
```bash
~/llvm/install/bin/clang-tidy -p build/compile_commands.json --checks="cppcoreguidelines*" --header-filter=".*" main.cpp
```

## Hints
Find out all possible or enabled checks

`~/llvm/install/bin/clang-tidy --list-checks --checks="*"`
