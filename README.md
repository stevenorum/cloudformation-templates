# cloudformation-templates
Various useful cloudformation templates.

Important note: I'm still working on minimizing the IAM permissions for some of the entities in the templates, so some of the policies included may be overly permissive.

## [codepipeline-codecommit.cf.json](https://github.com/stevenorum/cloudformation-templates/blob/master/templates/codepipeline-codecommit.cf.json)

Template for a CodePipeline that will automatically build a CodeCommit package using CodeBuild.  You need to already have a CodeCommit package with a buildspec.yml file; this will take care of the rest.

## [codepipeline-github.cf.json](https://github.com/stevenorum/cloudformation-templates/blob/master/templates/codepipeline-github.cf.json)

Template for a CodePipeline that will automatically build a GitHub package using CodeBuild.  You need to already have a GitHub package with a buildspec.yml file; this will take care of the rest.

Some manual work is required to enable the auth for this pipeline to talk to GitHub:

-Go to [Settings/Developer settings/Personal access tokens](https://github.com/settings/tokens/new) in GitHub to create a new OAuth token.

-Click "Generate new token"

-In "Token description", put "AWS CodeBuild/CodePipeline" or something else descriptive.

-Under "Select scopes", select "repo" and "admin:repo_hook"

-Click "Generate Token"

-Copy the token.  Keep it secret, keep it safe, and provide it as an unencrypted input to your CF stacks.  (Yeah, I know that means read access to your CF stack inputs is equivalent to admin access to all your GitHub repos, but that's what seems to currently be required.)

-Finally, follow the instructions in step 5 of ["Create a Build Project (Console)"](http://docs.aws.amazon.com/codebuild/latest/userguide/create-project.html#create-project-console) in the CodeBuild docs to directly give AWS access to GitHub.  (And yeah, I know this is just giving AWS the same permissions as those given by the aforementioned OAuth token, but it's what seems to currently be required.)

## [codepipeline-lambda.cf.json](https://github.com/stevenorum/cloudformation-templates/blob/master/templates/codepipeline-lambda.cf.json)

Template for a CodePipeline that will automatically deploy a Lambda function behind APIGateway.  The inputs to the stack are the same as for codepipeline-github.cf.json.  The package [stevenorum/example-lambda-website](https://github.com/stevenorum/example-lambda-website) is a simple website that the pipeline will successfully launch; you can use that as a basis to learn how to add more complex functionality.
