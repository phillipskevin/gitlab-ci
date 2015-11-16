## GitLab CI is an open-source continuous integration server

### Install GitLab CI
1. Install [Vagrant](http://www.vagrantup.com/downloads)
2. `git clone https://github.com/phillipskevin/gitlabci-installer`
3. `cd gitlabci-installer && vagrant up`

### Connect GitLab CI to GitLab

#### Add a deploy key to your project in GitLab for use by GitLab CI
This will allow GitLab to send project information to GitLab CI and trigger CI builds.

##### Create a private/public ssh key pair
1. `vagrant ssh`
2. `ssh-keygen -t rsa`
3. `sudo service ssh restart`

##### Add the public key as a deploy key in GitLab. 
4. `cat ~/.ssh/id_rsa.pub`
5. Copy the ssh key
6. From the project page in GitLab, click **Settings** > **Deploy Keys** > **+ New Deploy Key** (you will need master access to the project to complete this step)
7. Add the key and save

#### Add GitLab CI as an application that can use GitLab as an OAuth provider
This step allows users to log in to GitLab CI using their GitLab credentials.

1. Go to <GitLab URL>/profile/applications
2. Click **New Application**
3. The Redirect URI can be found at <GitLab CI URL>/help/oauth2 (it should be something like <GitLab CI URL>/user_sessions/callback)
4. Copy the `Application ID` and `Secret` to `/home/gitlab_ci/gitlab-ci/config/application.yml`
5. `sudo service gitlab_ci restart`
 
### Add a GitLab CI Runner
1. Use the [gitlab-ci-multi-runner install instructions](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner#installation) to install a runner for the operating system of your choice
2. When prompted for the gitlab-ci token, use the value given at <GitLab CI URL>/admin/runners

```
NOTE: if installing the runner on a machine other than the GitLab CI server, 
you will need to generate another SSH key for the runner and add it as a deploy key also.

Some runner installations run as user gitlab-runner. To generate an ssh key as this user:

sudo -H -u gitlab-runner bash -c 'ssh-keygen -t rsa'
```

### Add a project to CI
1. From the GitLab CI, find your project, and click the **Add project to CI** button
2. From the project page in GitLab CI, click on the runners link in the sidebar
3. Find the runner you want to use for your project and click **Enable for this project**
4. Create a [.gitlab-ci.yml](http://doc.gitlab.com/ci/yaml/README.html) file to the root of your project
5. Commit this project to your GitLab repo. This will trigger a build in GitLab CI.

### Example .gitlab-ci.yml file
Here is an example `.gitlab-ci.yml` file that will merge a branch with master and run `grunt test` in a virtual frame buffer, which will allow testing in a browser such as Firefox:

```
before_script:
  - git checkout master
  - git reset --hard origin/master
  - git merge @{-1} -m 'pre-merging into master'
  - nvm install 0.10
  - node --version
  - npm --version
  - npm install -g bower grunt-cli
  - bower --version
  - npm install
  - bower install
  - grunt --version

build_job:
  script:
    - xvfb-run grunt test
```
