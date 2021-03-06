CI development workflow
=======================

This document describes the rules that control the development of source code responsible for the testing and build of OpenContrail.

Commit message style guide
--------------------------

|  The commit summary/title line should be no longer than 50 characters and contain precise information about which repo elements has been changed, to the extent possible.
|  There should be a blank line after the summary.
|  The description text after the blank line should be wrapped at 72 characters at most.
|  The description should explain the rationale behind the change introduced in this commit (i.e. why do we need this change).
|  The description should also include corresponding Jira ticket ID (i.e. ``Closes-Jira-Bug: JD-xxx`` or ``Partial-Jira-bug: JCB-xxx``). Otherwise, commit will automatically and instantly get ``Code-Review -2`` vote.

Approval workflow for CI projects hosted on Gerrit
--------------------------------------------------


Each change to CI projects should go through a review process:

#. The author of the change adds a core reviewers group.
   Each CI project has a corresponding user group that contains core reviewers:

   - contrail-project-config-core
   - contrail-zuul-jobs-core
   - etc.

#. First core reviewer that accepts the change should place a ``Code-Review +2``
   vote. The reviewer here and below shouldn't be the author of the change.
#. Second core reviewer that accepts the change should place an ``Approved +1``
   vote that will result in a merge (if the CI tests pass).

The author of the change (regardless if he's in the core reviewers group), can place
a ``Code-Review -2`` vote under his review to prevent others from reviewing it. This
vote should come with a comment explaining the reason behind it.

Approval workflow for CI projects hosted on GitHub
--------------------------------------------------

Pull requests for repositories developed on GitHub should follow this general procedure:

#. Pull Requests should be created from a single-purpose branch created
   in a personal fork of the repo (not applicable in case of private repos)
#. Pull Requests should be merged by their author after acquiring at least one "Approve" review
#. Pull Requests should be merged using the "create merge commit" strategy
#. The author should request review from the ``codilime-contractors-devops`` team

In case of our customized forks of public projects, updating the code (rebasing our changes onto upstream)
should be carried out after consulting with the infra team. These include especially:

- Juniper/zuul-jobs, a fork of https://github.com/openstack-infra/zuul-jobs
- Juniper/zuul, a fork of https://git.zuul-ci.org/zuul

Emergency procedures - Gerrit
-----------------------------

In an emergency, when the change is required to fix an error affecting
majority of the CI jobs or daily build jobs, the author can merge the change on
his own by placing both `CR+2` and `A+1` votes under his change. This will still
require successfully passing the check and gate pipelines.

If the CI system is broken beyond all recognition and the jobs are failing even for the
change that is supposed to fix the system, the CI can be bypassed by placing
a `Verified+2` vote and manually submitting the change for merging. This can be done:
- via the Gerrit webUI using an account with `Project-bootstrappers` membership
- via CLI, by logging in to the zuulv3.opencontrail.org and issuing Gerrit CLI commands.

Testing changes in contrail-zuul-jobs
-------------------------------------

The contrail-zuul-jobs repo contains roles and jobs for various projects, so
it is impractical to execute all the affected jobs in the check pipeline for
this repo. The only checks that are performed are unit tests and linting of
the Ansible code. If you're preparing a change that modifies a job for a
project you should test it as a dependent change.

Suppose you're modifying a role which is used in the `contrail-sanity-centos7-kolla-ocata`
job. This job is a part of the `systests` project template defined in contrail-project-config.
The template is applied to e.g. the `contrail-controller` project, which means the job
will be run if any change is made to the project. So to test your changes for the abstract
role you need to do the following:

#. Create your change for the contrail-zuul-jobs project (change A; changes to the role).
#. Create a dummy change (e.g. add an empty file) for the related project
   (change B in `contrail-controller`). In the commit message for change B, specify change A as a
   dependent change using the "Depends-On: <change-id>" directive, where `change-id` is the
   value of the `Change-Id` header of change A.
#. Place a comment under change A that contains a reference to change B, so
   that reviewers can verify that the changed job passed.
#. Make sure that the `check` pipeline (all jobs run for that pipeline) for the change B pass.

Transitional changes in CI
--------------------------

Because Zuul doesn't allow cycles in change dependencies (using the ``Depends-On:`` tags),
sometimes it is necessary to perform a transitional change in CI jobs. When one
wants to make a breaking change in one of the repos used in CI, he first
modifies the CI jobs to handle both versions of the repo (before and after
the breaking change), then merges the change in the repo, which now can pass CI
and then removes the support for the code before the change.
In case this is not possible, one can use the steps described in
"Emergency procedures" section.
