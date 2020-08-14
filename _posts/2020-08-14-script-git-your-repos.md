---
title: How fast copy old projects to git repos
author: Chaves
date: 2020-08-14 02:00:00 +0000
categories: [Blogging, Git]
tags: [.net, azure, git, powershell]
---

I founded on a spare hard drive some old projects that I would like to keep safe. So I decided to move all content for Azure Git repositories - now they are a click distance away if someday I need to get some libs or remember some special architecture made. - probably not, just a handy code or methods.
The hard thing with this process is that you already have a folder structure with your code... and you can't git clone into a non-empty folder.... but you can with some dark magic... which isn't the way I wanted to approach this issue, so here are the steps that I made:

## Create Repos
I started to create all the git repo necessary with the following nomenclature: client.project.type_of_application

## Run powershell script
After the git repos are created and authentication with your git repo is established here's that code I use:

```powershell
#First let's delete any .git folder that had existed on the past (by older repos)
Remove-Item .git -Recurse -Force -Confirm:$false

#delete any packages folders that might have been forgotten (saving space)
Remove-Item packages -Recurse -Force -Confirm:$false

#delete any .vs folder (unwanted files)
Remove-Item .vs -Recurse -Force -Confirm:$false

#delete the temp folder (in case something runs bad after retry the temp folder shouln't exists)
Remove-Item temp -Recurse -Force -Confirm:$false

#perform the git clone of the empty repo into a temp folder
git clone https://gchaves@dev.azure.com/xxx/Project-archive/_git/myproject.application temp

#move the .get (struct folder) from the empty to project dir (saw this technique on stackoverfloww)
Move-Item -Force -Path temp/.git  -Destination .git

#dete the temp folder => at this stage, should be empty
Remove-Item temp -Recurse -Force -Confirm:$false

#all all the files that are at project folder
git add .

#commit
git commit -m "all file to be archive" -a

#push to the new repo :-)
git push
```

Hope that this could be a helper in some way. 💪👍