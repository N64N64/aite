# Aite

I'd like to say I'm really sorry you have to use this to build my projects. I really am. But come on, makefiles suck. You can't really blame me for that, can you? I promise this isn't that hard to install.

### Warning / Disclaimer

This is a wip, I don't care about backwards compatibility, blahblahblah

# Dependencies

* macOS
* LuaJIT (`brew install luajit`)
* dpkg-deb (`brew install dpkg-deb`)

# How to install

```bash
cd /some/directory
git clone https://github.com/rweichler/aite
cd /usr/local/bin
ln -s /some/directory/aite/main.lua aite
```

Now you have the `aite` command.

## `targets.lua` example

```lua
-- this is called when `make` is
-- called with no arguments
function default()
    -- cleanup old layout folder
    os.pexecute("rm -rf layout")

    -- setup compiler
    local b = builder('apple')
    b.compiler = 'clang'
    b.src = fs.scandir('*.m') --compile all .m files in current directory
    b.frameworks = {
        'UIKit',
        'Foundation'
    }
    b.output = 'layout/MobileSubstrate/DynamicLibraries/my_sick_tweak.dylib'

    -- compile
    local objs = b:compile()
    -- link
    b:link(objs)

    -- setup debber
    local d = debber()
    d.input = 'layout'
    d.output = 'lmfao.deb'
    d.packageinfo = {
        Name = 'LMFAO',
        Package = 'com.apple.lmfao',
        Version = 1.0,
    }

    -- create deb file
    d:make_deb()
end
```

If your project was just a bunch of `.m` files, it would compile all of them and then link them with UIKit, and then give you a dylib.

Then, it would make a deb, that you could put on your repo.

## Using builder

So there are two ways to do this.

* `local b = builder()`
* `local b = builder('apple')`

I'd recommend always having 'apple' in there. That way you get these:

* `b.frameworks`: Apple frameworks you want to link with. Public or private.
* `b.archs`: table listing of the archs you want to use (e.g. `armv7`, `arm64`, `x86_64`)
* `b.sdk`: name of the SDK you wanna link against (e.g. `iphoneos` or `macosx`)
* `b.sdk_path`: the full path to the iPhoneOS8.1.sdk (or whatever), in case if you don't have Xcode.


These are the ones included by default, no matter what:

* `b.compiler`: e.g. `gcc` or `clang`
* `b.src`: Table of all the source files you'll be compiling
* `b.build_dir`: The folder where you want all the ugly `.o` files to go.
* `b.output`: Where the executable should go. Default is `a.out`. Add `.dylib` to the end to make a dylib that can be dynamically loaded.
* `b.defines`: Table listing of what you want `#define`d at compile time.

Advanced ones: 

* `b.linker`: The linker. If this isn't set, it will use `b.compiler`.
* `b.include_dirs`: table listing all of the folders you wanna include from (like rpetrich's iphoneheaders or something)
* `b.library_dirs`: table listing of all the folders you wanna look in for libraries.
* `b.libraries`: table listing of the libraries you want to link to.

## Using debber

Create it using this:

```lua
local d = debber()
d.input = 'layout' -- folder where the "layout" of the deb will be
d.output = 'package.deb' -- self-explanitory
d.packageinfo = { -- equivalent of the DEBIAN/control file
    Name = 'Yeee',
    Package = 'yee.yee.yee',
    Version = '1.0',
}
```

Run it using this:

```lua
d:make_deb()
```


## Things to keep in mind

* This only works with C/C++/Objective-C code. Logos isn't supported yet.
* In my example I only had one builder. But you can have as many as you want! If you have `c` and `cpp` files that require different compilers, then you can just use two builders to compile them, and then `table.merge` them and pass them to a linker.
