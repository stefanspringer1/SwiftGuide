# Swift Guide

This is a little guide for the programming language [Swift](https://www.swift.org), about the installation and how to get started with the language. There is also an additional section on versioning of dependent packages and an overview of "dangerous" operations or situations which could cause a program to crash.

_All instructions and information are without any guarantee._

## Platforms guide

### Windows

**tl;tr** The core requirements for developing Swift programs on Windows are:

- **Visual Studio Installation** for enabling native development against the Windows SDK
- **Developer mode** to allow symbolic links
- **Admin rights** for installing the actual Swift toolchain
- **Visual Studio Code** as IDE

#### Setup and platform-specific hints

- the instructions on https://www.swift.org/getting-started, section "On Windows" are decisive; _**the components are listed here again (as of early 2023) with additional comments including the use of an IDE**_
- an installation of **Visual Studio** (as of early 2023: if possible version 2022, but at least version 2019, the Community Edition is sufficient) is required with certain Visual Studio components (Windows 10 SDK and C++ Build Tools); _usually, the parallel installation of several Visual Studio versions should be avoided_, if necessary execute commands in the command line in the "x64 Native Tools Command Prompt" of the corresponding Visual Studio installation (this generally applies if tools such as Git or Python are used as Visual Studio components have been installed and are not generally accessible in the command line); for license issues see (for Visual Studio 2022) [the according documentation](https://visualstudio.microsoft.com/license-terms/vs2022-ga-community)
- note that **Git** (required for the Swift Package Manager) must be version 2 or higher, an older Git version 1.x is not sufficient
- a **Credential Manager** must be enabled for Git if the Swift Package Manager needs to pull packages from private repositories, see e.g. [the instructions for GitHub](https://docs.github.com/de/get-started/getting-started-with-git/caching-your-github-credentials-in-git) or [the instructions for Microsoft Azure](https://learn.microsoft.com/en-us/azure/devops/repos/git/set-up-credential-managers?view=azure-devops)
- note that for the [Swift REPL](https://developer.apple.com/swift/blog/?id=18) ("[Read–eval–print loop](https://en.wikipedia.org/wiki/Read–eval–print_loop)”) a **Python** must be available in a version (as of early 2023) from 3.7
- **Python** is also used to configure the LLVM debugger and is therefore necessary for debugging
- to simplify some of the installations, it is recommended to use the **Windows Package Manager**
- for developer mode: _more precisely:_ the **privilege for setting symbolic links** (SeCreateSymbolicLinkPrivilege) is required (and activating developer mode is only one possible way to get there, a restart is needed; for directly setting the required privilege see notes below), see https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/create-symbolic-links; Test by using the "mklink" command in the command line (`mklink newfile oldfile`, creating a symbolic link file named `newfile` pointing to `oldfile`)
- note that the installation of the actual **Swift Toolchain** requires admin rights, especially _because additions are made to the Visual Studio installation;_ other installation parts are installed to `%SystemDrive%\Library` and to `%PROGRAMFILES%\swift`, the paths corresponding to `%PROGRAMFILES%\swift\icu-69.1\usr\bin`, `%PROGRAMFILES%\swift\runtime-development\usr\bin`, and `%SystemDrive%\Developer\Toolchains\unknown-Asserts-development.xctoolchain\usr\bin` are added to the PATH environment variable (in the future probably no insertion into the Visual Studio installation necessary any more cf. https://github.com/apple/swift/pull/63887)
- **after an update of Visual Studio** it might be necessary to repair the files copied into the Visual Studio installation, using the installed toolchain, see [how to repair an installation](https://support.microsoft.com/en-us/windows/repair-apps-and-programs-in-windows-e90eefe4-d0a2-7c1b-dd59-949a9030f317)
- you can build **your own installation process for the toolchain** as a replacement for the official toolchain installer by using the directories as installed by the installer, setting the appropriate paths, and copying some files into the Visual Studio installation as decribed below _(this copying of files into the Visual Studio installation does not have to be repeated for newer versions of Swift)_
- **IDE** (integrated development environment): **Visual Studio Code** with the [Swift extension](https://marketplace.visualstudio.com/items?itemName=sswg.swift-lang), useful settings for Visual Studio Code see in the section "Suggested settings for Visual Studio Code" below; see also [VS Code Swift extension lesser known features](https://opticalaberration.com/2022/11/vscode-features.html), there e.g. "Local editing of packages" (important for the simultaneous further development of dependent packages); _There should only be one Visual Studio installation (as of early 2023),_ and important tools such as Git must also be accessible outside of Visual Studio's "x64 Native Tools Command Prompt"
- you should **avoid PowerShell on Windows** for Swift development (or also in the general case), especially If you are working with a CI pipeline, because there are some hints that [this might be problematic](https://github.com/orgs/community/discussions/26933) (PowerShell is the default for GitHub Windows CI pipelines, configure the pipeline to use `cmd` or maybe even better `bash`, `bash` is available for GitHub Windows CI pipelines)
- for **profiling purposes,** the appropriate tools of the Xcode installation in macOS are currently to be used (however, profiling results can vary in principle depending on the platform)

**files copied by the toolchain installer into the Visual Studio installation** (the environment variables `UniversalCRTSdkDir`, `VCToolsInstallDir`, and `UCRTVersion` are set within the x64 Native Tools Command Prompt of Visual Studio[^1], and `SDKROOT` would be set to point to `%SystemDrive%\Library\Developer\Platforms\Windows.platform\Developer\SDKs\Windows.sdk\usr\share` for the standard installation):

[^1]: The corresponding values can be extracted from the registry entries `HKLM\SOFTWARE\[Wow6432Node]\Microsoft\Microsoft SDKs\Windows\v10.0\InstallationFolder`, `HKLM\SOFTWARE\[Wow6432Node]\Microsoft\Microsoft SDKs\Windows\v10.0\ProductVersion`, and `HKLM\SOFTWARE\Microsoft\Windows Kits\Installed Roots\KitsRoot10`, cf. [the installer code](https://github.com/compnerd/swift-installer-scripts/blob/main/platforms/Windows/CustomActions/SwiftInstaller/Sources/swift_installer.cc).

```batch
copy /Y %SDKROOT%\usr\share\ucrt.modulemap "%UniversalCRTSdkDir%\Include\%UCRTVersion%\ucrt\module.modulemap"
copy /Y %SDKROOT%\usr\share\visualc.modulemap "%VCToolsInstallDir%\include\module.modulemap"
copy /Y %SDKROOT%\usr\share\visualc.apinotes "%VCToolsInstallDir%\include\visualc.apinotes"
copy /Y %SDKROOT%\usr\share\winsdk.modulemap "%UniversalCRTSdkDir%\Include\%UCRTVersion%\um\module.modulemap"
```

**directly setting the SeCreateSymbolicLinkPrivilege / SE_CREATE_SYMBOLIC_LINK privilige:** the SE_CREATE_SYMBOLIC_LINK privilege can be set using the `gpedit.msc` tool (start it via the context menu of the Windows Explorer as administrator); if you have the Home edition of Windows, you first have to get this tool from Microsoft using the following script (open the command line window as administrator):

```Batch
@echo off 
pushd "%~dp0" 
dir /b %SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientExtensions-Package~3*.mum >List.txt 
dir /b %SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientTools-Package~3*.mum >>List.txt 
for /f %%i in ('findstr /i . List.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i" 
pause
```

#### Why is Visual Studio necessary?

_Question 1:_ Why is Visual Studio required for Swift?

_Answer:_ This is due to the dependency of some necessary components (the Windows SDK and ucrt have a dependency on the headers and import libraries of VCRUNTIME for development, which are not part of the Microsoft Visual C++ Redistributable).

_Question 2:_ Why isn't this necessary for other natively compiling languages like Rust or Go, but for Swift?

_Answer:_ The special thing about Swift on Windows is that Swift is implemented as a language with "equal rights to C++" so to speak, with direct access to all Windows libraries. The Swift runtime and standard library are implemented using direct calls into the Windows APIs, not emulating things like process startup, allowing a more "bare metal" environment compared to other environments. Swift on Windows is therefore a "true" native solution for Windows programming. This kind of system integration does not exist with Rust and Go.

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

## Suggested settings and tips for Visual Studio Code

### Inlay hints only with keyboard shortcut

Set Editor › Inlay Hints: offUnlessPressed.

_Without this setting you see a lot of type hints which makes the Swift code look very ugly at some places._

### Detect encoding

Set "files.autoGuessEncoding":true.

### Double-click a file in the navigator to keep it open

Unless you set "workbench.editor.enablePreview": false, a file only simply clicked in the navigator will be replaced in the editor pane by the next one clicked. Double-click a file to let it stay in the editor pane until you close it.

## Getting started with Swift

Please also consult the official [documentation overview](https://www.swift.org/documentation/) and the "open source efforts" section on [swift.org](https://www.swift.org).

- **minimal Swift package** (using the Swift Package Manager): creation of a new executable Swift package via `swift package init --type executable` (without the `--type` argument, a library package is created) within a directory newly created for the package, a possible minimal program is already created or alternatively consists of the code `print("Hello")` within a file `main.swift`, for further functionalities in general. at least `import Foundation`; note that a new project created inside Xcode might instead use the Xcode project format which is not useful for Linux or Windows
- you can also compile a program in the form of **a single Swift file** via `swiftc` or [use it as a script](https://rderik.com/blog/using-swift-for-scripting/), on a Mac or iPad you might also use **Swift Playgrounds** to play around with the language (see [there](https://www.apple.com/swift/playgrounds)), and there is also the **REPL** ("[Read–eval–print loop](https://en.wikipedia.org/wiki/Read–eval–print_loop)”) (to start the REPL under Windows, you need to provide — as of early 2023 — some extra arguments, cf. https://www.swift.org/getting-started)
- for **Swift Package Manager** ("SPM") see [Introduction](https://www.swift.org/getting-started/#using-the-package-manager) and [Documentation](https://github.com/apple/swift-package-manager/tree/main/Documentation) and the explanations of the versioning concept in the section "Versioning of dependent packages" below, there in particular the section "Tested version combinations" on the question of whether the `Package.resolved` file should be versioned; get a visual dependencies graph for a package by `swift package show-dependencies --format dot | dot -Tsvg -o graph.svg` (the `dot` commands necessitates the installation of graphviz)
- note that building a Swift package via `swift build` (called at the top level of the package directory) without further specification uses a quick incremental build and prepares the program for debugging, creating a so-called **debug version.** While useful during development, a debug version is significantly slower than the so-called release version and also uses more memory; a **release version** that does without debug additions and for which a whole module optimization is used, among other things, is created with `swift build -c release` (or with the corresponding setting or the corresponding call in the IDE); the built program is located in the corresponding subdirectory `.build/[debug|release]` of the package directory
- if unclear problems arise during the build, it may help to add the **verbose specification `-v` in the build command**
- on platforms other than macOS, tools for the various types of **profiling** as they come the form of "[Instruments](https://www.avanderlee.com/debugging/xcode-instruments-time-profiler/)" as part of the Xcode installation (e.g. for detecting reference cycles or measuring energy consumption), are (as of early 2023) not yet available; if you want to carry out the relevant analysis, you have to do it under macOS
- the official **Swift book:** https://docs.swift.org/swift-book/documentation/the-swift-programming-language/, especially important for getting started: [Optionals](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics) and [Optional Chaining](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/optionalchaining) and the very useful [`guard` statement](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow#Early-Exit) in this context, which avoids many `if ... else ...` constructs (note that the Swift book in the Apple book store is — as of early 2023 — not updated, so better use the mentioned online version) 
- some **new language features** might first have to be activated via the ["enableUpcomingFeature" option](https://github.com/apple/swift-evolution/blob/main/proposals/0362-piecemeal-future-features.md) before they are activated by default in a newer Swift version (this might be for compatibiliy reasons and is then mentioned on the according [evolution page](https://github.com/apple/swift-evolution/tree/main/proposals) of the feature)
- **platform-specific code** (or code specifically for the debug version) can be compiled using the appropriate [Compiler Directives](https://www.swiftbysundell.com/articles/using-compiler-directives-in-swift/ ) (full listing [in the Swift documentation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/statements/))
- the realization of the complete **equality of the standard Swift libraries on different platforms** (analogous to "[.NET Core](https://de.wikipedia.org/wiki/.NET_(Platform)#History)" from 2016) [starts in 2023]( https://www.swift.org/blog/future-of-foundation/), _until then it should always be tested (apart from further tests) whether a Swift program compiles for all target platforms_
- you can **develop for Linux on a non-Linux system** in [Visual Studio Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers), see [the description of how this works with Swift](https://github.com/swift-server/vscode-swift/blob/main/docs/remote-dev.md)
- **documentation in code** according to [DocC](https://www.swift.org/documentation/docc/); include the `--include-extended-types` option to also document extensions to types from other modules (`swift package generate-documentation --include-extended-types`)
- for executing the **unit tests** of a Swift package, use the tests tab in Xcode or the test icon in Visual Studio Code (on the left) or in the command line e.g. `swift test --parallel --xunit-output test-log.xml` (as of early 2023, the `--parallel` argument is necessary for `--xunit-output` to work)
- consider the **platform-specific hints** in the section "Platforms guide" above
- consider the section below about comparing Swift against Java and C# regarding **“problematic” events**
- for **benchmarking,** you might consider [this package](https://github.com/ordo-one/package-benchmark)

## Versioning of dependent packages

_This section describes how the Swift Package Manager proceeds, its procedure corresponds to a [common practice elsewhere](https://en.wikipedia.org/wiki/Dependency_hell#Solutions). A procedure that makes sense in this context is explained._

_See also the relevant Swift Package Manager documentation: ["Building Swift Packages or Apps that Use Them in Continuous Integration Workflows"](https://github.com/aplple/swift-package-manager/blob/main/Documentation/ContinousIntegration.md)._

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
    <span style="color:Gray">**Reason why a general use of precise version numbers is [not practical](https://en.wikipedia.org/wiki/Dependency_hell#Problems):** The package versions must be compared between all packages used, so the same packages may be drawn from several packages used. Example: The program requires Package A and Package B, and both Package A and Package B require Package C. But Package A may require a slightly newer version of Package C than Package B requires. This means that the package manager gets a number of version conditions and has to find a solution (or issue an appropriate error message). If you would generally only specify precise versions for all packages (over several levels!), then that would be difficult to handle. In other words: The package manager will then often not be able to find a solution. It should be remembered that packages from other sources are also commonly used (however "official" they may be).</span>

### Tested version combinations

- Normally, as a "valid" program, the sources of a program are not passed on, but rather a CI pipeline is built with a) automatic fetching of the latest packages (via "swift package update"), if necessary, and b) automatically tested executables. This is the usual practice, and the quick fetching and testing of all new fixes in their combination ("swift package update") [the heart of Continuous Integration](https://youtu.be/_w6TwnLCFwA).
- What you can (and should) do additionally: For the successfully tested executables, note the precise package versions drawn for them. This information is in the "Package.resolved" file, which is often excluded from versioning. You should definitely remember these combinations for productively used executables (in the form of the “Package.resolved” file).
- This precise version information can be used to get exactly this combination (via the set tag) again when building later ([build with given "Package.resolved" file](https://github.com/apple/swift-package-manager/blob/main/Documentation/Usage.md#resolving-versions-packageresolved-file) or just fetch the packages via "[swift package resolve](https://github.com/apple/swift-package-manager/blob/main/Documentation/Usage.md#resolving-versions-packageresolved-file)" and note whether the "Package.resolved" file changes).
- A note can be made by including the "Package.resolved" file in the versioning (**a new package created via "swift package init [--type executable]" does _not_ exclude the "Package.resolved" file from versioning**), a correspondingly tagged release version of the source code thus contains the specification of the package combination used. A build via "swift build [-c release]" should not change the "Package.resolved" file (the package versions listed there could then be used), the command "git status -s" should therefore have an empty return.
- Fixed version numbers in the form of a versioned Package.resolved is only an indication of a working version combination, so to speak as a fallback if the actual version restrictions (in Package.swift) are not sufficient (because semantic versioning is not always perfect in practice). Actual builds that you use might come from a CI pipeline which updates packages freely.

### Concurrent development of dependent packages

- Packages can also be referenced locally via `file://...` to be able to easily develop dependent packages at the same time
- cf. [the feature "Local editing of packages"](https://opticalaberration.com/2022/11/vscode-features.html) of [Swift extension for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=sswg.swift-lang)

## Comparing Swift against Java and C# regarding “problematic” events

**tl;dr** _Swift tries to be safe (you might say “harmless” i.e. avoiding dangerous operations, e.g. avoiding null pointer exceptions) while also being very efficient and guaranteeing correct results. An exception mechanism of the same kind as in Java or C# (where you can catch almost any error) would be difficult in Swift as Swift (on purpose[^2]) does not use a tracing garbage collector (so discarding part of the stack would leave you with memory leaks). So instead of silently failing (e.g. letting numbers overflow silently as in Java and C#) and giving wrong results, the philosophy is that it is better to let the whole process fail in cases where other solutions would be overly inefficient or overly complex. As a guideline, think “Swift is mostly harmless” and learn the patterns necessary to write robust code.[^3]_

[^2]: Swift uses an optimized reference counting instead of a tracing garbage collector which results in significantly less memory used and also makes a Swift program deterministic, avoiding the non-reproducibility of some errors you might encounter when using a tracing garbage collector. Swift might even be used for "close to the metal" situations.

[^3]: You may disagree with the design choices of the Swift language creators, but their belief is that those design choices result in better software at the end.

This is an overview of “dangerous” operations or situations which could cause a Swift program to crash, in comparison to Java and C#. (Note that even if your program does not crash, your program still might do unexpected things e.g. because of an unnoticed overflow of a number value.)

- None of these systems can handle problems within a program caused by excessive “memory hunger” (or too low memory on the system), aborts because of a high recursion level (or generally a “stack overflow”) or problems in the virtual machine or corresponding problems caused by compiler errors.
- Apart from these points, “everything” can be intercepted (“catched”) in “managed code” systems such as Java or C#.
- However, neither Java nor C# enforces catching all possible explicitly thrown exceptions (there are so-called “unchecked” exceptions whose handling is not enforced), and arithmetic operations or poorly implemented standard library APIs can throw implicit errors. A general `try`/`catch` wrapping must then be carried out for each job, for example.
- In Swift, such a general wrapping in `try`/`catch` (in Swift this is actually `do`/`catch`) “to catch everything” as in Java or C# is not possible, but only a few “dangerous” operations (e.g. forced unwrapping of optionals) or easily recognizable “unsafe” constructions or functions (e.g. `UnsafeMutableRawPointer`) have to be dispensed with or used in the usual "sensible" way (e.g. for substrings, only use ranges you previously found in the same string) in order to achieve the same situation as for Java or C# where you apply a general `try`/`catch` (because there are no unchecked exceptions in Swift). Apart from the dangers of overflow or underflow of numbers (these are often overlooked, especially since the according standard behavior differs from Java and C#, see the following table), the programmer is usually aware of the danger of these operations and they are then at best not used or only used very cautiously (possible overflows or underflows of numbers can also be handled, cf. the same table). However (as of early 2023) no automatic check for such dangerous operations by the compiler is possible.

Details:

“&#9760;” denotes a “dangerous” Swift operation in the following table.

|Event | Java  | C# | Swift|
|-------- | -------- | --------| --------|
| problem caused by virtual machine errors or compiler errors | uncatchable[^4] | uncatchable | uncatchable |
| too low memory on the system | uncatchable | uncatchable | uncatchable |
| excessive “memory hunger” while enough memory on the system (or paging) | uncatchable crash[^5] | uncatchable | uncatchable, no crash |
| stack overflow | uncatchable | uncatchable | uncatchable |
| explicitly thrown exception | handling of a “checked” exception is enforced by the compiler, but not of an “unchecked” exception (results in a crash if unhandled) | handling of a “checked” exception is enforced by the compiler, but not of an “unchecked” exception (results in a crash if unhandled) | there are only “checked” exceptions |
| null pointer exception | unchecked Exception, but catchable | unchecked Exception, but catchable | only possible if non-null assumption is explicitly enforced (“forced unwrapping” of an optional) &#9760;[^6], then uncatchable |
| floating point division by 0 | no error (result: infinite) | no error (result: infinite) | no error (result: infinity)[^7] |
| integer division by 0 | catchable (unchecked) | catchable (unchecked) | uncatchable &#9760; |
| number over-/underflow | no error (operators are “overflow operators”) | no error (operators are “overflow operators”) | uncatchable error &#9760;, but overflow operators (with prefix "\&") and controlled operations (e.g. `addingReportingOverflow`) available[^7] |
| array index not allowed[^8] | catchable (unchecked) | catchable (unchecked) | cannot be caught &#9760; (same for any `Collection`) |
| problems analogous to the array index problem caused by poorly formulated APIs | yes, even as unchecked exceptions, but catchable | yes, even as unchecked exceptions, but catchable | in general not present (corresponding operations return optional values) |
| unsafe[^9] operations | special case | special case| corresponding constructions and functions are easily recognizable as unsafe &#9760; by naming conventions (e.g. `UnsafeMutableRawPointer`), problems cannot be caught |

[^4]: “Catchable” means that a crash can be prevented using `try`/`catch` (or in Swift: `do`/`catch`).

[^5]: “Crash” can mean the termination of the program run by a virtual machine with a corresponding message; in any case, the program is aborted.

[^6]: Force-unwrapping an optional might be a sign that the code is mal-constructed, but there are sensible uses like implicitly unwrapped optional property [for inhertance reasons](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting#Unowned-References-and-Implicitly-Unwrapped-Optional-Properties).

[^7]: Behavior regarding arithmetic can additionally be changed using compiler flags (this may result in IEEE conformity being broken).

[^8]: Index access is generally to be replaced with other methods; avoiding such errors by the use of dependent types is currently not possible in any of the systems mentioned; with Swift, index access can be made an unsafe[^9] operation using a compiler flag.

[^9]: Definition: “unsafe” operations have an undefined behavior for some inputs, example: "pointer arithmetic".
