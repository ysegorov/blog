+++
path = "/2016/jenkinsfile/"
title = "Pipeline As a Code With Jenkins"
date = 2016-12-27
[extra]
modified = 2016-12-27T10:45:00+00:00
+++
# Pipeline as a code with Jenkins


[Jenkins][1] has a pretty nice [Pipeline][2] plugin to manage build, test and
deploy steps in a single file named `Jenkinsfile`. This file should be stored
alongside with the project code just like `python`, `javascript`, `html` or
whatever files.

The only problem with `Jenkinsfile` is that it must be written in [Groovy][4].
To me this is the same problem as with `Vagrantfile` which must be written in
[Ruby][5].
There is something strange forcing users to study language in order to have
proper configuration file.

It took some time to prepare workable `Jenkinsfile` which works for
[Multibranch Workflows][3] so I decided to create a [gist][6] with it.

Here it is:

{{ gist(url="https://gist.github.com/ysegorov/a23e7324e264b9afd932bd8b90862305") }}


It works pretty good for me providing slack notifications to production or
staging channels and having links to [Bitbucket][7] diff page to see changes
commited between builds.

This `Jenkinsfile` is suitable for `python` or `nodejs` projects and requires
just minimal tuning - to provide proper *repository name and url* and *build,
test and deploy* commands for the project.
Deployment happens only for `develop` (staging server) and `master` (production
server) branches.

Hope somebody will find this [gist][6] usefull.


[1]: https://jenkins.io/
[2]: https://jenkins.io/solutions/pipeline/
[3]: https://jenkins.io/blog/2015/12/03/pipeline-as-code-with-multibranch-workflows-in-jenkins/
[4]: http://www.groovy-lang.org/
[5]: https://www.ruby-lang.org/en/
[6]: https://gist.github.com/ysegorov/a23e7324e264b9afd932bd8b90862305
[7]: https://bitbucket.org
