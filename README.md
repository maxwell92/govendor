# 简明使用教程

1. 使用`go get -u github.com/kardianos/govendor`将该包下载到$GOPATH/src，尝试运行govendor命令，如果不能运行，说明该包还未安装。在src/kardianos/govendor目录下，运行`go build main.go`，并将生成的二进制文件main重命名为govendor并复制到/usr/bin下。

2. 以一个项目govendor-example为例，它建立在$GOPATH/src下，首先对它进行`govendor init`操作，然后`govendor list`列出所需要的第三方包：

![](http://7xiwbf.com1.z0.glb.clouddn.com/2govendorlist.png)

提示缺失第三方包github.com/gorilla/mux，使用`govendor get`下载该包并再次`govendor list`，得到：

![](http://7xiwbf.com1.z0.glb.clouddn.com/2govendorlist2.png)

标志e表示mux包已经存在于本地，但未包含，提示缺失第三方包github.com/gorilla/context，再次使用`govendor get`下载该包：

![](http://7xiwbf.com1.z0.glb.clouddn.com/2govendorlist3.png)

使用命令`govendor add +external`将这两个包添加到项目里(这两个包会导入到vendor目录下)：

![](http://7xiwbf.com1.z0.glb.clouddn.com/2govendorlist5.png)

这个时候可以查看vendor/vendor.json文件，得到第三方包的基本信息：

![](http://7xiwbf.com1.z0.glb.clouddn.com/2vendorjson.png)

如果需要获取指定版本的包，可以使用命令`govendor fetch golang.org/x/net/context@=v1`，更新context包为v1.1。这个包会存在vendor目录里，再次查看vendor.json：

![](http://7xiwbf.com1.z0.glb.clouddn.com/2vendorjson2.png)


`govendor fetch`提供了三种指定包版本的方式：

```
govendor fetch golang.org/x/net/context@a4bbce9fcae005b22ae5443f6af064d80a6f5a55
govendor fetch golang.org/x/net/context@v1   # Get latest v1.*.* tag or branch.
govendor fetch golang.org/x/net/context@=v1  # Get the tag or branch named "v1".
```

这里第一种方式里，后面的长字符串指的是commit的序号，可以在相应的项目commit记录里找到，它也与vendor/vendor.json里面的值对应起来：

![](http://7xiwbf.com1.z0.glb.clouddn.com/commitserial.png) 
![](http://7xiwbf.com1.z0.glb.clouddn.com/2govenderjson3.png)


可以采用`govendor update`或`govendor remove`来更新或移除特定的包。update会将该包的最新版本(commit)导入，remove会将vendor目录下的包删除。

*ISSUE*:
假设使用命令`govendor fetch xxxx@=vx`失败，比如遇到网路问题没能下载成功，但是通过`govendor list`和`cat vendor/vendor.json`会发现版本已经更改为了vx，具体的是version显示新版本，versionExact显示旧版本。







# The Vendor Tool for Go
`go get -u github.com/kardianos/govendor`

New users please read the [FAQ](doc/faq.md)

Package developers should read the [developer guide](doc/dev-guide.md).

For a high level overview read the [whitepaper](doc/whitepaper.md)

Uses the go1.5+ vendor folder. Multiple workflows supported, single tool.

[![Build Status](https://travis-ci.org/kardianos/govendor.svg?branch=master)](https://travis-ci.org/kardianos/govendor)
[![GoDoc](https://godoc.org/github.com/kardianos/govendor?status.svg)](https://godoc.org/github.com/kardianos/govendor)

 * Copy existing dependencies from $GOPATH with `govendor add/update`.
 * If you ignore `vendor/*/`, restore dependencies with `govendor sync`.
 * Pull in new dependencies or update existing dependencies directly from
	remotes with `govendor fetch`.
 * Migrate from legacy systems with `govendor migrate`.
 * Supports Linux, OS X, Windows, probably all others.
 * Supports git, hg, svn, bzr (must be installed an on the PATH).

## Notes

 * The project must be within a $GOPATH.
 * If using go1.5, ensure you `set GO15VENDOREXPERIMENT=1`.

### Quick Start, also see the [FAQ](doc/faq.md)
```
# Setup your project.
cd "my project in GOPATH"
govendor init

# Add existing GOPATH files to vendor.
govendor add +external

# View your work.
govendor list

# Look at what is using a package
govendor list -v fmt

# Specify a specific version or revision to fetch
govendor fetch golang.org/x/net/context@a4bbce9fcae005b22ae5443f6af064d80a6f5a55
govendor fetch golang.org/x/net/context@v1   # Get latest v1.*.* tag or branch.
govendor fetch golang.org/x/net/context@=v1  # Get the tag or branch named "v1".

# Update a package to latest, given any prior version constraint
govendor fetch golang.org/x/net/context

# Format your repository only
govendor fmt +local

# Build everything in your repository only
govendor install +local

# Test your repository only
govendor test +local

```

## Sub-commands
```
	init     Create the "vendor" folder and the "vendor.json" file.
	list     List and filter existing dependencies and packages.
	add      Add packages from $GOPATH.
	update   Update packages from $GOPATH.
	remove   Remove packages from the vendor folder.
	status   Lists any packages missing, out-of-date, or modified locally.
	fetch    Add new or update vendor folder packages from remote repository.
	sync     Pull packages into vendor folder from remote repository with revisions
  	             from vendor.json file.
	migrate  Move packages from a legacy tool to the vendor folder with metadata.
	get      Like "go get" but copies dependencies into a "vendor" folder.
	license  List discovered licenses for the given status or import paths.
	shell    Run a "shell" to make multiple sub-commands more efficient for large
	             projects.

	go tool commands that are wrapped:
	  `+<status>` package selection may be used with them
	fmt, build, install, clean, test, vet, generate, tool
```

## Status

Packages can be specified by their "status".
```
	+local    (l) packages in your project
	+external (e) referenced packages in GOPATH but not in current project
	+vendor   (v) packages in the vendor folder
	+std      (s) packages in the standard library

	+excluded (x) external packages explicitely excluded from vendoring
	+unused   (u) packages in the vendor folder, but unused
	+missing  (m) referenced packages but not found

	+program  (p) package is a main package

	+outside  +external +missing
	+all      +all packages
```

Status can be referenced by their initial letters.

 * `+std` same as `+s`
 * `+external` same as `+ext` same as `+e`
 * `+excluded` same as `+exc` same as `+x`

Status can be logically composed:

 * `+local,program` (local AND program) local packages that are also programs
 * `+local +vendor` (local OR vendor) local packages or vendor packages
 * `+vendor,program +std` ((vendor AND program) OR std) vendor packages that are also programs
	or std library packages
 * `+vendor,^program` (vendor AND NOT program) vendor package that are not "main" packages.

## Package specifier

The full package-spec is:
`<path>[::<origin>][{/...|/^}][@[<version-spec>]]`

Some examples:

 * `github.com/kardianos/govendor` specifies a single package and single folder.
 * `github.com/kardianos/govendor/...` specifies `govendor` and all referenced
	packages under that path.
 * `github.com/kardianos/govendor/^` specifies the `govendor` folder and all
	sub-folders. Useful for resources or if you don't want a partial repository.
 * `github.com/kardianos/govendor/^::github.com/myself/govendor` same as above
	but fetch from user "myself".
 * `github.com/kardianos/govendor/...@abc12032` all referenced packages at
	revision `abc12032`.
 * `github.com/kardianos/govendor/...@v1` same as above, but get the most recent
	"v1" tag, such as "v1.4.3".
 * `github.com/kardianos/govendor/...@=v1` get the exact version "v1".

## Packages and Status

You may specify multiple package-specs and multiple status in a single command.
Commands that accept status and package-spec:

 * list
 * add
 * update
 * remove
 * fetch

You may pass arguments to govendor through stdin if the last argument is a "-".
For example `echo +vendor | govendor list -` will list all vendor packages.

## Ignoring build tags and excluding packages
Ignoring build tags is opt-out and is designed to be the opposite of the build
file directives which are opt-in when specified. Typically a developer will
want to support cross platform builds, but selectively opt out of tags, tests,
and architectures as desired.

To ignore additional tags edit the "vendor.json" file and add tag to the vendor
"ignore" file field. The field uses spaces to separate tags to ignore.
For example the following will ignore both test and appengine files.
```
{
	"ignore": "test appengine",
}
```

Similarly, some specific packages can be excluded from the vendoring process.
These packages will be listed as `excluded` (`x`), and will not be copied to the
"vendor" folder when running `govendor add|fetch|update`.

Any sub-package `foo/bar` of an excluded package `foo` is also excluded (but
package `bar/foo` is not). The import dependencies of excluded packages are not
listed, and thus not vendored.

To exclude packages, also use the "ignore" field of the "vendor.json" file.
Packages are identified by their name, they should contain a "/" character
(possibly at the end):
```
{
	"ignore": "test appengine foo/",
}
```
