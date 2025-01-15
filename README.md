# Challenger

This repo contains code for automating the creation of customized code challenge
repos, inviting applicants to the repos, and team members to the repo (to review
the results).

We assume a base repository already built out - ideally this is a repo that contains
some functionality that can be added to or refactored, preferably in the same general
domain the candiate will be working in if they get the job.

Each applicant will get a custom repo (eg: coding-challenge-for-cselmer) set up
with Github issues for them to address, and Github pull requests for them to
review and provide feedback on.

These custom repos are set up in the account tied to the access token. This has
been a personal account in the past, since there can be many of these repos
created. Personal accounts have unlimited private repos, while organizations
are charged on tiers. At the time of this repo creation, the organization was at the
threshold of the next payment tier.


## Setup

Rename .env-sample to .env and add your Github access token. Set up an
access token here: https://github.com/settings/tokens


## Config

The customized repos are based on an existing repo configured specifically for this
coding challenge.

The pull requests created in the customized repos are actual pull requests on
the above repo. Additional PRs can be added by creating a PR on the above repo
and configuring the PR information in pulls.yml

Issues are created from the content in issues.yml

## Commands

`rake issue_coding_challenge_to[:github_id]`
Creates the custom repo for the specified github_id, including issues and pull
requests, and sends a Github collaboration invite to the user.

Since their Github account may not be tied to an email account they check
regularly, it is a good practice to also email them a direct link to the
repository.

`rake send_team_invites_for[:github_id]`
Invites team members (specified in config.yml) to the repository.
As with the previous command, it is a good practice to also email the team members
the link directly.

There are other commands that manage parts of both of the above commands, but
should not generally need to be called directly.
