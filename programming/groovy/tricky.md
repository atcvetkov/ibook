<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [get the first item if exists or null if empty](#get-the-first-item-if-exists-or-null-if-empty)
- [elegant way to merge Map<String, List<String>> structure by using groovy](#elegant-way-to-merge-mapstring-liststring-structure-by-using-groovy)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


### get the first item if exists or null if empty
```groovy
assert [:]?.find{true} == null
assert []?.find{true} == null
assert ['a']?.find{true} == 'a'
['a': '1'].find{ true }.each { println it.key + ' ~> ' + it.value }
println (['a': '1'].find{ true }.getClass())

// result
// a ~> 1
// class java.util.LinkedHashMap$Entry
```


### [elegant way to merge Map&#60;String, List&#60;String&#62;&#62; structure by using groovy](https://stackoverflow.com/q/62466451/2940319)

<table>
<tr> <td> original Map structure </td> <td> wanted result</td> </tr>
<tr> <td>
<pre lang="groovy"><code lang="groovy">
Map&#60;String, List&#60;String&#62;&#62; case_pool = [
  dev : [
    funcA : ['devA'] ,
    funcB : ['devB'] ,
    funcC : ['devC']
  ],
  'dev/funcA' : [
    funcA : ['performanceA']
  ],
  'dev/funcA/feature' : [
    funcA : ['performanceA', 'feature']
  ],
  staging : [
   funcB : ['stgB'] ,
   funcC : ['stgC']
  ]
]
</code></pre>
</td> <td>
<pre lang="groovy"><code>
String branch = 'dev/funcA/feature-1.0'

result:
[
  funcA: [ "devA", "performanceA", "feature" ],
  funcB: [ "devB" ],
  funcC: [ "devC" ]
]
</code></pre>
</td>
</tr>
</table>


|  <div style="width:50%"> original map structure</div>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | wanted result                                                                                                                                                                                  |
| : --                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | : --                                                                                                                                                                                           |
| Map&#60;String, List&#60;String&#62;&#62; case_pool = [<br> &nbsp;&nbsp;dev : [<br> &nbsp;&nbsp;&nbsp;&nbsp;funcA : ['devA'] ,<br> &nbsp;&nbsp;&nbsp;&nbsp;funcB : ['devB'] ,<br> &nbsp;&nbsp;&nbsp;&nbsp;funcC : ['devC']<br> &nbsp;&nbsp;],<br> &nbsp;&nbsp;'dev/funcA' : [<br> &nbsp;&nbsp;&nbsp;&nbsp;funcA : ['performanceA']<br> &nbsp;&nbsp;],<br> &nbsp;&nbsp;'dev/funcA/feature' : [<br> &nbsp;&nbsp;&nbsp;&nbsp;funcA : ['performanceA', 'feature']<br> &nbsp;&nbsp;],<br> &nbsp;&nbsp;staging : [<br> &nbsp;&nbsp;&nbsp;&nbsp;funcB : ['stgB'] ,<br> &nbsp;&nbsp;&nbsp;&nbsp;funcC : ['stgC']<br> &nbsp;&nbsp;]<br> ] | String branch = 'dev/funcA/feature-1.0'<br> <br> // will final get result of: <br> // " 'dev' + 'dev/funcA' + 'dev/funcA/feature' ":<br> [<br> &nbsp;&nbsp;funcA: [ "devA", "performanceA", "feature" ],<br> &nbsp;&nbsp;funcB: [ "devB" ],<br> &nbsp;&nbsp;funcC: [ "devC" ]<br> ] |



- original map structure:

```groovy
Map<String, List<String>> case_pool = [
  dev : [
    funcA : ['devA'] ,
    funcB : ['devB'] ,
    funcC : ['devC']
  ],
  'dev/funcA' : [
    funcA : ['performanceA']
  ],
  'dev/funcA/feature' : [
    funcA : ['performanceA', 'feature']
  ],
  staging : [
   funcB : ['stgB'] ,
   funcC : ['stgC']
  ]
]
```

- method 1st: by using loop

```groovy
String branch = 'dev/funcA/feature-1.0'
def result = [:].withDefault { [] as Set }
case_pool.keySet().each {
  if ( branch.contains(it) ) {
    case_pool.get(it).each { k, v ->
      result[k].addAll(v)
    }
  }
}
println 'result: ' + result
```

- method 2nd: by using closure

```groovy
String branch = 'dev/funcA/feature-1.0'
def result = [:].withDefault { [] as Set }
case_pool.findAll{ k, v -> branch.contains(k) }.collectMany{ k, v -> v.collect{ c, l ->
    result[c].addAll(l)
}}
println 'result: ' + result
```

- method 3rd: by using closure elegantly

```groovy
def result = case_pool.inject([:].withDefault { [] as Set }) { result, key, value ->
  if (branch.contains(key)) {
    value.each { k, v -> result[k] += v }
  }; result
}
println 'result: ' + result
```



