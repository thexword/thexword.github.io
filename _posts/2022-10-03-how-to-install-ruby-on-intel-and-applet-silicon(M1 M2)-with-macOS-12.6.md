---
layout: post
title: "How to Install Ruby on Intel and Apple Silicon (M1 M2) With macOS 12.6"
date: 2022-10-03 09:02:00 +0800
categories: jekyll
---

参考文章：

[how-to-install-ruby-on-macos-12-6-apple-silicon](https://www.rubyonmac.dev/how-to-install-ruby-on-macos-12-6-apple-silicon)

原文：

If you’ve recently updated your Mac to macOS 12.6, or if you updated the Apple command line tools and/or Xcode to version 14 on macOS 12.5 or higher, and are unable to install Ruby due to an error that says “ld: symbol(s) not found for architecture arm64”, this guide is for you.

You can find out which version of the command line tools you’re using by running brew config, then look for the lines that start with CLT: and Xcode:.

TL;DR
If you’re a [Ruby on Mac](https://www.blogger.com/blog/post/edit/6156948649504557105/2127861233798747434#) Basic customer, download the latest version, and follow the installation instructions. For existing Prime and Ultimate customers, you can update to the latest version by simply running romup in your Terminal.

For non-customers who use ruby-install, you can install Ruby by adding the --enable-shared flag, like this:

`ruby-install 3.1.2 -- --enable-shared`

If you use a different tool, such as asdf or rbenv, you should be able to install 2.7.6 or 3.1.2 without issues. If you are having issues, or if you’re trying to install older Ruby versions, keep reading.

Also, I was able to install Ruby 3.2.0-preview2 without having to add any special flags, which tells me that this bug has been fixed upstream by the Ruby team. Hopefully, they will release 2.7.7 and 3.1.3 with this fix as well.

UPDATE: [PR 6440 in the Ruby GitHub repo](https://www.blogger.com/blog/post/edit/6156948649504557105/2127861233798747434#) backports this fix to 3.1 and 2.7. This means that when 3.1.3 and 2.7.7 are released soon, you’ll be able to install them without any workarounds.

My troubleshooting journey
Like many people, I upgraded my Macs to macOS 12.6 yesterday. I have two Intel Macs, and one M1 MacBook Air. After the upgrade, all my existing Ruby projects were still working fine. Then I noticed an email from [F5Bot](https://www.blogger.com/blog/post/edit/6156948649504557105/2127861233798747434#) with a new [Reddit post](https://www.blogger.com/blog/post/edit/6156948649504557105/2127861233798747434#) matching the keyword “install ruby”.

In the post, Ed Mangimelli explains that he was having trouble installing Ruby 2.6.x after the update to 12.6, so I tried to reproduce the issue on all my Macs. Knowing that the oldest version of Ruby that can be installed natively (i.e. without Rosetta) on Apple Silicon is 2.6.8, I first tried to install 2.6.10 with ruby-install, the Ruby installation tool I normally use.

Why 2.6.10? Because it’s the latest in the 2.6.x series, and it’s recommended to update your projects to the latest patch version to keep them secure and up to date.

Important side note
Ruby 2.6.x reached end of life in March 2022, so it should not be used in production for any critical projects. Instead, I recommend updating your Ruby project to use at least 2.7.6 or 3.1.2.

In some cases, all that might be needed is to update the Ruby version in your Gemfile and/or .ruby-version file, cd out and back into your project, and run bundle. Then see if your project still works.

This assumes you have already installed 2.7.6 or 3.1.2, and ran gem install bundler, which [Ruby on Mac](https://www.blogger.com/blog/post/edit/6156948649504557105/2127861233798747434#) automates for you.

Similarly, if your project is using a version of 2.6.x lower than 2.6.8, that doesn’t mean you have to use that version. I keep seeing a lot of people try to install Ruby 2.6.6 on Apple Silicon, and recommendations to add the -Wno-error=implicit-function-declaration flag.

While that may allow Ruby to be installed, it can lead to other issues down the line. It’s also dangerous because it overrides macOS defaults.

Benoit Daloze, one of the rbenv and ruby-build maintainers, [explains](https://www.blogger.com/blog/post/edit/6156948649504557105/2127861233798747434#) in more detail why this flag should not be used:

First, it does not seem advisable to disable something the OS does by default. Actually it can be a pretty dangerous workaround, because then the compiler does not know precise argument types and could end up passing arguments incorrectly, which could cause all kinds of problems.

Also, recent CRuby enables that flag by default, and it would be very bad to disable something CRuby enables on purpose.

Since you can install 2.6.8 and greater natively on Apple Silicon without any dangerous hacks, and since there shouldn’t be any breaking changes between older 2.6.x versions and newer ones, you should be able to get your project up and running by simply updating the version in your Gemfile and/or .ruby-version.

Back to the troubleshooting
Installing 2.6.10 with ruby-install failed with this error on my M1 Air:

```
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make[2]: *** [../../.ext/arm64-darwin21/strscan.bundle] Error 1
make[1]: *** [ext/strscan/all] Error 2
make: *** [build-ext] Error 2
!!! Compiling ruby 2.6.10 failed!
```

On my Intel MacBook Air, it worked fine, but it failed on my Intel iMac with this error:

```
linking shared-object openssl.bundle
ld: warning: -undefined dynamic_lookup may not work with chained fixups
installing default openssl libraries
extracting ripper.y from ../.././parse.y
compiling compiler ripper.y
ripper.y:762.9-16: syntax error, unexpected identifier, expecting string
make[2]: *** [ripper.c] Error 1
make[1]: *** [ext/ripper/all] Error 2
make: *** [build-ext] Error 2
!!! Compiling ruby 2.6.10 failed!
```

That didn’t make sense, so I ran through my usual troubleshooting steps. The first thing I check after a macOS update is whether Homebrew is happy:

`brew doctor`

It said “Your system is ready to brew,” so the next step was to reinstall all the Ruby-related tools so they can be compiled against the latest Apple command line tools that were installed as part of the update to 12.6. Since I use chruby and ruby-install with the fish shell, these are the tools I reinstalled:

`brew reinstall automake bison gdbm libffi libyaml openssl@1.1 readline chruby ruby-install chruby-fish gcc`

Then I tried installing 2.6.10 again, and it worked this time on the Intel iMac.

Next, I tried installing 3.1.2 on my M1 Air with ruby-install, but it failed with a different error:

```
linking shared-object -test-/arith_seq/extract.bundle
Undefined symbols for architecture arm64:
  "_rb_arithmetic_sequence_extract", referenced from:
      _arith_seq_s_extract in extract.o
  "_rb_ary_new_capa", referenced from:
      _arith_seq_s_extract in extract.o
  "_rb_ary_store", referenced from:
      _arith_seq_s_extract in extract.o
  "_rb_define_singleton_method", referenced from:
      _Init_extract in extract.o
  "_rb_path2class", referenced from:
      _Init_extract in extract.o
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make[2]: *** [../../../../.ext/arm64-darwin21/-test-/arith_seq/extract.bundle] Error 1
make[1]: *** [ext/-test-/arith_seq/extract/all] Error 2
make: *** [build-ext] Error 2
```

The next step was to rule out the tool used for installing Ruby and see if I can compile Ruby manually:

```
brew install wget
cd ~/src
wget https://cache.ruby-lang.org/pub/ruby/3.1/ruby-3.1.2.tar.xz
tar -xzvf ruby-3.1.2.tar.xz
cd ruby-3.1.2
./configure --with-opt-dir="$(brew --prefix openssl@1.1):$(brew --prefix readline):$(brew --prefix libyaml):$(brew --prefix gdbm)" --prefix=/Users/moncef/.rubies/ruby-3.1.2
make
make install
```

It failed with the same error!

So then I try rbenv, and it was able to install 3.1.2 just fine, but I notice that it’s using OpenSSL 3, so I try ruby-install again with OpenSSL 3, but that still doesn’t work.

Then I compare the configuration options between ruby-install and rbenv and I notice that rbenv has enable shared turned ON, but ruby-install has it turned OFF.

With a carefully crafted search query in DuckDuckGo, I found this [Ruby bug report](https://www.blogger.com/blog/post/edit/6156948649504557105/2127861233798747434#) with macOS 13 Ventura Beta which was using Xcode 14 beta. macOS 12.6 uses the Xcode 14 command line tools too.

In the bug report, they provide a fix by passing in some configuration options, one of which was --enable-shared. Aha! Just like rbenv is doing. But when I tried to use both flags as written in the bug report, it didn’t work:

`--with-out-ext=+,bigdecimal --enable-shared`

So I thought maybe it’s supposed to be 2 separate flags for excluding the extensions:

`--with-out-ext=+ --with-out-ext=bigdecimal`

And that worked!

But then I thought is there really an extension called +? So, I removed it and it still worked. So it looks like only bigdecimal needs to be excluded. But then I thought, isn’t that gonna cause problems with Rails app or other Ruby projects? And sure enough, my Rails apps didn’t work anymore because they were looking for bigdecimal.

This is another example of the dangers of blindly adding configuration options when installing Ruby. Just because Ruby installed successfully doesn’t mean everything else will be fine, and most people will not realize that the issues are due to the way Ruby was compiled.

So I took that flag out and just used the --enable-shared flag, and that still worked! And my Rails apps still work as well.

So then I tried installing 2.6.9 with this flag:

`ruby-install 2.6.9 -- --enabled-shared`

But it failed with this error:

```
linking static-library libruby.2.6-static.a
linking shared-library libruby.2.6.dylib
Undefined symbols for architecture arm64:
  "__mh_execute_header", referenced from:
      _rb_dump_backtrace_with_lines in addr2line.o
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [libruby.2.6.dylib] Error 1
!!! Compiling ruby 2.6.9 failed!
```

So then I tried reverting to the previous command line tools, as Ed suggested in his Reddit post. The only difference is I’m only downloading the standalone command line tools, and not the full Xcode app.

Note that it should never be necessary to download the huge 7GB Xcode app unless you’re building macOS and iOS apps. In other words, if you’re never going to use the Xcode app itself, all you need are the standalone Command Line Tools.

[Download “Command Line Tools for Xcode 13.4”](https://www.blogger.com/blog/post/edit/6156948649504557105/2127861233798747434#). I think you can sign in with your existing Apple ID without having to create a new developer account.
Install them, but keep the DMG when it asks you if you want to Trash it, just in case you need to install them again. I also recommend downloading “Command Line Tools for Xcode 14” as well, so you can switch between both versions easily.
In your terminal, run xcode-select -p. If it says /Library/Developer/CommandLineTools, you’re good to go. Otherwise, change the location with sudo xcode-select -s /Library/Developer/CommandLineTools/
Check the path again with xcode-select -p
Check that Homebrew is using the correct CLT with brew config. You should see CLT: 13.4.0.0.1.1651278267. If not, remove the CLT with sudo rm -rf /Library/Developer/CommandLineTools and reinstall them.
Reinstall all Ruby-related tools with Homebrew. In my case, it’s these:

`brew reinstall automake bison gdbm libffi libyaml openssl@1.1 readline chruby ruby-install gcc`

Then install Ruby 2.6.9: ruby-install 2.6.9
This worked for me, but 2.6.10 still failed for some reason.

Conclusion

After updating to macOS 12.6, you should still be able to install 2.7.x and 3.x versions of Ruby with rbenv or asdf. With ruby-install, you’ll need to add the --enable-shared flag.

If you’re having issues, check Homebrew’s health with brew doctor, fix any issues it reports, and try reinstalling the Ruby tools.

If you absolutely must use Ruby 2.6.x, then revert to the 13.4 CLT.