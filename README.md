# Replacing .NET Core Framework DLL Test

.NET Core provides a `System.Web.dll` but its only containing type is `HttpUtility`. The goal is to use a .NET Framework library that has dependencies on `System.Web` under .NET Core (due to licensing issues porting is not an option). Since `System.Web.dll` does not contain the necessary types the goal is to create a custom version of `System.Web.dll` that contains the types used by the .NET Framework library.

For this the framework `System.Web.dll` has to be replaced with the custom `System.Web.dll`. This project aims to do just that. The following projects setup is used:

- The `UILib` project contains a .NET Framework assembly that exposes a `MyControl` that inherits from `System.Web.UI.Control` (located in `System.Web.dll`). It simulates a UI library that is built for .NET Framework. This is the DLL that should work under .NET Core.
- The `System.Web.Impl`  project contains the implementation for the `Control` class that is missing in .NET Core.
- The `System.Web.Forward` project has the same assembly name as `System.Web` (public signed). It is meant to replace the framework's `System.Web.dll`. The `Control` class is forwarded to `System.Web.Impl`.
- The `Test` project references the other projects and tries to create a instance of `MyControl`

With a clean SDK the `Test` project won't compile. This is because the framework already contains a `System.Web.dll` and the compiler does not know which of the two versions to pick.

It is possible to get rid of this compiler error by removing (renaming works too) the reference assembly for `System.Web.dll` (`C:\Program Files\dotnet\packs\Microsoft.NETCore.App.Ref\5.0.0-preview.6.20305.6\ref\net5.0\System.Web.dll`). This allows the project to compile but at runtime the framework  assembly is still preferred over the custom one. The assemblies `.deps.json` contains an entry for `System.Web./4.0.1.0` but the framework assembly still has higher precedence.

The only way to fix this problem is to remove the `System.Web.dll` from the `deps.json` in the shared framework folder (`C:\Program Files\dotnet\shared\Microsoft.NETCore.App\5.0.0-preview.6.20305.6\Microsoft.NETCore.App.deps.json`). This, of course, requires providing a custom runtime, which is what we wanted to prevent.

