---
layout: post
title: The importance of being signed
---

If you are so unlucky in 2015 and still have to maintain and deploy desktop-based .NET applications (WinForms, WPF, Windows services etc), this post might be helpful.

Probably most people have run across the situation when an installed .NET desktop application is blocked, removed or killed by the currently running anti-virus software. This happens especially when the application does some *suspicious* networking (from the point of view of the anti-virus of course).
Even installers may be affected by this cruel behaviour - and you might get very unhelpful error messages during installation.

Fortunately, the solution to this problem is very simple - you just need to sign all app binaries with a code-signing certificate. It is not a 100% bulletproof solution, but it worked fine for me.

This applies not only to application' .dll & .exe files, but also to installers and all auxiliary installer libraries.
For example, in my WiX install projects I have a special .dll with custom actions - this have to be signed too!

The signing process can easily be automated inside a WiX project for VS (which is a normal MSBuild file):

1. Define some required properties (understood by WiX build system):

```
<PropertyGroup>
    <SignTarget>true</SignTarget>
    <SignTools>C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A\Bin</SignTools>
</PropertyGroup>
<ItemGroup>
    <SignTargetPath Include="$(TargetPath)" />
</ItemGroup>
```

2. Now you can pre-process and sign your artifacts in the corresponding build targets (BeforeSigning, SignCabs, SignMsi etc), e.g.:

```
<Target Name="SignMsi">
    <Exec Command="&quot;$(SignTools)\signtool&quot; sign /d Your_Description /s Certificate_Store_Name /n &quot;Your_Certificate_Subject_Name&quot; &quot;%(SignMsi.FullPath)&quot;" />
</Target>
```

The more detailed description of how to sign with WiX can found [here](http://wixtoolset.org/documentation/manual/v3/overview/insignia.html).
