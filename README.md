# Swift Guide

This is a little guide about the programming language [Swift](https://www.swift.org), about the installation and how to get started. There is also an additional section on versioning of dependent packages.

_All instructions and information are without any guarantee._

## Platforms guide

### Windows

**tl;tr** The core requirements for developing Swift programs on Windows are:

- **Visual Studio Installation** for enabling native development against the Windows SDK
- **Developer mode** for working Swift Package Manager (symlinks)
- **Admin rights** for installing the actual Swift toolchain
- **Visual Studio Code** as IDE

#### Setup

- the instructions on https://www.swift.org/getting-started, section "On Windows" are decisive; _**The components are listed again here (as of early 2023) with additional comments including the use of an IDE**_
- an installation of **Visual Studio** (as of early 2023: if possible version 2022, but at least version 2019) is required with certain Visual Studio components (Windows 10 SDK and C++ Build Tools); _usually, the parallel installation of several Visual Studio versions should be avoided_, if necessary execute commands in the command line in the "x64 Native Tools Command Prompt" of the corresponding Visual Studio installation (this generally applies if tools such as Git or Python are used as Visual Studio components have been installed and are not generally accessible in the command line)
- note that **Git** (required for the Swift Package Manager) must be version 2 or higher, an older Git version 1.x is not sufficient
- A **Credential Manager** must be enabled for Git if the Swift Package Manager needs to pull packages from private repositories
- note that for the [Swift REPL](https://developer.apple.com/swift/blog/?id=18) ("[Read–eval–print loop](https://en.wikipedia.org/wiki/Read–eval–print_loop)”) a **Python** must be available in a version (as of early 2023) from 3.7
- **Python** is also used to configure the LLVM debugger and is therefore necessary for debugging
- to simplify the installations, it is recommended to use the **Windows Package Manager**
- for developer mode: _more precisely:_ the **privilege for setting symbolic links** (SeCreateSymbolicLinkPrivilege) is required (and activating developer mode is only one possible way to get there), see https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/create-symbolic-links; Test by using the "mklink" command in the command line
- note that the installation of the actual **Swift Toolchain** requires admin rights, especially _because additions are made to the Visual Studio installation;_ other installation parts are installed to `%SystemDrive%\Library` and to `%PROGRAMFILES%\swift`, the paths corresponding to `%PROGRAMFILES%\swift\icu-69.1\usr\bin`, `%PROGRAMFILES%\swift\runtime-development\usr\bin`, and `%SystemDrive%\Developer\Toolchains\unknown-Asserts-development.xctoolchain\usr\bin` are added to the PATH environment variable
- **after an update of Visual Studio** it might be necessary to repair the files copied into the Visual Studio installation, using the installed toolchain, cf. [Repair apps and programs in Windows](https://support.microsoft.com/en-us/windows/repair-apps-and-programs-in-windows-e90eefe4-d0a2-7c1b-dd59-949a9030f317)
- you can build **your own installation process for the toolchain** as a replacement for the official toolchain installer by using the directories as installed by the installer, setting the appropriate paths, and copying some files into the Visual Studio installation as decribed below
- IDE: **Visual Studio Code** with the [Swift extension](https://marketplace.visualstudio.com/items?itemName=sswg.swift-lang), useful settings for Visual Studio Code see in the section "Suggested settings for Visual Studio Code" below; see also [VS Code Swift extension lesser known features](https://opticalaberration.com/2022/11/vscode-features.html), there e.g. "Local editing of packages" (important for the simultaneous further development of dependent packages); _There should only be one Visual Studio installation (as of early 2023),_ and important tools such as Git must also be accessible outside of Visual Studio's "x64 Native Tools Command Prompt".

**files copied by the toolchain installer into the Visual Studio installation** (the environment variables `UniversalCRTSdkDir`, `VCToolsInstallDir`, and `UCRTVersion` are set within the x64 Native Tools Command Prompt of Visual Studio[^1], and `SDKROOT` should be set to point to `%SystemDrive%\Library\Developer\Platforms\Windows.platform\Developer\SDKs\Windows.sdk\usr\share` for the standard installation):

[^1]: The corresponding values can be extracted from the registry entries `HKLM\SOFTWARE\[Wow6432Node]\Microsoft\Microsoft SDKs\Windows\v10.0\InstallationFolder`, `HKLM\SOFTWARE\[Wow6432Node]\Microsoft\Microsoft SDKs\Windows\v10.0\ProductVersion`, and `HKLM\SOFTWARE\Microsoft\Windows Kits\Installed Roots\KitsRoot10`, cf. [the installer code](https://github.com/compnerd/swift-installer-scripts/blob/main/platforms/Windows/CustomActions/SwiftInstaller/Sources/swift_installer.cc).

```batch
copy /Y %SDKROOT%\usr\share\ucrt.modulemap "%UniversalCRTSdkDir%\Include\%UCRTVersion%\ucrt\module.modulemap"
copy /Y %SDKROOT%\usr\share\visualc.modulemap "%VCToolsInstallDir%\include\module.modulemap"
copy /Y %SDKROOT%\usr\share\visualc.apinotes "%VCToolsInstallDir%\include\visualc.apinotes"
copy /Y %SDKROOT%\usr\share\winsdk.modulemap "%UniversalCRTSdkDir%\Include\%UCRTVersion%\um\module.modulemap"
```

#### Distribution of compiled programs under Windows

- there is (as of early 2023) for Swift programs **under Windows no static linking yet,** so to run a compiled Swift program, some DLLs must be available separately from the program, as explained below
- these **DLLs** are 1) installed in `%PROGRAMFILES%\swift`, 2) the Visual Studio C++ Redistributables in `[VisualStudioFolder]\VC\redist\...\x64\*.CRT`, cf. for the latter [Microsoft's documentation](https://learn.microsoft.com/en-us/visualstudio/releases/2022/redistribution#visual-c-runtime-files); the versions of the DLLs must match those with which the Swift program was compiled; Point "2)" can be omitted and instead a corresponding [Visual C++ Redistributable](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170)- installation are assumed
- these DLLs must be placed next to the executable or the relevant directories must be named in the PATH environment variable (different versions of the same DLLs in PATH should be avoided)
- for the DLLs included with such a program, reference should be made to the corresponding **licenses** or analogous documentation, if necessary, or a corresponding license (recommended: named and documented according to the licensed component) must be enclosed (the latter could e.g. for the Apache 2.0 license, as long as no static linking is done, note the runtime library exception of the Apache 2.0 license); please note a) [the Apache 2.0 license](https://github.com/apple/swift/blob/main/LICENSE.txt ) for [Swift](https://github.com/apple/swift) , b) [the Unicode License](https://www.unicode.org/license.txt) for the [International Components for Unicode “ICU”](https://icu.unicode.org), c) the [Documentation from Microsoft](https://learn.microsoft.com/en-us/visualstudio/releases/2022/redistribution#visual-c-runtime-files) on the Visual C++ Runtime Files; Point "c)" can be omitted if (see above) the installation of the corresponding Visual C++ Redistributables is required

### Linux

- see https://www.swift.org/getting-started or [Docker-Images](https://hub.docker.com/_/swift/?tab=tags)
- a credential manager must be activated for Git if the Swift Package Manager needs to pull packages from private repositories
- IDE: like Windows
- first steps with Swift: see Windows
- static linking via addition to the build command `-Xswiftc -static-executable` (everything) or `-Xswiftc -static-stdlib` (Swift standard libraries only)
- note the license questions mentioned under "Windows".

### macOS

- install [Xcode](https://developer.apple.com/xcode)
- a credential manager must be activated for Git if packages from private repositories have to be pulled from the Swift Package Manager (should not be explicitly necessary within Xcode)
- other IDE: see Windows
- first steps with Swift: see Windows
- the Swift runtime (Swift standard libraries) is part of the operating system on Apple platforms (this is possible because of the stable ABI already implemented there), so new features (language or standard libraries) may only be available there with new operating system versions and you sometimes have to use [`#available` annotations](https://www.avanderlee.com/swift/available-deprecated-renamed/) (for individual components, see also [the official Swift documentation](https ://docs.swift.org/swift-book/documentation/the-swift-programming-language/attributes/)) or (in the configuration for the package manager, for the whole package) with the [`platforms` flag ](https://github.com/apple/swift-package-manager/blob/main/Documentation/PackageDescription.md#package) (note: restrictions according to `platforms` only for platforms mentioned there)

## Suggested settings for Visual Studio Code

### Detect encoding

Set "files.autoGuessEncoding":true.

### Inlay hints only with keyboard shortcut

Set Editor › Inlay Hints: offUnlessPressed.

### Don't preview found files

Set "workbench.editor.enablePreview": false. Then the files stay open. (Alternative: Open a found file with a double click if you want it to stay open.)

## Getting started with Swift

- **minimal Swift program:** Creation of a new executable Swift package via `swift package init --type executable` within a directory newly created for the package, a possible minimal program is already created or alternatively consists of the code `print("Hello")` within a file `main.swift`, for further functionalities in general. at least `import Foundation`
- for **Swift Package Manager** see [Introduction](https://www.swift.org/getting-started/#using-the-package-manager) and [Documentation](https://github.com/apple/swift-package-manager/tree/main/Documentation) and the explanations of the versioning concept in the section "Versioning of dependent packages" below, there in particular the section "Tested version combinations" on the question of whether the `Package.resolved` file should be versioned
- note that building via `swift build` (called at the top level of the package directory) without further specification uses a quick incremental build and prepares the program for debugging, therefore creating such a **debug version* * While useful during development, a debug version is significantly slower than the so-called release version and also uses more memory; a **release version** that does without debug additions and for which a whole module optimization is used, among other things, is created with `swift build -c release` (or with the corresponding setting or the corresponding call in the IDE); the built program is located in the corresponding subdirectory `.build\[debug|release]` of the package directory
- if unclear problems arise during the build, it may help to add the **verbose specification `-v` in the build command**
- on platforms other than macOS, tools are provided for the various types of **on-the-fly analysis** that complement the capabilities of "[Instruments](https://www.avanderlee.com/debugging/xcode-instruments-time-profiler/)" correspond to the Xcode installation (e.g. energy consumption), not or (as of early 2023) not yet available; if you want to carry out the relevant analysis, you have to do it under macOS
- the official **Swift book:** https://docs.swift.org/swift-book/documentation/the-swift-programming-language/, especially important for getting started: [Optionals](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics) and [Optional Chaining](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/optionalchaining) and the very useful [`guard` statement](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow#Early-Exit) in this context, which many `if ... else ...` constructs are avoided
- **platform-specific code** (or code specifically for the debug version) can be compiled using the appropriate [Compiler Directives](https://www.swiftbysundell.com/articles/using-compiler-directives-in-swift/ ) (full listing [in the Swift documentation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/statements/))
- the **REPL** does not yet work properly under Windows (as of early 2023).
- the realization of the complete **equality of the standard Swift libraries on different platforms** (analogous to "[.NET Core](https://de.wikipedia.org/wiki/.NET_(Platform)#History)" from 2016) [planned from 2023]( https://www.swift.org/blog/future-of-foundation/), _until then it should always be tested (apart from further tests) whether a Swift program for all target platforms compiled_
- **Documentation in code** according to [DocC](https://www.swift.org/documentation/docc/)

## Versioning of dependent packages

_This section describes how the Swift Package Manager proceeds, its procedure corresponds to a [common practice elsewhere](https://en.wikipedia.org/wiki/Dependency_hell#Solutions). A procedure that makes sense in this context is explained._

_See also the relevant Swift Package Manager documentation: ["Building Swift Packages or Apps that Use Them in Continuous Integration Workflows"](https://github.com/apple/swift-package-manager/blob/main/Documentation/ContinousIntegration.md)._

### Explanation "semantic versioning"

- With "semantic versioning" there is a major, minor and patch version, e.g. "2.4.12" = major: 2, minor: 4, patch: 12, with the following interpretation:
- Patches only fix bugs or optimize (or maybe do code refactoring) but don't bring any changed or new features.
- New minor versions only extend functionalities, but remain backwards compatible.
- New major versions may contain breaking changes.

### Formulation of dependent packages with semantic versioning

- A program or library uses other packages.
- A minimum version is usually specified for each of these.
- The Package Manager then fetches it (using the command "[swift package update](https://github.com/apple/swift-package-manager/blob/main/Documentation/Usage.md#resolving-versions-packageresolved-file)" or building without an existing "Package.resolved" file, see below) a version that is as new as possible within the same major version.
- You can refine this information, e.g. only get new patches, but no new minor version.

### Why setting minimal versions makes sense

- Newer but still suitable versions of packages are automatically used (when updating the packages, see above). These may contain important bug fixes.
- A consistent statement of precise version numbers would be extremely difficult to handle in practice.
    <span style="color:Gray">**Reason why a general use of precise version numbers is [not practical](https://en.wikipedia.org/wiki/Dependency_hell#Problems):**
The package versions must be compared between all packages used, so the same packages may be drawn from several packages used. Example: The program requires Package A and Package B, and both Package A and Package B require Package C. But Package A may require a slightly newer version of Package C than Package B requires. This means that the package manager gets a number of version conditions and has to find a solution (or issue an appropriate error message). If you would generally only specify precise versions for all packages (over several levels!), then that would be difficult to handle. In other words: The package manager will then often not be able to find a solution. It should be remembered that packages from other sources are also commonly used (however "official" they may be).</span>

### Tested version combinations

- Normally, as a "valid" program, the sources of a program are not passed on, but rather a CI pipeline is built with a) automatic fetching of the latest packages (via "swift package update"), if necessary, and b) automatically tested executables. This is the usual practice, and the quick fetching and testing of all new fixes in their combination ("swift package update") [the heart of Continuous Integration](https://youtu.be/_w6TwnLCFwA).
- What you can (and should) do additionally: For the successfully tested executables, note the precise package versions drawn for them. This information is in the "Package.resolved" file, which is often excluded from versioning. You should definitely remember these combinations for productively used executables (in the form of the “Package.resolved” file).
- This precise version information can be used to get exactly this combination (via the set tag) again when building later ([build with given "Package.resolved" file](https://github.com/apple/swift-package-manager/blob/main/Documentation/Usage.md#resolving-versions-packageresolved-file) or just fetch the packages via "[swift package resolve](https://github.com/apple/swift-package-manager/blob/main/Documentation/Usage.md#resolving-versions-packageresolved-file)" and note whether the "Package.resolved" file changes).
- A note can be made by including the "Package.resolved" file in the versioning (**a new package created via "swift package init [--type executable]" closes the "Package.resolved" file initially _not_ from the versioning**), a correspondingly tagged release version of the source code thus contains the specification of the package combination used. A build via "swift build [-c release]" should not change the "Package.resolved" file (the package versions listed there could then be used), the command "git status -s" should therefore have an empty return.
- Fixed version numbers in the form of a versioned Package.resolved is only an indication of a working version combination, so to speak as a fallback if the actual version restrictions (in Package.swift) are not sufficient (because semantic versioning is not always perfect in practice). Actual builds that you use might come from a CI pipeline which updates packages freely.

### Concurrent development of dependent packages

- Packages can also be referenced locally via `file://...` to be able to easily develop dependent packages at the same time
- cf. [Feature "Local editing of packages"](https://opticalaberration.com/2022/11/vscode-features.html) of [Swift extension for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=sswg.swift-lang)
