---
layout:     page
title:      "Migrate to Git from SVN"
subtitle:   ""
date:       2021-03-17
author:     "Jim Ma"
header-img: "img/post-bg-06.jpg"
---
The command to migrate svn project to git 
```
1. svn co https://your/project
2. svn log -q | awk -F '|' '/^r/ {sub("^ ", "", $2); sub(" $", "", $2); print $2" = "$2" <"$2">"}' | sort -u > authors-transform.txt
3. git svn clone --stdlayout --authors-file=authors-transform.txt https://your/svn/project /your/local/gitrepo
```
Make sure you installed the latest git-svn, otherwise you will see couple of strange perl failures. 
If you already installed the latest version , you still see some failures and try it again, sometimes
the error will disappear.

After the git svn clone, you can add your remote repo and push your local repo.

If you want to add all the tags to the git repo. This shell will be helpful:
```
git for-each-ref refs/remotes/origin/tags | cut -d / -f 5- |
while read ref
do
git tag -a "$ref" -m "Tag $ref" "refs/remotes/origin/tags/$ref"
git push origin tag "$ref"
done
```
After all the tags are converted, ```git push --tags" to add these tags to remote repo. 

Hope you find this helps when you migrate your old svn repo to github or gitbucket.