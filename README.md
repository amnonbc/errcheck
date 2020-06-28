# errcheck

errcheck is a program for checking for unchecked errors in go programs.

![Go](https://github.com/amnonbc/errcheck/workflows/Go/badge.svg)

It is forked from https://github.com/kisielk/errcheck but unlike the original
allows you to write `defer foo.Close()`.

## Why a fork

`defer h.Close()` 
is idiomatic Go, which appears regularly in the standard library and text books.
If h is a read only descriptor, then nothing useful can be done if the Close fails.

This has been raised a number [of](https://github.com/kisielk/errcheck/issues/55) [times](https://github.com/kisielk/errcheck/issues/101) 
upstream - see https://github.com/kisielk/errcheck/issues/55
and https://github.com/kisielk/errcheck/issues/101, but these have been closed, as the author 
believes that these errors should always be checked, and if necessary deferred calls to Close
should be wrapped in a lambda. 
This [unecessarily complicates code](https://github.com/Marethyu12/gotube/pull/4/commits/8552c52ca02b81fd0a307784916bfe6393fcaa1e), 
and means that the Go standard
library will fail errcheck.

The Author is, of course, entitled to his opinion. But those who disagree with him have the right to fork and do otherwise.


## Install

    go get -u github.com/amnonbc/errcheck

errcheck requires Go 1.14 or newer and depends on the package go/packages from the golang.org/x/tools repository.

## Use

For basic usage, just give the package path of interest as the first argument:

    errcheck github.com/kisielk/errcheck/testdata

To check all packages beneath the current directory:

    errcheck ./...

Or check all packages in your $GOPATH and $GOROOT:

    errcheck all

errcheck also recognizes the following command-line options:

The `-tags` flag takes a space-separated list of build tags, just like `go
build`. If you are using any custom build tags in your code base, you may need
to specify the relevant tags here.

The `-asserts` flag enables checking for ignored type assertion results. It
takes no arguments.

The `-blank` flag enables checking for assignments of errors to the
blank identifier. It takes no arguments.


## Excluding functions

Use the `-exclude` flag to specify a path to a file containing a list of functions to
be excluded.

    errcheck -exclude errcheck_excludes.txt path/to/package

The file should contain one function signature per line. The format for function signatures is
`package.FunctionName` while for methods it's `(package.Receiver).MethodName` for value receivers
and `(*package.Receiver).MethodName` for pointer receivers. If the function name is followed by string of form `(TYPE)`, then
the the function call is excluded only if the type of the first argument is `TYPE`. It also accepts a special suffix
`(os.Stdout)` and `(os.Stderr)`, which excludes the function only when the first argument is a literal `os.Stdout` or `os.Stderr`.

An example of an exclude file is:

    io/ioutil.ReadFile
    io.Copy(*bytes.Buffer)
    io.Copy(os.Stdout)
    (*net/http.Client).Do

The exclude list is combined with an internal list for functions in the Go standard library that
have an error return type but are documented to never return an error.

When using vendored dependencies, specify the full import path. For example:
* Your project's import path is `example.com/yourpkg`
* You've vendored `example.net/fmt2` as `vendor/example.net/fmt2`
* You want to exclude `fmt2.Println` from error checking

In this case, add this line to your exclude file:
```
example.com/yourpkg/vendor/example.net/fmt2.Println
```


### The deprecated method

The `-ignore` flag takes a comma-separated list of pairs of the form package:regex.
For each package, the regex describes which functions to ignore within that package.
The package may be omitted to have the regex apply to all packages.

For example, you may wish to ignore common operations like Read and Write:

    errcheck -ignore '[rR]ead|[wW]rite' path/to/package

or you may wish to ignore common functions like the `print` variants in `fmt`:

    errcheck -ignore 'fmt:[FS]?[Pp]rint*' path/to/package

The `-ignorepkg` flag takes a comma-separated list of package import paths
to ignore:

    errcheck -ignorepkg 'fmt,encoding/binary' path/to/package

Note that this is equivalent to:

    errcheck -ignore 'fmt:.*,encoding/binary:.*' path/to/package

If a regex is provided for a package `pkg` via `-ignore`, and `pkg` also appears
in the list of packages passed to `-ignorepkg`, the latter takes precedence;
that is, all functions within `pkg` will be ignored.

Note that by default the `fmt` package is ignored entirely, unless a regex is
specified for it. To disable this, specify a regex that matches nothing:

    errcheck -ignore 'fmt:a^' path/to/package

The `-ignoretests` flag disables checking of `_test.go` files. It takes
no arguments.

## Cgo

Currently errcheck is unable to check packages that import "C" due to limitations in the importer when used with versions earlier than Go 1.11.

However, you can use errcheck on packages that depend on those which use cgo. In order for this to work you need to go install the cgo dependencies before running errcheck on the dependent packages.

See https://github.com/kisielk/errcheck/issues/16 for more details.

## Exit Codes

errcheck returns 1 if any problems were found in the checked files.
It returns 2 if there were any other failures.

# Editor Integration

## Emacs

[go-errcheck.el](https://github.com/dominikh/go-errcheck.el)
integrates errcheck with Emacs by providing a `go-errcheck` command
and customizable variables to automatically pass flags to errcheck.

## Vim

[vim-go](https://github.com/fatih/vim-go) can run errcheck via both its `:GoErrCheck`
and `:GoMetaLinter` commands.
