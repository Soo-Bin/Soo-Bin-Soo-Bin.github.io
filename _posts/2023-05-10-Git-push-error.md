---
title: GitLab pre-receive hook declined 에러
categories: [Git]
tags: [Git]
comments: true
---

`feature` 브랜치에서 작업을 끝낸 뒤, `master` 브랜치로 merge 하고 push를 하니까 `'pre-receive hook declined'` 에러가 발생했다.

```bash
$ git push origin master
remote: GitLab: You are not allowed to push code to protected branches on this project.
error: failed to push some refs to 'http://gitlab.xxxxxxxxx'
To http://gitlab.xxxxxxxxx
!	refs/heads/master:refs/heads/master	[remote rejected] (pre-receive hook declined)
```

해석해보면 push 하려는 코드가 Protected Branch의 정책에 맞지 않아서 거부되었다는 의미다.

GitLab에서는 사용자들이 저장소를 관리할 때 보안을 강화하기 위해 Protected Branch라는 기능을 제공한다. 프로젝트 생성 시 디폴트로 `master` 브랜치는 Protected Branch로 설정된다. 이 기능은 Maintainer에게만 허가되며, 일반적으로 Maintainer는 코드를 검토하고 브랜치에 대한 변경 사항을 결정하는 역할을 담당한다.

하지만, Protected Branches의 정책 때문에 Git Repository를 생성할 때 기본적으로 Maintainer에게만 push 권한이 부여되고 Developer에게는 권한이 부여되지 않는다. 따라서 Developer는 해당 브랜치에 push 하려고 하면 `'pre-receive hook declined'` 에러가 발생한다.

## 해결 방법

이 문제를 해결하려면 Maintainer가 Developer에게 push 권한을 부여해야 한다. 이를 위해

1. Maintainer는 GitLab의 설정에서 **Protected Branches의 권한을 수정**하거나,
2. Developer에게 **Maintainer로부터 권한을 직접 받도록 요청**할 수 있다.

본인은 Maintainer에게 부탁하여 해당 프로젝트 안에서 권한을 Developer에서 Maintainer로 변경하였다.
