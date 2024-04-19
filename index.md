---
site: sandpaper::sandpaper_site
---

[GitLab] is a web application for managing [Git] repositories.
Since it is build around Git, it is suitable to manage any project that works primarily with plain text files, for example software source code, TeX based documents, or meeting notes in Markdown.
With its built-in issue and wiki systems, it can, in certain cases, even be the right tool for managing a project without any files.

This lesson will give you a foundational understanding of GitLab’s features, so you can make informed decisions on how to use it as a tool.

Since GitLab interprets many of its text fields’ values as Markdown, more specifically [GitLab flavored Markdown][GitLabMarkdown], this lesson contains a rudimentary introduction to Markdown syntax, following the [CommonMark specification][CommonMark] on which the GitLab flavor is based.

Depending on previous knowledge of learners, the material can either be taught using solely the GitLab web interface or it can involve parts that teach the interaction with a local Git repository, using the Git command-line interface.

[CommonMark]: https://spec.commonmark.org/current/
[Git]: https://git-scm.com/
[GitLab]: https://about.gitlab.com/
[GitLabMarkdown]: https://docs.gitlab.com/ee/user/markdown.html

# Required Previous Knowledge

The material can be taught solely in the GitLab web interface.
In that case no previous knowledge is required.

Optionally, the interaction of local Git repositories with GitLab can be taught.
In that case a basic familiarity with the Git command-line interface is required.

To understand the material on GitLab’s continuous integration (CI) feature, basic knowledge of Bash and how [Docker] works is required.

[Docker]: https://www.docker.com/

::: instructor

# GitLab Instance

Not all GitLab instances have the same features and behavior.
Depending on version and configuration they may vary in many details.
If you plan to run this course on an instance other than gitlab.com, we recommend you go through the lesson once before teaching.
Even when teaching on gitlab.com this might be a good idea, because the platform is changing constantly.

Note, that the episode on CI does not work as described on gitlab.com because using the free shared CI runners requires the user to provide credit card information.

:::
