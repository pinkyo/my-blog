# 编写Shell脚本命令

## 参数处理

参数处理主要有两种实现方式：

### 1. 不使用getops

```bash

#!/bin/sh
# POSIX

die() {
    printf '%s\n' "$1" >&2
    exit 1
}

# Initialize all the option variables.
# This ensures we are not contaminated by variables from the environment.
file=
verbose=0

while :; do
    case $1 in
        -h|-\?|--help)
            show_help    # Display a usage synopsis.
            exit
            ;;
        -f|--file)       # Takes an option argument; ensure it has been specified.
            if [ "$2" ]; then
                file=$2
                shift
            else
                die 'ERROR: "--file" requires a non-empty option argument.'
            fi
            ;;
        --file=?*)
            file=${1#*=} # Delete everything up to "=" and assign the remainder.
            ;;
        --file=)         # Handle the case of an empty --file=
            die 'ERROR: "--file" requires a non-empty option argument.'
            ;;
        -v|--verbose)
            verbose=$((verbose + 1))  # Each -v adds 1 to verbosity.
            ;;
        --)              # End of all options.
            shift
            break
            ;;
        -?*)
            printf 'WARN: Unknown option (ignored): %s\n' "$1" >&2
            ;;
        *)               # Default case: No more options, so break out of the loop.
            break
    esac

    shift
done

# if --file was provided, open it for writing, else duplicate stdout
if [ "$file" ]; then
    exec 3> "$file"
else
    exec 3>&1
fi

# Rest of the program here.
# If there are input files (for example) that follow the options, they
# will remain in the "$@" positional parameters.

```

这种方式：
缺点——不能把`-xyz`解析为`-x -y -z`
优点——可以解析长的参数，如：`--file`

### 2. 使用getops

```bash

#!/bin/sh

# Usage info
show_help() {
cat << EOF
Usage: ${0##*/} [-hv] [-f OUTFILE] [FILE]...
Do stuff with FILE and write the result to standard output. With no FILE
or when FILE is -, read standard input.

    -h          display this help and exit
    -f OUTFILE  write the result to OUTFILE instead of standard output.
    -v          verbose mode. Can be used multiple times for increased
                verbosity.
EOF
}

# Initialize our own variables:
output_file=""
verbose=0

OPTIND=1
# Resetting OPTIND is necessary if getopts was used previously in the script.
# It is a good idea to make OPTIND local if you process options in a function.

while getopts hvf: opt; do
    case $opt in
        h)
            show_help
            exit 0
            ;;
        v)  verbose=$((verbose+1))
            ;;
        f)  output_file=$OPTARG
            ;;
        *)
            show_help >&2
            exit 1
            ;;
    esac
done
shift "$((OPTIND-1))"   # Discard the options and sentinel --

# Everything that's left in "$@" is a non-option.  In our case, a FILE to process.
printf 'verbose=<%d>\noutput_file=<%s>\nLeftovers:\n' "$verbose" "$output_file"
printf '<%s>\n' "$@"

# End of file

````
***如何使用getops*** 如果选项有值，就在选项后加`:`, 如`a:b:c`, 则a和b有值，而c无值。对于有值的选项处理，使用`$OPTARG`来取值。

优点：

- 可以处理`-xyz filename`这样的参数

缺点：

- 只能处理短的选项, 如`-h`

## 编写脚本内容

### 参数校验

使用`[[ "$a" == "" ]] || [[ "$b" == "" ]]`等判断，然后使用

```bash

echo "Error Message" >&2
exit 1

```

来实现错误退出


【参考文档】：

http://mywiki.wooledge.org/BashFAQ/035#getopts