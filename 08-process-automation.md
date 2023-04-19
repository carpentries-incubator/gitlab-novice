---
title: "Process Automation"
teaching: 0
exercises: 0
---

::: questions

- How can I use GitLab to automate processes?
- How can I host a static website directly from GitLab?

:::

::: objectives

- Setup a process to run automatically when a commit is pushed to a repository.
- Safely use secrets in a process automatically triggered by GitLab.
- Configure a GitLab repository to host a static website.

:::

We have setup our project, collaborated with our colleagues;
we fill our research diary every day. Now it is time to make use of it.

We want to share it with the world, because what we do is so important and everyone should now about it.
So we want to create a website based on the diary and update it whenever our Markdown files change.

Just as much based in reality as the rest of this lesson’s story, we know how to do that, but we do not want to do it by hand.
Instead, we are going to let GitLab automatically turn our Markdown files into HTML, whenever our repository changes.
To achieve it, we will use two GitLab features: CI/CD-Pipelines and Pages.

::: callout

### CI/CD or Continuous Integration/Continuous Deployment

The terms continuous integration (CI) and continuous deployment (CD) come from the fields of software engineering and software operation.

A software project uses CI, if it has setup a process that automatically runs some (or all) tests for a software project, whenever changes are committed or even just proposed (for example through a merge request) to the main branch of a repository.
In particular, when tests run already for proposed changes this can help prevent bugs to reach the code base, while freeing the developer of remembering to run all the tests, before creating a merge request.

CD is a process that updates the software on the production machines (deploys) as soon as a new change reaches the code base, generally after having passed the CI process.

Both processes can be implemented by running scripts, called jobs on GitLab, triggered either by changes to the code base or by the successful completion of other jobs, which lead to the CI/CD feature of GitLab.

:::

## Example Configuration

To configure our automatic process, we navigate to our project page and click the menu item labeled  “CI/CD” in the main menu on the left.

The page that this leads to invites us to “use a sample `.gitlab-ci.yml` template file to explore how CI/CD works.”
The GitLab CI is configured by providing a file called `.gitlab-ci.yml` in the root directory of a project’s repository.
Even though we want to learn how to configure CI in GitLab, we do not follow the invitation, because the example is quite elaborate and targets software developers specifically.
Instead, we will use the provided template for Bash.

In the list at the bottom of the page, we click on the button labeled “Use template” in the entry for Bash.

This leads us to an editor for the `.gitlab-ci.yml` file that is prepopulated with the selected template.
The file is expected to be written in the [YAML](https://yaml.org/) file format, hence the file extension `.yml`.
We will go through the example line by line. Afterwards we will adapt it to our needs.

The first few lines, starting with the symbol `#`, are comments notifying us of where to find further information.
The last of these lines is the exception, which is a comment on the first functional line:

```yaml
image: busybox:latest
```

This line states that the scripts that are provided later on should be executed in Docker containers build from the stated Docker image.
Busybox is a very small Linux distribution, reduced to the bare minimum, that provides bash, a shell.

::: callout

### Docker

TODO

:::

It is followed by two blocks starting with `before_script:` and `after_script:`.
We will ignore those.

The remaining four blocks, `build1:`, `test1:`, `test2:`, and `deploy1:`, each define a so called **job**.
A job represents one non-divisible unit of a CI/CD process.
Together the jobs defined in a project’s `.gitlab-ci.yml` file form a so called **pipeline**.

By default a pipeline has three **stages**, called `build`, `test`, and `deploy`, and their order is important.
The terminology comes from software development again.
A pipeline is executed by running all jobs of the first stage and, if they succeed, to then continue with jobs of the next stage, repeating this until a job fails or all stages complete.

The four jobs in the example are named similar to the stages they are assigned to.
The assignment is done through the keyword `stage:` that can in this example be found in the second line of each job’s definition.

The lines following the keyword `script:` in each section, define the script that will be executed as part of the respective job.
In this example, they are all `echo` statements, that output what follows.

## Our Own Configuration

We will now adapt the script to our requirements.

First we delete everything.

Then we define our own stages, because we do not develop software and want to have descriptive names for our use case:

```yaml
stages:
    - check
    - publish
```

In the first stage, we will do some testing, in the second we will create the HTML version of our diary and publish it.

We call the first job `check-for-mds` and in it make sure, that at least one Markdown file exists, because otherwise someone must have accidentally removed
them all.

```yaml
check-for-mds:
    image: bash:latest
    stage: check
    script:
        - compgen -G '*.md'
```

We use the docker image `bash`, because Bash provides the `compgen` command that allows us to check for the existence of files.
We also state that the job belongs to the stage `check`.
Finally, we provide the script.
It tests for the existence of any files with names ending in `.md`.

Whenever the job `check-for-mds` successfully completes, we want to create the HTML version of our research diary.
We define a second job and call it `publish-on-web`.

```yaml
publish-on-web:
    image:
        name: pandoc/core:latest
        entrypoint: ["/bin/sh", "-c"]
    stage: publish
    script:
        - pandoc *.md -o diary.html
    artifacts:
        paths:
            - diary.html
```

After giving the name of the job, we again provide an Docker image in which the job should run.
We use pandoc to convert our Markdown files into an HTML file, so we use the official pandoc Docker image.
The next line, with the keyword `entrypoint` is necessary, because the pandoc Docker image is configured to directly run pandoc, when started in a container (pandoc is configured as its entrypoint).
However, GitLab CI expects to run scripts in the Docker container in a shell.

::: callout

### Pandoc

Pandoc converts text documents from one format to another, for example from Markdown to HTML.
It supports many formats for the source documents and even more for the target document.

The [project webpage](https://pandoc.org/) provides a complete list (in text and graphical form).

The conversion is customizeable, for example through templates.

:::

Next, we specify the stage, followed by the script.
It runs pandoc on all files ending in `.md` in the current directory and instructs it to output a file `diary.html`.
Pandoc deduces from the file extension that it should be in HTML format.

The final three lines, starting with the keyword `artifacts`, specify that GitLab CI should save the file `diary.HTML` from the Docker container the job runs in and provide it for download on its web interface.

This completes our GitLab CI configuration.
There are a lot more configuration options, for example to have a process run only for commits to certain branches, that are documented in [GitLab’s Handbook](https://docs.gitlab.com/ee/ci/yaml/gitlab_ci_yaml.html)

We change the commit message to ”Configure CI", leave the branch as it is (`main`), and click the button labeled “Commit changes”.

::: callout

### Shared Runners

The software that runs CI jobs is called a runner.
GitLab provides so called shared runners on its platform for free (within certain usage limits).
It does, however, require the user to provide credit card information.
This is intended to prevent abuse of the free resource.

If no such information has been provided running the CI will always fail.

Alternatively, the organizers of a course based on this material could setup their own runner and help learners to configure their projects to use that runner.

On other instances the situation can be different.

:::

Clicking the button does not cause a change in page. Instead, we are informed at the top of the page about the status of the pipeline run that was initiated by committing the `.gitlab-ci.yml` to the repository.

To get a better overview, we select “CI/CD” from the main menu on the right side.
This brings us to the list of pipeline runs. We should see exactly one entry in the list, as we have just configured the CI.

A blue, red, or green box in the “Status” column informs on the left informs us about the state of the pipeline.
Blue indicates a running pipeline, red a failed pipeline, and green a pipeline that successfully completed.
Below the status box we have the runtime of the pipeline and below that information about when it was started.

In the ”Pipeline” column, we see the first line of the commit that triggered the pipeline run, the number of the pipeline run, for example `#11071`, the branch the commit is part of as well as the short identifier of the commit.
Most of these serve as links to pages for their respective entities.

The ”Triggerer” column tells us who caused the pipeline run.
In our case that’s the author of the commit.

The ”Stages” column visualizes the stages and their state.
In addition to the colors discussed above, there are stages colored in gray.
They are waiting for their predecessor stages (or need to be manually triggered, if configured that way).

We wait until our pipeline ran through.
It should complete without error.

::: instructor

The behavior of the CI is the most fragile step of this course.
There are some outside influences that can cause the pipeline to fail.
This needs to be handled on a case by case basis.

:::

We click on the button labeled by the three dots on the right of the entry.
In the menu that opens, we see that it provides a link to download artifacts of the pipeline.
We click on the link labeled “publish-on-web:archive”, which causes our browser to download a file called `artifacts.zip`.

In that archive, we find the file `diary.html` that was build by our second job, `publish-on-web`.

By opening the file in your browser, you can verify that it contains the contents of all Markdown our files.
(We ignore the fact, that the order might not make sense at all.
For that we would need to improve the script that builds the HTML file.)

## Publishing to the Web

So far we managed to create an HTML page from our Markdown files.
But we wanted to publish it on the web.

To achieve this goal, we can use a feature of GitLab called Pages.
It allows us to publish the results of a specific job of a CI pipeline to the web under the `gitlab.io` domain.

TODO: DESCRIBE HOW TO GET THERE

We change the configuration of our second job to look as follows:

```yaml
pages:
    image:
        name: pandoc/core:latest
        entrypoint: ["/bin/sh", "-c"]
    stage: publish
    script:
        - mkdir public
        - pandoc *.md -o public/diary.html
    artifacts:
        paths:
            - public
```

We change the job name to `pages`.
This name is a convention to tell GitLab, that we want this job to create the contents for our web page.
Then we create the HTML file in a directory called `public` and change the path that defines the job artifacts to `public` as well.
The directory `public` is, again by convention, the place from which GitLab will take the contents to be published.

TODO: DESCRIBE HOW TO SAVE

After committing our change, the pipeline will automatically run.
When it has successfully completed, we select Pages under the Settings heading of the menu on the left.
In the section called “Access pages”, we find a link to our web page.
