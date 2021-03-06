# tut 

<img src="https://api.travis-ci.org/tpolecat/tut.svg?branch=master"/><br>
[![Join the chat at https://gitter.im/tpolecat/tut](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/tpolecat/tut?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

<img alt="How'd you get so funky?" align=right src="tut.jpg"/>

**tut** is a very simple documentation tool for Scala programs that reads Markdown files and interprets code in `tut` sheds. So you add **tut** as an [sbt](http://scala-sbt.org) plugin and then you can write tutorials that are typechecked and run as part of your build. The idea is to have tutorial code that is never out of sync with the code it's documenting.

The current version is **0.3.2**, which runs on **Scala 2.10** and **2.11**.

##### Recent Notable Changes

- Version **0.3.2** adds support for compiler plugins and `scalacOptions` defined in `(Compile, doc)`; these are propagated to  **tut** REPL sessions. This has been tested with  [kind-projector](https://github.com/non/kind-projector) and works fine. Also new is the `invisible` modifier for interpreting a block but producing no output at all.
- Version **0.3.1** improves error reporting, removes confusing caching behavior, and removes the dependency on scalaz.
- version **0.3.0** was a **breaking change** with previous versions! **tut** now looks for `tut` sheds rather than `scala` sheds, so existing documentation will need to be modified when you upgrade.

##### Projects Using tut

These are just the ones I know about. If you are using **tut** for your doc give me a shout and I'll add to this list.

- The [doobie](https://github.com/tpolecat/doobie) database library uses **tut** for its [reference documentation](http://tpolecat.github.io/doobie-0.2.0/00-index.html).
- The [cats](https://github.com/non/cats) library for FP in Scala uses **tut** for numerous [examples](http://non.github.io/cats/) (in progress).

### How-To

**tut** looks for code in `tut` sheds and (by default) replaces it with what you would see if you had pasted the code into a REPL. As an example, the input file

    Here is how you add numbers:
    ```tut
    1 + 1
    ```

is rewritten as

    Here is how you add numbers:
    ```scala
    scala> 1 + 1
    res0: Int = 2    
    ```

The code runs from top to bottom (imports and definitions from earlier code blocks are available in subsequent blocks), with a new REPL session for each input file.

You can follow `tut` with any number of colon-prefixed modifiers to alter the way a block is interpreted.

- Normally an error in interpretation causes the buid to fail, but if you want to include an example that fails to compile you can add the `nofail` modifier.
- If you don't want REPL prompts or responses you can use the `silent` modifier. Code is still interpreted and errors will cause the build to fail, but no REPL output will appear. If you want no output at all (i.e., even your code is hidden) use `invisible`.
- If you don't want Scala syntax highlighting, use the `plain` modifier.

For example

    This won't compile:
    ```tut:nofail
    blech?
    ```

Note that if you want a code block that is not interpreted at all, just use a normal `scala` shed; **tut** doesn't touch these.

### Setting Up

Add the following to `project/plugins.sbt` in your project to add SBT shell commands:

```scala
resolvers += Resolver.url(
  "tpolecat-sbt-plugin-releases",
    url("http://dl.bintray.com/content/tpolecat/sbt-plugin-releases"))(
        Resolver.ivyStylePatterns)

addSbtPlugin("org.tpolecat" % "tut-plugin" % "0.3.2")
```

And add the following to `build.sbt` for the tut runtime, which must run alongside your code:

```scala
tutSettings
```

This will add the following to your SBT world:

- `tut` is a task that interprets **all files** in `tutSourceDirectory` (`src/main/tut` by default) and writes output to `target/<scala-version>/tut`. If the code fails to compile or otherwise barfs, you will get a message that directs you to the file and line where the failure happened, along with the REPL error, and the build will fail ... **except** for failures that are in a `tut:nofail` block, which are still reported in output but are ignored for the purposes of build success.
- `tutScalacOptions` is a list of scalac options to pass to REPL sessions; by default this value is taken from `scalacOptions in (Compile, doc)`. `tutPluginJars` is a list of plugin jarfiles to add to REPL sessions; by default this list is derived from plugins found in `libraryDependencies in (Compile, doc)`. It is unlikely that you will need to change either of these; typically you want **tut** to use the same settings you use in your library code.

### Integration with sbt-site

Tut is designed to work seamlessly with sbt-site so that your checked tutorials can be incorporated into your website.

Add the following to `project/plugins.sbt` in your project to add SBT shell commands:

```scala
addSbtPlugin("com.typesafe.sbt" % "sbt-site" % "0.8.1")
```

Then in your build sbt, link the files generated by tut to your site generation:

```scala
project("name").settings(site.addMappingsToSiteDir(tut, "tut"))
```

When the `buildSite` task is run in sbt, the typechecked tutorials from `src/main/tut` will be incorporated with the site generated by sbt-site in `target/site`.

### Particulars

- Each tutorial is an independent REPL session, and the code examples run from top to bottom.
- Code in `tut` sheds will be interpreted. Anything in your dependencies will be available. Interpreted code runs with the same classpath as `(Compile, doc)` with the addition of the **tut** runtime.
- Blank lines in sheds are ignored. Multi-line definitions work, but `:paste` style definitions (for mutual recursion for example) don't work [yet].
- ANSI escapes in REPL output are filtered out.

### Complaints and other Feedback

Feedback of any kind is always appreciated. 

Issues and PR's are welcome, or just find me on Twitter or `#scala` on FreeNode or on [gitter](https://gitter.im/tpolecat/tut).


