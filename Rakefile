# frozen_string_literal: true

require 'rubygems'
require 'bundler/setup'
Bundler.require(:default)
Dotenv.load
require 'yaml'

CONFIG = YAML.load_file('config.yml')

GITHUB_ORG = CONFIG['github_org']
CODE_CHALLENGE_REPO = CONFIG['base_repo_name'].freeze
CODE_CHALLENGE_REPO_FULL = "#{GITHUB_ORG}/#{CODE_CHALLENGE_REPO}".freeze
CODE_CHALLENGE_REPO_URL = "https://github.com/#{CODE_CHALLENGE_REPO_FULL}.git".freeze
CANDIDATE_BASE_REPO_NAME = CONFIG['candidate_base_repo_name'].freeze

TEAM_GITHUB_IDS = CONFIG['team_github_ids']

PULLS = CONFIG['pulls']
ISSUES = CONFIG['issues']

GITHUB_CLIENT = Octokit::Client.new(access_token: ENV.fetch('access_token', nil))

# rubocop:disable Style/TrivialAccessors
def github_id
  @github_id
end
# rubocop:enable Style/TrivialAccessors

def candidate_repo
  "#{CANDIDATE_BASE_REPO_NAME}_#{github_id}"
end

def candidate_repo_full
  "#{GITHUB_ORG}/#{candidate_repo}"
end

def candidate_repo_url
  "https://github.com/#{candidate_repo_full}.git"
end

def output(text)
  puts ">> #{text}"
end

desc 'Refresh local directories with most up-to-date content from base repo'
task :refresh_local_directories do
  output 'Removing previous versions...'
  `rm -rf repo-* #{CODE_CHALLENGE_REPO}`

  # re-clone and refresh main directory
  output 'Cloning coding-challenge repo...'
  `git clone #{CODE_CHALLENGE_REPO_URL}`

  output 'Copying coding-challenge repo into directory and removing .git directory...'
  `cp -r #{CODE_CHALLENGE_REPO} repo-main`
  `rm -rf repo-main/.git`

  # refresh feature directories
  PULLS.each do |pull|
    branch_name = pull['branch']
    dir_name = "repo-#{branch_name.gsub('/', '-')}"

    output "Checking out #{branch_name} branch"
    Dir.chdir(CODE_CHALLENGE_REPO) do
      `git checkout #{branch_name}`
    end

    output "Copying #{branch_name} repo into feature directory and removing .git directory..."
    `cp -r #{CODE_CHALLENGE_REPO} #{dir_name}`
    `rm -rf #{dir_name}/.git`
  end
  output 'Finished refreshing local directories'
end

desc 'Set the github_id instance variable'
task :set_github_id, [:github_id] do |_t, args|
  @github_id = args.github_id
end

desc 'Create repo with issues and pull requests for a candidate and send invite'
task :issue_coding_challenge_to, [:github_id] => [:set_github_id] do |_t, _args|
  output "Preparing coding challenge for #{github_id}..."
  Rake::Task[:reset_repo_for].invoke(github_id)
  Rake::Task[:send_candidate_invite_to].invoke(github_id)
end

desc 'Resets repo for a candidate'
task :reset_repo_for, [:github_id] => [:set_github_id] do |_t, _args|
  output "Resetting repo for #{github_id}..."
  Rake::Task[:delete_repo_for].invoke(github_id)
  Rake::Task[:setup_repo_for].invoke(github_id)
end

desc 'Deletes GitHub repo and local directory for a candidate'
task :delete_repo_for, [:github_id] => [:set_github_id] do |_t, _args|
  output "Deleting #{candidate_repo_full} repo from GitHub if it exists..."
  GITHUB_CLIENT.delete_repository(candidate_repo_full)
  output "Deleting #{candidate_repo} local directory if it exists..."
  `rm -rf #{candidate_repo}`
  output 'Done deleting.'
end

desc 'Creates GitHub repo with issues and pull requests for a candidate'
# rubocop:disable Metrics/BlockLength
task :setup_repo_for, [:github_id] => [:set_github_id] do |_t, _args|
  output "Creating #{candidate_repo_full} repo..."
  GITHUB_CLIENT.create_repository(candidate_repo,
                                  private: true,
                                  has_issues: true,
                                  has_projects: false,
                                  has_wiki: false,
                                  description: "#{GITHUB_ORG} Coding Challenge for #{github_id}")

  sleep(5) # wait for repo to be created
  ISSUES.each do |issue|
    output "Creating '#{issue['title']}' in repo..."
    GITHUB_CLIENT.create_issue(candidate_repo_full, issue['title'], issue['body'])
  end

  output 'Adding initial release content to repo and pushing...'
  `git clone #{candidate_repo_url}`

  Dir.chdir(candidate_repo) do
    `cp -r ../repo-main/ .`
    `git add .`
    `git commit -m 'Initial release'`
    `git push`
  end

  # Create and push branches and create pull requests
  PULLS.each do |pull|
    branch_name = pull['branch']
    dir_name = "repo-#{branch_name.gsub('/', '-')}"
    puts `pwd`

    output "Adding and commiting '#{branch_name}' content to repo, and pushing..."
    Dir.chdir(candidate_repo) do
      `git checkout main`
      `git checkout -b #{branch_name}`
      `cp -r ../#{dir_name}/ .`
      `git add .`
      `git commit -m "#{pull['commit']}"`
      `git push -u origin #{branch_name}`
    end
    sleep(2) # Wait for branch to be created
    output "Creating pull request for '#{branch_name}' branch..."
    GITHUB_CLIENT.create_pull_request(candidate_repo_full,
                                      'main',
                                      branch_name,
                                      pull['title'],
                                      pull['body'])
  end
  output 'Removing local directory...'
  `rm -rf #{candidate_repo}`
  output "Done setting up repo for #{github_id}."
end
# rubocop:enable Metrics/BlockLength

desc 'Send team invites to a candidate repo'
task :send_team_invites_for, [:github_id] => [:set_github_id] do |_t, _args|
  TEAM_GITHUB_IDS.each do |team_member|
    output "Inviting #{team_member} to #{candidate_repo_full}..."
    GITHUB_CLIENT.invite_user_to_repo(candidate_repo_full, team_member)
  end
  output 'Done sending invites to team.'
end

desc 'Send invite to candidate for their candidate repo'
task :send_candidate_invite_to, [:github_id] => [:set_github_id] do |_t, _args|
  output "Inviting #{github_id} to #{candidate_repo_full}"
  GITHUB_CLIENT.invite_user_to_repo(candidate_repo_full, github_id)
  output "Done sending invite to #{github_id}."
end
