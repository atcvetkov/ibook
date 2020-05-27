<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [.gitconfig](#gitconfig)
- [refs/meta/config](#refsmetaconfig)
- [reference](#reference)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

### .gitconfig
```bash
$ git config --global gitreview.username <UserName>
$ git config --global gitreview.remote origin
```
### [refs/meta/config](https://gerrit-review.googlesource.com/Documentation/config-project-config.html#_the_refs_meta_config_namespace)

#### get project.config
- clone the repo
  ```bash
  $ git clone <repo url>
  # or update the local repo to HEAD
  $ git pull [--rebase]
  ```
 
- checkout `meta/config`
  ```bash
  $ git fetch origin refs/meta/config:refs/remotes/origin/meta/config
  $ git checkout meta/config
  ```
  or
 
  ```bash
  $ git fetch ssh://localhost:29418/project refs/meta/config
  $ git checkout FETCH_HEAD
  ```

#### publish to remote
```bash
$ git add --all .
$ git commit -m "<add your comments here>"
```
- submit directly
  ```bash
  $ git push origin meta/config:meta/config
  ```
  or

  ```bash
  $ git push origin HEAD:refs/meta/config
  ```
- submit review

  ```bash
  $ git push origin HEAD:refs/for/refs/meta/config
  ```
  or

  ```bash
  $ git push origin meta/config:refs/for/refs/meta/config
  ```

#### update meta/config if remotes update
```bash
$ git fetch origin --force refs/meta/config:refs/remotes/origin/meta/config

$ git pull origin refs/meta/config
# or
$ git merge meta/config
```

#### useful refs
- sandbox: `refs/heads/sandbox/${username}/*`
- its-jira: `refs/heads/jira/jira-[0-9]{1,5}(_.*)?`

  ```bash
  [plugin "its-jira"]
    association = OPTIONAL
    branch = ^refs/heads/.*
    branch = ^refs/heads/stable-.*
    commentOnChangeAbandoned = false
    commentOnChangeCreated = true
    commentOnChangeMerged = true
    commentOnChangeRestored = false
    commentOnCommentAdded = false
    commentOnFirstLinkedPatchSetCreated = true
    commentOnPatchSetCreated = false
    commentOnRefUpdatedGitWeb = true
    enabled = enforced
  [commentlink "its-jira"]
    match = ^[ \\t]*([A-Za-z]*-[0-9]{1,5}):
    link = https://<jira-domain>:<jira-port>/browse/$1
  [commentlink "changeid"]
    match = (I[0-9a-f]{8,40})
    link = "#/q/$1"
  ```

- verified label

  ```bash
  [label "Verified"]
      function = MaxWithBlock
      defaultValue = 0
      copyAllScoresIfNoCodeChange = true
      value = -1 Fails
      value =  0 No score
      value = +1 Verified
  ```

#### integrate with Jira:

- freeze `master` branch
  - `project.config`
    ```bash
    [access "refs/for/refs/heads/master"]
      push = block group user/Marslo Jiao (marslo)
      push = block group Registered Users
      submit = block group Registered Users
      submit = block group group user/Marslo Jiao (marslo)
      addPatchSet = block group user/Marslo Jiao (marslo)
      addPatchSet = block group Registered Users
      pushMerge = block group user/Marslo Jiao (marslo)
      pushMerge = block group Registered Users
    ```
  - `groups`
      ```bash
      ...
      global:Project-Owners      Project Owners
      global:Registered-Users    Registered Users
      ...
      user:marslo                user/Marslo Jiao(marslo)
      ...
      ```

- freeze multiple branches (`stable` & `release`) for the specific account
  - `project.config`
      ```bash
      [access "^refs/for/refs/heads/(stable|release)$"]
        push = block group Registered Users
        submit = block group Registered Users
        addPatchSet = block group Registered Users
        pushMerge = block group Registered Users
      [access "^refs/heads/(stable|release)$"]
        read = group user/Marslo Jiao (marslo)
        push = +force group user/Marslo Jiao (marslo)
        pushMerge = group user/Marslo Jiao (marslo)
      ```
  - `groups`
      ```bash
      ...
      global:Project-Owners      Project Owners
      global:Registered-Users    Registered Users
      ...
      user:marslo                user/Marslo Jiao(marslo)
      ...
      ```

- restriction for branches (`feature1`, `feature2` and `master`) for only allow code review merge, forbidden code push
  - `project.config`
  ```bash
  [access "refs/*"]
    read = group Project Owners
    read = group user/Marslo Jiao (marslo)
  [access "refs/for/*"]
    addPatchSet = group Project Owners
    addPatchSet = group user/Marslo Jiao (marslo)
    push = group Project Owners
    push = group user/Marslo Jiao (marslo)
    pushMerge = group Project Owners
    pushMerge = group user/Marslo Jiao (marslo)
  [access "^refs/heads/(feature1|feature2|master)$"]
    push = block group Registered Users
    pushMerge = block group Registered Users
    submit = group Change Owner
  ```
  - `groups`
    ```bash
    ...
    global:Project-Owners      Project Owners
    global:Registered-Users    Registered Users
    ...
    user:marslo                user/Marslo Jiao(marslo)
    ...
    ```

- example of `project.config`
  - [project.config](https://gerrit.googlesource.com/gerrit/+/refs/meta/config/project.config)

  ```bash
  [project]
    description = Gerrit Code Review
  [access "refs/*"]
    owner = group google/gerritcodereview-maintainers@googlegroups.com
  [access "refs/heads/*"]
    label-Code-Review = -2..+2 group google/gerritcodereview-maintainers@googlegroups.com
    label-Code-Review = -2..+2 group polygerrit-maintainers
    label-Verified = -1..+1 group Change Owner
    label-Verified = -1..+1 group gerrit-verifiers
    label-Code-Style = -1..+1 group gerrit-verifiers
    label-Verified-Notedb = -1..+1 group gerrit-verifiers
    label-Library-Compliance = -1..+1 group gerrit-lib
    label-Library-Compliance = -1..+0 group google/gerritcodereview-maintainers@googlegroups.com
    submit = group Change Owner
    submit = group google/gerritcodereview-maintainers@googlegroups.com
    create = group google/gerritcodereview-maintainers@googlegroups.com
    abandon = group gerrit-verifiers
    editTopicName = +force group google/gerritcodereview-maintainers@googlegroups.com
    removeReviewer = group google/gerritcodereview-maintainers@googlegroups.com
    publishDrafts = group google/gerritcodereview-maintainers@googlegroups.com
  [access "refs/tags/*"]
    create = group gerrit-release-creators
    create = group google/gerritcodereview-maintainers@googlegroups.com
    createTag = group gerrit-release-creators
    createTag = group google/gerritcodereview-maintainers@googlegroups.com
    createSignedTag = group gerrit-release-creators
    createSignedTag = group google/gerritcodereview-maintainers@googlegroups.com
  [access]
    inheritFrom = Public-Projects
  [receive]
    rejectImplicitMerges = true
  [reviewer]
    enableByEmail = true
  [label "Verified"]
    function = MaxNoBlock
    copyAllScoresIfNoCodeChange = true
    value = -1 Fails
    value = 0 No score
    value = +1 Verified
    defaultValue = 0
  [label "Code-Style"]
    function = MaxWithBlock
    copyAllScoresIfNoCodeChange = true
    value = -1 Wrong Style or Formatting
    value = 0 No score
    value = +1 Style Verified
    defaultValue = 0
  [label "Library-Compliance"]
    function = MaxWithBlock
    copyAllScoresIfNoCodeChange = true
    copyAllScoresOnTrivialRebase = true
    value = -1 Do not submit
    value = 0 No score
    value = +1 Approved
    defaultValue = 0
  [access "refs/for/refs/meta/dashboards/*"]
    push = group google/gerritcodereview-maintainers@googlegroups.com
  [access "refs/meta/dashboards/*"]
    label-Code-Review = -2..+2 group google/gerritcodereview-maintainers@googlegroups.com
    label-Code-Review = -1..+1 group Registered Users
    label-Verified = -1..+1 group gerrit-verifiers
    label-Verified = -1..+1 group google/gerritcodereview-maintainers@googlegroups.com
    submit = group google/gerritcodereview-maintainers@googlegroups.com
    forgeAuthor = group google/gerritcodereview-maintainers@googlegroups.com
    label-Code-Style = -1..+1 group google/gerritcodereview-maintainers@googlegroups.com
  [access "refs/for/refs/meta/config"]
    push = group gerrit-verifiers
  [notify "polygerrit-reviews"]
    email = polygerrit-reviews@google.com
    type = all_comments
    type = submitted_changes
    header = cc
    filter = file:polygerrit-ui
  [access "refs/heads/infra/config"]
    push = group gerrit-tricium-admins
  ```

### reference
- [project owner guide](https://www.gerritcodereview.com/intro-project-owner.html)
- [Gerrit Code Review - Uploading Changes](https://www.gerritcodereview.com/user-upload.html)
- [gerrit/gerrit/refs/meta/config](https://gerrit.googlesource.com/gerrit/+/refs/meta/config)
- [gerrit 权限控制](https://blog.csdn.net/chenjh213/article/details/50571190)
- [its-jira plugin md](https://gerrit.googlesource.com/plugins/its-jira/+/refs/heads/stable-3.0/src/main/resources/Documentation/config.md)
- [Rule base configuration](https://review.opendev.org/plugins/its-storyboard/Documentation/config-rulebase-common.html)
