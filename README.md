Mono buildpack for heroku, built on every stable mono build.

[![](https://circleci.com/gh/AdamBurgess/heroku-buildpack-mono-builder.png?style=shield&circle-token=fe5a1697660ac8727b496f624407ea006b2069d7)](https://circleci.com/gh/AdamBurgess/heroku-buildpack-mono-builder)

#### what to use this for

C# console apps running on the latest (T-minus 2 hours) mono version.

Note: ASP and similar aren't supported. For that, use Heroku's unofficial official buildpack: [heroku/dotnet-buildpack](https://github.com/heroku/dotnet-buildpack).

#### how to use this

1. Put a solution in the root directory of your app
2. Use nuget to get dependencies
3. Add https://github.com/AdamBurgess/heroku-buildpack-mono.git to your BUILDPACK_URL
4. Add `mono ProjectName.exe` to your Procfile. Apps will be built into the root directory

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
