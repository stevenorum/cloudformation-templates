# cloudformation-templates
Various useful cloudformation templates.

## devops
### [codepipeline-basic.cf.json](https://github.com/stevenorum/cloudformation-templates/blob/master/templates/devops/codepipeline-basic.cf.json)

Template for a CodePipeline that will automatically build a CodeCommit package using CodeBuild.  You need to already have a CodeCommit package with a buildspec.yml file; this will take care of the rest.
