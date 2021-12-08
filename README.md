# dotfiles


* git-flow
* alwaysontop
* git aliases

```
# Git Aliases
# See https://github.com/yuanqing/git-cheatsheet
# 
alias gs='git status '
alias ga='git add '
alias gb='git branch '
alias gc='git commit'
alias gcm='git commit --amend -m'
alias gcl='git clone'
alias go='git checkout'
alias gco='git checkout'
alias gd='git diff'
alias gi='git init'
alias gf='git fetch'
alias gdc='git diff --cached'
alias gpom='git push origin master'
alias grao='git remote add origin'
alias gm='git merge'
alias gmf='git merge --no-ff'
alias gk='gitk --all&'
alias gx='gitx --all'
alias gpr='git diff -p -R --no-color | grep -E "^(diff|(old|new) mode)" --color=never | git apply';
```

* bash-git-prompt

```
## Configuration of bash-git-prompt
# Set config variables first
GIT_PROMPT_ONLY_IN_REPO=1

GIT_PROMPT_FETCH_REMOTE_STATUS=1   # uncomment to avoid fetching remote status
GIT_PROMPT_SHOW_UPSTREAM=1 # uncomment to show upstream tracking branch
GIT_PROMPT_SHOW_UNTRACKED_FILES=no # can be no, normal or all; determines counting of untracked files

# GIT_PROMPT_START=...    # uncomment for custom prompt start sequence
# GIT_PROMPT_END=...      # uncomment for custom prompt end sequence
source ~/.bash-git-prompt/gitprompt.sh
```


* Some utilitary functions to review

```
function mylines()
{
	git log --author="mallen" --pretty=tformat: --numstat | gawk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s removed lines: %s total lines: %s\n", add, subs, loc }'
}

# Read the function log to understand its behaviour
# No input argument
function gitmergedevelop()
{
	echo "-----------------------------------------------------------------------"
	echo "Merging branch develop into master (no ff), rebasing develop to master"
	echo "-----------------------------------------------------------------------"
	git status;
	echo "-----------------------------------------------------------------------"
	echo "Do you wish to continue? ENTER to continue, CTRL+C to stop"
	echo "-----------------------------------------------------------------------"
	read
	BRANCH=`gs | grep "On branch" | cut -d" " -f4`
	if [ "${BRANCH}" == "develop" ]
	then
		# Regularize current branch
		git pull
		if [ "$?" == "0" ]
		then
			git checkout master
		else
			echo "ERROR at checkout master"
			return 1;
		fi
		if [ "$?" == "0" ]
		then
			git pull
		else
			echo "ERROR when pulling master"
			return 1;
		fi
		if [ "$?" == "0" ]
		then
			git merge -m "ENH: Merge branch develop" --no-ff develop
		else
			echo "ERROR when merging develop into master (no-ff)"
			return 1;
		fi
		if [ "$?" == "0" ]
		then
			git checkout develop
		else
			echo "ERROR when checkout develop"
			return 1;
		fi
		if [ "$?" == "0" ]
		then		
			git rebase master
		else
			echo "ERROR when rebasing develop on master"
			return 1;
		fi
	else
		echo "ERROR : you are not on branch develop"
	fi
}

# No input argument
function gitpushmasterdevelop
{
	git checkout develop
	git push
	git checkout master
	git push
	git checkout develop
}

# Create a release branch
# Only operates from develop branch
# Read function log to understand its behaviour
function gitmkrelease
{
	if [ "$#" != "1" ]
	then
		echo "-- You should provide a version number (VX-Y-Z), aborting"
		return 1
	fi
	BRANCH=`gs | grep "On branch" | cut -d" " -f4`
	if [ "${BRANCH}" == "develop" ]
	then
		# Check version name is of the form : VX-Y-Z 
		VERSION=V`echo ${1} | sed -nre 's/^[^0-9]*(([0-9]+\-)*[0-9]+).*/\1/p'`
		if [ "${VERSION}" == "${1}" ]
		then 
			RELEASE_BRANCH=release_${VERSION}
			echo "-- Starting release branch ${RELEASE_BRANCH}"
			echo "-- First, check current state"
			git status | grep "working directory clean"

			if [ ! "$?" == "0" ]
			then
				echo "---- working directory not clean, aborting"
				return 1
			else
				echo "---- creating branch ${RELEASE_BRANCH}"
				git checkout -b ${RELEASE_BRANCH}
				if [ ! "$?" == "0" ]
				then 
					echo "---- problem creating branch ${RELEASE_BRANCH}"
					return 1
				fi
				git push --set-upstream origin ${RELEASE_BRANCH}
				if [ ! "$?" == "0" ]
				then 
					echo "---- error when pushing ${RELEASE_BRANCH}"
					return 1
				fi
			fi
		else
			echo "-- You should provide a valid version number (VX-Y-Z), aborting"
			return 1
		fi
	else 
		echo "-- You are not on branch develop, aborting"
		return 1
	fi
}

# Tag an existing release branch
# Only operates from a release_VX-Y-Z named branch
# Only operates if working dir is clean 
# No input argument
function gittagrelease
{
	echo "-- Trying to tag current release branch"
	# Get branch name
	branch_name=$(git symbolic-ref -q HEAD)
	if [ "$?" != "0" ]
	then
		echo "-- You are not currently on a git branch"
		return 1
	fi
	branch_name=${branch_name##refs/heads/}
	branch_name=${branch_name:-HEAD}

	RELEASE=`echo ${branch_name} | cut -d"_" -f1`
	VERSION=`echo ${branch_name} | cut -d"_" -f2`

	if [ "${RELEASE}x" != "releasex" ]
	then
		echo "-- You are not on a release branch : ${RELEASE}"
		return 1
	fi	 

	CHECK_VERSION=V`echo ${VERSION} | sed -nre 's/^[^0-9]*(([0-9]+\-)*[0-9]+).*/\1/p'`
	if [ "${CHECK_VERSION}" == "${VERSION}" ]
	then
		echo "-- tagging version ${VERSION}"
		git tag -a ${VERSION} -m "ENH: tag for release version ${VERSION}"
	else
		echo "-- Current branch name seems malformed : ${branch_name}"
	fi
	echo "-- If you tagged the wrong commit, use : "
	echo "---- git tag --delete ${VERSION}"
	echo "---- git push --delete origin ${VERSION}"
}

function gitrmtag
{
	echo "-- Trying to remove tag ${1}"
	if [ -z ${1} ]
	then
		echo "-- Input tag is empty"
		return 1
	fi

	# Local removal	
	git tag --delete ${1}
	if [ "$?" != "0" ]
	then
		echo "-- Problem removing local tag"
		return 1
	fi

	# Remote removal
	git push --delete origin ${1}
	if [ "$?" != "0" ]
	then
		echo "-- Problem removing remote tag"
		return 1
	fi
}
```
