#!/bin/bash
# Show the permissions on all the directories leading from / to
# the parameter.
# Usage: $0 [-h] [-v] <file_or_dir> <...>
#
#
# determine my name
me=$0
me=${me##*/}

#
# Verbose if -v
verbose=0

# To ensure that we pathlld / at the very end if we need to
startedatslash=0

help () {
    echo "${me} [-h] [-v]  <file> ... " >&2
    echo "${me}: Show mount state, permissions and ownerships for" >&2
    echo "${me}: each object in the path for <file>." >&2
    echo "${me}:     -h  This help." >&2
    echo "${me}:     -v  Turn on verbose output to STDERR." >&2
    exit 2
}

# I had to change this in two places 
# once too often, but rewrites eliminated
# the second place
function showMount () {
    if [[ $( mountpoint -q "${1}";echo $?) ]] ; then
	mount | egrep " on ${1} "
    fi
}

function myVerbose () {
    [[ $verbose -ne 0 ]] && echo "$@" >&2
}

# The recursive meat of the whole thing.
function pathlld ()
{
    # These are MINE! 
    local target parent
    target="$1"
    myVerbose "Entering pathlld target=\"$target\""
    # Here is a trick - if we see a '/' in the $target, there
    # is more in the filepath, than a simple object (there is
    # a parent dir to object relationship) so I set $parent
    # to the filepath less the trailing '/' and what follows,
    # then call pathlld on what's left.
    # 
    case "$target" in
	/ ) :;; 
	*/* ) parent="${target%/*}"
	      if [[ -z "$parent" ]] ; then
		  [[ $startedatslash ]] && parent="/" 
		  [[ $startedatslash ]] || return
		  startedatslash=0
	      fi
	      myVerbose "pathlld, target=\"$target\", parent=\"$parent\""
	      pathlld "$parent";;
	* ) :;;
    esac
    myVerbose "Unwinding, target=\"$target\", parent=\"$parent\""
    /bin/ls -ld "$target"
    [[ -L "$target" ]] && /bin/ls -ldL "$target"
    showMount "$target"
}				# end of function pathlld

# ===> Execution starts here <===

# Parse command options
while getopts "hv" options; do
  case $options in
    v ) verbose=1;;
    h ) help;;
    \? ) help;;
    * ) help
        exit 1;;

  esac
done
shift $(($OPTIND - 1))

# Make sure we got at least one filename or directory name

if [[ -z "$1" ]] ; then
    echo "$me: Nothing to do, use $me -h for help. Here's the help:" >&2
    help
fi

while [[ $# -ne 0 ]] ; do # loop through all the parameters 
    target="${1}"
    shift
    if [[ "${target#/}" = "$target" ]] ; then
	startedatslash=0
	else
	startedatslash=1
    fi
    myVerbose "main, target=\"$target\", startedatslash=\"$startedatslash\""
    # strip trailing /, unless that's all there is
    if [[ "$target" != "/" ]] ; then
	target=${target%/}
    fi
    
    myVerbose "main target=\"$target\"" 
    pathlld "$target"
done

exit 0
