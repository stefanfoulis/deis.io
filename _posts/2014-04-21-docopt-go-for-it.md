---
layout: post
title: doctopt-go for it!
author: mboersma
meta:
  - name: description
    content: As of today the Deis team is helping maintain the docopt-go project on GitHub.
  - name: keywords
    content: deis, docopt, golang, docopt-go
---

```go
go import "github.com/docopt/docopt-go"
```

> Don't write parser code: a good help message already has all the necessary information in it.

That is the philosophy of [docopt][1] in a nutshell.

Back during the Stone Age of [Deis][2]—almost eight lunar months ago—I was [enthused][3] about docopt [for Python][4]. Think about what commands and options your tool needs, type them out into docstrings, and you're basically done with the user input and validation side of things.

We Deis maintainers are enthusiastic [Gophers][5] as well as Pythoneers. So I'm doubly glad we chose docopt, because the [docopt-go][6] package for Go makes porting the `deis` CLI from Python to Go much easier.

As of today the Deis team is helping maintain the [docopt-go][6] project on GitHub. Recent improvements include:

*   `go get github.com/docopt/docopt-go` now works
*   no `os.Exit()` if optional parameter `exit` is `false`
*   tests run at [Travis CI][7]
*   docs hosted at [GoDoc.org][8]

I'll be among the Deis project attendees at [GopherCon][5] this week, so please ask me about [docopt-go][6]!

 [1]: http://docopt.org/
 [2]: http://deis.io/
 [3]: http://deis.io/command-line-bliss-with-docopt/
 [4]: https://github.com/docopt/docopt
 [5]: http://www.gophercon.com/about/
 [6]: https://github.com/docopt/docopt.go
 [7]: https://travis-ci.org/docopt/docopt.go
 [8]: http://godoc.org/github.com/docopt/docopt.go