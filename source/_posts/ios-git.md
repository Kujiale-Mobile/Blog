title: iOS模块化：子模块版本管理实践
date: 2019-06-24
author: 波波
subtitle: 随着项目业务模块越来越多，代码越来越复杂，很多公司都会对项目进行拆解，将各个业务模块、基础模块、中间层、UI组件、功能插件等拆分到独立的 repo 中，本篇教你如何做子模块的管理
tags: [iOS, 模块化，git]

categories: iOS
---

> 背景：随着项目业务模块越来越多，代码越来越复杂，很多公司都会对项目进行拆解，将各个业务模块、基础模块、中间层、UI组件、功能插件等拆分到独立的 repo 中。根据拆分粒度以及项目复杂度的不同，会多出来少则十几个，多则上百个 repo 出来。    
无论多出来多少个子 repo ，如何管理这些子 repo 的版本，就会是一个避不开的问题。
# 子模块版本管理
拆分出来的子 repo ，我们统称它为子模块，集成这些子项目的 repo，我们称为主项目。

在 iOS 项目中，子模块的版本管理，每个公司可能都有自己的做法，以下介绍的是我司在 Coohom 项目组的子模块版本管理实践。

## 版本号
参考 [Semantic Versioning 2.0.0](https://semver.org/)，对应 git flow 中的各个开发阶段，我们为子模块制定了版本号生成规则。

+ develop branch 的子模块版本号：`a.b.c.alpha-d-hash`
+ release branch 的子模块版本号：`a.b.c.beta-d-hash`
+ master branch 的子模块版本号：`a.b.c`

> a, b, c 分别对应 major, minor, patch，-d-hash 为 git describe 命令所获取的字符串

这样子的版本号设计，除了可以在主项目中清晰的跟踪子模块的版本升级，还能知道当前依赖的子模块版本是处于什么阶段。

为了方便将子模块的版本更新部署自动化，我们还为子模块版本号更新制定了版本号自增规则（这种细节本文不介绍，逻辑可以直接看文末的脚本demo）。

## Dev 模式
我们使用 `cocoapods` 来集成子模块，如果子模块只能通过版本号来集成的话，当需要修改子模块的时候，主项目就需要更新子模块来验证修改后的代码，这样会带来频繁的子模块版本更新，并在升级版本中浪费大量时间。于是我们为子模块设计了 Dev 模式。

在 Dev模式下，子模块代码会通过 `Development Pods` 的方式集成进项目，这种方式集成子模块，可以直接修改子模块代码，不需要走子模块版本更新的流程，就可以在主项目中验证修改后的代码。

为了在 Dev 模式和版本集成中方便切换，以及为了给后续 CI 开发服务，我们外层封装了一下 `cocoapods` 的 `pod(name = nil, *requirements)` 方法，创建了我们自己的  `submodule(name, version, mode, params)` 方法（具体的方法实现可以看文末的demo）。

以下是 submodule 方法在 Podfile 中的使用方式：
```ruby
    require_relative './PodScripts/submodule.rb'

  # Dev, Alpha, Beta, Release
    submodule 'MobileiOSBase', '~> 0.6.0', CooHom::Mode::Release, :subspecs => [
      'Base',
      'ServiceBase',
      'ViewBase',
      'CHAnalytics',
      'AccountService'
    ]
```
`CooHom::Mode` 有4个值：`Dev, Alpha, Beta, Release`，分别对应子模块的 Dev 模式，alpha 版本，beta 版本，release 版本。

## 子模块在主项目 git flow 的各个阶段中的状态
![子模块在主项目 git flow 的各个阶段中的状态](https://user-images.githubusercontent.com/4279515/60317132-44d2e600-99a0-11e9-96f8-a6ee5ac38b72.png)

### 原则
1. Dev 模式集成的子模块不允许合入 develop，只允许在本地 feature 开发中使用。
2. release 不允许存在 alpha 版本的子模块，所以在开出 release 分支之前，都必须要确定所有 alpha 版本的子模块进入 beta 版本。
3. 上传 app store 的 ipa，所有依赖的子模块都必须是 release 版本。

## 相关脚本设计
> 人为去保证上述开发流程的运行，浪费人力不说，还很容易出错，所以相关的CI脚本设计，git hook 设计是必不可少的。

### git hooks
1. 主项目 feature branch push 的时候，需要检查是否有处于Dev模式的子模块，如果有，push失败。
2. 主项目创建并 push release branch 的时候，需要检查是否有 alpha 版本的子模块，如果有，branch 创建失败。

### CI / CD

#### 主项目CI / CD
1. feature branch 的 merge request 触发，需要检查是否有处于 Dev 模式的子模块，如果有，无法 merge。
2. 在 develop 生成 dev 的 ipa 时触发，先检查是否有处于 Dev 模式的子模块，如果有，无法生成 ipa。
3. 在 release 生成 beta 的 ipa 时触发，先检查是否有处于 Dev 模式或者 alpha 版本的子模块，如果有，无法生成 ipa。
4. 上传 testflight 的时候触发，先检查是否有非 release 版本的子模块，如果有，无法生成 ipa。

#### 子模块CI / CD
1. feature branch 的 merge request 触发，pod lib lint ，如果失败，不能 merge ，需要确保 develop 上的代码能够正常发布版本。
2. develop 每一次 commit 触发，为这次 commit 打上 alpha 版本的 tag。
3. release 每一次 commit 触发，为这次 commit 打上 beta 版本的 tag。
4. release 每一次 commit 触发，为这次 commit 打上 release 版本的 tag。
5. 打 tag 触发，根据 tag ，修改 podspec 文件版本并发布 podspec。

***[脚本Demo](https://github.com/huangqiaobo/scripts-demo-for-submodules)***