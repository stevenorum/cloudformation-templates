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

In this template, at the end of the CloudFormationPolicy, there's a list of several <service>:* permissions that are granted.  Those should be the permissions necessary to create the CloudFormation stack, as well as any permissions that IAM roles in the stack itself need to inherit.  Unfortunately, it will likely have to be somewhat broad, due to the difficult of specifying resource-specific permissions for resources that haven't yet been created.

## [cloudhsm.cf.json](https://github.com/stevenorum/cloudformation-templates/blob/master/templates/cloudhsm.cf.json)

### \<Important warning>

**WARNING:** Most of the CF templates I write generate resources that are either below AWS's free-tier usage levels, or are very cheap (I had one beer with some friends last night and even before tip it cost more than my monthly AWS bill).  CloudHSM is **not** a cheap service.  Each HSM costs $1.45-$2.05 per hour, depending on the region, or between $1,000 and $1,500 per month.  Make sure to only have them running while you're actively using them.  Once you've [activated a CloudHSM cluster](https://docs.aws.amazon.com/cloudhsm/latest/userguide/activate-cluster.html), you can delete all the HSMs in the cluster and create replacements later and the keys, users, etc. that you initially created will still be there.

**In addition,** this template creates an EC2 instance so that you can communicate with the HSM.  By default it uses an m4.large instance, which costs ~$0.10 per hour, or ~$70 per month.  You can stop this instance when you aren't using it, and you can also substitute a smaller (and cheaper) instance type if you won't be running resource-intensive software.

### </Important warning> \<We now return to your regularly scheduled broadcasting>

This template uses CloudFormation custom resources to automatically set up an [AWS CloudHSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/introduction.html) cluster, even though that service isn't actually integrated with CloudFormation.

Hardware Security Modules are a good technology to be familiar with these days.  It seems that every other day I get a LinkedIn message from a recruiter from a cryptocurrency startup that's looking for devs for HSM stuff.  However, the bar to learning about HSMs can be pretty high.  The off-the-shelf price of an HSM is in the thousands or tens of thousands of dollars.  As mentioned before, CloudHSM isn't cheap compared with the rest of the stuff I post code for, but it's affordable to play around with if you're careful to not leave instances up and running long-term.

This template will create a CloudHSM cluster containing a single HSM.  In more detail, it will [create a VPC and subnets](https://docs.aws.amazon.com/cloudhsm/latest/userguide/prerequisites.html), it will [create a cluster and HSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/create-cluster-and-hsm.html) connected to that VPC, it will [initialize the cluster](https://docs.aws.amazon.com/cloudhsm/latest/userguide/initialize-cluster.html) for you, it will [launch a client instance](https://docs.aws.amazon.com/cloudhsm/latest/userguide/launch-client-instance.html), and it will [install and configure the client software](https://docs.aws.amazon.com/cloudhsm/latest/userguide/install-and-configure-client.html) on that instance.  At that point, you just need to log onto the client instance and follow the steps listed in /home/ec2-user/README.txt to [activate the cluster](https://docs.aws.amazon.com/cloudhsm/latest/userguide/activate-cluster.html).

Because of the slightly brittle way this template handles the CloudHSM resources, manual cleanup may be required when deleting the stack.  Although it should ideally work seamlessly through CF, I recommend deleting the HSM and cluster through the CloudHSM console instead of relying on CF to handle it.  This is not just recommended but strictly required if you delete and recreate your HSM while the CF stack is up, as CF will be ignorant of the new HSM's ID.
