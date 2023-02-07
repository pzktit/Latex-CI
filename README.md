# LaTeX CI

This template repository for *LaTeX* documents processing using *Docker* containers and command line, [*VSCode* Devcontainers](https://code.visualstudio.com/docs/devcontainers/containers) and/or [*GitHub*'s](https://github.com/) continuous integration and deployment using [*GitHub* actions](https://docs.github.com/en/actions).

The motivation for this work comes from [merkez/latex-on-ci-cd](https://github.com/merkez/latex-on-ci-cd) and [xu-cheng/latex-docker](https://github.com/xu-cheng/latex-docker).

## How this work?

Documents are processed with the up-date *TeXLive* distribution enclosed in a Docker container [pzktit/TeXLiveDev](https://github.com/pzktit/TeXLiveDev).
This container can be used in three ways:

1. as a standalone tool for document preparation in controlled and easily replicable environment,
2. as a development container for *VSCode*.
3. as a *GitHub* action engine that builds document in response to events that happen in the repository, for instance, on updating a specified branch.   

The [TeXLiveDev container](https://github.com/pzktit/TeXLiveDev) contains the most recent distribution of *TeXLive*, and it is updated once a month.
It is indirectly derived from [Microsoft's Ubuntu devcontainer](mcr.microsoft.com/devcontainers/base:ubuntu), so all tools required for communication with *VSCode* client are in place.
As a result, a [VSCoder-server](https://code.visualstudio.com/docs/remote/vscode-server) is **not** installed at the container bootstrap stage (which is the case for most other containerized *TeXLive* images).
Also, two helpful *VSCode* extensions, namely [james-yu.latex-workshop](https://github.com/James-Yu/LaTeX-Workshop) and [valentjn.vscode-ltex](https://github.com/valentjn/vscode-ltex) are installed in the container.
The *vscode-ltex* uses [LTeX](https://valentjn.github.io/ltex/) spell and grammar checker.
Usually *LTeX* is downloaded (ca. 200 MB) by the extension at its first use.
However, it makes the instantiation process of the Devcontainer for the new document quite long and requires network access.
Two avoid this complication, the *LTeX* system is installed inside the container, and the extension is informed where it can be found (see entry ``"ltex.ltex-ls.path": "/usr"`` in file ``.devcontainer/devcontainer.json``).
Such setup speedups the first bootstrap of the devcontainer instance so greatly, that the devcontainer can be deleted after its use (see the entry ``"runArgs": ["--rm", "--name=texlivedev-sample"]`` in file ``.devcontainer/devcontainer.json``) without a significant loss in performance.

Please note that *VSCode* extensions are installed in the help of the [VSCode-server](https://code.visualstudio.com/docs/remote/vscode-server), which is closed source tool and some legal limitations may apply.

## How to use it?



## Tweaking



