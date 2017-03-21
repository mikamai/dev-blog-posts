---
ID: 244
post_title: Fast Capistrano tasks completion
author: Emanuele Serafini
post_date: 2014-05-30 10:22:48
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/05/30/fast-capistrano-tasks-completion/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/87287430324/fast-capistrano-tasks-completion
tumblr_mikamayhem_id:
  - "87287430324"
---
<p>Starting with the assumption that I&rsquo;m lazy and I make a lot of typos, I believe that command completion tools in a shell is indispensable.<br />
My current bash configuration is pretty simple, for completion I&rsquo;m using 
bash-completion installed from Homebrew. I already get completions for almost everything I want out of the box but I am missing completions for Capistrano tasks.</p>

<p>Here’s what I came up with, without running the slow  command <code>cap -vT</code>:</p>

<pre><code>_cap_envs()
{
  local cur
  cur=${COMP_WORDS[COMP_CWORD]}

  if [[ -d "config/deploy/" ]]; then
    envs=$(ls config/deploy/*.rb | xargs -n1 basename | sed -e 's/..*$//')
    COMPREPLY=( $(compgen -W "${envs}" -- $cur) )
    COMPREPLY=( "${COMPREPLY[@]/%/ }" )
  fi
}

_cap_deploy_tasks_list()
{
  local task
  task=${COMP_WORDS[COMP_CWORD]#deploy:}

  DEPLOY_OPTS='
    check
    check:
    cleanup
    cleanup_assets
    cleanup_rollback
    compile_assets
    finished
    finishing
    finishing_rollback
    log_revision
    migrate
    normalize_assets
    published
    publishing
    revert_release
    reverted
    reverting
    rollback
    rollback_assets
    set_current_revision
    started
    starting
    symlink:
    updated
    updating'

  case "${task}" in
    check:* )
      COMPREPLY=($(compgen -W "directories linked_dirs linked_files make_linked_dirs" -- ${task#check:} ))
      ;;
    symlink:* )
      COMPREPLY=($(compgen -W "linked_dirs linked_files release shared" -- ${task#symlink:} ))
      ;;
    * )
      COMPREPLY=($(compgen -W "${DEPLOY_OPTS}" -- $task ))
      ;;
  esac
  COMPREPLY=( "${COMPREPLY[@]/%/ }" )
  return 0
}

_cap_nginx_tasks_list()
{
  local task
  task=${COMP_WORDS[COMP_CWORD]#nginx:}

  COMPREPLY=($(compgen -W "start stop stop restart setup" -- $task))
  COMPREPLY=( "${COMPREPLY[@]/%/ }" )
}

_cap_puma_tasks_list()
{
  local task
  task=${COMP_WORDS[COMP_CWORD]#puma:}

  COMPREPLY=($(compgen -W "start stop restart hot_restart setup" -- $task))
  COMPREPLY=( "${COMPREPLY[@]/%/ }" )
}

_cap_tasks_list()
{
  local cur
  cur=${COMP_WORDS[COMP_CWORD]}

  COMPREPLY=($(compgen -W "deploy deploy: nginx: puma:" -- $cur))
}

_cap_tasks()
{
  local cur
  cur=${COMP_WORDS[COMP_CWORD]}

  case "${cur}" in
    deploy:* ) _cap_deploy_tasks_list ;;
    nginx:* ) _cap_nginx_tasks_list ;;
    puma:* ) _cap_puma_tasks_list ;;
    * ) _cap_tasks_list ;;
  esac
}

_cap()
{
  if [[ ! -f ./Capfile ]]; then
    return
  fi
  COMPREPLY=()

  local pos
  pos=${COMP_CWORD}

  case "${pos}" in
    1 ) _cap_envs ;;
    2 ) _cap_tasks ;;
  esac
}
complete -F _cap -o nospace cap
</code></pre>

<p>Completion now includes default capistrano 3 tasks and I&rsquo;ve added some tasks as examples. Paste this code inside your <code>.bash_profile</code> and fell free to add your personal capistrano tasks.</p>