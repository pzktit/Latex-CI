# LaTeX CI

This is template repository that demonstrates *LaTeX* documents processing using *Docker* containers. 
Thre flavours are demonstrated: command line, [*VSCode* Devcontainers](https://code.visualstudio.com/docs/devcontainers/containers) and/or [*GitHub*'s](https://github.com/) continuous integration and deployment using [*GitHub* actions](https://docs.github.com/en/actions).

The motivation for this work comes from [merkez/latex-on-ci-cd](https://github.com/merkez/latex-on-ci-cd) and [xu-cheng/latex-docker](https://github.com/xu-cheng/latex-docker).

## How this work?

Documents are processed with the up-date *TeXLive* distribution enclosed in a Docker container [pzktit/TeXLiveDev](https://github.com/pzktit/TeXLiveDev).
This container can be used in three ways:
1. as a standalone tool for document preparation in a controlled and easily replicable environment,
2. as a development container for *VSCode*,
3. as a *GitHub* action engine that builds document in response to events that happen in the repository, for instance, on updating a specified branch.   

Please note that the last option is more suitable for provision of the document evergreen version than for its development, as the times between the change of the document source and the resulting output can be significant. On the other hand, that method frees the authors/developers from repetitive deployment of the subsequent copies of the document.
In other words, the third option should be used in conjunction with other approaches.

The [TeXLiveDev container](https://github.com/pzktit/TeXLiveDev) contains the most recent distribution of *TeXLive*, and it is updated once a month.
There are many *TexLive* containers, however their use as devcontainers is cumbersome.
First, the *DevContainer* extension requires a *VSCode-server* inside container.
As a result, the server is installed inside container when it is instantiated and that takes some time.
Second, the comfortable authoring requires the spell and grammar check engine.
The very good extension for LaTeX documents is based on a service that should be installed inside container.
Again, that tool is usually installed at the moment of container instantiation.
As the container preparation takes significant time, they are not removed after use, and they are cached by Docker - one copy per document opened in devcontainers.
These copies have to be removed manually by the user.

The [pzktit/TeXLiveDev](https://github.com/pzktit/TeXLiveDev) container resolves that difficulty.
It is indirectly derived from [Microsoft's Ubuntu devcontainer](mcr.microsoft.com/devcontainers/base:ubuntu), so all tools required for communication with *VSCode* client are in place.
As a result, a [VSCoder-server](https://code.visualstudio.com/docs/remote/vscode-server) is **not** installed at the container bootstrap stage (which is the case for most containerized *TeXLive* images used as Devcontainers).
Also, two helpful *VSCode* extensions, namely [james-yu.latex-workshop](https://github.com/James-Yu/LaTeX-Workshop) and [valentjn.vscode-ltex](https://github.com/valentjn/vscode-ltex) are installed in the container.
The *vscode-ltex* extension internally uses [LTeX](https://valentjn.github.io/ltex/) spell and grammar checker engine.
Usually *LTeX* is downloaded (ca. 200 MB) by the extension ([valentjn.vscode-ltex](https://github.com/valentjn/vscode-ltex)) when it is installed in *VSCode*.
However, it makes the devcontainer instantiation process quite long and requires network access.
Two avoid this complication, the *LTeX* system is installed inside the container, and the extension is informed where the engine can be found (see entry ``"ltex.ltex-ls.path": "/usr"`` in file ``.devcontainer/devcontainer.json``).
Such setup speedups the first bootstrap of the devcontainer instance so greatly, that it can be removed after its use (see the entry ``"runArgs": ["--rm", "--name=texlivedev-sample"]`` in file ``.devcontainer/devcontainer.json``) without a significant loss in performance.
However, in that case all your *VSCode*'s workspace customizations will be lost.
If you are going to open the document many times, then you can safely remove the ``--rm`` option. The preinstallation of extensions and *LTeX* makes that disk space footprint of container instances is not high.
It may be checked with
```
docker system df
```

Please note that *VSCode* extensions are installed with the help of the [VSCode-server](https://code.visualstudio.com/docs/remote/vscode-server), which is closed source tool and some legal limitations may apply.

## How to use it?

### Prerequisites

You need *VSCode* and *git* to use continuous deployment.
Start with cloning this repository to your system
```
git clone https://github.com/pzktit/LatexCI
cd LatexCI
```
Additionally, for local building of documents, you need installation of *Docker* in your system.
Please pull the image of the [container used for development](https://github.com/pzktit/TeXLiveDev)
```sh
docker pull ghcr.io/pzktit/texlivedev
```
Continuous integration and deployment requires a GitHub repository (and an account if you don't have one).
In other methods, its creation is optional, although advised.
Let us further assume that your GitHub account is ``fikumiku`` and the newly created repository is named ``akuku``.
In the cloned folder of the template issue
```sh
git remote -v
```
It should display ``https://github.com/pzktit/LatexCI`` for both ``push`` and ``fetch`` actions.
Then issue **one** of the following commands
```sh
git remote set-url origin git@github.com:/fikumiku/akuku.git
git remote set-url origin https://github.com/fikumiku/akuku
```
Please note that the second option implies using Personal Access Tokens which may be a little cumbersome, but it is the most flexible way of access management.
The first option uses SSH, and you can install trusted public keys on your GitHub account for passwordless authentication.

### Command line

The following command can be used to build the document
```
docker run --rm --name=latexbuild -ti -v ${PWD}:/home/vscode ghcr.io/pzktit/texlivedev latexmk -lualatex main.tex
```
The above translates to the following ``docker-compose.yml``
```yml
version: '3'
services:
  latexbuild:
    image: ghcr.io/pzktit/texlivedev
    container_name: latexbuild
    volumes:
      - '.:/home/vscode'
    stdin_open: true
    tty: true
    command: 'latexmk -lualatex main.tex'
```
The build process is now much simpler
```sh
docker compose run --rm latexbuild
```
Different methods of document building can be specified as other services and correspondingly referred in the build command.

The commands inside the container are issued on behalf of ``vscode`` user
```sh
docker compose run --rm latexbuild id
```
and the files left in your folder are owned by that user. That may cause some problems.
Thus, if you are working on an account with a different ``UID`` and ``GID``, try the following command and config file
```sh
UGID="$(id -u):$(id -g)" docker compose run --rm latexbuild
```
```yml
version: '3'
services:
  latexbuild:
    image: ghcr.io/pzktit/texlivedev
    container_name: latexbuild
    user: ${UGID}
    volumes:
      - '.:/home/vscode'
    stdin_open: true
    tty: true
    command: 'latexmk -lualatex main.tex'
```

### Code and Devcontainer


Start *VSCode* in your local folder 
```
code . 
```
The following description will work only if you have ``Dev Containers`` extension installed.
Open folder in a container (agree on the *VSCode*'s proposal or click green icon in left down corner and select ``Reopen in container``) and then open a LaTeX source file. You will see on ``Primary Side Bar`` (usually the most left) an icon with "``TeX``" letters. This means that ``Latex Workshop`` extension is working, and its goodies are for your disposal.
Please see [its manual](https://github.com/James-Yu/LaTeX-Workshop) for information what power is at your fingertips.
By default, (see ``.devcontainer/devcontainer.json``)
the ``LuaLaTeX`` is used to build documents.
The extension also offers others predefined build schemes, although it is advisable to configure it in ``devcontainer.json``. That way, you and your coauthors will use the same environment for compilation.
Definition of not used build schemes may be removed in more restrictive authoring environment configurations. 


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

The ``workflow_dispatch`` permits launching the build task from [GitHub's GUI of the repository](https://github.com/fikumiku/akuku/actions/workflows/compileandrelease.yml) (please change link accordingly to your repository name).
The option ``push`` triggers event of pushing some data. The commented out option triggers build when whatever data is pushed to the ``main`` branch.
The ``tags`` option requires pushing a tag that matches to a predefined mask, e.g.
```
git tag v0.0.4 ; git push --tags 
```

The resulting document is published on repository's GitHub page, so you have to enable this feature (on page ``https://github.com/fikumiku/akuku/`` select ``Seetings``, then ``Pages`` on the left and enable whatever method you want.)
The built document should be available at link ``https://fikumiku.github.io/akuku/main.pdf``.


## Tweaking

See ``Dockerfiles`` of [TeXLiveDev](https://github.com/pzktit/TeXLiveDev) and [TeXLiveCI](https://github.com/pzktit/TeXLiveCI) images.
