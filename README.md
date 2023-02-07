# LaTeX CI

This template repository for *LaTeX* documents processing using *Docker* containers and command line, [*VSCode* Devcontainers](https://code.visualstudio.com/docs/devcontainers/containers) and/or [*GitHub*'s](https://github.com/) continuous integration and deployment using [*GitHub* actions](https://docs.github.com/en/actions).

The motivation for this work comes from [merkez/latex-on-ci-cd](https://github.com/merkez/latex-on-ci-cd) and [xu-cheng/latex-docker](https://github.com/xu-cheng/latex-docker).

## How this work?

Documents are processed with the up-date *TeXLive* distribution enclosed in a Docker container [pzktit/TeXLiveDev](https://github.com/pzktit/TeXLiveDev).
This container can be used in three ways:
1. as a standalone tool for document preparation in controlled and easily replicable environment,
2. as a development container for *VSCode*,
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

Please note that *VSCode* extensions are installed with the help of the [VSCode-server](https://code.visualstudio.com/docs/remote/vscode-server), which is closed source tool and some legal limitations may apply.

## How to use it?

### Prerequisites

You need *VSCode* and *git* to use continuous deployment.
Start with cloning this repository to your system
```
git clone https://github.com/pzktit/LatexCI
cd LatexCI
```
Additionally, for local building of documents, you need installation of *Docker* on your local system.
Please pull the image of the [container used for development](https://github.com/pzktit/TeXLiveDev)
```sh
docker pull ghcr.io/pzktit/texlivedev
```
For continuous integration and deployment you have to create a GitHub repository (and an account if you don't have one).
For other methods its creation is optional, although advised.
Let us further assume that your GitHub account is ``fikumiku`` and the newly created repository is named ``akuku``.
In the cloned folder of the template issue
```sh
git remote -v
```
It should display ``https://github.com/pzktit/LatexCI`` for both ``push`` and ``fetch`` actions.
Then issue *one* of the following commands
```sh
git remote set-url origin git@github.com:/fikumiku/akuku.git
git remote set-url origin https://github.com/fikumiku/akuku.git
```
Please note that the second option implies using Personal Access Tokens which may be a little cumbersome, but it is the most flexible way of access management.
The first option uses SSH, and you can install trusted public keys on your GitHub account for passwordless authentication.

### Command line

### Code and Devcontainer

Start *VSCode* in your local folder 
```
code . 
```


### Continuous integration and deployment

Start *VSCode* in your local folder 
and synchronize with your remote repository using a built-in *git* interface
(if you know nothing about *git*, [please read this documentation](https://docs.github.com/en/get-started/getting-started-with-git)).
Please ignore the offer of opening the folder in Remote Container. In fact, if you are not going to use Devcontainers, you can safely remove the folder ``.devcontainer``.

The document building and deploying process is controlled by the file ``.github/workflows/compileandrelease.yml``.
The section ``on:`` determines conditions in which the build process will be triggered.

```yml
on:
  push:
    tags:
      - 'v*.*.*'
#     branches:
#       - main
  workflow_dispatch:
```

The ``workflow_dispatch`` permits launching task from [GitHub's GUI of the repository](https://github.com/fikumiku/akuku/actions/workflows/compileandrelease.yml) (please change link accordingly to your repository name).
The option ``push`` triggers event of pushing some data. The commented out option triggers build when whatever data is pushed to the ``main`` branch.
The ``tags`` option requires pushing a tag that matches to a predefined mask, e.g.
```
git tag v0.0.4 ; git push --tags 
```

The resulting document is published on repository's GitHub page, so you have to enable this feature (on page ``https://github.com/fikumiku/akuku/`` select ``Seetings``, then ``Pages`` on the left and enable whatever method you want.)
The built document should be available at link ``https://fikumiku.github.io/akuku/main.pdf``.


## Tweaking



