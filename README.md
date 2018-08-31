# Functional Bash

#### _This is a collection of Bash functions to make bash more functional._

Filter(_callback_,arr[]) => results[]
```bash
filter() {
    arr=(${@:2})
    ret=()
    inc=1
    for i in ${arr[@]}; do
        [[ $($1 $i) -eq 0 ]] && { ret[$inc]=$i ; inc=$(($inc+1)) }
    done
    echo ${ret[@]}
}
```
Call via `filter funcName "${arrName[@]}"`

map(_callback_, arr[]) => results[]
```bash
#Usage map callback ${arr[@]}
map() {
    outarr=()
    tmparr=("${@:2}")
    for item in ${tmparr[@]}; do
        outarr=( ${outarr[@]} $($1 $item) )
    done
    echo ${outarr[@]}
}
```
Call via `map funcName "${arrName[@]}"`

referential map(_callback_, arr[@]) => results[]
```bash
#Usage refmap callback arr[@] => []
refmap() {
    outarr=()
    tmparr=("${!2}")
    for item in ${tmparr[@]}; do
        outarr=( ${outarr[@]} $($1 $item) )
    done
    echo ${outarr[@]}
}
```

chain(_callback[]_, arr[]) => results[]
```bash
chain() {
    funcs=("${!1}")
    input=("${!2}")
    res=(${input[@]})
    for func in ${funcs[@]}; do
            res=( $(map $func res[@]) ) #NOTE: uses referential map. 
#Altern.    res=( $(map $func ${res[@]} )
    done
    echo "${res[@]}"
}
```
Call via `chain funcs[@] arr[@]`

referential foldr(_callback_ [init] arr[]) => result
```bash
#Usage foldr callback [$init] arr[@]
reffoldr() {
    fun=${1}
    [[ $# -eq 3 ]] && init=("${2}")
    arr=("${!3:-${!2}}")
    res=${init:-0}
    for i in ${arr[@]}; do
        res=( $($fun $res $i ) )
    done
    echo ${res[@]}  
}
```

foldr(_callback_ arg[@]) => result
```bash
#Usage foldr callback ${arr[@]}
foldr() {
    fun=${1}
    arr=("${@:2}")
    res=${init:-0}
    for i in ${arr[@]}; do
        res=( $($fun $res $i ) )
    done
    echo ${res[@]}  
}
```

unfold(_callback_ [init] limit [step]) => []
```bash
#usage: unfold callback [$init] $limit [$step] => []
unfold() {
    fun=${1}
    [[ $# -eq 3 ]] && init=("${2}") || init=0
    limit=${3:-$2}
    [[ $# -eq 4 ]] && inc=("${4}") || step=1
    out=()
    range=( $(seq $init $step $limit)  ) 
    for i in ${range[@]} ; do
        out=( ${out[@]} $($fun $i) )
    done
    echo ${out[@]}
}
```

zip(arr1[], arr2[], [tuplesep]) => result[]
```bash
zip() {
    arr1=( "${!1}" )
    arr2=( "${!2}" )
    out=()
    
    [[ ${#arr1[@]} -ge ${#arr2[@]} ]] && use=2 || use=1
    [[ ${#arr1[@]} -eq ${#arr2[@]} ]] && use=1

    index=0
    sep="${3:-" "}" #separator for resulting tuples. $3 if set. else :
    while [[ $index -ne $( [[ $use -eq 1 ]] && echo ${#arr1[@]} || echo ${#arr2[@]} ) ]] ; do
        out=( ${out[@]} \'${arr1[$index]}$sep${arr2[$index]}\' )
        index=$(($index+1))
    done
    echo "${out[@]}"
}
```
Call via `zip arr1[@] arr2[@] [tuplesep]`

zipWith(_callback_, arr1[], arr2[], [tuplesep]) => result[]
```bash
zipWith() {
    func=$1
    arr1=( "${!2}" )
    arr2=( "${!3}" )
    tuplesep="${4:-":"}"
    tmpres=$(zip arr1[@] arr2[@] $tuplesep)
    res=()
    for i in ${tmpres[@]}; do
        arg1=$(echo $i|cut -d "'" -f 2|cut -d"$tuplesep" -f 1)
        arg2=$(echo $i|cut -d"$tuplesep" -f 2|cut -d"'" -f 1)
        res=( ${res[@]} $($func $arg1 $arg2) )
    done
    echo ${res[@]}
}
```
Call via `zipWith funcname arr1[@] arr2[@] [tuplesep]`

forEach(_callback_, arr[]) => void
```bash
forEach() {
    arr=(${@:2})
    for i in ${arr[@]}; do
        $1 $i
    done
}
```

_Special forEachParallel._  
_Uses the `coproc` directive in bash to run each function call asynchronously_
forEachParallel(_callback_, arr[]) => void
```bash
forEachParallel() {
#$1=callback $2..N array to act on.
    arr=(${@:2}) #Copy $2..N into own array
    for i in ${arr[@]}; do
        coproc $1 $i 
    done
}
```
