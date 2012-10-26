# Jenkins Gitlab Hook Plugin

Enables Gitlab web hooks to be used to trigger SMC polling on Gitlab projects

Plugin details can be found at https://wiki.jenkins-ci.org/display/JENKINS/Gitlab+Hook+Plugin

## Why?

For [Gitlab](http://gitlabhq.com) there is an existing solution that might work for you.
You can just use the notifyCommit hook on [Git plugin](https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin) like this:

```
http://yourserver/jenkins/git/notifyCommit?url=<URL of the Git repository for the Gitlab project>
```

But, with a large number of projects that are mostly polling (no hooks), the project might actually be built with a great delay (5 to 20 minutes).
You can find more details about notifyCommit and this issue [here](http://kohsuke.org/2011/12/01/polling-must-die-triggering-jenkins-builds-from-a-git-hook).

That is where this plugin comes in.
It gives you the option to use build\_now or notify\_commit hook, whichever suits your needs better.

### Build now hook

Add this web hook on your Gitlab project: 

```
http://yourserver/jenkins/gitlab/build_now
```

Plugin will parse the Gitlab payload and extract the branch for which the commit is being pushed and changes made.
It will then scan all Git projects in Jenkins and start the build for those that:

* match url of the Gitlab repo
* match committed Gitlab branch

Notes:

* for branch comparison, it will take into account both the branch definition and the strategy (this is different from the original notifyCommit)
* the project must be enabled
* you don't have to setup polling for the project

### Notify commit hook

Add this web hook on your Gitlab project: 

```
http://yourserver/jenkins/gitlab/notify_commit
```

The procedure is the same as for the build now hook, the difference is that this hook schedules polling of the project, much like the original notifyCommit.

Additional notes:

* the project must be configured not to skip notifyCommit

### Hook data related

Gitlab uses JSON POST to send the information to the defined hook.
The plugin expects the request to have the appropriate structure, like this example:

```json
{
        :before => "95790bf891e76fee5e1747ab589903a6a1f80f22",
         :after => "da1560886d4f094c3e6c9ef40349f7d38b5d27d7",
           :ref => "refs/heads/master",
       :user_id => 4,
     :user_name => "John Smith",
    :repository => {
               :name => "Diaspora",
                :url => "localhost/diaspora",
        :description => "",
           :homepage => "localhost/diaspora",
            :private => true
    },
       :commits => [
        [0] {
                   :id => "450d0de7532f8b663b9c5cce183b...",
              :message => "Update Catalan translation to e38cb41.",
            :timestamp => "2011-12-12T14:27:31+02:00",
                  :url => "http://localhost/diaspora/commits/450d0de7532f...",
               :author => {
                 :name => "Jordi Mallach",
                :email => "jordi@softcatala.org"
            }
        },

        ....

        [3] {
                   :id => "da1560886d4f094c3e6c9ef40349...",
              :message => "fixed readme",
            :timestamp => "2012-01-03T23:36:29+02:00",
                  :url => "http://localhost/diaspora/commits/da1560886d...",
               :author => {
                 :name => "gitlab dev user",
                :email => "gitlabdev@dv6700.(none)"
            }
        }
    ],
   total_commits_count => 3
}
```

## Dependencies

* [Ruby runtime](https://github.com/jenkinsci/jenkins.rb) version 0.10 or higher
* [Git plugin](https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin) version 1.1.24 or higer
