# Contributing

When contributing to this repository, please first discuss the
change you wish to make via issue, email, or any other method with
the owners of this repository before making a change.

Please note we have a Code of Conduct, please follow it in all
your interactions with the project.

## Pull Request Process

1. Ensure that the tests (in `test/run`) still pass.

2. Ensure that any new ops files have tests for baseline validity
   in the `test/run` suite, and that those new tests (if any)
   pass.

3. Ensure that a fair portion of k8s deployments (combinations of
   ops files and variable settings) still deploy.  The definition
   of "a fair portion" is deliberately vague; please use your best
   judgment, and reach out if in doubt about how much testing you
   ought to do.

4. Provide the context of the discussion with the repository
   owners and core team members that lead to the submission of the
   pull request.  This may be as simple as a link to an issue.

5. After review and approval, your Pull Request will be merged by
   a repository owner.
