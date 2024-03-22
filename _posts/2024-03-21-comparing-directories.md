---
layout: post
title: "comparing directories and binaries with powershell"
date: 2024-03-22 08:00:00 +0100
image: fs-cmp/fs-cmp-logo.jpg
tags: one-liner powershell
---

Yesterday, I needed to compare binaries deployed on different environments. I
thought of two ways how to deal with this:
1. Compare DLLs and EXEs versions via
   ```[System.Diagnostics.FileVersionInfo]::GetVersionInfo``` function.
2. Use a ```Get-FileHash``` function to compare the hashes of the files.

Here is the one-liner :smiling_imp: I prepared to get the information. The steps how I prepared and used the command are described next :information_desk_person:.

{% include code-button.html %}
```powershell
ls . -Recurse | ?{$_.Extension -in ".dll",".exe"} | foreach {Get-FileHash $_.fullname | select *,@{n='RelativePath';e={$_.Path.Replace("$($pwd)","")}} -ExcludeProperty Path | sort -property RelativePath} | convertto-json | set-clipboard
```

I decided to use the second approach, because version information is not so
reliable and in some cases might not be updated. On the other hand, the hash is
changing each time a file changes.

So I started building a one-liner which could be used for comparison.

First, I needed to list all files within a directory. Simple:
```powershell
ls . -Recurse
```

I was not really interested in configuration files, nor other non-executable
files. This was an ASP.NET page containing only DLL executables so my filter was
again simple:
```powershell
ls . -Recurse -Include *.dll
```

Unfortunately, it was problematic for me to include ```*.exe``` as well.
According to [superuser: How do I get get-childitem to filter on multiple file
types?](https://superuser.com/questions/318197/how-do-i-get-get-childitem-to-filter-on-multiple-file-types)
I found that getting all files and directories and then filtering them might be
even faster, thus I used this approach.
```powershell
ls . -Recurse | ?{$_.Extension -in ".dll",".exe"} 
```

So, now we have all DLL and EXE files, we'll pipe them to the ```Get-FileHash```, easy.
```powershell
ls . -Recurse | ?{$_.Extension -in ".dll",".exe"} | foreach {Get-FileHash $_.fullname}
```

We could just, pipe the output to the clipboard or to a file and we're done.
Afterwards we can use our favorite text editor and compare the hashes between the environments.

Sadly, in my case the files were not in the same path, thus it was not so easy.
Still, the directory structure should be the same. So I decided to get a
relative path instead of full path. To achieve that I used:
```powershell
select *,@{n='RelativePath';e={$_.Path.Replace("$($pwd)","")}} -ExcludeProperty Path | sort -property RelativePath}
```

And after that I've converted output to JSON which can be easily compared. You
can check the output here (bogus, but you'll get the message).

![](/assets/pictures/fs-cmp/fs-cmp-winmerge.jpg)