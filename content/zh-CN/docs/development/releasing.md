# 发布

本文档描述了发布 Electron 版本的过程。

## 决定从哪个版本发布

- **如果发布beta版本，**请从`master`运行以下脚本。
- **如果发布稳定版本，**请从你要试图稳定的分支运行下列脚本。

## 找出需要哪个版本更改

运行`npm run prepare-release -- --notesOnly`来查看自动生成的发布说明。 The notes generated should help you determine if this is a major, minor, patch, or beta version change. 请参考[版本变更规则](../tutorial/electron-versioning.md#semver)以获取更多信息。

**注意：** 如果从一个分支发布，例如1-8-x，使用 `git checkout 1-8-x` 检出分支而不是 `git checkout -b remotes/origin/1-8-x`。 The scripts need `git rev-parse --abbrev-ref HEAD` to return a short name, e.g. no `remotes/origin/`

## Set your tokens and environment variables

You'll need Electron S3 credentials in order to create and upload an Electron release. Contact a team member for more information.

There are a handful of `*_TOKEN` environment variables needed by the release scripts. Once you've generated these per-user tokens, you may want to keep them in a local file that you can `source` when starting a release. * `ELECTRON_GITHUB_TOKEN`: Create as described at https://github.com/settings/tokens/new, giving the token repo access scope. * `APPVEYOR_TOKEN`: Create a token from https://windows-ci.electronjs.org/api-token If you don't have an account, ask a team member to add you. * `CIRCLE_TOKEN`: Create a token from "Personal API Tokens" at https://circleci.com/account/api

## 运行 prepare-release 脚本

The prepare release script will do the following: 1. Check if a release is already in process and if so it will halt. 2. 创建一个发布分支。 3. Bump the version number in several files. See [this bump commit](https://github.com/electron/electron/commit/78ec1b8f89b3886b856377a1756a51617bc33f5a) for an example. 4. Create a draft release on GitHub with auto-generated release notes. 5. 推送release分支 6. 调用API以运行release构建

Once you have determined which type of version change is needed, run the `prepare-release` script with arguments according to your need: - `[major|minor|patch|beta]` to increment one of the version numbers, or - `--stable` to indicate this is a stable version

例如：

### Major 版本更改

```sh
npm run prepare-release -- major
```

### Minor 版本更改

```sh
npm run prepare-release -- minor
```

### Patch 版本更改

```sh
npm run prepare-release -- patch
```

### Beta 版本更改

```sh
npm run prepare-release -- beta
```

### 促进 beta 稳定

```sh
npm run prepare-release -- --stable
```

Tip: You can test the new version number before running `prepare-release` with a dry run of the `bump-version` script with the same major/minor/patch/beta arguments, e.g.:

```sh
$ ./script/bump-version.py --bump minor --dry-run
```

## 等待构建 :hourglass_flowing_sand:

`prepare-release` 脚本将通过 API 调用触发生成。要监视生成进度, 请参阅以下页面:

- [circleci.com/gh/electron/electron](https://circleci.com/gh/electron) 对于 OS X 和 Linux
- [windows-ci.electronjs.org/project/AppVeyor/electron](https://windows-ci.electronjs.org/project/AppVeyor/electron) 对于 Windows

## 编译发布说明

编写发行说明是在生成运行时保持忙碌的好方法。 有关以前的技术, 请参阅 [ 发布页 ](https://github.com/electron/electron/releases) 上的现有版本。

Tips: - Each listed item should reference a PR on electron/electron, not an issue, nor a PR from another repo like libcc. - No need to use link markup when referencing PRs. Strings like `#123` will automatically be converted to links on github.com. - To see the version of Chromium, V8, and Node in every version of Electron, visit [atom.io/download/electron/index.json](https://atom.io/download/electron/index.json).

### Patch 发布

对于 ` 修补程序 ` 版本, 请使用以下格式:

```sh
## Bug 修复

* 修复跨平台问题. #123

### Linux

* 修复 Linux 问题. #123

### macOS

* 修复 macOS 问题. #123

### Windows

* 修复 Windows 问题. #1234
```

### Minor 发布

对于 ` 次要 ` 版本, 如 ` 1.8.0 `, 请使用以下格式:

```sh
## 升级

- 升级 Node 从 `oldVersion` 到 `newVersion`. #123

## API 变更

* 变更内容. #123

### Linux

* 变更 Linux 内容. #123

### macOS

* 变更 macOS 内容. #123

### Windows

* 变更 Windows 内容. #123
```

### Major 发布

```sh
## 升级

- 升级 Chromium 从 `oldVersion` 到 `newVersion`. #123
- 升级 Node 从 `oldVersion` 到 `newVersion`. #123

## 破坏性 API 变更

* 变更内容. #123

### Linux

* 变更 Linux 内容. #123

### macOS

* 变更 macOS 内容. #123

### Windows

* 变更 Windows 内容. #123

## 其它变更

- 其它变更内容. #123
```

### Beta 发布

使用与上述建议相同的格式，但在变更日志的开头添加以下内容：

```sh
**注意：** 这是一个测试版本并很可能会有一些不稳定和/或复原。

请为您在其中找到的任何错误提出新问题。

This release is published to [npm](https://www.npmjs.com/package/electron)
under the `beta` tag and can be installed via `npm install electron@beta`.
```

## 编辑发布草稿

1. 访问 [发行页面](https://github.com/electron/electron/releases) 然后你将看到一个新的带有发行说明的草稿版本。
2. 编辑版本并添加发行说明.
3. Uncheck the `prerelease` checkbox if you're publishing a stable release; leave it checked for beta releases.
4. 点击 'Save draft'. **不要点 'Publish release'!**
5. 等待所有生成通过, 然后再继续。
6. In the `release` branch, verify that the release's files have been created:

```sh
$ git rev-parse --abbrev-ref HEAD
release
$ npm run release -- --validateRelease
```

## 合并临时分支（仅pre-2-0-x分支）

Once the release builds have finished, merge the `release` branch back into the source release branch using the `merge-release` script. If the branch cannot be successfully merged back this script will automatically rebase the `release` branch and push the changes which will trigger the release builds again, which means you will need to wait for the release builds to run again before proceeding.

### 合并回 master

```sh
npm run merge-release -- master
```

### 合并回旧版本分支

```sh
npm run merge-release -- 1-7-x
```

## 发布版本

Once the merge has finished successfully, run the `release` script via `npm run release` to finish the release process. 这个脚本会完成下列内容：1. Build the project to validate that the correct version number is being released. 2. Download the binaries and generate the node headers and the .lib linker used on Windows by node-gyp to build native modules. 3. Create and upload the SHASUMS files stored on S3 for the node files. 4. 创建并上传储存在GitHub发布中的SHASUMS256.txt文件。 5. Validate that all of the required files are present on GitHub and S3 and have the correct checksums as specified in the SHASUMS files. 6. 在Github上发布release 7. 删除 `release` 分支。

## 发布到 npm

在发布到npm之前，您会需要作为Electron登入npm。 Optionally, you may find [npmrc](https://www.npmjs.com/package/npmrc) to be a useful way to keep Electron's profile side-by-side with your own:

```sh
$ sudo npm install -g npmrc
$ npmrc -c electron
Removing old .npmrc (default)
Activating .npmrc "electron"
```

The Electron account's credentials are kept by GitHub. "Electron - NPM" for the URL "https://www.npmjs.com/login".

```sh
$ npm login
用户名: electron
密码:
电子邮件: (这是公共的) electron@github.com
```

发布版本到npm。

```sh
$ npm whoami
electron
$ npm run publish-to-npm
```

Note: In general you should be using the latest Node during this process; however, older versions of the `publish-to-npm` script may have trouble with Node 7 or higher. If you have trouble with this in an older branch, try running with an older version of Node, e.g. a 6.x LTS.

## 手动修复发行版的缺失二进制文件

在发布版本受损的情况下，则可能需要重新上传已发布版本的二进制文件。

第一步是转到[Releases](https://github.com/electron/electron/releases)页面，并使用 `SHASUMS256.txt`校验文件和删除损坏的二进制文件。

然后手动为每个平台创建分发并上传它们：

```sh
# 检出要重新上传的版本。
git checkout vTHE.RELEASE.VERSION

# 发布构建，指定一个目标体系结构。
./script/bootstrap.py --target_arch [arm|x64|ia32]
./script/build.py -c R
./script/create-dist.py

# 明确允许覆盖已发布的版本.
./script/upload.py --overwrite
```

重新上传所有发行版之后，再次发布以上载校验和文件：

```sh
npm run release
```