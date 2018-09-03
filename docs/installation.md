---
id: installation
title: Installation
---

## CLI

The recommended way to install the scalafmt command line tool is with
[Coursier](#coursier).

### Coursier

<div class="sidenote">
To install Coursier see <a href="https://github.com/coursier/coursier#command-line" target="_blank">here</a>
</div>

Create a standalone executable in `/usr/local/bin/scalafmt` with (sudo if
necessary):

```sh
coursier bootstrap com.geirsson:scalafmt-cli_2.12:@STABLE_VERSION@ \
  -r bintray:scalameta/maven \
  -o /usr/local/bin/scalafmt --standalone --main org.scalafmt.cli.Cli
scalafmt --version # should be @STABLE_VERSION@
```

Alternatively you can create a slim 15 KiB bootstrap script with:

```sh
coursier bootstrap com.geirsson:scalafmt-cli_2.12:@STABLE_VERSION@ \
  -r bintray:scalameta/maven \
  -o scalafmt --main org.scalafmt.cli.Cli
./scalafmt --version # should be @STABLE_VERSION@
```

It is **recommended** to put this bootstrap script in your code repository to
make sure everyone on your team, as well as CI, uses the same scalafmt version.
To configure which files to format, see [project](configuration.md#project).

To customize the JVM options, use the Coursier option `--java-opt`, more info
with

```sh
coursier bootstrap --help | grep -A 1 "\-\-java-opt"
```

### Pre-release

Our CI publishes a pre-release version of scalafmt to Bintray on every merge
into master. To use a pre-release, replace @STABLE_VERSION@ with the version
here:

<a href='https://bintray.com/scalameta/maven/scalafmt-cli/_latestVersion'>
    <img src='https://api.bintray.com/packages/scalameta/maven/scalafmt-cli/images/download.svg'>
</a>

If you use coursier to install a pre-release, be sure to include the flag -r
bintray:scalameta/maven so that the artifact can be resolved.

If you use sbt to install a pre-release, be sure to add the following setting

```scala
resolvers += Resolver.bintray("scalameta", "maven")
```

### Nailgun

Nailgun is recommended if you want to integrate scalafmt with a text editor like
vim/Emacs/Atom/Sublime/VS Code.

- Make sure you have a nailgun client installed. For example with
  `brew install nailgun`.
- Create a standalone executable in `/usr/local/bin/scalafmt_ng` with (sudo if
  necessary)

```sh
coursier bootstrap --standalone com.geirsson:scalafmt-cli_2.12:@STABLE_VERSION@ \
  -r bintray:scalameta/maven \
  -o /usr/local/bin/scalafmt_ng -f --main com.martiansoftware.nailgun.NGServer
scalafmt_ng & // start nailgun in background
ng ng-alias scalafmt org.scalafmt.cli.Cli
ng scalafmt --version # should be @STABLE_VERSION@
```

Nailgun keeps scalafmt running on a local server to avoid the JVM startup
penalty and also so scalafmt can benefit from JIT. This makes scalafmt up to 10x
faster when formatting a single file from the CLI. The downside to Nailgun is
that the setup is complicated and the long-running server needs to be restarted
once in awhile.

### Homebrew

You can install scalafmt via Homebrew using a custom formula

```sh
brew install --HEAD olafurpg/scalafmt/scalafmt
scalafmt --version // should be @STABLE_VERSION@

// to upgrade between releases
brew upgrade scalafmt
```

### --help

```scala mdoc:passthrough
println(website.plaintext(org.scalafmt.cli.CliArgParser.buildInfo))
```

```scala mdoc:passthrough
println(website.plaintext(org.scalafmt.cli.CliArgParser.scoptParser.usage))
```

## IntelliJ

<div class="sidenote">
  Please upvote <a href="https://youtrack.jetbrains.com/issue/SCL-13658">this Jetbrains ticket</a>
  about adding built-in support for Scalafmt in the IntelliJ Scala plugin.
</div>

[Here is the plugin](https://plugins.jetbrains.com/plugin/8236?pr=). You can
install it directly from within IntelliJ:

- open `Settings > Plugins`
- open `Browse repositories`
- search for `scalafmt`
- restart IntelliJ.

The default shortcut is `Ctrl + Shift + L`. Undo works, but not redo.

The plugin determines which style to use in this order:

1. `.scalafmt.conf` in the project's root directory, if it exists
1. `$HOME/.scalafmt.conf`, if it exists
1. Otherwise, uses `default` style.

For details on how `.scalafmt.conf` should look like, see
[Configuration](configuration.md). The scalafmt IntelliJ plugin has a "Format on
save" setting.

- To enable for current project: `Settings > Tools > Scalafmt`
- To enable for all future project:
  `File > Other settings > Default settings > Scalafmt`

## sbt

You can choose between

- [sbt-scalafmt](#sbt-scalafmt) (sbt 1.0 only)
- [neo-sbt-scalafmt](#neo-sbt-scalafmt) (sbt 0.13 and sbt 1.0)

### sbt-scalafmt

```scala
// In project/plugins.sbt. Note, does not support sbt 0.13, only sbt 1.0.
addSbtPlugin("com.geirsson" % "sbt-scalafmt" % "1.5.1")
```

The sbt plugin is enabled by default for the Test and Compile configurations. To
enable the plugin for integration tests

```scala
inConfig(IntegrationTest)(scalafmtConfigSettings)
```

and then use `it:scalafmt` to format.

Pro tip. To share configuration across projects, you can define a setting in
`build.sbt` to generate `.scalafmt.conf` programmatically on sbt load.

```scala
// define setting key to write configuration to .scalafmt.conf
SettingKey[Unit]("scalafmtGenerateConfig") :=
  IO.write( // writes to file once when build is loaded
    file(".scalafmt.conf"),
    """style = IntelliJ
      |# Your configuration here
      """.stripMargin.getBytes("UTF-8")
  )
```

### neo-sbt-scalafmt

[lucidsoftware/neo-sbt-scalafmt](https://github.com/lucidsoftware/neo-sbt-scalafmt)
is an sbt plugin that

- supports both sbt 0.13 and 1.0.0
- supports any version of scalafmt
- runs in-process
- uses SBT's update resolutions mechanism, which tends to be slow for large
  multi-module builds.

## Gradle

It is possible to use scalafmt in gradle with the following externally
maintained plugins:

- [Spotless](https://github.com/diffplug/spotless/tree/master/plugin-gradle#applying-scalafmt-to-scala-files)
- [gradle-scalafmt](https://github.com/alenkacz/gradle-scalafmt)

## Maven

It is possible to use scalafmt in Maven with the following externally maintained
plugin:

- [mvn_scalafmt](https://github.com/SimonJPegg/mvn_scalafmt)

## Mill

Mill have scalafmt support built-in:

- [scalafmt module](http://www.lihaoyi.com/mill/page/configuring-mill.html#reformatting-your-code)

## Vim

- Make sure you have the [CLI](#cli) installed and working.
- install [vim-autoformat](https://github.com/Chiel92/vim-autoformat)
- add to your `.vimrc`

```
noremap <F5> :Autoformat<CR>
let g:formatdef_scalafmt = "'scalafmt --stdin'"
let g:formatters_scala = ['scalafmt']
```

<div class="sidenote">
You pay the JVM startup penalty on every format unless you're using
<a href="#nailgun">Nailgun</a>.
</div>

## Standalone library

Add to your dependencies

```scala
libraryDependencies += "com.geirsson" %% "scalafmt-core" % "@STABLE_VERSION@"
// Scala.js
libraryDependencies += "com.geirsson" %%% "scalafmt-core" % "@STABLE_VERSION@"
```

Use the API like this:

```scala mdoc
org.scalafmt.Scalafmt.format("""
      object FormatMe { List(Split(Space, 0).withPolicy(SingleLineBlock(close)), Split(Newline, 1).withPolicy{ case Decision(t@FormatToken(_, `close`, _), s) => Decision(t, List(Split(Newline, 0)))}.withIndent(2, close, Right)) }
""").get
```

Obtain a configuration object with `parseHoconConfig`

```scala mdoc
val config = org.scalafmt.Scalafmt.parseHoconConfig("align=most").get
org.scalafmt.Scalafmt.format("""
    object Align {
        val x = 1
        val xx = 2
    }
""", config).get
```

To format code with top-level statements like `*.sbt` files

```scala mdoc
val configForSbt = org.scalafmt.Scalafmt.configForSbt(config)
org.scalafmt.Scalafmt.format("""
    val x = 1
    val xx = 2
""", configForSbt).get
```

The Scalafmt public API consists only of methods in`org.scalafmt.Scalafmt`. In
particular, case classes in `org.scalafmt.config` are subject to binary and
source breaking changes on any release.

## Help Wanted

- Ensime
- Scala IDE ([help wanted!](https://github.com/scalameta/scalafmt/issues/125))
- Your favorite editor? Join the gitter channel.

[intellij-ticket]: https://youtrack.jetbrains.com/issue/SCL-13658