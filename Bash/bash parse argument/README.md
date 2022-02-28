# bash parse argument

Created: February 1, 2022 5:22 PM
Tags: Bash, Git

### Bash Space-Separated (e.g., `-option argument`)

```
cat >/tmp/demo-space-separated.sh <<'EOF'
#!/bin/bash

POSITIONAL=()
while [[ $# -gt 0 ]]; do
  key="$1"

  case $key in
    -e|--extension)
      EXTENSION="$2"
      shift # past argument
      shift # past value
      ;;
    -s|--searchpath)
      SEARCHPATH="$2"
      shift # past argument
      shift # past value
      ;;
    -l|--lib)
      LIBPATH="$2"
      shift # past argument
      shift # past value
      ;;
    --default)
      DEFAULT=YES
      shift # past argument
      ;;
    *)    # unknown option
      POSITIONAL+=("$1") # save it in an array for later
      shift # past argument
      ;;
  esac
done

set -- "${POSITIONAL[@]}" # restore positional parameters

echo "FILE EXTENSION  = ${EXTENSION}"
echo "SEARCH PATH     = ${SEARCHPATH}"
echo "LIBRARY PATH    = ${LIBPATH}"
echo "DEFAULT         = ${DEFAULT}"
echo "Number files in SEARCH PATH with EXTENSION:" $(ls -1 "${SEARCHPATH}"/*."${EXTENSION}" | wc -l)
if [[ -n $1 ]]; then
    echo "Last line of file specified as non-opt/last argument:"
    tail -1 "$1"
fi
EOF

chmod +x /tmp/demo-space-separated.sh

/tmp/demo-space-separated.sh -e conf -s /etc -l /usr/lib /etc/hosts

```

### **Output from copy-pasting the block above**

```
FILE EXTENSION  = conf
SEARCH PATH     = /etc
LIBRARY PATH    = /usr/lib
DEFAULT         =
Number files in SEARCH PATH with EXTENSION: 14
Last line of file specified as non-opt/last argument:
#93.184.216.34    example.com

```

### **Usage**

```
demo-space-separated.sh -e conf -s /etc -l /usr/lib /etc/hosts

```

---

### Bash Equals-Separated (e.g., `-option=argument`)

```
cat >/tmp/demo-equals-separated.sh <<'EOF'
#!/bin/bash

for i in "$@"; do
  case $i in
    -e=*|--extension=*)
      EXTENSION="${i#*=}"
      shift # past argument=value
      ;;
    -s=*|--searchpath=*)
      SEARCHPATH="${i#*=}"
      shift # past argument=value
      ;;
    -l=*|--lib=*)
      LIBPATH="${i#*=}"
      shift # past argument=value
      ;;
    --default)
      DEFAULT=YES
      shift # past argument with no value
      ;;
    *)
      # unknown option
      ;;
  esac
done
echo "FILE EXTENSION  = ${EXTENSION}"
echo "SEARCH PATH     = ${SEARCHPATH}"
echo "LIBRARY PATH    = ${LIBPATH}"
echo "DEFAULT         = ${DEFAULT}"
echo "Number files in SEARCH PATH with EXTENSION:" $(ls -1 "${SEARCHPATH}"/*."${EXTENSION}" | wc -l)
if [[ -n $1 ]]; then
    echo "Last line of file specified as non-opt/last argument:"
    tail -1 $1
fi
EOF

chmod +x /tmp/demo-equals-separated.sh

/tmp/demo-equals-separated.sh -e=conf -s=/etc -l=/usr/lib /etc/hosts

```

### **Output from copy-pasting the block above**

```
FILE EXTENSION  = conf
SEARCH PATH     = /etc
LIBRARY PATH    = /usr/lib
DEFAULT         =
Number files in SEARCH PATH with EXTENSION: 14
Last line of file specified as non-opt/last argument:
#93.184.216.34    example.com

```

### **Usage**

```
demo-equals-separated.sh -e=conf -s=/etc -l=/usr/lib /etc/hosts

```

---

To better understand `${i#*=}` search for "Substring Removal" in [this guide](http://tldp.org/LDP/abs/html/string-manipulation.html). It is functionally equivalent to ``sed 's/[^=]*=//' <<< "$i"`` which calls a needless subprocess or ``echo "$i" | sed 's/[^=]*=//'`` which calls *two* needless subprocesses.

---

### Using bash with getopt[s]

getopt(1) limitations (older, relatively-recent `getopt` versions):

- can't handle arguments that are empty strings
- can't handle arguments with embedded whitespace

More recent `getopt` versions don't have these limitations. For more information, see these [docs](https://mywiki.wooledge.org/BashFAQ/035#getopts).

---

### POSIX getopts

Additionally, the POSIX shell and others offer `getopts` which doen't have these limitations. I've included a simplistic `getopts` example.

```
cat >/tmp/demo-getopts.sh <<'EOF'
#!/bin/sh

# A POSIX variable
OPTIND=1         # Reset in case getopts has been used previously in the shell.

# Initialize our own variables:
output_file=""
verbose=0

while getopts "h?vf:" opt; do
  case "$opt" in
    h|\?)
      show_help
      exit 0
      ;;
    v)  verbose=1
      ;;
    f)  output_file=$OPTARG
      ;;
  esac
done

shift $((OPTIND-1))

[ "${1:-}" = "--" ] && shift

echo "verbose=$verbose, output_file='$output_file', Leftovers: $@"
EOF

chmod +x /tmp/demo-getopts.sh

/tmp/demo-getopts.sh -vf /etc/hosts foo bar

```

### **Output from copy-pasting the block above**

```
verbose=1, output_file='/etc/hosts', Leftovers: foo bar

```

### **Usage**

```
demo-getopts.sh -vf /etc/hosts foo bar

```

The advantages of `getopts` are:

1. It's more portable, and will work in other shells like `dash`.
2. It can handle multiple single options like `vf filename` in the typical Unix way, automatically.

The disadvantage of `getopts` is that it can only handle short options (`-h`, not `--help`) without additional code.

There is a [getopts tutorial](http://wiki.bash-hackers.org/howto/getopts_tutorial) which explains what all of the syntax and variables mean. In bash, there is also `help getopts`, which might be informative.

[https://stackoverflow.com/questions/192249/how-do-i-parse-command-line-arguments-in-bash](https://stackoverflow.com/questions/192249/how-do-i-parse-command-line-arguments-in-bash)