Maton takes the security of the Maton CLI seriously.

If you believe you have found a security vulnerability in Maton CLI, please report it to us privately through one of the following channels:

* Use this repository's [private vulnerability reporting][] feature.
  * Include a description of your investigation of Maton CLI's codebase and why you believe an exploit is possible.
  * Proofs-of-concept and links to specific code are greatly encouraged.

* Email **support@maton.ai** with the same information.

**Please do not report security vulnerabilities through public GitHub issues, discussions, or pull requests.**

A dependency having a CVE does not mean `maton` has a vulnerability. We use [`govulncheck`](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck) to determine whether vulnerable symbols are actually reachable from `maton`'s code. If you are reporting a dependency CVE, please include evidence that the issue is exploitable in `maton`: a call chain into the affected symbols or a proof of concept. Reports that only list a dependency version and CVE without demonstrating impact will be closed.

Thanks for helping keep Maton CLI and its users safe.

  [private vulnerability reporting]: https://github.com/maton-ai/cli/security/advisories
