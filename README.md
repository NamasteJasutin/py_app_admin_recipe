# Run Py2App and PyInstaller with Administrative Privilages on MacOs using AppleScript as a middleman.
MacOS .App Administrator Recicipe

I simply wasn't able to find this online and have found a solution leveraging AppleScript as a middleman.

AppleScript can create an Application from the IDE. They can also be signed, which I never did myself so I don't have the exact details. Some tips are at the bottom.

With a simple `do shell script quoted form of myPath & "Contents/MacOS/start" with administrator privileges` and some hacking, I could work my way around.

#
# For PyInstaller, follow these steps:
Create a single binary application for your Python app using:

`python3 -m PyInstaller -F --windowed main.py --osx-bundle-identifier \"{com.identifier.appname}\" --add-binary {file}:.`

Use `--windowed` if you use PyQt.

See [PyInstaller documentation](https://pyinstaller.readthedocs.io/en/stable/usage.html) for more info.

If you need binaries to be included, add each file with `--add-binary {fileHere}:.` to include it in the same root as `main.py`.

Test if it runs from a terminal before you continue. You'll have to follow the tracebacks or errors if you have any. Google is the best place.



From the PyInstaller folder, download `main.scpt.txt` and save as `main.scpt`.

Open `main.scpt` with AppleScript.

From this piece `"Contents/Resources/Scripts/App"` change `App` to the name of your PyInstaller binary.

We will place the binary later in the `Scripts` folder.

In the menu click File, then Export; as File Format, select `Application`.

Press save (signing needs to be done after!)



In Finder, move to your Application, press right-click then `Show Package Contents`.

Navigate to `Contents/Resources/Scripts/`. Place your PyInstaller app in this folder next to `main.scpt`.


Folder contents of `main.app` should look like this:
```
└── Contents
    ├── Info.plist
    ├── MacOS
    │   └── applet
    ├── PkgInfo
    └── Resources
        ├── Scripts
        │   ├── main.scpt
        │   └── App <- This is your Python binary
        ├── applet.icns
        ├── applet.rsrc
        └── description.rtfd
            └── TXT.rtf
```

If you did all the steps correct, taken I was able to explain well enough, you should be able to go up untill you see your Application.

Now double-click and 🤞!

If it works; 🎉

If not; Did you make sure PyInstaller made you a working binary?

Did you change `App` to what is the name of your binary?

🤷 Try to use logging in python.



Now, if you make any updates to your Python file, update your binary inside the Application by using Show Package Contents.

Change the Info.plist to bump the version number there as this is what MacOs uses to show the version of your app.

Place binary in `Scripts` folder.

Sign again.

Done.

#
# For Py2App, follow these steps:

*This one is fun.* Make sure you have a setup.py for your application.

If not, visit [Py2App Tutorial](https://py2app.readthedocs.io/en/latest/tutorial.html)

Make sure you set binaries to be included in `DATA_FILES` in `setup.py`.

In options, you can add `plist` or `iconfile` as dictionary items, if they need to be included.

See [Py2App Options](https://py2app.readthedocs.io/en/latest/options.html#option-reference).

Run `python3 setup.py py2app`.

This will create an Application in /dist/, which I'll call `Your.app` from now.

Please test the application first. If anything fails at this step, you should debug this before you continue.



Now we'll have to modify and mix this with the AppleScript.



From the Py2App folder, download `main.scpt.txt` and save as `main.scpt`.

Open `main.scpt` with AppleScript.

Don't change anything.

In the menu click File, then Export; as File Format, select `Application`.

Press name it something other than the Application you already have and press save (signing needs to be done after!)



Open 2 Finder windows and navigate to both your Py2App Applciation and in the other window the one you just exported.

Navigate the Application's files by clicking Show Package Contents on both of them.

**In 'Your.app':**
Navigate to `Contents/MacOS/`. Rename `applet` to `start`.

From `main.app/Contents/MacOS` copy `applet` into `Your.app/Contents/MacOS/`.

From `main.app/Contents/Resources/` copy the `Scripts` folder into `Your.app/Contents/Resources/`.

This will copy the AppleScript linker applet that refers to main.scpt, which refers back to the original applet that is now `start`, with `administrator privileges`.

Folder contents of `Your.app` should look like this:
```
└── Contents
    ├── Frameworks
    │   ├── Python.framework
    │   │   ├── Python -> Versions/Current/Python
    │   │   ├── Resources -> Versions/Current/Resources
    │   │   └── Versions
    │   │       ├── 3.7
    │   ├── libcrypto.1.1.dylib
    │   ├── libssl.1.1.dylib
    │   ├── libtcl8.6.dylib
    │   └── libtk8.6.dylib
    ├── Info.plist
    ├── MacOS
    │   ├── applet <- applet copied from AppleScript
    │   ├── python <- from py2app
    │   └── start <- original applet from py2app renamed
    ├── PkgInfo
    └── Resources
        ├── Scripts
        │   └── main.scpt <- AppleScript file
        ├── __boot__.py
        ├── __error__.sh
        ├── __pycache__
        │   └── site.cpython-37.pyc
        ├── applet.icns
        ├── include
        │   └── python3.7m
        │       └── pyconfig.h
        ├── lib
        │   ├── python3.7
... ommitted ...
```

If you did all the steps correct, taken I was able to explain well enough, you should be able to go up untill you see your Application.

Now double-click and 🤞!

If it works; 🎉!

If not; 🤷 Try to use logging in python.

Did it work when you created the executable from py2app?


#
# Add icon to your .App

If you want to change the icon, make sure you have it in .icns Apple Icon format.

Go into your Application Package Contents, navigate to `Contents/Resources/`.

Replace `applet.icns` with your own.


#
# Signing
I don't know anything of signing myself, but from what I know, these tools where used to sign the ones I made;

[Entitlements.plist](https://gist.githubusercontent.com/txoof/0636835d3cc65245c6288b2374799c43/raw/8b181198b374ca269a2347857f5526620458b3ac/
entitlements.plist)

[SDNotary](https://latenightsw.com/sd-notary-notarizing-made-easy/)