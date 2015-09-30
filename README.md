Mono buildpack for heroku, built on every stable mono build.

[![](https://circleci.com/gh/AdamBurgess/heroku-buildpack-mono-builder.png?style=shield&circle-token=fe5a1697660ac8727b496f624407ea006b2069d7)](https://circleci.com/gh/AdamBurgess/heroku-buildpack-mono-builder)

#### what to use this for

C# console apps running on the latest (T-minus 2 hours) mono version. (or, any mono version available, starting from 4.0.4*)

Note: If you're running on ASP, use Heroku's unofficial official buildpack instead: [heroku/dotnet-buildpack](https://github.com/heroku/dotnet-buildpack).

You can still run webservers; self-hosting OWIN and NancyFx work perfectly. I recommend [Nowin](//github.com/Bobris/Nowin).

#### how to use this

1. Put a solution in the root directory of your app
2. Use nuget to get dependencies
3. Run `heroku buildpacks:set https://github.com/AdamBurgess/heroku-buildpack-mono` to set the buildpack
4. Optionally configure the version of mono using a `.mono` file.
5. Add `mono ProjectName.exe` to your Procfile. Apps will be built into the root directory

#### more options

Create a `.mono` file to configure more options:

````bash
# put a tag name here that has a corresponding release to specify a version*
MONO_VERSION=96e40c5793ff
# specify the build you want, either minimal (default) or full
# see below for explanation
MONO_TYPE=full
# if this is set, the cache is not used/cleaned.
# the cache is used to 1) store mono builds and 2) store nuget packages
MONO_CACHE=nope
````

#### how this works

1. The latest built version for mono is downloaded from Github's Releases
2. Nuget dependencies are restored
3. The solution is built using mono's `xbuild`

#### how mono is built

1. Someone pushes to [mono/mono](//github.com/mono/mono)
2. Mono's CI server, [Jenkins](//jenkins.mono-project.com/job/test-mono-mainline/label=debian-amd64/), compiles and runs the test suite for mono.
3. A [scheduled task](//github.com/AdamBurgess/heroku-buildpack-mono-watcher) checks Jenkins every hour to see if a new build has completed and passed all tests.
4. If it has, it creates a new commit in [another project](//github.com/AdamBurgess/heroku-buildpack-mono-builder) which makes [Circle CI](//circleci.com/gh/AdamBurgess/heroku-buildpack-mono-builder) start building Mono.
5. Once completed, it then creates a commit, tag, and release here, which is fetched by Heroku on building your app.

#### full vs minimal builds
The builder creates two version of mono:
1. Full build. ~95mb slug size increase.
  * Includes everything, including libgdiplus, boehm, libraries (e.g. for use in mkbundle).
2. Minimal build. ~30mb slug size increase.
  * Removes shared libraries, boehm, includes, System.Windows.Forms

For most, the minimal build will work fine, and it is the default. Enable the full build if you need its resources or are having problems.

#### * I want version [x] of Mono. How can I get it?

Create a pull request in [AdamBurgess/heroku-buildpack-mono-builder](//github.com/AdamBurgess/heroku-buildpack-mono-builder) changing the `latest` file to the tag or commit that you want, and I'll merge and see that it builds successfully.
