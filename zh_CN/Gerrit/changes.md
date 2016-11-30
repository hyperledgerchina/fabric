# Submitting a Change to Gerrit

请在提交改动之前阅读以下的要求，这个指南适用于刚开始接触开源项目的开发者，也适用于对开源项目经验丰富的开发者。

##改动要求

This section contains guidelines for submitting code changes for review.
For more information on how to submit a change using Gerrit, please
see [Gerrit](gerrit.md).

改动以git commit的方式提交，每个提交必须包含以下信息：
- 一个少于72个字符的简短描述标题，紧接着一个空白行。
-一个改动逻辑或者改动原因的简单描述，紧接着一个空白行。
- a Signed-off-by line, followed by a colon (Signed-off-by:)
- a Change-Id identifier line, followed by a colon (Change-Id:). Gerrit won't
  accept patches without this identifier.

A commit with the above details is considered well-formed.

All changes and topics sent to Gerrit must be well-formed. Informationally,
`commit messages` must include:

* **what** the change does,
* **why** you chose that approach, and
* **how** you know it works -- for example, which tests you ran.

Commits must [build cleanly](../dev-setup/build.md) when applied in top of each
other, thus avoiding breaking bisectability. Each commit must address a single
identifiable issue and must be logically self-contained.

For example: One commit fixes whitespace issues, another renames a
function and a third one changes the code's functionality.  An example commit
file is illustrated below in detail:

```
A short description of your change with no period at the end

You can add more details here in several paragraphs, but please keep each line
width less than 80 characters. A bug fix should include the issue number.

Fix Issue # 7050.

Change-Id: IF7b6ac513b2eca5f2bab9728ebd8b7e504d3cebe1
Signed-off-by: Your Name <commit-sender@email.address>
```
Each commit must contain the following line at the bottom of the commit
message:

```
Signed-off-by: Your Name <your@email.address>
```

The name in the Signed-off-by line and your email must match the change
authorship information. Make sure your :file:`.git/config` is set up
correctly. Always submit the full set of changes via Gerrit.

When a change is included in the set to enable other changes, but it
will not be part of the final set, please let the reviewers know this.
